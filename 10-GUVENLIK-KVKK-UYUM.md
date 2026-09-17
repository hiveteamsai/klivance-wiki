# 10 — GÜVENLİK, KVKK VE UYUM

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
>
> ⚠⚠ **FOUNDER KURALI (2026-09-11, taşa kazınmış):** *"Ben hukuki boyutu düşün demedikçe sen
> DÜŞÜNME."* Bu dosya **hukuki yorum üretmez**; **kodda duran kapıları** ve **verilmiş
> kararları** belgeler. Kodda duran kapılar (NC/ND süzgeci, lisans alanına birebir kopya,
> StatPearls görünmezliği) düşünme değil **veri ve koddur**: koşar, yorumlanmaz.

---

## 1. UYGULAMA GÜVENLİĞİ

### 1.1 Yanıt başlıkları — `main._security_headers_mw`

| Başlık | Değer | Not |
|---|---|---|
| `X-Content-Type-Options` | `nosniff` | Her yanıtta |
| `Referrer-Policy` | `no-referrer` (sır taşıyan yollar) / `strict-origin-when-cross-origin` | ⚠⚠ Bkz. §1.2 |
| `X-Frame-Options` | `DENY` | ⚠ `_XFO_EXEMPT` hariç (ödeme dönüş yolları) |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | HTTPS ya da `COOKIE_SECURE` iken |

**⚠ CSP BİLİNÇLİ OLARAK YOK.** Uygulama satır-içi `<script>`/`<style>` yoğundur (landing,
`_CHAT_BODY`, `_REF_BODY`, `calc_body`, inline dönüşüm çağrıları) — bir CSP eklemek sayfaları
**sessizce kırardı**. Eklenecekse önce **`Content-Security-Policy-Report-Only`** ile ölçülmeli.

**⚠ `X-Frame-Options: DENY` ödeme dönüş yollarında UYGULANMAZ:** iyzico akışı bugün üst-düzey
yönlendirme olsa da 3DS/sağlayıcı tarafı çerçeve kullanırsa **para yolu kırılmasın**.
Kapı: `xfo_muafiyet_verify`.

### 1.2 ⚠⚠ SIR TAŞIYAN YOLDA `no-referrer` ŞART (#230b)

`strict-origin-when-cross-origin` **aynı-köken gezinmede TAM URL'i** referrer olarak gönderir
⇒ `/dogrula/<token>` → `/dogrulandi` geçişinde **token GA4'ün `page_referrer`ına düşer** ve
#210b sızıntısı **arka kapıdan geri gelir**.

⚠ Rota kendi başlığını yazsa bile **middleware SONRA koşup ezer** — tek doğru yer burasıdır.
⚠ **Yeni sır taşıyan rota `_NOREF_YOLLAR`a EKLENMELİ.**
Kapı: `scratchpad/sir_yolu_izsiz_verify.py`.

### 1.3 Diğer sertleştirmeler

| Önlem | Yer | Gerekçe |
|---|---|---|
| `/docs`, `/redoc`, `/openapi.json` **KAPALI** | `main.py` (`KLIVANCE_DOCS=1` ile yalnız yerelde) | Şema tüm admin rotalarını + form alan adlarını (44 KB) oturumsuz herkese veriyordu = **saldırı yüzeyi haritası** |
| Container **root değil** | `Dockerfile` — uid 10001 | RCE/path-traversal doğrudan root vermesin. Uygulama hiçbir yere yazmıyor (dosyalar DB'de bytea, `.pyc` kapalı) |
| `FORWARDED_ALLOW_IPS="*"` | `Dockerfile` | ⚠ **BİLİNEN RİSK, bilinçli**: container'a doğrudan erişen istemci IP'sini uydurabilir. Render ağ yalıtımı engeller ama **savunma TEK KATMAN**. ⚠ **DARALTMAYA KALKMA (ölçmeden)** — Render/Cloudflare çıkış IP'leri sabit değil; yanlış daraltma **sessizce** eski davranışa döner |
| Hız sınırı | `webutil._rate_ok(doctor_id, limit)` · `RATE_MAX` · `SUGGEST_RATE_MAX` | Süreç-içi, uç-başına sıkı tavan (M12) |
| Token'lar DB'de **SHA-256 özetli** | `auth._tok_hash` | Bkz. `07` §1.2 |
| ILIKE kaçışı | `reference._like_escape` | `a_pirin`→ASPIRIN yanlış kart / tek `%`→seq-scan |
| Staging noindex | `_noindex_staging_mw` | `onrender.com` Google'a düşmesin |
| Sürüm damgası | `/health` | ⚠ **SAHTE sürüm UYDURULMAZ** — ikisi de yoksa `bilinmiyor` |

Kapılar: `guvenlik_verify` · `xss_reference_verify` · `like_escape_verify` ·
`sir_yolu_izsiz_verify` · `statik_mime_verify` · `tip_ipucu_kapisi`.

---

## 2. ŞİFRELEME — hasta dosya kütüphanesi

`saglik/filecrypto.py` — **Fernet** (AES128-CBC + HMAC), `MultiFernet` ile rotasyonlu.

```
app.case_file.content_enc = Fernet(KLIVANCE_FILE_KEY).encrypt(bytes)
```

| Kural | Detay |
|---|---|
| **Nerede** | **DB bytea** — diskte DEĞİL (Render diski ephemeral) |
| **Anahtar** | `KLIVANCE_FILE_KEY` (birincil/yazma) + `KLIVANCE_FILE_KEY_OLD` (virgülle ayrık eski) |
| **Yoksa** | İlk çağrıda **net hata** → rota `err=svc`. **Sessiz düz-metin saklama YOK** |
| ⚠⚠ **Kaybı** | **KALICI VERİ KAYBI** — tüm hasta dosyaları okunamaz olur. Render dışında da yedeklenmeli |
| **Rotasyon** | 1) yeni anahtar → `KLIVANCE_FILE_KEY=yeni`, `_OLD=eski` · 2) `python scripts/rotate_file_key.py` · 3) `_OLD` kaldır |
| **Hata sınıfı** | `FileKeyError(RuntimeError, ValueError)` — bilerek **ikisi birden** (V10: bozuk anahtar ham `ValueError` fırlatınca istek 500 ile patlıyordu) |

---

## 3. PII REDAKSİYONU — `saglik/redact.py`

**Deterministik KVKK güvenlik ağı.** LLM talimatına (MAP promptu) **EK olarak**, DB'ye yazılan
/ Anthropic'e tekrar giden metinden Türk kimlik verisini temizler.
Saf fonksiyon (yalnız `re`) → import döngüsü yok.

```python
redact_pii(text) -> (temizlenmiş_metin, flags_dict)
```

| Veri | Davranış | Gerekçe |
|---|---|---|
| **TC Kimlik No** | **REDAKTE** `[TC]` | 11 hane + **resmî checksum** (rastgele 11 haneyi değil) |
| **Ayraçlı TC** | **REDAKTE** | ⚠ S6 (2026-07-29): insanlar "123 456 789 01" / "123.456.789.01" yazıyor; bitişik-hane regex'i **kaçırıyordu** → kimlik numarası redakte edilmeden DB'ye ve Anthropic'e gidiyordu. **Yanlış-pozitif freni üç katmanlı**: (1) her grup ≥2 hane (tansiyon "120 80" zincirlenmez), (2) toplam **tam 11**, (3) checksum. Telefonlar `0` ile başladığı için elenir |
| **Telefon** (05xx / +90 5xx) | **REDAKTE** `[TEL]` | |
| **E-posta** | **REDAKTE** `[E-POSTA]` | |
| **Ad-soyad örüntüsü** | **YALNIZ BAYRAK** (redakte etmez) | Klinik eponimleri bozmasın (*Parkinson Hastalığı*, *Tip 2 Diyabet*) |

⚠ **`redact.py` DURUYOR ve 3 gerçek çağıranı var — SİLME.**
⚠ `scratchpad/kvkk_takma_kod_temizle.py` **toplu `--apply` YASAK** (gerçek ad artık meşru;
kapsamsız apply'ı kod zaten reddeder) — yalnız tek hesaplık KVKK m.11 talebi için.

Kapı: `redact_ilac_verify` (SKIP).

---

## 4. HASTA KİMLİĞİ — KURAL 2026-08-05'TE TERSİNE DÖNDÜ

**Founder kararı + avukat onayı (`_gorev.txt` #118b):**

| Eski | Yeni |
|---|---|
| Takma kod **ZORUNLU** | Hekim **gerçek ad girebilir**; takma kod **isteğe bağlı** |
| Kayıtta **3 onay** kutusu | **2 onay** (hasta-kimliği-girmeme taahhüdü **kaldırıldı**) |

⚠ Eski `consents` satırlarındaki `patient: true` **SİLİNMEZ** (ts+IP ispat kaydı); yalnız
yeni kayıtlar iki anahtar yazar.

⚠⚠ **DEĞİŞMEYEN OLGU:** gerçek ad girilen kayıt KVKK **m.6 özel nitelikli veri** olur ve
klinik metinle birlikte **ABD'deki model sağlayıcısında** işlenir. Bu olgu değişmedi;
**ifşa metinlerinden çıkarılması AYRI bir founder kararıdır (#119b), olgunun kendisi değil.**

⚠ *"Kimlik İŞLENMEZ / yalnız takma kod"* diyen her yüzey bu kararla **yanlış** oldu —
**yeni yüzey yazarken bu dili ÜRETME.**

### 4.1 Hasta kartı taşıdıkları

`case_note`: **alerji** (whitelist çip — kimlik DEĞİL) · **doğurganlık**
(`gebe_olabilir` / `emziriyor`) · **boy** · tahliller · ziyaret notları.
CDS bunları kullanır (bkz. `05-CDS-YANIT-ZINCIRI.md` §10).

### 4.2 Dosya kütüphanesi kuralı (2026-07-14 değişti)

Eski *"yüklenen dosya SAKLANMAZ"* kuralı **kaldırıldı** (kullanıcı kararı).
Artık hastaya bağlı tahlil/rapor dosyaları **KALICI + ŞİFRELİ** saklanır.
Hekim dosyayı **silebilir** (soft delete).

⚠ **Dosyalara kimlik YAZILMAZ** (KVKK; gizlilik metni ifşa eder).
Analiz **map-reduce**: her dosya Haiku ile çıkarım (`extracted_text` cache, `mode='map'` →
kotaya sayılmaz) → tek Sonnet sentezi (1 kota).

---

## 5. YASAL METİNLER

| | |
|---|---|
| **Veri sorumlusu** | **ALFA 4N SAN. TİC. LTD. ŞTİ.** — VKN 0510246142, Biga/Çanakkale, KEP alfa4n@hs01.kep.tr |
| Kaynak | `saglik/app/legal_body.py` → `_LEGAL` (TR) + `LEGAL_EN` |
| Rota | `GET /yasal/{slug}` |
| *"Taslak niteliğindedir"* ibaresi | **KALDIRILDI** (founder 2026-07-23) |
| Avukat teyidi | ✅ **VAR** (founder 2026-09-11: *"her şey için teyit var"*) |
| VERBIS durumu | **Ölçülmedi** |

### ⚠⚠ YASAL METİNDE `anthropic` / `ABD` / `m.9` / `açık rıza` = **0** — VE BU BİR FOUNDER KARARIDIR

2026-08-05, **iki kez teyit edildi**; itiraz iletildi, karar tekrarlandı → `_gorev.txt` #119b.
**Eksik sanıp EKLEME.**

⚠ Kararın **çürütmediği** olgu: aktarım gerçekten oluyor (prod DB Oregon/ABD, klinik metin
ABD'deki sağlayıcıda işleniyor) — karar **İFŞAYA** dair, **OLGUYA** değil.

### ⚠⚠ Yasal metne dair hiçbir iddiayı bu dokümandan devralma — **METİNDEN ÖLÇ**

Bağlayıcı ölçüt: **`scratchpad/_selim_yasal_render.py`** (exit kodu döner, **iki yönlü öz-testli**).
Bu satır bir kez iki bayat iddianın kaynağı oldu.

### 5.1 İFŞA KURALLA ATOMİKTİR

| İfşa | Kural |
|---|---|
| Gizlilik metni (TR+EN) yönetici arşivini ve korunan soruları *"hesap kapanışında silinir"*in **İSTİSNASI** olarak ve süresi **"süresiz"** yazar | O ikisinde job gerekmez |
| Dönem **toplam kullanım hacmi** maddesi | **Yoksa `KLIVANCE_BUTCE_MOD=uygula` KODDA REDDEDİLİR** (`credits.butce_ifsa_var()`) |
| **Adil kullanım** ibaresi | "Sınırsız" ilanı + gizli tavan ⇒ **zorunlu**. İbareyi kaldıran tavanı da kaldırmalı |

⚠ **Süre taahhüdü yazacaksan purge'ü AYNI COMMIT'te bağla** — 30 günlük sözün job'u **var ve
koşuyor** (`purge-retention.yml` günlük + açılış backstop).

---

## 6. HESAP YAŞAM DÖNGÜSÜ (2026-07-30 denetimi → **BAĞLAYICI**)

### 6.1 Yüzey küçük ve öyle KALMALI

| Yön | Fonksiyon |
|---|---|
| **2 doğum** | `auth.create_doctor` · `auth.upsert_google_doctor` |
| **2 ölüm** | `store.admin_archive_and_delete` · `store.purge_due_accounts` |

⚠ **Yeni `doctor_id`'li tablo eklersen CASCADE'i AÇIKÇA yaz** — yoksa KVKK silmesi FK
hatasıyla **bloklanır**.

### 6.2 ⚠⚠ YENİ KİŞİ/OLAY TABLOSU KURULMAZ

`app.person` / `app.account_event` **REDDEDİLDİ**:
- *"Geri döndü"* → arşivde **okuma anında** `lower(email)` JOIN'i
- *"Tekrar kayıt"* → **mevcut HMAC** (`store.returned_doctor_ids`)

⇒ **sıfır yeni kalıcı veri, sıfır yeni ifşa yükümlülüğü.**

### 6.3 ⚠⚠ HMAC TUZU — TEK NOKTA ARIZASI

`app.setting 'trial_salt'`. Kaybolursa:
1. Deneme freni **düşer** (herkes yeniden deneme alır)
2. Panel **"hepsi ilk kez geldi"** der

**Tuzu yedeklemeden DB restore etme.**

⚠⚠ **Kanonik e-posta = Python `.strip().lower()`; HMAC'i ASLA SQL'de üretme** (Türkçe İ).

### 6.4 "Silinmiş" panelde İKİ AYRI ŞEY — ayrımı bozma

| Yol | Kopya | Nerede görünür |
|---|---|---|
| Admin silmesi (`admin_archive_and_delete`) | **Kalıcı snapshot** | `/admin/arsiv` |
| Hekimin KVKK talebi (`purge_due_accounts`) | **Kopya ALMAZ** (bilinçli) | Görünmez — yalnız kimliksiz sayaç `app.setting 'kvkk_purge_n'` + **"alt sınırdır"** ibaresi |

Başlık **SATIR değil KİŞİ** sayar.

### 6.5 Guard: bekleyen KVKK talebi olan hesap ARŞİVLENEMEZ

`api_admin_delete` → `?del=kvkk`.
**Gerekçe:** m.7 talebi işlenirken kalıcı kopya almak hakkı **fiilen bertaraf eder**.
Guard **silmeyi** değil **kopya almayı** engeller.

### 6.6 Arşiv snapshot minimizasyonu

| Atılır | Kalır (bilinçli) |
|---|---|
| `password_hash` · `verify_token` · `reset_token` · `signup_ip` · `gclid(_at)` · consents içindeki **IP** | `email` · `billing` · `phone` · onay kaydı (**mali eşleşme + ispat**) |

⚠⚠ `to_jsonb(d)` her şeyi otomatik alır → **yeni `app.doctor` kolonu eklersen arşive gitmeli
mi diye KARAR VER.**

### 6.7 `app.question_kept.asked_at` = `date` (timestamp DEĞİL)

Saniye hassasiyeti **küçük tabanda yeniden-kimliklendirme vektörüdür**.
Kimlik bağı **YOK** ve öyle kalmalı.

### 6.8 Retention job

```
.github/workflows/purge-retention.yml   cron "17 3 * * *"  (her gün 03:17 UTC)
  → scripts/purge_retention.py
  → grace (≤30 gün) dolan silme-talepli hesaplar + eski soft-delete dosya tombstone'ları
```
**+ `migrate.py` açılış backstop'u** (deploy seyrekleşse de silme sözü 30 günde tutulur).
Gerekli secret: `DATABASE_URL`.

Kapılar: `retention_verify` · `hesap_yasam_dongusu_verify` (ikisi de **CI SUITES**).

### 6.9 ⚠ Açık kalem (kod kalemi)

`iyzico_payment` + `topup_grant` CASCADE'i **ödeme satırlarını götürür**, metin *"mali kayıtlar
10 yıl"* der. ✅ **Avukat teyidi VAR** (founder 2026-09-11) → **soru değil, kod kalemi.**

---

## 7. ANONİM SORU İSTATİSTİĞİ — regresyon freni

`app.query_stat`: **`doctor_id` YOK, ham soru metni YOK** → `/admin` "Soru istatistiği".

⚠⚠ **REGRESYON FRENİ: ham/redakte soru metni ya da `doctor_id` EKLENİRSE KVKK ifşası
ZORUNLU olur** (aydınlatma metni **AYNI commit'te**).

⚠ `app.question_insight` (soru analitiği) bu tabloya **dokunmaz** — ayrı yoldan gider
(`app.message`). Bkz. `09-ADMIN-VE-ANALITIK.md` §2.

---

## 8. FDA CDS UYUMU

Yürürlükteki rehber: **29 Ocak 2026**.

| | |
|---|---|
| Riskli kriter | **Kriter 4** — hekim dayanağı **bağımsız gözden geçirebilmeli** + öneriye **birincil dayanmamalı** |
| ⚠⚠ | **"Tanı koymaz" TEK BAŞINA muafiyet SAĞLAMAZ** — FDA **fonksiyona** bakar |
| ⚠⚠ | 2026 değişikliği muafiyet değil, **geri çekilebilir enforcement discretion** |

**Mimari karşılıkları:**
1. Her iddia **atıflı** → `cds/kaynaklar.py`
2. Atıf sadakati **ölçülüyor** → `cds/faithfulness.py`
3. **Rozet = modele giden kanıt** → sahte provenans yok
4. `dose_check` / `redflag` **tek-çıktı yönlendirmesinden kaçınır**
5. Landing'deki **Kriter-3/4 kalkan cümleleri YERİNDE** (commit `c98b72e`) — **SİLME**

Kaynak: `docs/rakip-bosluk-arastirma-2026-07-16.md`.

---

## 9. TİCARİ İLETİ (İYS) — mail sınırı

Bkz. `07-KIMLIK-ERISIM-ODEME.md` §8. Özet:

| Kural | Madde |
|---|---|
| Bilgilendirme/aktivasyon maili İYS'siz gidebiliyor | **m.6/1** — *"temin edilen hizmetin KULLANIMINA yönelik"* |
| ⚠⚠ Maile **plan/indirim/fiyat/satın-alma CTA'sı** girdiği gün istisna **DÜŞER** | onay + İYS şart (toplu gönderimde ceza **×10**) |
| ⚠ **m.6/2'ye yaslanma** | *"hizmet özendirilemez"* der; aktivasyon maili tam da özendirir |
| Tek satır kimlik zorunlu | **m.8/2** (ticaret unvanı + MERSİS + info@klivance.com) |
| Görünür iptal linki zorunlu | **m.9/4** — `List-Unsubscribe` başlığı **tek başına yetmez** |
| İşlemsel mailler ret kapsamı dışında | **m.9/5** |

---

## 10. LİSANS KAPILARI (kod — yorum değil)

| Kapı | Ne yapar |
|---|---|
| `ingest_europepmc.commercial_ok()` | **NC/ND'yi ingest kapısında eler** |
| `core.corpus.license` | Kaynağın **kendi sayfasından BİREBİR** yazılır |
| `app/gizli_kaynak.py` | StatPearls adının **hiçbir yüzeyde** görünmemesi (sunucu-taraflı süzgeç) |
| `bookshelf_lisans_kapisi_verify` | *"OA subset"* ≠ ticari |
| `bookshelf_kimlik_kapisi_verify` | Kimlik disiplini |
| `inis_kaynak_beyani_verify` | Landing'de kaynak beyanı |
| `_derya_lisans_bozma_tatbikati` (SKIP) | Bozma tatbikatı |

⚠ **Reklamda kaynak adı ÜÇ yerde bulunur: görsel pikselleri / gövde metni / platform alanı —
üçünü de tara.**

---

## 11. AVUKAT TEYİDİ — TAŞA KANLA YAZILMIŞ KURAL

**Founder 2026-09-11:** *"avukattan her şey için teyit var."*

2026-09-11'e kadar açılmış **HER** hukuk/lisans kalemi avukat onaylıdır — teyit **MEVCUT
DURUŞU** kapsar (kullandıklarımız **VE** reddettiklerimiz):

StatPearls (görünmez, atıfsız kullanım) · TİTCK KÜB · ICD-11 CC BY-ND · EUCAST ·
TEMD/TKD/ESC/SIGN/KDIGO redleri · TGA · CDC "endorsement" ibaresi · KVKK ifşa metinleri
(#119b) · yasal metinler + VERBIS · adil kullanım maddesi (#396b) · `iyzico_payment`
CASCADE/10 yıl · hasta kimliği (#118b) · mali kayıt saklama.

### ⚠⚠ KURAL

Bu kalemleri:
- bir daha founder'a/avukata **SORU olarak ÇIKARMA**
- **"avukat teyidi bekliyor" YAZMA**
- iş **BLOKLAMA**
- reddedilmiş kaynağı **"artık teyitli" diye AÇMA**

**Teyit izin DEĞİL, duruş onayıdır.**

**Teknik kurallar DEĞİŞMEDİ** (hukuk değil ürün disiplini): lisans kaynağın sayfasından
birebir yazılır · NC/ND ingest kapısında elenir · StatPearls görünmez kalır.

**Yeni kaynakta lisans belirsizse:** fail-closed (indirme) + `_gorev.txt`'e kalem;
founder'a **tek satır "karar"** — avukata yönlendirme **YOK**.

---

## 12. GÜVENLİK/UYUM KAPILARI

| Alan | Kapı |
|---|---|
| Başlıklar / sır | `guvenlik_verify` · `sir_yolu_izsiz_verify` · `xfo_muafiyet_verify` · `statik_mime_verify` |
| XSS | `xss_reference_verify` · `en_enjeksiyon_verify` |
| SQL | `like_escape_verify` · `bayat_conn_verify` · `async_db_semantik_verify` |
| KVKK / yaşam döngüsü | `retention_verify` · `hesap_yasam_dongusu_verify` · `_selim_yasal_render` · `yas_asimi_verify` |
| Hasta verisi | `cases_maske_tatbikati` (SKIP) · `akis_kvkk_tatbikat` (SKIP) · `parite_maske_verify` · `kimlik_kapisi_verify` (SKIP) |
| Admin denetimi | `admin_denetim_verify` · `api_admin_uclari_verify` |
| Lisans | `bookshelf_lisans_kapisi_verify` · `bookshelf_kimlik_kapisi_verify` · `inis_kaynak_beyani_verify` |
| Güvenlik tavanı | `guvenlik_tavan_verify` (SKIP — API anahtarı ister) |
| Ödeme dürüstlüğü | `callback_verify` · `iptal_maliyet_verify` |

---

**Sonraki:** [`11-TEST-VE-CI-KAPILARI.md`](11-TEST-VE-CI-KAPILARI.md)
