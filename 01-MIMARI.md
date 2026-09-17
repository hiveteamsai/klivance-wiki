# 01 — SİSTEM MİMARİSİ

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> Sayılar bu tarihte **ölçülmüştür**; "son durum" iddiası kurmadan önce yeniden ölç
> (`18-DEVIR-NOTLARI.md` §6 reçeteyi verir).

---

## 1. Yığın

| Katman | Teknoloji | Not |
|---|---|---|
| Dil | **Python 3.12** (`.venv`) | |
| Web | **FastAPI 0.139** + uvicorn (yerel) / **gunicorn + UvicornWorker** (prod) | |
| Veritabanı | **PostgreSQL** 16 (yerel Docker `saglik-pg`) / 18 (prod Render Oregon) | `psycopg` 3.3 + `psycopg_pool` |
| LLM | **Anthropic Claude** (`anthropic` 0.116) | Haiku / Sonnet / Opus kademesi |
| Parola | **Argon2id** (`argon2-cffi`) | |
| Şifreleme | **Fernet** (`cryptography`) — AES128-CBC + HMAC | Hasta dosyaları at-rest |
| Ödeme | **iyzipay** 1.0.46 (aktif) · **stripe** 15.3 (pasif) | |
| Şablon | **Yok** — HTML Python string sabitlerinde | Jinja/React yok; kasıtlı |
| Frontend | **Vanilla JS**, build adımı yok | Tek istek = tek HTML |
| Konteyner | Docker (`python:3.12-slim`), root olmayan kullanıcı uid 10001 | |

**Bilinçli sadelik:** derleme adımı, node_modules, SPA ve şablon motoru **yoktur**.
Sayfa gövdeleri Python string sabitleridir (`*_body.py`). Bu, i18n'i (§6) ve CI'da
"gövde ölçmeyi" mümkün kılan tercihti — değiştirmek 320 doğrulama kapısının bir kısmını
kırar.

### 1.1 Çalıştırma

```bash
# repo kökünden (D:\sağlık) — MUTLAKA modül yolu ile
./.venv/Scripts/python.exe -m uvicorn saglik.app.main:app --host 127.0.0.1 --port 8000
```

⚠⚠ **`saglik/` dizininin İÇİNDEN çalıştırma.** `saglik/http.py` stdlib `http`'yi gölgeler
→ uvicorn çöker.

```bash
# prod (Dockerfile CMD)
python -m saglik.app.migrate && \
gunicorn saglik.app.main:app -k uvicorn.workers.UvicornWorker \
  -w ${WEB_WORKERS:-2} -b 0.0.0.0:${PORT:-8000} --timeout ${WEB_TIMEOUT:-300} \
  --forwarded-allow-ips '*'
```

---

## 2. Katman haritası

```
┌──────────────────────────────────────────────────────────────────────────┐
│  saglik/app/            WEB UYGULAMA KATMANI  (FastAPI · 88 modül)       │
│  ─────────────────────────────────────────────────────────────────────  │
│  main.py        → app nesnesi · middleware · kalan rotalar · kur() sırası│
│  webutil.py     → PAYLAŞILAN SÖZLÜK (esc/head/nav/en/get_lang/_doctor/   │
│                   _rate_ok/_profil_eksik/_erisim_402/_gunluk_fren_429)   │
│  *_routes.py    → rota modülleri (@_rota + kur(app))                    │
│  *_body.py      → SAF sayfa gövdeleri (HTML/CSS/JS sabitleri, import≈0)  │
│  store.py       → tüm DB işlemleri (101 fonksiyon)                       │
│  auth.py        → Argon2id + sunucu-taraflı oturum                       │
│  credits.py     → ERİŞİM MODELİ (isim tarihsel; kredi YOK)               │
│  iyzico*.py     → ödeme · billing.py → Stripe                            │
│  migrate.py     → deyim-bazlı idempotent şema migrasyonu (açılışta)      │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────────┐
│  saglik/cds/            KLİNİK KARAR-DESTEK MOTORU  (37 modül)            │
│  ─────────────────────────────────────────────────────────────────────  │
│  chat.py        → orkestratör: classify → route → answer → ölç           │
│  engine.py      → route(): anahtarsız KB araması (RAG), LLM YOK          │
│  retrieval.py   → kanıt çekimi + TÜRKÇE TERİM KÖPRÜSÜ                    │
│  reference.py   → ilaç kartı + etkileşim tarama, LLM YOK                 │
│  llm.py         → Anthropic sarmalayıcı: model kademesi, cache, maliyet  │
│  prompts.py     → DONUK sistem promptları (prompt-caching için sabit)    │
│  kaynaklar.py   → ATIF ZİNCİRİNİN TEK DOĞRU KAYNAĞI                      │
│  faithfulness.py→ atıf sadakati (LLM'siz, deterministik, salt-gözlem)    │
│  pipeline.py    → ikinci-geçiş güvenlik ajanları (doz · kırmızı bayrak)  │
│  renal/sgk/recete_kontrol/etkilesim_coklu → LLM'siz klinik motorlar      │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────────┐
│  saglik/  (kök)  +  saglik/connectors/     ALTYAPI + VERİ BAĞLAYICILARI  │
│  ─────────────────────────────────────────────────────────────────────  │
│  db.py          → connect() @contextmanager + havuz                      │
│  config.py      → .env + sources.yaml + AÇILIŞTA YANLIŞ-YAPILANDIRMA UYARISI│
│  filecrypto.py  → Fernet (MultiFernet, rotasyonlu)                       │
│  redact.py      → deterministik PII redaksiyonu (KVKK güvenlik ağı)      │
│  kimlik.py      → "bu metin gerçekten ilaç adı mı" TEK KAPI              │
│  http.py        → TLS parmak izi taklidi + rate limit + retry            │
│  ingest_state.py→ watermark + koşu logu + idempotent run_sync            │
│  connectors/    → openfda · dailymed · enforcement · clinicaltrials ·    │
│                   icd11 · titck · orphanet · mhra                        │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────────────┐
│  PostgreSQL     app (32 tablo) · core (22 tablo) · raw (1 tablo)         │
└──────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Bağımlılık YÖNÜ (kırılmaz kural)

```
main.py  ──imports──▶  *_routes.py  ──imports──▶  webutil.py / store.py / cds/*
   ▲                                                      │
   └──────────────── ASLA ────────────────────────────────┘
```

**Hiçbir modül `main.py`'den import ETMEZ** — dairesel olur. `main.py` tersine, taşınan
adları **fasad** olarak yeniden dışa verir; bu fasadlar doğrulama scriptlerinin
`main.<ad>` erişimini ayakta tutar.

⚠⚠ **FASAD SİLME KURALI:** `main.py`'de kullanılmayan bir import "ölü" görünür ama
olmayabilir. Silmeden önce **AST ile** dış tüketiciyi tara (`from…import` çok satırlı
parantezli **dahil** + `mod.attr` + `getattr`/`hasattr`). Satır-bazlı regex parantezli
import bloğunu ilk satırda keser ve dinamik erişimi hiç görmez — bu tuzak bir kez yaşandı.

Bilinen zorunlu fasadlar:
| Ad | Neden silinemez |
|---|---|
| `_CALC_BODY` | `calc_ask_verify` (CI SUITES) import zinciri teyidi olarak okur |
| `chat_routes` blok (`SAFETY_DAILY_CAP` vb.) | `main_fix_verify` (SUITES) + `guvenlik_tavan_verify` (SKIP) okur |
| `_tr_tutar_coz` | `admin_panel_verify` + `admin_pano_verify` `M._tr_tutar_coz` ile çağırır |
| `cases_body` 5 sabiti | `cases_govde_verify` `hasattr(sys.modules[...])` ile çiviler |

⚠ `cases_routes` için **bilerek fasad KONMADI** (0 dış tüketici ölçüldü). "Tutarlılık olsun"
diye ekleme — ölü yeniden-dışa-verim, main.py'yi okuyanı yanıltır.

### 2.2 Saf modüller (import ≈ 0, bilinçli)

Bu modüller hiçbir şey import etmez ve etmemelidir — böylece hiçbir kapının yanına
istemeden bağımlılık taşınmaz:

`form_listeler.py` · `ornek_sorular.py` · `cases_body.py` · `abonelik_body.py` ·
`cds/isaretler.py` · `cds/kritik_goruntu.py` · `app/kritik_serit.py` ·
`app/on_okuma_kayit.py` · `cds/fenotip_capa.py` (üretilmiş dosya)

---

## 3. Rota kaydı — neden `include_router` yok

```python
# *_routes.py içinde (app burada YOK — dairesel olurdu)
@_rota("get", "/chat", response_class=HTMLResponse)
def chat_page(request: Request): ...

def kur(app) -> None:
    """main.py çağırır; kayıt listesi gerçek app.get/post PUBLIC API'siyle uygulanır."""
```

`_rota(...)` rotayı bir **kayıt listesine** yazar; `kur(app)` çağrılınca gerçek kayıt
`app.get/post` ile yapılır. Sonuç: **`app.routes` DÜZ** kalır ve rota envanteri okuyan
araçlar körelmez. `include_router` ölçülerek reddedildi.

⚠⚠ **SIRA = EŞLEŞME SIRASI.** `main.py`'deki `kur(app)` çağrı sırası, aynı deseni
yakalayan rotalarda kimin kazandığını belirler. `main.py:1005–1196` arasındaki blok
**sıralıdır ve yorumlarla gerekçelendirilmiştir** — taşırken oku.

Sıra (main.py'den, satır numarasıyla):
```
1005 _auth_kur          1156 _admin_kur
1008 _cihaz_kur         1164 _abonelik_kur
1009 _analiz_kur        1172 _panel_kur
1024 _chat_kur          1174 _admin_hekim_kur
1025 _turn_kur          1178 _admin_hasta_kur
1100 _cal_kur           1180 _recete_kur
1115 _ref_kur           1184 _recete_case_kur
1124 _cases_kur         1185 _sgk_case_kur
1125 _enabiz_kur        1186 _sgk_ara_kur
                        1187 _hastalik_kur
                        1188 _deger_onay_kur
                        1189 _rad_case_kur
                        1190 _on_okuma_kur
                        1191 _on_okuma_kaydet_kur
                        1196 _etk_yorum_kur
```

⚠ `_on_okuma_kaydet_kur` **ayrı bir `kur`dur** (dosya tavanı aşıldığı için bölündü) —
unutulursa rota **sessizce yok olur**.

---

## 4. İstek yaşam döngüsü

### 4.1 Middleware yığını (ölçülmüş sıra — `app.user_middleware`)

Starlette'te **son eklenen en dışta** çalışır. `main.py`'deki tanım sırası ile yürütme
sırası **terstir**; aşağıdaki tablo **yürütme** sırasıdır (istek yukarıdan aşağı iner,
yanıt aşağıdan yukarı çıkar):

| # | Middleware | main.py | Görev |
|---|---|---|---|
| 1 | `_security_headers_mw` | 657 | Güvenlik yanıt başlıkları (M10) |
| 2 | `_css_yorum_mw` | 615 | `<style>` yorumlarını soyar (#93b) |
| 3 | `_noindex_staging_mw` | 564 | `onrender.com` Google'a indekslenmesin |
| 4 | `_notrack_mw` | 479 | `klv_notrack=1` → `_ANALYTICS_TAG`'in **TAMAMINI** HTML'den soyar |
| 5 | `_acq_mw` | 429 | Edinim kaynağını **sunucuda** yakala (JS'e bağlı değil) |
| 6 | `i18n_mw` | 399 | `lang=en` iken app HTML'ini `EN_PAIRS` ile çevirir + `<html lang>` |
| 7 | `profil_kapisi_mw` | 341 | Girişli ama profili eksik hekimi `/onay`'a yollar |

⚠⚠ **`_notrack_mw` sonucu:** `/admin*` 200 yanıtında `klv_notrack=1` **otomatik yazılır**
→ **founder'ın kendi cihazı GA4'e HİÇ düşmez.** Dolayısıyla *"GA4'te yok"* bir şeyin
olmadığının kanıtı **değildir**. Yeni analytics parçası `_ANALYTICS_TAG`'in **İÇİNE**
eklenir (dışına eklenen soyulmaz).

⚠ **`profil_kapisi_mw`** altı maddelik bir sözleşmedir; kural TEK YERDE `_profil_eksik(d)`,
ortağı `/onay`ın topladığı kümedir. Bozmadan önce `.claude/agents/kamil.md` oku.
Kapı: `scratchpad/kayit_zorunlu_verify.py`.

### 4.2 Açılış (startup)

1. `migrate.py` — **deyim-bazlı** idempotent şema migrasyonu.
   - SQL dosyaları deyimlere ayrılır (dolar-tırnak/string/yorum farkında ayırıcı).
   - **Her deyim KENDİ transaction'ında** koşar → kilit hemen bırakılır.
   - Her deyimin özeti `app.schema_migration`'a yazılır → sonraki açılışlar **atlar**
     (kararlı durumda **sıfır DDL, sıfır kilit**).
   - `lock_timeout` (vars. 3 sn) → kilit alınamazsa migrate **patlar, siteyi kilitlemez**.

   > Neden: eskiden ≈47 DDL TEK transaction'da her açılışta koşuyordu.
   > `ALTER TABLE … ADD COLUMN IF NOT EXISTS` hiçbir şey yapmasa **bile** ACCESS EXCLUSIVE
   > kilidi alır ve transaction sonuna kadar bırakmaz → deploy sırasında **site geneli donma**.

2. `_threadpool_hizala` (`@app.on_event("startup")`) — threadpool ↔ DB havuzu oranını hizalar.
3. `config._uyari_bas()` (import anında) — eksik/çelişkili env'i **stderr'e gürültülü** basar.
4. `purge_due_accounts` backstop.

### 4.3 Tipik CDS isteği (`POST /api/chat`)

```
İstemci (chat_body.js)
   │  fetch POST /api/chat  {question, thread_id?, case_id?, attachment?}
   ▼
middleware yığını (§4.1)
   ▼
chat_routes.api_chat  (async)
   ├─ _doctor(request)                 kimlik
   ├─ ÜÇLÜ ERİŞİM ZİNCİRİ (sırası sabit):
   │    1. webutil._erisim_402         FAIL-CLOSED  → 402
   │    2. butce_kapisi._butce_fren    dönem bütçesi → 402 (gozlem modunda sayar)
   │    3. webutil._gunluk_fren_429    FAIL-OPEN     → 429
   ├─ _chat_hazirlik_sync  (run_in_threadpool — AĞIR İŞ EVENT LOOP'TA ÇALIŞMAZ)
   │    ├─ cds.dil.yanit_dili(soru)    YANIT DİLİ ≠ ARAYÜZ DİLİ
   │    ├─ _hasta_baglami()            case_ctx (parts boş olsa da record gitmeli)
   │    └─ cds.chat.respond_stream ──▶ §5
   ├─ StreamingResponse                yanıt akar; _heartbeat_stream ping
   └─ store.record_usage               ⚠ İPTALDE DE YAZILIR
```

⚠⚠ **Ağır iş event loop'ta çalışmaz** (`run_in_threadpool`) — iki eşzamanlı dosya analizi
siteyi **herkes için** donduruyordu. Kapı: `threadpool_kapisi`.

⚠ **Akış iptalinde koşulsuz iade YASAK** — teslim edilen karaktere göre ücretlendirilir
(`STREAM_ABORT_FREE_CHARS`), iptalde de `record_usage` yazılır; yoksa kaçak **ölçülemez**.

---

## 5. CDS zinciri (özet — ayrıntı `05-CDS-YANIT-ZINCIRI.md`)

```
soru
 │
 ├─ 1. KAPSAM KİLİDİ      llm.classify (Haiku, temp=0, İ-normalize)
 │      KLINIK | KAPSAMDISI | COZULEMEDI
 │      → KAPSAMDISI ise LLM'e HİÇ gitmez, prompts.REFUSAL döner
 │
 ├─ 2. RAG                engine.route(...)  — **LLM YOK, anahtarsız**
 │      ├─ retrieval.py   TÜRKÇE TERİM KÖPRÜSÜ (aşağıda)
 │      ├─ reference.py   ilaç kartı (find_drugs 4 katmanlı sıralama)
 │      └─ core.corpus    FTS (tsv GIN)
 │
 ├─ 3. AKIL YÜRÜTME       llm.answer_stream(DOCTOR_SYSTEM + kanıt + geçmiş)
 │      sistem promptu ÖNBELLEKLİ (donuk taban), mesajlar değişken
 │
 ├─ 4. İKİNCİ GEÇİŞ       pipeline.py — dar kapsamlı, ucuz denetim ajanları
 │      dose_check · redflag · whatmissed · council · deep_analyze
 │
 └─ 5. ÖLÇÜM              faithfulness (salt-gözlem) · metering · store.record_usage
```

⚠⚠ **TÜRKÇE'NİN ÜÇ TUZAĞI** (`cds/retrieval.py`): eklemeli dil · ICD başlığı virgülden
sonra nitelik taşır · kısaltmalar. **Köprü çökerse TR'de kanıt paketi TAMAMEN BOŞ döner.**
"Daha çok belge" hedef **değildir** — EŞİTLİK şartı korunur (LIKE/parça değil), gündelik
cümle köprü **üretmez**. Kapı: `scratchpad/rag_kopru_verify.py`.

### 5.1 Derya/Cahit ayrımı — hata avlarken ÖNCE bunu yap

| Soru | Katman | Dosya |
|---|---|---|
| Kanıt paketi **boş mu geldi**? | veri seçimi | `cds/retrieval.py`, `cds/engine.py`, `cds/reference.py` |
| Paket **doluyken mi kullanılmadı**? | yanıt üretimi | `cds/chat.py`, `cds/prompts.py`, `cds/pipeline.py` |

Bu ayrımı yapmadan kazarsan **yanlış katmanda** kazarsın.

---

## 6. i18n mimarisi

| Yüzey | Yöntem |
|---|---|
| Uygulama HTML'i | `i18n_mw` → `EN_PAIRS` (TR→EN, **uzun-önce sıralı**) + `<html lang>` swap |
| Landing + legal | **Kendi İngilizce sürümünü servis eder** (`LANDING_HTML_EN`, `LEGAL_EN`) → middleware no-op |
| `/hesaplayicilar` | `CALC_LANG` ile **JS tarafında** iki dilli → `i18n_mw`'den **MUAF** |
| `/reference` | `REF_LANG` sunucudan enjekte (bu sayfada `KLV_LANG` **YOK**) |
| Runtime JS metni | `KLV_LANG` (cookie'den) ile dallan |

`get_lang(request)`: cookie `klv_lang` > `Accept-Language` (birincil `tr`→tr, değilse en) > tr.

### ⚠⚠ APOSTROF TUZAĞI (sessiz P0 — bir kez tüm sayfa JS'ini düşürdü)

`en()` EN modunda **TÜM HTML'i (script blokları dahil)** `EN_PAIRS` ile değiştirir. Bir EN
çevirisi **düz kesme (`'`)** içeriyorsa ve TR anahtarı **tek-tırnaklı bir JS string**
içindeyse → string kapanır → `SyntaxError` → **o sayfanın TÜM JS'i çöker** (sessiz; sayfa
boş yüklenir).

**Kural: `EN_PAIRS` İngilizce değerlerinde kıvrık kesme `'` (U+2019) kullan.**
Doğrulama: `head()+nav(doc,'chat','en')+_CHAT_BODY` → `en()` → `<script>` çıkar → `node --check`.

### ⚠⚠ YANIT DİLİ ≠ ARAYÜZ DİLİ (founder 2026-08-15, #275b)

Modele giden dil **SORUDAN** tespit edilir: `cds/dil.py::yanit_dili` (LLM'siz; belirsizse
önceki turlar → `get_lang`). `chat_routes`ta **5 üretici çağrı** ondan geçer; hata
metni/paneller UI dilinde kalır. Kapı: `yanit_dili_verify` (CI).

⚠ Yeni TR literal'i yazarken `EN_PAIRS` anahtarlarıyla **çakışmayacak** şekilde yaz
(ör. "Kaynaklar" yerine "Tıbbi kaynaklar taranıyor").

---

## 7. Modül kesimi (faz) tarihi — neden bu kadar çok dosya var

`main.py` bir zamanlar **8.490 satırdı**. Dokuz fazda kesildi; her kesim ayrı bir gerekçeyle
yapıldı ve gerekçe dosyanın **kendi başlığına** yazıldı:

| Faz | Çıkan | Modül |
|---|---|---|
| — (2026-07-29) | saf sabitler | `ui_css` · `i18n_pairs` · `chat_body` · `cal_body` · `ref_body` · `legal_body` |
| 1 | landing gövdesi | `landing.py` |
| 2a | paylaşılan sözlük | `webutil.py` |
| 2b | admin yüzeyi | `admin_routes.py` |
| 3 | kimlik/oturum | `auth_routes.py` |
| 4 | hasta defteri gövdesi | `cases_body.py` |
| 5 | hesap/ödeme | `account_routes.py` |
| 6 | sohbet + CDS api | `chat_routes.py` |
| 7 | hasta defteri rotaları | `cases_routes.py` |
| 8 | referans + ücretsiz araçlar | `ref_routes.py` |
| 9 | takvim | `cal_routes.py` |

⚠ **Dosya tavanı vardır** ve CI'da ölçülür: `scratchpad/dosya_boyut_verify.py`.
Tavanı aşan modül bölünür (`on_okuma.py` → `on_okuma_kaydet.py` böyle doğdu).
**Satır/tavan sayısını buraya yazma — o scriptten oku.**

---

## 8. Dış bağımlılıklar (ağ)

| Servis | Ne için | Kritiklik | Anahtar |
|---|---|---|---|
| **Anthropic** | Tüm LLM çağrıları | **Ürünün çekirdeği** — yoksa CDS yanıt üretemez | `ANTHROPIC_API_KEY` |
| **iyzico** | TR kredi kartı tahsilatı | Para yolu | `IYZICO_API_KEY` · `IYZICO_SECRET_KEY` · `IYZICO_BASE_URL` |
| **Stripe** | ABD kolu | Pasif | `STRIPE_SECRET_KEY` · `STRIPE_WEBHOOK_SECRET` |
| **Resend (SMTP)** | Doğrulama + makbuz + bilgilendirme | Yoksa **sessizce atlar**, link log'a düşer | `KLIVANCE_SMTP_*` |
| **Google OAuth** | "Google ile devam et" | Yoksa **buton gizlenir** | `GOOGLE_OAUTH_CLIENT_*` |
| **GA4 + Meta Pixel** | Ölçüm | Yalnız production (`KLIVANCE_COOKIE_SECURE=1`) | `META_PIXEL_ID` |
| **Google Ads API** | Reklam yönetimi (scriptler) | App runtime **import etmez** | `GOOGLE_ADS_*` |
| **Meta Marketing API** | Reklam yönetimi (scriptler) | App runtime **import etmez** | `META_*` |
| **RxNorm / RxClass (NLM)** | Ad çözümleme son-çaresi | ⚠ **hot-path'te ağ ÇAĞRILMAZ** | anahtarsız |

⚠⚠ **`KLIVANCE_COOKIE_SECURE` ÇİFT GÖREVLİDİR**: oturum çerezinin `Secure` bayrağı **ve**
TÜM GA4/Meta ölçümü buna bağlıdır. Render panelinden silinirse site çalışmaya devam eder
ama (a) oturum çerezi düz HTTP'ye de gider, (b) **reklam ölçümü komple susar** — ve hiçbir
yerde tek satır log çıkmaz. `config.startup_warnings()` açılışta bağırır.

---

## 9. Reddedilmiş mimari kararlar (tekrar önerme)

| Karar | Gerekçe |
|---|---|
| `include_router` | Rota envanteri okuyan iki ağ sessizce körelir |
| `app/db.py` | `connect()` **`saglik/db.py`**'de; çıplak ad bir kez bir aracı yanılttı |
| Şablon motoru / SPA | Gövde ölçen 320 kapıyı kırar |
| `app.person` / `app.account_event` | Sıfır yeni kalıcı veri ilkesi; sorular mevcut tablolardan türetilir |
| `.rail-dar` (64px admin dalı) | 2026-08-08 founder kararıyla kaldırıldı; **TEK genişlik, admin dahil** |
| `ts_headline`'a dokunmak | Yerel PG segfault'u **ortam** sorunudur, ürün regresyonuyla kapatılmaz |
| Kısa ray etiketi ("Sor"/"Ask") | Bir hedefe **TEK ad**: "Klivance'a Sor" bileşiği |

---

**Sonraki:** [`02-DIZIN-VE-DOSYA-ENVANTERI.md`](02-DIZIN-VE-DOSYA-ENVANTERI.md)
