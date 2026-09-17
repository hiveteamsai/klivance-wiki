# 05 — CDS YANIT ZİNCİRİ (`saglik/cds/`)

> Ürünün çekirdeği. Bu dosyadaki her invariant **yanlış güven** üretmemek içindir:
> bu üründe "yanlış cevap" tehlikelidir, **"yanlış güven" daha tehlikelidir.**
>
> Ölçüm anı: 2026-09-17 · commit `cb16758d`. Fonksiyon imzaları:
> [`15b-FONKSIYON-REFERANSI-CDS.md`](15b-FONKSIYON-REFERANSI-CDS.md)

---

## 1. Zincirin tamamı

```
                       HEKİMİN SORUSU
                            │
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 0. YANIT DİLİ TESPİTİ          cds/dil.py :: yanit_dili                   │
│    LLM'siz. Sorunun dilinden; belirsizse önceki turlar → get_lang.        │
│    ⚠⚠ YANIT DİLİ ≠ ARAYÜZ DİLİ (founder 2026-08-15, #275b)                │
└───────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 1. KAPSAM KİLİDİ               cds/llm.py :: classify   (Haiku, temp=0)   │
│    → KLINIK | KAPSAMDISI | COZULEMEDI                                     │
│    KAPSAMDISI ⇒ LLM'e HİÇ gitmez, prompts.REFUSAL döner (para yanmaz)     │
│    ⚠ Türkçe İ normalizasyonu ZORUNLU · CLASSIFIER_SYSTEM ÖNCEKİ TUR bloklu │
└───────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 2. RAG — KANIT PAKETİ          cds/engine.py :: route()   **LLM YOK**     │
│    mod = interaction | drug | clinical | hastalik | timeline              │
│    ├─ retrieval.py   TÜRKÇE TERİM KÖPRÜSÜ + FTS (core.corpus.tsv GIN)     │
│    ├─ reference.py   find_drugs (4 KATMANLI SIRALAMA = hasta güvenliği)   │
│    ├─ hastalik_kanit.py  monograf kanıt paketi                            │
│    └─ _patient_med_block  hasta bağlamı etkileşim taraması                │
│    çıktı: format_result(res) → KANIT METNİ  +  _sources_from(res) → ROZET │
└───────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 3. AKIL YÜRÜTME     cds/llm.py :: answer_stream(DOCTOR_SYSTEM, messages)  │
│    sistem promptu ÖNBELLEKLİ (donuk taban) · mesajlar değişken            │
│    system_suffix = ÖNBELLEKSİZ ek (dil talimatı, rol eki, görsel kuralı)  │
│    → yanıt AKAR (SSE); _heartbeat_stream ping; iptalde record_usage YAZILIR│
└───────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 4. KAPANIŞ NORMALİZASYONU      cds/kapanis.py :: kapanis_normalize        │
│    SUNUCUDA DETERMİNİSTİK (prompt dilek, sunucu karar) — iki hâlli        │
└───────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 5. İKİNCİ GEÇİŞ (ayrı uçlar)   cds/pipeline.py                            │
│    dose_check · redflag_scan · what_missed · deep_analyze · council ·     │
│    drug_card · simplify · case_timeline · goruntu_on_okuma                │
└───────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────▼──────────────────────────────────────────────┐
│ 6. ÖLÇÜM                                                                  │
│    faithfulness.check/log (SALT-GÖZLEM) · metering · store.record_usage · │
│    soru_kayit (#715b K1) · aklama_monitor (salt-gözlem)                   │
└──────────────────────────────────────────────────────────────────────────┘
```

Giriş noktaları: `chat.respond()` (senkron) ve `chat.respond_stream()` (akış ikizi).
Üründe kullanılan **`respond_stream`**'dir.

---

## 2. Model kademesi ve maliyet

`cds/llm.py` — model kimlikleri ve fiyat tablosunun **tek yeri**.

| Rol | Env | Varsayılan | Nerede |
|---|---|---|---|
| Kapsam kilidi | `KLIVANCE_CLASSIFIER_MODEL` | `claude-haiku-4-5` | `classify` |
| Ana yanıt | `KLIVANCE_ANSWER_MODEL` | `claude-opus-4-8` | `answer` / `answer_stream` |
| MAP (belge çıkarım) | `KLIVANCE_MAP_MODEL` | `claude-sonnet-5` | `extract` |
| Ek eşleme | `KLIVANCE_ATTACH_MAP_MODEL` | — | `_map_attachments` |
| Eleştirmen | `KLIVANCE_CRITIC_MODEL` | `claude-opus-4-8` | `deep_analyze` |
| Orta kademe | `KLIVANCE_MID_MODEL` | `claude-sonnet-5` | triyaj, `what_missed` |
| Konsey | `KLIVANCE_COUNCIL_MODEL` | `claude-opus-4-8` | `council` |
| Görüntü | `KLIVANCE_GORUNTU_MODEL` | `claude-sonnet-5` | `goruntu_on_okuma` |
| Efor | `KLIVANCE_EFFORT` | `high` | düşünme rezervi |

⚠ **`llm.fiyatsiz_modeller()`** — `PRICES` tablosunda karşılığı olmayan **etkin** modelleri
döndürür. Boş değilse maliyet ölçümü sessizce yanlıştır. Kapı: `maliyet_korlugu_verify`.

### 2.1 Düşünme (thinking) tuzağı

**Sonnet-5'te `thinking` verilmezse adaptive VARSAYILAN AÇIK** ve `max_tokens`'i yer.
`extract` / `structured` / `translate` / `critic` gibi **salt-çıktı** çağrılarında
`llm._no_thinking(model)` ile kapatılır.
⚠ **Haiku'ya `_no_thinking` GEÇİRME → HTTP 400.**

### 2.2 Prompt caching tabanı

`DOCTOR_SYSTEM` önbelleklenebilir taban olarak tasarlandı: **donuk metin sistemde**,
değişken içerik (kanıt paketi, soru) **mesajlarda**.

⚠⚠ **MİNİMUM ÖNBELLEKLENEBİLİR ÖNEK SAYISINI EZBERE YAZMA.** Modele bağlıdır ve nesiller
arası **monotonik değildir**. `scratchpad/token_butce_verify.py::_CACHE_MIN` haritasından
oku (bilinmeyen model → en temkinli değer). Yanlış sabitin yönü "güvenli" değildir: fazla
temkinli eşik, promptu kısaltmak isteyeni durdurur = **ters yönde yanlış alarm.**
✅ `llm.count_tokens()` gerçek Anthropic ucunu sarar → taban **ölçümle** doğrulanır.

### 2.3 Bütçe kapısı (çağrı öncesi aritmetik)

`llm.butce_kapisi(max_tokens, effort, akis=)` — saf fonksiyon, ağ/DB yok.
`dusunme_rezervi(effort)` + `METIN_TABANI_TOKEN` hesabını **çağrı öncesi** doğrular.
⚠ Bilinmeyen efor → **EN BÜYÜK** rezerv (temkinli).

---

## 3. Sistem promptları (`cds/prompts.py`)

**Donuk metinlerdir** — değiştikçe cache bozulur. Değişken içerik sisteme değil mesajlara konur.

| Sabit | Kullanım |
|---|---|
| `DOCTOR_SYSTEM` | Ana yanıt. **Zorunlu iskelet burada tanımlı.** |
| `HASTALIK_SYSTEM` | Hastalık monografi (`_hastalik_system_kur()` — DOCTOR_SYSTEM'den blok değiştirerek) |
| `DERIN_SYSTEM` | Derin analiz (**yalnız UZUNLUK/TARZ bloğunun İLK CÜMLESİ** değişir) |
| `ANALIZ_OZET_SYSTEM` | Hasta dosyası klinik özeti |
| `CLASSIFIER_SYSTEM` | Kapsam kilidi (İ-güvenli + ÖNCEKİ TUR bloklu) |
| `DRUGCARD_SYSTEM` | Tek ilaç kartı |
| `ETKILESIM_YORUM_SYSTEM` | `/etkilesim` AI yorumu |
| `REFUSAL` | Kapsam dışı yanıtı |
| `RAD_MAP_BLOK` | YOL 1 radyoloji raporu kuralı (iki MAP promptunun **ortak** bloğu) |
| `ONOKUMA_BASLIKLAR` | YOL 2 görüntü ön-okuma iskeleti |

⚠ `_hastalik_system_kur()` / `_derin_system_kur()` / `_analiz_ozet_kur()` **beklenen önek
yoksa `RuntimeError` fırlatır** — yani `DOCTOR_SYSTEM`'i bozarsan açılışta patlar, sessizce
yanlış prompt üretmez. Bu bilinçli bir fail-fast'tir.

### 3.1 Zorunlu yanıt iskeleti (UI buna güvenebilir)

| TR | EN |
|---|---|
| **Kısa yanıt** | **Bottom line** |
| **Gerekçe** | **Rationale** |
| **⚠ Dikkat** | **⚠ Caution** |
| **Pratik öneri** | **Practical guidance** |
| *kapanış satırı* | *closing line* |

### 3.2 Kapanış satırı — İKİ HÂLLİ (founder 2026-09-11, #769c)

| Durum | TR | EN |
|---|---|---|
| Yanıtta **en az bir** kaynak etiketi var | `Atıflıdır.` | `Cited.` |
| Hiç yok | `Karar-destek amaçlıdır; son karar hekimindir.` | `For decision support only; …` |

- `Atıfsızdır.` kuyruğu **KALKTI**.
- ⚠⚠ **"Atıflı" hükmü GİZLEME ÖNCESİ metne bakar** → yalnız StatPearls'e dayanan yanıt da
  atıflıdır (StatPearls etiketi modele gider, hekimden gizlenir).
- **Sunucu karar verir**, prompt yalnız dilektir: `cds/kapanis.kapanis_normalize`.

### 3.3 Prompt'tan KALDIRILAN ve KALAN — ayrımı bozma

**Founder kararı 2026-08-07 (kesin):** ürünün **kendi altyapısından** söz eden itiraf
cümleleri yanıttan kaldırıldı ("Bilgi tabanımda bu soruya özgü kayıt yok", "kanıt paketinde
yok"…).

⚠⚠ **İKİ ŞEYİ AYIRMAK ŞART** — çünkü bu depoda FAIL-OPEN yasaktır:

| | Durum |
|---|---|
| **(A) BİZİM altyapımızdan söz eden META-İTİRAF** | **KALKTI** — hekimin okuduğu metni kirletir; ona hastası hakkında değil **bizim veritabanımız** hakkında bilgi verir |
| **(B) KLİNİK güvenlik taşıyan uyarı** | **KALDI** — yalnız dili altyapı yerine klinik gerekçeye çevrildi |

Örnek dönüşüm: *"bu doz kanıt paketinden değil"* → *"bu yerleşik bir başlangıç dozudur;
güncel KÜB/kılavuzla teyit edin"*.

⚠ **KAPSAM:** bu karar YALNIZ modelin ürettiği yanıt metnini bağlar. `/etkilesim` sayfasının
altı durumu, `pipeline` status sözleşmesi (`unavailable`/`clean`) ve `chat_body` fail-open UI
dalları **AYRI YÜZEYLERDİR ve KAPSAM DIŞIDIR** — onlarda "çalıştırılamadı" demek
**dürüstlüktür**, itiraf değil.

### 3.4 Yönlendirme yasağı (founder 2026-08-15)

`prompts.YONLENDIRME_YASAGI_URETICI` (üretici prompt) + `YONLENDIRME_YASAGI_DENETCI`
(denetçi prompt — *yönlendirme YOKLUĞU eksiklik değildir*).

**YASAK:** "uzmana danışın" · "… konsültasyonu düşünülebilir" · "karar hekimindir, ona bırakın".
Fiilî girişim, **yönetim adımı** olarak yazılır.

- Ton = **konunun profesörü**.
- Tıp öğrencisi de hekimle **aynı derinlikte** yanıtlanır.
- `rol_eki(unvan, lang)` **her unvanda BOŞ** döner (rol eskalasyon dalı 2026-08-08'de kapandı).
- Öz-beyan ("psikologum/öğrenciyim") perspektifi **değiştirmez** (founder ikinci teyit).

Kapılar: `yonlendirme_yasagi_verify` · `rol_eskalasyon_verify` (ikisi de CI SUITES).
Ürün ağacındaki tek kaynak: `cds/aklama.py`.

### 3.5 Terim kuralı (founder 2026-09-01)

`prompts.TERIM_AJAN` — Türkçe yanıtta İngilizce "agent" **"ajan" değil "ilaç"** olarak
çevrilir. Kapı: `terim_ajan_verify` (SUITES).

---

## 4. ATIF ZİNCİRİ — `cds/kaynaklar.py` (TEK DOĞRU KAYNAK)

Bu modül "kaynak adı → etiket + kimlik disiplini + link kaydı" eşlemesini tutar.
`faithfulness` desen/tip kümelerini, `chat_body` atıf regex'i + etiket haritasını
**buradan** alır (`chat_routes` enjekte eder).

```python
KAYNAKLAR = {
  "statpearls":  {"etiket": "StatPearls", "kimlik": "tam", "link": None, "gizli": True},
  "europepmc":   {"etiket": "Europe PMC", "kimlik": "tam", "link": "europepmc"},
  "pubmed":      {"etiket": "PubMed",     "kimlik": "tam", "link": "pubmed"},
  "openFDA":     {"etiket": "openFDA",    "kimlik": "ad",  "link": "reference"},
  "FAERS":       {"etiket": "FAERS",      "kimlik": None,  "link": None},
  …  # toplam ~45 kaynak
}
```

| Alan | Anlam |
|---|---|
| `etiket` | Hekime görünen ad. **Rozetler ETİKETE göre tekilleştirilir** (`addSources`) |
| `kimlik` | `tam` (doc_id zorunlu) · `ad` (ad örtüşmesi) · `None` (kimliksiz rozet) |
| `link` | Derin bağlantı kurucusu (`europepmc`/`pubmed`/`reference`) ya da `None` = **ölü span** |
| `gizli` | `True` → gösterimde adı **basılmaz** (kanıt modele gitmeye devam eder) |

### ⚠⚠ Kuralları

1. **Yeni kaynak YALNIZ `KAYNAKLAR`a eklenir.** Eskiden üç elle liste vardı; yeni kaynak
   girince dördü birden sessizce düşüyordu.
2. **`cf` PUBLIC = SÖZLEŞME.** Kendi `.lower()`ını yazma —
   `'TİTCK'.lower()` = `ti̇tck` ≠ `cf('TİTCK')` = `titck`. Kümeyi **ürettiğin fonksiyonla**
   ona bak: `kimlik_turleri(mod, norm=)`.
3. **Ayrı yayın = ayrı etiket.** İki kaynağı tek etikete bağlamak iki ayrı yayını tek rozete
   indirir (SAMHSA TIP 42/45 ve altı CDC kaynağı bu yüzden ayrı).
4. **`link: None` çoğu kaynakta ZORUNLU, tercih değil.** Sebep: `doc_id` başlık slug'ıdır,
   `core.corpus`ta `url` kolonu yoktur, href kurucu yalnız pubmed/europepmc/openFDA için
   vardır ve **URL sürüklenmesi gerçektir** (ölçüldü). **Başlıkla arama-linki EKLEME** —
   yanlış makaleye götürmek, denetlenebilirlik vaadini bozar.
5. **Monitör salt-gözlemdir**, istemciye **sızmaz** → yanlış alarm hekimi değil **monitörü** bozar.

Kapılar: `atif_zinciri_verify` (CI) · `atif_arayuz_kapsam_verify` · `rozet_kapsam_verify` ·
`atif_indeksi_verify` · `atif_kimlik_kapisi`.

### 4.1 StatPearls — P0 lisans duruşu

`core.corpus`ta `license='CC BY (StatPearls Publishing)'` yazılıdır ama **gerçek lisans
CC BY-NC-ND 4.0**'dır (NBK430685).

**Founder kararı 2026-08-31:** *"kullanalım, bilgileri alalım, ama hekim karşısına StatPearls
diye çıkmasın."* ⇒ görünür atıf da düşer.

| | |
|---|---|
| Kanıt modele gider mi? | **EVET** |
| Hekim adını görür mü? | **HAYIR** — `gizli: True`, `app/gizli_kaynak.py` sunucu-taraflı süzgeç |
| Eksik sanıp atıf ekle? | **HAYIR** — bilerek alındı |
| "Atıflı" hükmüne sayılır mı? | **EVET** — hüküm gizleme ÖNCESİ metne bakar |

⚠ Bu bayrak lisans sorununu **çözmez** (ihlal göstermekten değil KULLANMAKTAN doğar).
Hukuk kalemi **kapalıdır** (avukat teyidi var, founder 2026-09-11); teknik kural açıktır.

---

## 5. ROZET SÖZLEŞMESİ — sahte provenans ve tersi

**Kural:** Rozet, **modele giden kanıtla AYNI kümeden** üretilir.
- Olmayan kart rozet **almaz** → *sahte provenans*.
- Olan kart rozet **almalı** → yoksa `citedSources` süzgeci **meşru atfı eler**, hekim
  uydurma sanır ve `faithfulness` masum modeli `⚠UNSUPPORTED` damgalar.

`chat._kart_rozetleri(card)` **tek doğru kaynaktır** — `drug` ve `clinical` dalları ikisi de
onu çağırır. Ölçülmüş altı "ayak":

| Ayak | Ne zaman atlandı |
|---|---|
| `tr_brands` | 2026-07-29 |
| `tr_kub` | 2026-07-29 |
| **`faers`** | 10 gün eksik kaldı — 15 klinik sorgunun 6'sında FAERS satırı modele gidiyor, rozeti üretilmiyordu |
| **`variants`** | 2026-08-02 (gerçek üretim sorgusuyla bulundu) |
| **`docs`** | Aynı sınıf, aynı gün kapatıldı |
| + | Yeni kaynak türü buraya eklenince **iki dal da kendiliğinden alır** |

⚠ Bu küme `kaynaklar.py`'den **türetilemez**: `KAYNAKLAR` "kaynak adı → etiket" eşler,
`_kart_rozetleri` "**KART ALANI** → hangi kaynak" kararı verir. Farklı sorulardır.
Kapı `rozet_kapsam_verify`: kanıt **metnini** değiştiren her kart alanı ya rozet üretir ya
`MUAF` listesinde **geçerli bir gerekçeyle** durur; yeni alan ikisini de atlarsa CI kırmızı.

---

## 6. ATIF SADAKATİ — `cds/faithfulness.py`

LLM'siz, deterministik, **SALT-GÖZLEM** (aşama-1: yanıtı DEĞİŞTİRMEZ, kotayı etkilemez).

**KVKK:** yalnız atıf **etiketlerini** okur/loglar — hasta içeriği/kimlik okumaz, loglamaz.

### ⚠⚠ NE ÖLÇER, NE ÖLÇMEZ — bu ayrım modülün TANIMIDIR, eksiklik değil

| | |
|---|---|
| **ÖLÇER** | **KİMLİK DİSİPLİNİ** — atıftaki kaynak/kimlik, RAG'in fiilen bulduğu kümede var mı (uydurma doc_id, kayıtsız tür, çok kısa ad → yakalanır) |
| **ÖLÇMEZ** | **ANLAMSAL DESTEK** — atıf doğru kaynağa yapılıp iddia o kaynakta yoksa göremez, ve göremeyeceği **yapısaldır**: `check()` yalnız kaynak etiket listesine bakar, **belgenin metnine hiç bakmaz** |
| **ÖLÇMEZ** | **ATIF YOĞUNLUĞU** — vekil-bağımlı ve kırılgan; bilerek buraya sokulmadı |

Ölçüldü: aynı `[statpearls:25943]` etiketiyle makul iddia ve klinik olarak **yanlış** iddia
**birebir aynı sonucu** verir.

### 6.1 Boş kanıt paketi — üç hâl, ikisi eskiden ayırt edilemiyordu

| Hâl | İmza |
|---|---|
| (A) **Dürüst**: model itiraf etti, atıf yapmadı | `cited=0, unsupported=[]` |
| (B) **Kendinden emin**: itiraf etmedi, yine de kesin konuştu | **AYNI İMZA** |

Üstelik `log()` şartı `if cited or unsupported` olduğu için **en riskli hâl Render'da hiç iz
bırakmıyordu** — sıklığı ölçülemiyordu. Artık `check()` **`kaynak_yok`** döndürür → "sadık"
ile "ÖLÇÜLEMEDİ" ayrışır.

---

## 7. İKİNCİ GEÇİŞ — `cds/pipeline.py`

Ana yanıt (Opus) üretilir; ardından **dar kapsamlı, ucuz** ajanlar ikinci kez denetler.
Bu ajanlar **tedavi önermez / tanı koymaz** — yalnız riski yakalar. "Tanı koymaz"ı güçlendirir.

| Fonksiyon | Model | Uç | Ne yapar |
|---|---|---|---|
| `dose_check` | Haiku | `POST /api/dose-check` | Yanıttaki dozları güvenlik açısından denetler |
| `redflag_scan` | Haiku | `POST /api/redflag` | Kaçmış aciliyet örüntüsü tarar |
| `what_missed` | Sonnet | `POST /api/whatmissed` | "Şeytanın avukatı" — atlanan klinik noktalar (kanıt-farkında) |
| `deep_analyze` | Opus×2 + Sonnet | `POST /api/deep-analyze` | max-efor yanıt → critic → sorun varsa revize |
| `council` | Sonnet + Opus×3 | `POST /api/council` | triyaj → 3 paralel branş → moderatör sentez |
| `drug_card` | — | `POST /api/drug-card` | Tek ilaç yapılandırılmış atıflı kart |
| `simplify` | Haiku | `POST /api/simplify` | Hasta diline sadeleştirme |
| `case_timeline` | Sonnet | `POST /api/cases/{cid}/timeline` | Kronolojik trend özeti |
| `goruntu_on_okuma` | Sonnet | `POST /api/goruntu-onoku` | **YOL 2** — doğrulanmamış makine ön-okuması |

### 7.1 ⚠⚠ FAIL-OPEN YASAK — sözleşme

Hata / JSON-parse hatası / hız sınırında:
```python
{"status": "unavailable", "critic_status": ...}   # ASLA "clean"
```
UI **amber "elle doğrulayın"** basar, **asla yeşil tik**.

⚠ Yeni güvenlik katmanı eklerken **anahtarı gerçekten DÖNDÜR** — `redflag_scan`da `status`
hiç yoktu ve bu, sessizce fail-open demekti.

### 7.2 Ücretsiz güvenlik uçları

`dose-check` / `redflag` / `simplify` → **0 maliyet ve `_email_gate`'in DIŞINDA.**
Bu **doğrudur**: güvenlik katmanı kota/duvar arkasına konmaz.
Fren kredi değil **günlük tavan** (`_gunluk_tavan`), doğrulanmamış deneme hesabında dar.
⚠⚠ Fren **kapatmıyor, azaltıyor** — "kapandı" deme. Kapı: `guvenlik_tavan_verify`
(⚠ bir kez 3 gün CI'da hiç koşmadı; SKIP_SUITES'te çünkü `ANTHROPIC_API_KEY` ister).

### 7.3 Görüntü ön-okuma (YOL 2) — fail-closed dereceli süzgeç

`pipeline.dogrula(text, lang, conn, gorsel_n, belge_kanit)` model çıktısını
**FAIL-CLOSED** süzgeçten geçirir ve `(metin, sebep)` döner. İçindeki denetimler:

| Denetim | Fonksiyon | İki yönlü öz-test |
|---|---|---|
| Accession/protokol kimliği sil | `_accession_sil` | `accession_oz_test()` — kimliği **yakalar**, klinik kullanımı **yakalamaz** |
| "~N mm (kalibrasyonsuz)" yer tutucusu | `olcu_sablonu_soy` | `olcu_sablonu_oz_test()` — yer tutucuyu yakalar, **gerçek ölçüye dokunmaz** |
| Kesin değişim hükmü kalıpları | `degisim_bul` | olumsuzlama-farkında; **metne dokunmaz, salt gözlem** |

⚠ `_yedek_blok(d)` — kritik katman çökünce basılan **degrade** metin. **İSTİSNA ÜRETEMEZ.**

Sözleşme: `docs/goruntuleme-sozlesme-2026-09-13.md`. Env `KLIVANCE_GORUNTU_ONOKUMA` yokken
**kapalı**. `DOCTOR_SYSTEM` **dokunulmaz**. `extracted_text`e **yazılmaz** (ayrı kolon).

---

## 8. LLM'SİZ KLİNİK MOTORLAR

Bunlar model çağırmaz; deterministiktir ve hızlıdır. Ürünün "ucuz ve doğru" katmanı.

| Modül | Yüzey | Ne |
|---|---|---|
| `reference.py` | `/reference`, ilaç kartı | TR→EN ilaç köprüsü (~130 ad) + ek-soyma + kelime-sınırı-öncelikli arama; `interaction_check` 4 etiket bölümü + sınıf haritası |
| `etkilesim_coklu.py` | `/etkilesim` | 2–8 ilaç etiket taraması |
| `renal.py` | `/bobrek-doz` | Böbrek doz kontrolü |
| `recete_kontrol.py` | hasta kartı sekmesi | 4 kontrol: etkileşim · böbrek · alerji · duplikasyon |
| `sgk.py` | SGK Ek-4/A | **"SGK ÖDER" DEMEZ** · üç durum ayrı · yürürlük tarihi zorunlu |
| `fenotip.py` | nadir hastalık | HPO fenotip → aday hastalık |
| `rad_cikar.py` | YOL 1 | Radyoloji **rapor metninden** yapılandırılmış lezyon satırı |
| `kritik_goruntu.py` | #782c | Zaman-kritik aday kademesi (saf modül) |

### 8.1 ⚠⚠ `find_drugs` SIRALAMA KURALI = HASTA GÜVENLİĞİ

Yanlış sıralama **YANLIŞ İLAÇ** kartı gösterir. Ölçülmüş vakalar:

| Sorgu | Yanlış sonuç |
|---|---|
| `prednol` | → LOTEPREDNOL |
| `seretide` | → ENALAPRIL |
| `diane` | → ZINC OXIDE (`'zinc'` ⊂ `ZİNCİRLİ`) |
| `voltaren` | → boş kart |
| `coumadin` | → boş kart |

**Dört katman + Türkçe İ normalizasyonu + iki katmanlı yedeklilik + `_klinik_dolu`/RxNorm
son-çaresi.** Tam anlatım: `.claude/agents/derya.md`.

⚠ **İçeriği boş eşleşme "bulundu" sayılmaz** (`_klinik_dolu`) — DailyMed satırı yalnız indeks
+ link taşır. Boş eşleşmede RxNorm son-çaresine inilir ve **yalnız gerçekten daha zengin
sonuç gelirse takas edilir** (takas yoksa `resolved_via` temizlenir).

⚠ Dokunursan: `drug_match_verify.py` + `siralama_bozma_tatbikati.py` (2. KB ister, CI'da atlanır).
⚠ Hot-path'te `resolve_ingredient` (**ağlı**) **çağrılmaz**.

### 8.2 ⚠⚠ `/etkilesim` dürüstlük sözleşmesi

Altı durum, **hiçbirinde yeşil/aklama dili YOK** (CI'da taranır — `etkilesim_govde_verify`).

- Sayaç motorun döndürdüğü **karttan** okunur.
- **"eledim" ≠ "bulunamadı"** → AMBER fail-closed.
- `kind="sinif"` etiketlenir.
- `id="etk-sonuc"` **silinmez**.
- `_kirp` kesik alıntıyı `…` ile **işaretler**.
- `_cf_hiza` uzunluk korur · `_kelime_sinirina_yuvarla` pasajı kelimeye yuvarlar
  (**sunum değil KAYNAK düzeltmesi**).

⚠⚠ **SONDA TUZAĞI: `?a=X&b=X` ile ölçme** — sayaç basılmaz; markayı **alakasız** kontrol
ilacıyla eşleştir.
⚠⚠ **48'lik marka listesiyle kapsam hükmü kurma** (seçim yanlılığı; taban Wilson alt sınırı).
⚠⚠ **`_MARKA_ETKEN`e KAYNAKSIZ SATIR YAZMA** (≤12 kayıt, CI iddiası).

---

## 9. TÜRKÇE TERİM KÖPRÜSÜ — `cds/retrieval.py`

**Türkçe'nin üç tuzağı:**
1. **Eklemeli dil** — "diyabetli hastada" ≠ "diyabet"
2. **ICD başlığı virgülden sonra nitelik taşır** — "Diyabet, tip 2" gibi
3. **Kısaltmalar** — "KOAH", "HT", "DM"

**Köprü çökerse TR'de kanıt paketi TAMAMEN BOŞ döner.**

⚠⚠ **"DAHA ÇOK BELGE" HEDEF DEĞİLDİR.** EŞİTLİK şartı korunur (LIKE/parça değil); gündelik
cümle köprü **üretmez**. Kapı: `scratchpad/rag_kopru_verify.py` · `terim_kopru_verify` ·
`konu_alakasi_verify`.

⚠ Bilinen açık: TR sorguda kanıtın konu-dışı gelmesi — üç yaygın hipotezin **üçü de çürüdü**;
köprü doğru çalışıp **slot sırasında eziliyor** (`docs[:max_docs]`).
Memory: `klivance-kanit-alakasi`.

---

## 10. HASTA BAĞLAMI

`chat._case_ctx_block(case_ctx, lang)` — pseudonim hasta bağlamını prompta çevirir.

⚠⚠ **`parts` boş olabilir, `record` yine de GİTMELİ.** Yoksa tahliller / ziyaret notları /
alerji-teratojen-renal talimatları prompta **hiç girmez**.

`case_note` taşıdıkları ve CDS'e etkisi:

| Alan | CDS etkisi |
|---|---|
| **alerji** (whitelist çip — kimlik DEĞİL) | Çapraz-reaktivite: penisilin ↔ sefalosporin / sülfonamid / NSAİİ → **KONTRENDİKE** |
| **doğurganlık** (`gebe_olabilir` / `emziriyor`) | Teratojen uyarısı |
| **boy/kilo** | Doz hesabı |
| **eGFR / kreatinin** | Renal doz |

Kullanan modlar: Sor · ikinci-göz · panel · derin · redflag · dose_check.

---

## 11. DENEY / ÖLÇÜM KURALI (founder — para)

⚠⚠ **FOUNDER KURALI (2026-08-03):** test soruları hariç, ücretli API ile (Anthropic/Gemini/vb.)
**para yakmak YASAK**; test soruları da **ekip toplamı günde ≤10**. Her test çağrısı raporda
adet + maliyetle **beyan edilir**. 10'u aşacak ya da test-dışı her çağrı **founder onayı** ister.
(Müşterinin ürün kullanımı bu kuralın dışındadır.)

⚠⚠ **FOUNDER KURALI (2026-09-06):** **abonelikle yapılabilen hiçbir şey ücretli API ile
yapılmaz.** Deney/ölçü/metin/sınıflandırma → alt ajan. Ürün API yolu yalnız **kod yolundan
geçmesi ZORUNLU olan** ön-kayıtlı tek doğrulama koşumu içindir, o da API'ye mümkün olduğunca
aynı (mesajlar ürünün derleyicisinden).

Tam kural seti: `.claude/agents/cahit.md` "DENEY KURALI".

---

## 12. CDS zincirine dokunurken koşulacak kapılar

| Dokunduğun yer | Kapı |
|---|---|
| `kaynaklar.py` / yeni kaynak | `atif_zinciri_verify` · `rozet_kapsam_verify` · `atif_arayuz_kapsam_verify` |
| `prompts.DOCTOR_SYSTEM` | `yonlendirme_yasagi_verify` · `rol_eskalasyon_verify` · `prompt_adim_kota_verify` · `token_butce_verify` (SKIP) |
| `retrieval.py` terim köprüsü | `rag_kopru_verify` (SKIP — KB ister) · `terim_kopru_verify` (SKIP) · `konu_alakasi_verify` |
| `reference.find_drugs` | `drug_match_verify` (SKIP) · `siralama_bozma_tatbikati` (SKIP) · `ad_cozumleme_kapisi` |
| `pipeline.py` | `analiz_aklama_monitor_verify` · `derin_prompt_verify` · `kritik_goruntu_verify` |
| Yanıt dili | `yanit_dili_verify` |
| Akış | `akis_parite_verify` · `akis_kopma_verify` · `iptal_maliyet_verify` |
| Görüntüleme | `goruntuleme_kapisi_verify` · `goruntu_onokuma_verify` · `rad_*_verify` (7 takım) |

---

**Sonraki:** [`06-BILGI-TABANI-VE-KAYNAKLAR.md`](06-BILGI-TABANI-VE-KAYNAKLAR.md)
