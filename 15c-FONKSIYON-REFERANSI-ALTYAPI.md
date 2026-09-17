# ALTYAPI + VERİ BAĞLAYICILARI — fonksiyon referansı


> **Üretilmiş dosya — elle düzenleme.** Kaynak: Python `ast` ile kaynak ağacın tamamı.
> Ölçüm anı: 2026-09-17 · commit `cb16758d`. Yeniden üretmek için export kökündeki
> `veri/` JSON'larını besleyen çıkarıcıyı koştur (bkz. `18-DEVIR-NOTLARI.md` §6).

**22 modül · 101 fonksiyon/metot.** Kapsam: ALTYAPI + VERİ BAĞLAYICILARI (`saglik/*.py`, `saglik/connectors/`, `saglik/ads/`)

Okuma anahtarı: `_` ile başlayan ad = modül-içi (dışarıdan çağırma); `async def`
işaretlidir. **Modül başlığındaki `⚠⚠` satırları sözleşmedir** — bu referans yalnız
imzayı ve ilk satırı taşır, bir fonksiyona dokunmadan önce dosyanın kendi başlığını oku.

---

## `saglik/__init__.py`

`3 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
saglik — açık sağlık bilgi tabanı ingestion pipeline'ı.
```

</details>

## `saglik/ads/__init__.py`

`2 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Reklam entegrasyonları (Google Ads · Meta Marketing API). Kimlikler env'de; Claude okumaz.
```

</details>

## `saglik/ads/google_ads.py`

`76 satır` · `4 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Google Ads API entegrasyonu — istemci fabrikası + salt-okuma yardımcıları.

Kimlik bilgileri env'den (bkz. config.google_ads_config). google-ads kütüphanesi app runtime
için ŞART DEĞİL — yalnız reklam scriptleri/işleri import eder (lazy import → app'i ağırlaştırmaz).

Kullanım:
    from saglik.ads.google_ads import build_client, get_customer_info
    client, customer_id = build_client()
    info = get_customer_info(client, customer_id)   # {name, currency, time_zone, status, ...}
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `build_client` | `()` | 16 | (GoogleAdsClient, hedef_customer_id) döndürür. Config eksikse RuntimeError. |
| `get_customer_info` | `(client, customer_id: str) -> dict` | 29 | Hesabın temel bilgisi (GAQL customer sorgusu) — para birimi/saat dilimi TEYİDİ + token testi. |
| `list_accessible_customers` | `(client) -> list[str]` | 51 | Bu kimlikle erişilebilen hesap ID'leri (tireSİZ). Bağlantı/kimlik doğrulama duman-testi. |
| `list_campaigns` | `(client, customer_id: str, limit: int = 50) -> list[dict]` | 58 | Hesaptaki kampanyalar (GAQL) — {id, name, status, channel}. İlk yönetim iskeleti için okuma. |

## `saglik/config.py`

`263 satır` · `18 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Yapılandırma: .env + sources.yaml yükleme.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ROOT` | `Path(__file__).resolve().parent.parent` | 11 |
| `_PAZAR_SAGLAYICI` | `{'tr': 'iyzico', 'intl': 'stripe'}` | 81 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `database_url` | `() -> str` | 16 |  |
| `anthropic_key` | `() -> str \| None` | 23 | Anthropic API anahtarı (.env'den). Yoksa None — çağıran taraf zarifçe uyarır. |
| `stripe_secret_key` | `() -> str \| None` | 29 |  |
| `stripe_webhook_secret` | `() -> str \| None` | 34 |  |
| `payment_provider` | `() -> str` | 39 | VARSAYILAN ödeme sağlayıcısı: 'iyzico' (TR-önce) \| 'stripe'. |
| `pay_routing` | `() -> bool` | 52 | PAZARA GÖRE SAĞLAYICI YÖNLENDİRMESİ açık mı? (env `KLIVANCE_PAY_ROUTING=1`) |
| `provider_for_market` | `(pazar: str \| None) -> str` | 84 | Pazar kodu ('tr'\|'intl') → ödeme sağlayıcısı. Yönlendirme kapalıysa ya da pazar |
| `iyzico_config` | `() -> dict \| None` | 93 | iyzico üye işyeri anahtarları (.env). Eksikse None → arayüz zarifçe uyarır. |
| `file_encryption_key` | `() -> str` | 108 | Hasta dosya kütüphanesi Fernet BİRİNCİL (yazma) anahtarı (url-safe base64, 32 byte). |
| `file_encryption_keys` | `() -> list[str]` | 122 | Tüm Fernet anahtarları — BİRİNCİL önce, ardından ESKİ anahtar(lar) (rotasyon için). |
| `app_base_url` | `() -> str` | 136 | Checkout dönüş URL'leri için (sonda / yok). Env yoksa localhost. |
| `google_ads_config` | `() -> dict \| None` | 141 | google-ads istemci yapılandırması (env). Zorunlu alan eksikse None → arayüz zarifçe uyarır. |
| `google_oauth` | `() -> dict \| None` | 167 | Google OAuth istemci bilgisi (env). Eksikse None → "Google ile devam et" gösterilmez. |
| `smtp_config` | `() -> dict \| None` | 177 | SMTP ayarları (e-posta doğrulama maili için). Eksikse None → mailer zarifçe atlar. |
| `startup_warnings` | `() -> list[str]` | 189 | SESSİZ YANLIŞ-YAPILANDIRMA AVCISI — eksik/çelişkili env'i açılışta GÖRÜNÜR yapar. |
| `_uyari_bas` | `() -> None` | 228 | İçe aktarmada (yani uygulama/işçi açılışında) uyarıları stderr'e GÜRÜLTÜLÜ basar. |
| `sources` | `() -> dict` | 251 | sources.yaml içeriğini {source_id: config} olarak döner. |
| `source` | `(source_id: str) -> dict` | 258 |  |

## `saglik/connectors/__init__.py`

`32 satır` · `1 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Connector kayıt defteri: source_id → connector sınıfı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `REGISTRY` | `{'openfda': OpenFDAConnector, 'clinicaltrials': ClinicalTrialsConnector, 'titck': TitckConnector, 'icd11': ICD11Connecto` | 12 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `get_connector` | `(source_id: str) -> BaseConnector` | 24 |  |

## `saglik/connectors/base.py`

`77 satır` · `0 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Tüm connector'ların ortak arayüzü.
```

</details>

### `class BaseConnector(ABC)`  <sub>satır 12</sub>

Bir kaynağı çeker (fetch) → ham katmana yazar → normalize eder.

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, source_id: str)` | 44 |  |
| `fetch` | `(self, limit: int \| None = None) -> Iterator[tuple[str, dict]]` | 55 | (source_key, ham payload) üretir. |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 59 | Bir ham kaydı core.* tablolarına yazar. |
| `ingest` | `(self, limit: int \| None = None) -> int` | 62 | fetch → raw'a yaz → normalize. İşlenen kayıt sayısını döner. |

## `saglik/connectors/clinicaltrials.py`

`149 satır` · `1 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ClinicalTrials.gov v2 connector.

API: https://clinicaltrials.gov/api/v2/studies

⚠⚠ İMLEÇ = TARİH (`YYYY-MM-DD`), pageToken DEĞİL — 2026-07-29'da bu yüzden değişti.
   Eski tasarım `nextPageToken`'ı KALICI su işareti olarak `core.sync_state`'e yazıyordu.
   Ama o token OPAK ve GEÇERSİZLEŞEBİLİR bir değer: 12 saatte bir koşan cron, saatler önce
   alınmış bir token'la geri geldiğinde API **HTTP 400 "Incorrect pageToken format"**
   veriyor (ölçüldü: 352 ms'de düşüyor) → sync-feed işi kırmızı. Geçersizleşen bir değeri
   kalıcı watermark yapmak yapısal hataydı.

   Yeni sözleşme:
     · `cursor` = TAM İŞLENMİŞ son güncelleme tarihi (ISO `YYYY-MM-DD`). Tarih ESKİMEZ.
     · Sorgu `AREA[LastUpdatePostDate]RANGE[cursor,MAX]` + `sort=LastUpdatePostDate:asc`.
     · pageToken YALNIZ tek koşu İÇİNDE sayfalama için kullanılır, ASLA saklanmaz.
     · İmleç DAHİL edicidir → sonraki koşu son günü YENİDEN işler (~1 günlük fazlalık).
       Bu BİLİNÇLİ: upsert idempotent, ama BOŞLUK bırakmak veri kaybıdır. Güvenlik > tasarruf.

⚠ GÜN SINIRINDA KESİLİR, ORTASINDA DEĞİL. Sebep: imleç gün çözünürlüğünde olduğu için bir
  günü yarım bırakırsak sonraki koşu o günü baştan alır ve AYNI YERDE TAKILIRIZ (ilerleme
  sıfır, iş sessizce durur). Ölçüldü: hafta içi ~800-1.050 çalışma/gün, hafta sonu 0 →
  "tam gün" batch'i sınırlı ve güvenli.

⚠ BOOTSTRAP = SON 30 GÜN, tüm arşiv DEĞİL. Bu cron'un işi GÜNCEL KALMAK. Tüm veri 596.277
  kayıt; 8.000/gün kapasiteyle tam tarama ~75 gün sürer ve her tükenişte baştan başlar.
  Tarihsel doldurma AYRI bir toplu iştir (KÜB partisi gibi) — cron'a yüklenmemeli.
  Eski/opak bir imleç (legacy pageToken) bulunursa da bootstrap'a düşülür → prod'daki
  bozuk token bu değişiklikle KENDİLİĞİNDEN temizlenir.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_TARIH` | `re.compile('^\\d{4}-\\d{2}-\\d{2}$')` | 38 |
| `BOOTSTRAP_GUN` | `30` | 39 |
| `SAYFA` | `200` | 40 |

### `class ClinicalTrialsConnector(BaseConnector)`  <sub>satır 49</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `_ertesi` | `(gun: str) -> str` | 56 | TAM işlenmiş bir günün ERTESİ günü. |
| `_baslangic` | `(self, cursor: str \| None) -> str` | 66 |  |
| `fetch` | `(self, limit: int \| None = None, cursor: str \| None = None) -> Iterator[tuple[str, dict]]` | 75 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 125 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_gun` | `(study: dict) -> str \| None` | 43 | Çalışmanın `lastUpdatePostDate` alanı (YYYY-MM-DD) — yoksa None. |

## `saglik/connectors/dailymed.py`

`133 satır` · `1 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
DailyMed SPL connector (NLM) — resmî FDA ilaç etiketi İNDEKSİ + resmî link.

API: https://dailymed.nlm.nih.gov/dailymed/services/v2/spls.json  (sayfalı; ~158k SPL)
Kapsam kararı (2026-07-19): liste kaydı setid + title + published_date verir; TAM klinik
metin (endikasyon/uyarı/etkileşim) SPL XML'inde — 158k XML'i parse etmek openFDA'nın zaten
yaptığını tekrarlar. Bu yüzden connector İNDEKS + RESMÎ LİNK yazar (kanıtlanmış TİTCK-KÜB
deseni): title'dan jenerik/marka/üretici çıkarır, resmî etiket URL'sini warnings'e link
olarak koyar. Tam-metin çıkarım Faz-3 (KÜB extraction PoC deseni). Değer: geniş marka/jenerik
kapsamı (RxNorm çözümlemesini + kart-köprüsünü besler) + otoriter FDA etiket linki.

Lisans: SPL etiketleri KAMU MALI (public domain, FDA). Ticari kullanım serbest.
INCREMENTAL: cursor = sayfa numarası (opak). Idempotency normalize ON CONFLICT'ten.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_MFR` | `re.compile('\\[([^\\]]+)\\]\\s*$')` | 23 |
| `_PAREN` | `re.compile('\\(([^)]+)\\)')` | 24 |
| `_FORMS` | `('TABLET', 'CAPSULE', 'INJECTION', 'SOLUTION', 'SUSPENSION', 'CREAM', 'OINTMENT', 'POWDER', 'GEL', 'SPRAY', 'PATCH', 'LO` | 26 |

### `class DailyMedConnector(BaseConnector)`  <sub>satır 64</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `fetch` | `(self, limit: int \| None = None, cursor: str \| None = None) -> Iterator[tuple[str, dict]]` | 67 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 100 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_parse_title` | `(title: str) -> tuple[str \| None, list[str], str \| None]` | 31 | DailyMed title → (jenerik, [marka], üretici). Biçimler: |

## `saglik/connectors/enforcement.py`

`90 satır` · `1 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
openFDA Drug Enforcement (geri çağırma / FDA RES) connector.

API: https://api.fda.gov/drug/enforcement.json  (~17.8k kayıt, haftalık güncel, 2004-günümüz)
Hasta-güvenliği sinyali: kalite/sterilite/kontaminasyon nedeniyle geri çekilen ilaç ürünleri.
ABD-merkezli (RES); TR-ilgisi sınırlı ama Class-I (yaşamı tehdit) kayıtlar evrensel uyarı değeri.

core.drug_recall'e yazar (openFDA drug/label connector'ının kardeşi; aynı skip-tabanlı sayfalama).
Lisans: kamu malı (ABD Hükümeti). INCREMENTAL: cursor=skip. Idempotency PRIMARY KEY upsert'ten.
```

</details>

### `class EnforcementConnector(BaseConnector)`  <sub>satır 24</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `fetch` | `(self, limit: int \| None = None, cursor: str \| None = None) -> Iterator[tuple[str, dict]]` | 27 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 63 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_first` | `(v)` | 18 |  |

## `saglik/connectors/icd11.py`

`194 satır` · `1 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
WHO ICD-11 connector (hastalıklar — omurga kodlama).

Kimlik: OAuth2 client_credentials. https://icd.who.int/icdapi adresinden ücretsiz
`client_id` + `client_secret` alınır, .env'e konur (ICD11_CLIENT_ID / ICD11_CLIENT_SECRET).

Akış:
  1. Token al (icdaccessmanagement.who.int/connect/token, scope=icdapi_access)
  2. MMS linearizasyon kökünden başla → child URI'lerini izleyerek ağacı gez (BFS)
  3. Her varlığı core.disease'e + code_map(system=icd11)'e yaz

Gezinme büyük (~35k varlık, her biri bir istek) — nazik rate limit + --limit desteği.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `TOKEN_URL` | `'https://icdaccessmanagement.who.int/connect/token'` | 22 |
| `API_BASE` | `'https://id.who.int'` | 23 |

### `class ICD11Connector(BaseConnector)`  <sub>satır 33</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, source_id: str)` | 34 |  |
| `_get_token` | `(self) -> str` | 42 |  |
| `_headers` | `(self, lang: str) -> dict` | 61 |  |
| `_get` | `(self, uri: str, lang: str) -> dict` | 69 |  |
| `_key_of` | `(uri: str) -> str` | 84 |  |
| `_load_done_frontier` | `(self) -> tuple[set[str], deque[str]]` | 87 | Kaydedilmiş payload'lardan resume sınırını kur — API'yi tekrar gezmeden. |
| `fetch` | `(self, limit: int \| None = None) -> Iterator[tuple[str, dict]]` | 114 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 155 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_val` | `(field) -> str \| None` | 26 | ICD-11 alanları {'@language':..,'@value':..} biçiminde gelir. |

## `saglik/connectors/mhra.py`

`159 satır` · `3 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
MHRA Drug Safety Update (GOV.UK) connector → core.corpus (`mhra_dsu`). #763c-A, 2026-09-11, Derya.

KAYNAK: https://www.gov.uk/drug-safety-update — MHRA'nın ilaç güvenlik uyarıları (doz / kontrendikasyon /
eşik değişiklikleri; TR'de aynı moleküller). Atom feed `/drug-safety-update.atom?page=N` (50 giriş/sayfa,
en yeni önce) + GOV.UK içerik API'si `/api/content/drug-safety-update/<slug>` (JSON: title · description ·
details.body HTML · details.metadata.therapeutic_area · first_published_at · public_updated_at ·
withdrawn_notice). HTML kazıma YOK; iki uç da robots.txt `User-agent: *` bloğunda serbest (ölçüldü 11.09).

LİSANS (kaynağın kendi sayfasından, birebir — `scratchpad/_derya_kaynak_arastirma_2026-09-11.md`):
  gov.uk/drug-safety-update footer: "All content is available under the Open Government Licence v3.0,
  except where otherwise stated". OGL v3 (nationalarchives.gov.uk/doc/open-government-licence/version/3,
  WebFetch 11.09): "exploit the Information commercially and non-commercially" · atıf şartı "acknowledge
  the source of the Information in your product or application by including or linking to any
  attribution statement specified" · varsayılan atıf: "Contains public sector information licensed under
  the Open Government Licence v3.0."
  ⚠ Lisans metni `core.corpus.license`a OLDUĞU GİBİ yazılır; "except where otherwise stated" — makale
    düzeyinde farklı beyan görülürse (ölçülmedi, 0 örnek) o makale elle incelenir.

İMLEÇ: OPAK (Atom sayfa numarası, `DURABLE_CURSOR=False`). Tükenince `run_sync` NULL'a çeker = bir sonraki
koşum 1. sayfadan (en yeniden) başlar → güncellemeler yeniden süpürülür; idempotentlik `core.corpus`
`(source, doc_id)` ON CONFLICT upsert'inden. ⚠ `limit` sayfa ortasında dolarsa imleç AYNI sayfada kalır
(o sayfa bir sonraki koşumda yeniden işlenir — idempotent, en çok 49 fazladan istek); `--batch` 50'nin
katı verilirse hiç olmaz.
NAZİK: `sources.yaml rate_limit_per_min: 40` (≥1,5 s/istek, Client RateLimiter) + tanıtıcı UA.
⚠ BULUT-IP ENGELİ ÖLÇÜLMEDİ (yerelden 200; cron runner'ından HEAD atılmadı) — sync-feed matrisine
  ekleme kararı Ömer'in. ⚠ Yalnız yerel DB (connect() .env DATABASE_URL).
GERİ ÇEKİLME DAMGASI (#763c-c, Ömer onayı 11.09): INSERT `retraction_status='not_applicable'` +
  `checked_at=now()` + `retraction_source` gerekçesi yazar — kurum güvenlik uyarısı dergi makalesi
  değildir (cdc_yellowbook/livertox/lactmed ile aynı sınıf). Damgasız bırakmak her MHRA künyesini
  amber 'olculemedi' yapardı (engine 2026-08-03 dersi). ⚠ Bu dosya `geri_cekilme_verify._IZINLI`
  kümesinde GEREKÇELİ kayıtlıdır; kolonlara dokunan yeni connector oraya satır eklemeden CI kırmızı olur.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `UA` | `'Klivance-KB/1.0 (+https://klivance.com; info@klivance.com)'` | 41 |
| `ATOM_YOL` | `'/drug-safety-update.atom'` | 42 |
| `ICERIK_YOL` | `'/api/content/drug-safety-update/{slug}'` | 43 |
| `SAYFA` | `50` | 44 |
| `LISANS` | `'Open Government Licence v3.0 (OGL, MHRA/GOV.UK) — kaynağın kendi sayfası (gov.uk/drug-safety-update): "All content is a` | 46 |
| `_ENTRY` | `re.compile('<entry>(.*?)</entry>', re.S)` | 53 |
| `_ALAN` | `{'id': re.compile('<id>(.*?)</id>', re.S), 'updated': re.compile('<updated>(.*?)</updated>', re.S), 'title': re.compile(` | 54 |
| `_HREF` | `re.compile('<link[^>]*rel="alternate"[^>]*href="([^"]+)"')` | 60 |
| `_ETK` | `re.compile('<[^>]+>')` | 61 |
| `_BLOK` | `re.compile('</?(p\|div\|br\|li\|h[1-6]\|tr\|section\|table\|ul\|ol\|dd\|dt)\\b[^>]*>', re.I)` | 62 |

### `class MhraDsuConnector(BaseConnector)`  <sub>satır 110</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, source_id: str)` | 114 |  |
| `fetch` | `(self, limit: int \| None = None, cursor: str \| None = None) -> Iterator[tuple[str, dict]]` | 122 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 148 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `atom_girdileri` | `(xml: str) -> list[dict]` | 65 | Atom sayfası → [{id, updated, title, summary, href, slug}] (sıra korunur). |
| `html_metin` | `(parca: str) -> str` | 78 | İçerik API'sinin HTML gövde parçası → düz metin (blok → satır, varlıklar açılır). |
| `govde_kur` | `(payload: dict) -> tuple[str, str]` | 88 | (title, body) — içerik API JSON'undan; başlık = makale başlığı (ilaç + konu, ayırt edici). |

## `saglik/connectors/openfda.py`

`130 satır` · `1 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
openFDA Drug Label connector.

API: https://api.fda.gov/drug/label.json
Sayfalama: skip + limit (limit<=100, skip<=25000). Büyük hacim için
`search_after` yerine tarih aralığıyla parçalama gerekir; Faz 1 pilotu skip kullanır.
```

</details>

### `class OpenFDAConnector(BaseConnector)`  <sub>satır 25</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `fetch` | `(self, limit: int \| None = None, cursor: str \| None = None) -> Iterator[tuple[str, dict]]` | 28 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 73 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_first` | `(payload: dict, key: str) -> str \| None` | 17 | openFDA alanları liste döner; ilk elemanı düz metne çevirir. |

## `saglik/connectors/orphanet.py`

`74 satır` · `0 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Orphanet nadir hastalık connector (Orphadata product1).

Kaynak: https://www.orphadata.com/data/xml/en_product1.xml (~11.6k nadir hastalık, tek büyük XML)
Değer: ddx "zebra" kör noktası — ICD-11 KB'sinde yaygın hastalıklar var, nadir hastalıklar zayıf.
Orphanet ad + eş anlamlı + ICD-10 xref ile nadir hastalık TANIMA'yı güçlendirir (klinik tanım
metni product1'de YOK; o ayrı üründe ve lisansı farklı → yalnız ad/eş-anlam/xref alınır).

Lisans: **CC-BY-4.0** (XML'in kendisi doğrular; ticari kullanım açıkça serbest, atıf gerekir).
Sayfalama YOK (tek dosya) → INCREMENTAL=False, her koşu tam-yenileme (idempotent upsert).
Streaming iterparse (sabit bellek; dosya ~30MB). core.disease'e yazar (source_id='orphanet').
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_UA` | `{'User-Agent': 'Klivance-KB/1.0 (clinical decision support; info@klivance.com)'}` | 20 |

### `class OrphanetConnector(BaseConnector)`  <sub>satır 23</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `fetch` | `(self, limit: int \| None = None, cursor: str \| None = None) -> Iterator[tuple[str, dict]]` | 26 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 51 |  |

## `saglik/connectors/titck.py`

`161 satır` · `2 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
TİTCK KÜB/KT connector (Türkiye ilaçları).

API yok; sitenin DataTables server-side endpoint'i kullanılır — bu HTML kazıma
DEĞİL, sayfalı JSON. Akış:
  1. /kubkt sayfasını çek → session cookie + inline `_token` (Laravel CSRF)
  2. /getkubktviewdatatable'a DataTables POST payload'ı ile sayfa sayfa istek
  3. name/element/firmName + KÜB/KT PDF linklerini normalize et

Nazik davran: rate_limit_per_min düşük tutulur (sources.yaml).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `COLUMNS` | `['name', 'element', 'firmName', 'confirmationDateKub', 'confirmationDateKt', 'documentPathKub', 'documentPathKt']` | 22 |
| `TOKEN_RE` | `re.compile('_token:\\s*"([^"]+)"')` | 25 |
| `HREF_RE` | `re.compile('href="([^"]+)"')` | 26 |

### `class TitckConnector(BaseConnector)`  <sub>satır 44</sub>

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `_get_token` | `(self) -> str` | 48 |  |
| `_payload` | `(self, token: str, start: int, length: int) -> dict` | 55 |  |
| `fetch` | `(self, limit: int \| None = None) -> Iterator[tuple[str, dict]]` | 71 |  |
| `normalize` | `(self, conn, source_key: str, payload: dict) -> None` | 100 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_extract_href` | `(cell_html: str \| None) -> str \| None` | 29 | documentPath* alanları <div><a href="...pdf">...</a></div> içerir. |
| `_clean` | `(s: str \| None) -> str \| None` | 37 | Boşlukları kırp + HTML entity'leri çöz (&apos; → '). |

## `saglik/db.py`

`162 satır` · `9 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
PostgreSQL erişimi ve ham katmana yazma.

BAĞLANTI HAVUZU (2026-07-29, denetim bulgusu V1): eskiden her `connect()` YENİ bir
bağlantı açıyordu (TCP + TLS + auth el sıkışması; Render'da 25-60 ms). main.py'de 70+
`with connect()` var, tek sohbet turu birkaç bağlantı açıyor ve biri LLM streaming'i
boyunca açık kalıyordu → `WEB_WORKERS` × eşzamanlı istek arttığında Postgres
`FATAL: too many connections` veriyordu.

⚠ ÇAĞRI API'Sİ DEĞİŞMEDİ: `with connect() as conn:` aynen çalışır (commit'te commit,
istisnada rollback — psycopg.connect() ile birebir aynı semantik). Havuz yalnız
bağlantının nereden geldiğini değiştirir.

- Kill-switch: `KLIVANCE_DB_POOL=0` → eski davranış (her çağrıda yeni bağlantı).
- Boyut: `KLIVANCE_DB_POOL_MIN` (vars. 1) / `KLIVANCE_DB_POOL_MAX` (vars. 8) — WORKER BAŞINA.
- Havuz kurulamazsa (modül yok / DB kapalı) sessizce doğrudan bağlantıya düşer.
- `connect_direct()` = havuzu BYPASS eden kaçış yolu (migrate, bakım scriptleri).
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_pool_reset` | `(conn) -> None` | 34 | Havuza dönen bağlantıyı TEMİZ bırak. Kritik: oturum-düzeyi `SET` (ör. retrieval'ın |
| `_make_pool` | `()` | 44 |  |
| `_get_pool` | `()` | 64 | Süreç-başına tekil havuz (tembel kurulur). Kullanılamıyorsa None → doğrudan bağlantı. |
| `_after_fork` | `() -> None` | 88 | gunicorn worker fork'unda: ÇOCUK ebeveynin soketlerini ASLA kullanmamalı. |
| `connect` | `()` | 101 | Havuzdan bir bağlantı ödünç al (yoksa doğrudan aç). Semantik psycopg.connect() ile aynı: |
| `connect_direct` | `()` | 125 | Havuzu BYPASS eden bağlantı — migrate/bakım scriptleri gibi uzun süren, havuz slotunu |
| `close_pool` | `() -> None` | 132 | Havuzu kapat (uygulama kapanışı / test temizliği). İdempotent. |
| `apply_schema` | `(schema_path: str) -> None` | 143 |  |
| `upsert_raw` | `(conn, source_id: str, source_key: str, payload: dict) -> None` | 151 | Ham kaydı idempotent olarak yazar (aynı anahtar → günceller). |

## `saglik/etiket_tarih.py`

`143 satır` · `4 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Etiket TAZELİĞİ — ham kaynak payload'ından tarih + sürüm çıkarımı (#70).

Hekime "bu bilgi ne zamana ait" diyebilmek için `core.drug` üç kolon taşır:
  · `label_date`      — tarih (DATE)
  · `label_date_kind` — o tarihin NE ANLAMA geldiği (aşağı bkz)
  · `label_version`   — kaynağın kendi sürüm sayacı (varsa)

⚠⚠ ÜÇ KAYNAĞIN TARİHİ ÜÇ FARKLI ŞEY DEMEK — tek etiketle sunmak sessiz yanlış beyandır:
  · openfda  `effective_time`      = etiketin YÜRÜRLÜK tarihi        → kind='yururluk'
  · dailymed `published_date`      = SPL sürümünün YAYIMLANMA tarihi → kind='yayim'
  · titck    `confirmationDateKub` = KÜB ONAY tarihi                 → kind='onay'
`label_date_kind` bu yüzden ZORUNLU eştir: tarih yazılıyorsa türü de yazılır, tersi de.
Yüzeye çıkaran katman (ref_body — Kamil) etiketi BU kolondan üretmeli, kaynak adından
tahmin ETMEMELİ.

⚠ `label_version` = kaynağın SPL sürüm sayacı (openfda `version`, dailymed `spl_version`;
  ikisi de AYNI şeyi sayar, bu yüzden `kind` eşi YOK). TİTCK'te karşılığı yoktur → NULL.
  ⚠⚠ SÜRÜM ÜRÜNLER ARASINDA KIYASLANAMAZ — aynı setid'in KENDİ geçmişinde artar.
  "sürüm 9 > sürüm 2 demek daha taze" ÇIKARIMI YANLIŞTIR; sürüme göre SIRALAMA YAPMA.
  Değeri şurada: tarih AYNI kalırken sürüm artmışsa revizyon olmuştur — tarih kolonunun
  tek başına göremediği hâl budur.

──────────────────────────────────────────────────────────────────────────────
HİJYEN FRENİ — NİYE VAR (yerelde tam taramayla ölçüldü, 2026-08-03)
Hekime "yürürlük: 2029" ya da "onay: 0022" basmak kaynağa güveni tek başına bitirir;
bu üründe yanlış cevaptan tehlikelidir çünkü YANLIŞ GÜVEN üretir. Ölçülen kirlilik:
  · openfda  effective_time: 6 satır biçimsiz ('2010110126', '201001004' gibi 9-10 hane),
             7 satır GELECEK tarihli (20260930 … 20290530), 3 satır <1990 (en eski 19781018)
  · titck    confirmationDateKub: biçim 15.536/15.536 TEMİZ — ama DEĞERLERDE 58 satır
             <1990 ve en eskisi **0022-04-15**. ⚠ Bu, deponun kendi dersinin tarih
             yüzeyindeki hâli: "biçim geçerli" ≠ "değer makul". Regex'e bakıp
             "TİTCK temiz" demek yanlış olurdu.
  · dailymed published_date: 158.014/158.014 biçim temiz, 0 gelecek, 0 <1990.
Fren: biçim regex'i + `[ALT_SINIR, bugün]` aralığı. Eleme SESSİZ DEĞİL — çağıran bir
`red` kodu alır ve sayabilir (kaç satır, NEDEN elendi).

⚠ ÜST SINIR "bugün" OLDUĞU İÇİN SONUÇ ZAMANA BAĞLIDIR: bugün elenen 20260930 kaydı
  Ekim 2026'da kabul edilir. Bilinçli — "gelecekte yürürlüğe girecek etiket" bugün
  gösterilemez. Bu yüzden `bugun` PARAMETRE: süit tarihi çiviler, üretim `date.today()`
  kullanır (yoksa süit takvim döndükçe kendiliğinden kırmızıya döner).

⚠ `%b` KULLANMA — Python `strptime` LC_TIME'a duyarlıdır; Türkçe locale'de 'Oct'
  ayrıştırılamaz. Aynı gerekçeyle Postgres `to_date` da kullanılmaz: ay adı ASCII
  sözlükten çözülür (`_AY`), böylece davranış makineden bağımsızdır.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ALT_SINIR` | `date(1990, 1, 1)` | 55 |
| `SURUM_UST` | `2147483647` | 59 |
| `_AY` | `{'Jan': 1, 'Feb': 2, 'Mar': 3, 'Apr': 4, 'May': 5, 'Jun': 6, 'Jul': 7, 'Aug': 8, 'Sep': 9, 'Oct': 10, 'Nov': 11, 'Dec': ` | 61 |
| `KAYNAK` | `{'openfda': ('effective_time', 'yururluk', re.compile('^\\d{8}$')), 'dailymed': ('published_date', 'yayim', re.compile('` | 65 |
| `SURUM_ALANI` | `{'openfda': 'version', 'dailymed': 'spl_version'}` | 72 |
| `KINDLER` | `frozenset((k for _, k, _ in KAYNAK.values()))` | 74 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_ham` | `(payload: dict, alan: str) -> str \| None` | 77 |  |
| `_cevir` | `(source_id: str, s: str) -> date \| None` | 85 | Biçimi doğrulanmış metni date'e çevirir. Locale'e DOKUNMAZ. |
| `coz_tarih` | `(source_id: str, payload: dict, bugun: date \| None = None) -> tuple[date \| None, str \| None, str \| None]` | 103 | (tarih, kind, red_kodu) döndürür. Tarih varsa kind DOLU, red None. |
| `coz_surum` | `(source_id: str, payload: dict) -> tuple[int \| None, str \| None]` | 129 | (sürüm, red_kodu). Red: 'kaynak_yok' · 'alan_bos' · 'bicim' · 'aralik'. |

## `saglik/filecrypto.py`

`69 satır` · `4 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Hasta dosyası uygulama-katmanı şifrelemesi (Fernet, AES128-CBC + HMAC).

Kalıcı dosya kütüphanesi at-rest ŞİFRELİ tutar (KVKK). Anahtar(lar): KLIVANCE_FILE_KEY
(birincil/yazma) + opsiyonel KLIVANCE_FILE_KEY_OLD (eski/rotasyon; bkz. config).
Anahtar yoksa ilk çağrıda net hata → route err=svc.

Rotasyon: MultiFernet yazmada DAİMA birincil anahtarı kullanır, okumada tüm anahtarları
sırayla dener → eski anahtarla şifreli veri hâlâ çözülür. `scripts/rotate_file_key.py`
tüm satırları birincil anahtara yeniden şifreler.
```

</details>

### `class FileKeyError(RuntimeError, ValueError)`  <sub>satır 20</sub>

Şifreleme ANAHTARI sorunu (eksik / bozuk / yanlış uzunlukta).

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_mf` | `() -> MultiFernet` | 31 |  |
| `encrypt_bytes` | `(data: bytes) -> bytes` | 48 | Ham dosya baytlarını şifrele (BİRİNCİL anahtar) → DB bytea'ya yazılacak token. |
| `decrypt_bytes` | `(token: bytes) -> bytes` | 53 | Şifreli token'ı çöz (tüm anahtarlar denenir) → orijinal baytlar. |
| `reencrypt_bytes` | `(token: bytes) -> bytes` | 62 | Var olan token'ı (herhangi bir anahtarla çözüp) BİRİNCİL anahtara yeniden şifrele. |

## `saglik/http.py`

`102 satır` · `0 fonksiyon` · `3 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HTTP istemcisi — tarayıcı TLS parmak izi taklidi + rate limit + retry.

Bazı kaynaklar (ör. clinicaltrials.gov) Cloudflare bot koruması ardında;
düz httpx/requests TLS parmak izi 403 yer. curl_cffi Chrome'u taklit ederek geçer.
```

</details>

### `class RateLimiter(—)`  <sub>satır 19</sub>

Dakikada N istekle basit throttle.

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, per_min: int)` | 22 |  |
| `wait` | `(self) -> None` | 26 |  |

### `class HTTPError(Exception)`  <sub>satır 33</sub>

Yeniden denenebilir HTTP hatası (429/5xx).

### `class Client(—)`  <sub>satır 37</sub>

Nazik HTTP istemcisi: Chrome TLS taklidi + rate limit + üstel backoff.

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, base_url: str = '', rate_limit_per_min: int = 60, timeout: float = 30.0)` | 40 |  |
| `_url` | `(self, path: str) -> str` | 46 |  |
| `get_json` | `(self, path: str, params: dict \| None = None, headers: dict \| None = None) -> dict` | 57 |  |
| `get_text` | `(self, path: str, params: dict \| None = None) -> str` | 68 | Ham metin (ör. token çıkarmak için sayfa HTML'i). Session cookie'leri saklar. |
| `post_json` | `(self, path: str, data: dict \| None = None, headers: dict \| None = None) -> dict` | 83 |  |
| `close` | `(self) -> None` | 94 |  |
| `__enter__` | `(self) -> 'Client'` | 97 |  |
| `__exit__` | `(self, *exc) -> None` | 100 |  |

## `saglik/ingest_state.py`

`129 satır` · `6 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Sürekli besleme hattı — watermark (core.sync_state) + koşu logu (core.ingest_log) +
artımlı/idempotent orchestrator (run_sync).

Idempotency connector normalize'lerinin ON CONFLICT upsert'lerinden gelir; bu katman
"nereye kadar çekildi" imlecini (cursor) ve her koşunun gözlemlenebilir logunu ekler.

Kullanım (CLI):  python scripts/sync_feed.py <source> --batch 500
Kullanım (kod):  from saglik.ingest_state import run_sync; run_sync("openfda", batch=500)
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `get_state` | `(conn, source_id: str) -> dict` | 17 | Kaynağın watermark durumu. Yoksa sıfır-durum döner (cursor=None → baştan). |
| `advance_state` | `(conn, source_id: str, cursor, last_key, added: int, success: bool = True) -> None` | 29 | Watermark'ı ilerlet: cursor + last_key + kümülatif toplam + zaman damgaları (upsert). |
| `begin_run` | `(conn, source_id: str, cursor_before) -> int` | 47 | Koşu logu satırı aç (status='running'). Döner: run_id. |
| `finish_run` | `(conn, run_id: int, status: str, records: int, cursor_after, error: str \| None = None) -> None` | 56 | Koşu logunu kapat (status/records/cursor_after/error + finished_at). |
| `run_sync` | `(source_id: str, batch: int = 500, commit_every: int = 50) -> dict` | 66 | Bir kaynağı artımlı çek: watermark'tan devam et → batch kadar kayıt upsert et → |
| `_close` | `(connector) -> None` | 124 |  |

## `saglik/kimlik.py`

`107 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
İLAÇ KİMLİĞİ KAPISI — bir metin gerçekten bir ilaç adı mı?

TEK DOĞRU KAYNAK. Üç ayrı yer bunu çağırır, hiçbiri KENDİ kopyasını yazmaz
(`ci_kapi_verify` dersi: kapıyı taklit eden test yeşil basarken kapı kırmızıydı):
  · `saglik/connectors/*` — YAZMA anında (çöp DB'ye hiç girmesin)
  · `scripts/extract_kub_batch.py --onayla` — ONAY anında (çöp klinik yüzeye çıkmasın)
  · `scripts/cop_generic_temizle.py` — TEMİZLİK anında (girmiş olanı geri al)

⚠⚠ NEDEN VAR — ölçülen olay: `core.drug.generic_name` hekime KİMLİK olarak basılır
(`/etkilesim` → "Bu ikisi aynı etken maddeyi içeriyor: X") ve ARANABİLİRDİR. TİTCK
çok bileşenli ürünlerde etken madde kolonuna ilaç adı yerine şunları yazıyor:
    "KÜB'E BAKINIZ"                 → belgeye atıf
    "BILIM İLAÇ SAN. VE TIC. A.Ş."  → ruhsat sahibi firma  (içerik: LAKSOTEK/bisakodil)
    "8699525710048"                 → barkod               (içerik: ZOMRANIP-T göz damlası)
DailyMed/openFDA'da da başlık ayrıştırma artıkları var ('7%', '3350', NDC kodu).
Canlıda ölçüldü (2026-08-01): `bilim` arayan hekime **"Bu ikisi aynı etken maddeyi
içeriyor: BILIM İLAç SAN. VE TIC. A.Ş."** basılıyordu — `prednol→LOTEPREDNOL`
ailesinden YANLIŞ KİMLİK hatası.

⚠⚠ KAPI ŞART, TEMİZLİK TEK BAŞINA KALICI DEĞİL: `dailymed` ve `openfda`
`.github/workflows/sync-feed.yml` matrisinde ve cron **12 saatte bir DOĞRUDAN PROD'a**
yazıyor; connector'lar `ON CONFLICT ... generic_name = EXCLUDED.generic_name` yapıyor
→ yalnız satırı temizlemek çöpü bir sonraki koşumda GERİ GETİRİR.

⚠ KÖK NEDEN KAYNAK VERİDİR, MODEL DEĞİL — LLM bu alana hiç dokunmaz.

⚠ DAR TUT: geniş regex kurt masalı okur. Desenler 4.741 anahtarın TAMAMINA karşı iki
yönlü ölçüldü → 24 yakalandı, YANLIŞ ALARM 0. Özellikle elenen tuzaklar:
  · 'bkz' ALT-DİZGE olarak aranamaz → LEBRIKIZUMAB-LBKZ meşru bir ilaçtır
  · 'içerir' fiil imzası sayılamaz  → "…41,7 G EKSTRE (DER 1:38,5) IÇERIR." meşru
  · uzunluk ölçüt DEĞİL → 454 karakterlik amino asit bileşimleri MEŞRU
  · kısalık ölçüt DEĞİL → 'ÜRE' (3 krk) meşru bir etken maddedir
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_COP_ATIF` | `re.compile('\\b(bkz\|bak[iı]n[iı]z)\\b')` | 63 |
| `_COP_FIRMA` | `re.compile('(san\\.?\\s*ve\\s*ti[cs]\|sanayi\\s*ve\\s*ticaret\|\\ba\\.\\s*ş\\.\|ltd\\.?\\s*şti\|\\bholding\\b)')` | 65 |
| `_COP_HARFSIZ` | `re.compile('^[^a-zçğıöşü]*$')` | 68 |
| `_COP_CUMLE` | `re.compile('\\b(içerdiğinden\|içeriyor\|kullanılır\|edilmiştir)\\b')` | 73 |
| `_COP_DESENLER` | `(('atıf', _COP_ATIF), ('firma', _COP_FIRMA), ('harfsiz', _COP_HARFSIZ), ('cümle', _COP_CUMLE))` | 75 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `cop_generic_key` | `(key: str) -> list[str]` | 79 | `key` bir ilaç adı DEĞİLSE eşleşen çöp sınıflarını döndür (temizse boş liste). |
| `temiz_generic` | `(ad: str \| None) -> str \| None` | 85 | Connector YAZMA kapısı: çöp kimlikse **None** döndür (satırı DÜŞÜRME). |

## `saglik/redact.py`

`114 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Deterministik kimlik (PII) redaksiyonu — KVKK güvenlik ağı.

LLM talimatına (MAP promptu) EK olarak, DB'ye yazılan / Anthropic'e tekrar giden metinden
Türk kimlik verisini DETERMİNİSTİK olarak temizler. Saf-fonksiyon (yalnız `re`) → import
döngüsü yok. redact_pii(text) -> (temizlenmiş_metin, flags_dict).

Politika (conservatif, yanlış-pozitif klinik veriyi bozmasın):
- TC Kimlik No: 11 hane + resmi checksum → REDAKTE ([TC]).  (rastgele 11 haneyi değil.)
- Telefon (05xx / +90 5xx): REDAKTE ([TEL]).
- E-posta: REDAKTE ([E-POSTA]).
- Ad-soyad örüntüsü (2+ ardışık Baş-harfli sözcük): yalnız BAYRAK (redakte etmez) — klinik
  epinomileri (Parkinson Hastalığı, Tip 2 Diyabet) yanlış redakte etmemek için.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_TC_RE` | `re.compile('(?<!\\d)(\\d{11})(?!\\d)')` | 19 |
| `_TC_SEP_RE` | `re.compile('(?<!\\d)(\\d{2,11}(?:[ .\\-]\\d{2,11}){1,3})(?!\\d)')` | 25 |
| `_PHONE_RE` | `re.compile('(?<!\\d)(?:\\+?90[\\s-]?)?0?5\\d{2}[\\s-]?\\d{3}[\\s-]?\\d{2}[\\s-]?\\d{2}(?!\\d)')` | 27 |
| `_EMAIL_RE` | `re.compile('[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}')` | 28 |
| `_NAME_RE` | `re.compile('\\b([A-ZÇĞİÖŞÜ][a-zçğıöşü]{1,})(\\s+[A-ZÇĞİÖŞÜ][a-zçğıöşü]{1,}){1,2}\\b')` | 30 |
| `_CLINICAL_STOP` | `{'parkinson', 'alzheimer', 'hashimoto', 'crohn', 'graves', 'cushing', 'addison', 'wilson', 'behçet', 'behcet', 'tip', 'd` | 32 |
| `_CLINICAL_SUFFIX` | `{'hastalığı', 'hastalık', 'hastaligi', 'hastalik', 'sendromu', 'sendrom', 'tiroiditi', 'sendromudur', 'hastalığ', 'belir` | 40 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_tc_valid` | `(s: str) -> bool` | 44 | Resmi TC Kimlik No checksum doğrulaması (yanlış-pozitifleri eler). |
| `redact_pii` | `(text: str, names: bool = False, ilac_adlari = None)` | 54 | Metinden PII'yi temizle. Döner: (temizlenmiş_metin, {tur: adet}). |

## `saglik/web/__init__.py`

`1 satır` · `0 fonksiyon` · `0 sınıf`

