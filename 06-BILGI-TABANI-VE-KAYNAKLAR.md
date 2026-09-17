# 06 — BİLGİ TABANI, KAYNAKLAR VE VERİ HATTI

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
>
> ⚠⚠ **BU DOSYADA KB SATIR SAYISI YOKTUR — bilinçli.** Hacimler bir kez bayatladı ve
> **prod KB ≠ yerel KB**'dir. Sayı gerekiyorsa ölç; reçete §7'de.

---

## 1. Veri nerede — topoloji (founder: *"D:\sağlık datalar burada, hiç unutma"*)

```
┌─────────────────────────────────┐        ┌─────────────────────────────────┐
│  YEREL  (D:\sağlık)             │        │  PROD  (Render / Oregon)        │
│  Docker `saglik-pg`             │        │  PostgreSQL 18                  │
│  ┌───────────────────────────┐  │        │  ┌───────────────────────────┐  │
│  │ core.*  BİLGİ TABANI      │──┼──(1)──▶│  │ core.*                    │  │
│  │ app.*   (yerel test)      │  │  elle  │  │ app.*   CANLI KULLANICI   │  │
│  │ raw.*   ~4 GB  PROD'A     │  │        │  │                           │  │
│  │         ALINMAZ           │  │        │  └───────────▲───────────────┘  │
│  └───────────────────────────┘  │        │              │                  │
└─────────────────────────────────┘        └──────────────┼──────────────────┘
                                                          │ (2) cron, 12 saatte bir
                          ┌───────────────────────────────┴──────────────────┐
                          │ .github/workflows/sync-feed.yml                  │
                          │ secret DATABASE_URL = **RENDER**                 │
                          │ openfda + clinicaltrials → DOĞRUDAN PROD'A       │
                          └──────────────────────────────────────────────────┘
```

**İki ayrı besleme yolu — karıştırma:**

| # | Yol | Ne taşır | Sonuç |
|---|---|---|---|
| **(1)** | **YEREL → PROD, elle** | `scripts/migrate_kb_delta.py` · `migrate_corpus.py` (dry-run varsayılan) | Bulut-IP engelli (TİTCK) ya da tek seferlik toplu veri |
| **(2)** | **CRON → PROD (yerel pay ALMAZ)** | `openfda` + `clinicaltrials` | **Prod bu ikisinde yerelden İLERİDE** |

### ⚠⚠ PROD DB politikası

`PROD_DATABASE_URL` `.env`'de **vardır** → kural teknik zorunluluk değil **POLİTİKA**:

- Prod DB'ye **founder onayı olmadan BAĞLANILMAZ.**
- Onaylıysa **salt-okunur**: `SET default_transaction_read_only=on`.
- ⚠ *"Prod'da X yok"* iddiası **proddan okunarak** kurulur. Yerelden çıkarım bir kontrol
  grubunu sessizce çürüttü ve **2 gün fark edilmedi**.
- ⚠ *"Yerelde X"* prod için kanıt **değildir**. Onaysızsa iddia **canlı yüzeyden**
  (curl/HTTP) ölçülerek kurulur.

---

## 2. Şema `core` — bilgi tabanı

| Tablo | İçerik |
|---|---|
| `core.drug` | İlaç kartı (jenerik, markalar, etiket bölümleri, kaynak, tarih/sürüm) |
| `core.disease` | Hastalık (ICD-11 kodu, başlık, Orphanet) |
| `core.corpus` | **Klinik literatür korpusu** + `tsv` GIN indeksi (FTS) + `license` |
| `core.code_map` | Kod eşlemeleri (ICD-11, RxNorm) |
| `core.adverse_event` | FAERS yan etki |
| `core.drug_disease` | İlaç ↔ hastalık bağı |
| `core.name_alias` | Ad takma/eşanlam |
| `core.drug_recall` | openFDA enforcement (geri çağırma) |
| `core.drug_class_cache` | RxClass ATC/MoA önbelleği |
| `core.kub_extract` | TİTCK KÜB yapılandırılmış çıkarımı |
| `core.sgk_odeme` + `core.sgk_odeme_surum` | SGK Ek-4/A liste durumu |
| `core.retracted_pub` | Geri çekilme hattı |
| `core.symptom` + `core.symptom_disease` | HPO fenotip → nadir hastalık |
| `core.trial` + `core.trial_disease` | ClinicalTrials.gov |
| `core.corpus_disease` | Korpus ↔ hastalık bağı |
| `core.sync_state` | **Besleme watermark'ı** (cursor **OPAK**) |
| `core.ingest_log` | Koşu logu |
| `core.faers_fetch` | FAERS çekim durumu |

Kolon kolon: [`04-VERITABANI.md`](04-VERITABANI.md).

---

## 3. Connector mimarisi — `saglik/connectors/`

### 3.1 Sözleşme (`base.BaseConnector`)

```python
class BaseConnector(ABC):
    INCREMENTAL: bool = False      # fetch(cursor=) ile watermark'tan devam edebilir mi
    DURABLE_CURSOR: bool = False   # imleç eskimez mi (tükenmede korunur)

    def fetch(self, limit=None) -> Iterator[tuple[str, dict]]: ...   # (source_key, payload)
    def normalize(self, conn, source_key, payload) -> None: ...      # core.* tablolarına yaz
    def ingest(self, limit=None) -> int: ...                         # fetch → raw → normalize
```

Idempotency `normalize`'in `ON CONFLICT` upsert'lerinden gelir → **yeniden çalıştırmak güvenli.**

### ⚠⚠ İMLECİN İKİ TÜRÜ VAR — hangisini yazdığını BİL

| Tür | `DURABLE_CURSOR` | Tükenmede | Örnek |
|---|---|---|---|
| **GEÇİCİ / OPAK** (varsayılan) | `False` | `run_sync` imleci **NULL'a çeker** = "başa dön, güncellemeleri yeniden süpür" | ClinicalTrials `nextPageToken` |
| **KALICI** | `True` | İmleç **korunur**; `exhausted=True` yaparken `next_cursor` ANLAMLI bırakılmalı | tarih / sıra no |

⚠ Opak bir değeri kalıcı watermark yapmak **hatadır**: ClinicalTrials'ın `nextPageToken`'ı
6 saat sonra **HTTP 400 "Incorrect pageToken format"** veriyordu ve besleme işi kırmızıya
düşüyordu. Tersi de hata: kalıcı imleci NULL'lamak, her yetişmeden sonra **tüm arşivi baştan**
taratır.

⚠ `core.corpus`a yazan connector `retraction_status` / `retraction_checked_at` /
`retraction_source` kolonlarına **dokunuyorsa** `scratchpad/geri_cekilme_verify._IZINLI`'ye
**gerekçeli satır** eklemeden CI kırmızı olur (izinli-küme kapısı, bilinçli — örnek
`connectors/mhra.py`, #763c-c).

### 3.2 Kayıt defteri (`connectors/__init__.REGISTRY`)

| `source_id` | Sınıf | Erişim | Lisans (sources.yaml'dan) |
|---|---|---|---|
| `openfda` | `OpenFDAConnector` | api | public domain (US Government) |
| `dailymed` | `DailyMedConnector` | api | public domain (FDA SPL) |
| `enforcement` | `EnforcementConnector` | api | public domain (US Government) |
| `clinicaltrials` | `ClinicalTrialsConnector` | api | public domain |
| `icd11` | `ICD11Connector` | api (**OAuth2 client_credentials**) | WHO terms (ücretsiz, kayıt gerekli) |
| `titck` | `TitckConnector` | **scrape** (DataTables server-side endpoint) | kamu kurumu — nazik erişim |
| `orphanet` | `OrphanetConnector` | bulk (tek büyük XML) | **CC-BY-4.0** |
| `mhra_dsu` | `MhraDsuConnector` | api (Atom + GOV.UK content API) | **OGL v3.0** |

Konfigürasyon: **`sources.yaml`** (base_url, endpoint, auth, rate_limit, lisans, docs).

### 3.3 Orchestrator — `saglik/ingest_state.py`

```python
run_sync(source_id, batch=500)   # idempotent · İSTİSNA FIRLATMAZ = cron-güvenli
```
- Watermark: `core.sync_state` (`get_state` / `advance_state`)
- Koşu logu: `core.ingest_log`
- CLI: `python scripts/sync_feed.py <source> --batch 500`
- Cron: `.github/workflows/sync-feed.yml` (12 saatte bir)

⚠ **Cron kapsamı dışında kalanlar:** `titck` (bulut-IP engeli) · `icd11` (OAuth).

⚠⚠ **Çıkış kodu vermeyen script CI'da HEP YEŞİLDİR** — sync-feed'de tam olarak bu yaşandı:
haftalarca yeşil görünen **ölü bir hat**. Kapı: `sync_feed_exit_verify`.

---

## 4. LİSANS DİSİPLİNİ — kırılmaz kurallar

### K1 — Lisans, kaynağın KENDİ sayfasından doğrulanır ve `core.corpus.license`'a **OLDUĞU GİBİ** yazılır
Elle "CC BY" yazmak, P0 StatPearls hatasının ta kendisiydi.
Doğru deseni `scripts/ingest_europepmc.commercial_ok()` gösterir: **NC/ND'yi ingest kapısında eler.**

### K2 — "OA subset" ≠ "ticari kullanılabilir"; **arşiv lisans DEĞİLDİR**
Bookshelf yolu bu yüzden **kapandı**.
Kapılar: `bookshelf_lisans_kapisi_verify` · `bookshelf_kimlik_kapisi_verify`.

### K3 — Yeni kaynakta lisans belirsizse **fail-closed** (indirme) + `_gorev.txt`'e kalem

### K4 — StatPearls: görünmez kalır (bkz. `05-CDS-YANIT-ZINCIRI.md` §4.1)

### 4.1 REDDEDİLEN kaynaklar — **tekrar önerme**

| Kaynak | Ret gerekçesi |
|---|---|
| **DDInter** | CC-BY-**NC** |
| **DrugBank (tam)** | Ücretli |
| **WHO ATC dosyası** | Lisans |
| **MedlinePlus** | Karışık lisans |
| **PMC doğrudan** | Lisans |
| **USPSTF** | Lisans |
| **NCBI Bookshelf yolu** | "OA subset" ≠ ticari; arşiv lisans değildir |
| **TEMD / TKD / ESC / SIGN / KDIGO** | Lisans |
| **TGA** | Lisans |

**Sonuç:** ticari-kullanılabilir açık **etkileşim DB'si YOKTUR** → etiket-tarama +
sınıf-bayrağı **doğru yaklaşımdır**. Bu bir eksiklik değil, ölçülmüş bir kısıttır.

### 4.2 Taşınmayanlar — ÖLÇÜLDÜ, kazanç 0, **tekrar önerme**

`DailyMed` · `name_alias` · `code_map`-rxnorm → **prod'a taşınmaz.**
Gerekçeler ve çürütülmüş akıl yürütmeler: `.claude/agents/derya.md`.

### 4.3 Reklamda güvenli kaynaklar (ölçüldü)

✅ openFDA · FAERS · Europe PMC · LactMed · LiverTox · Orphanet

⚠ Reklam maruziyeti: ücretli yüzeyler **temiz**, **Instagram organik gönderileri hâlâ kirli**
(founder elle silecek).
⚠ **Kaynak adı ÜÇ yerde bulunur: görsel pikselleri / gövde metni / platform alanı — üçünü de tara.**

### 4.4 Belirsiz kalanlar (kod kalemi, hukuk kalemi değil)

| Kaynak | Durum |
|---|---|
| TİTCK | Ticari izin **doğrulanamadı** (FSEK) — #763c-HUKUK |
| ICD-11 | **CC BY-ND 3.0 IGO** — kod + başlık + URI **birlikte** şartı — #765c |

⚠ **AVUKAT TEYİDİ KURALI (founder 2026-09-11):** 2026-09-11'e kadar açılmış her hukuk/lisans
kalemi **avukat onaylıdır** ve teyit MEVCUT DURUŞU kapsar (kullandıklarımız **ve**
reddettiklerimiz). Bu kalemleri bir daha soru olarak çıkarma, "avukat teyidi bekliyor" yazma,
iş bloklama, reddedilmiş kaynağı "artık teyitli" diye açma. **Teyit izin değil, duruş onayıdır.**

---

## 5. KORPUS KAYNAKLARI (`core.corpus.source`)

`cds/kaynaklar.KAYNAKLAR` kayıt defterinden — etiket ve link disiplini orada.

| Grup | `source` anahtarları |
|---|---|
| **Klinik ansiklopedi** | `statpearls` *(gizli)* · `livertox` · `lactmed` |
| **Literatür** | `europepmc` *(linkli)* · `pubmed` *(linkli)* |
| **Seyahat/enfeksiyon** | `cdc_yellowbook` |
| **Bağımlılık** | `tip42` · `tip45` (SAMHSA — **ayrı etiketler**) |
| **Bakteri / AMR** | `cdc_mmwr` · `ecdc_amr` · `ecdc_azlist` · `cdc_hcp` |
| **Aşı / virüs / parazit** | `cdc_vaccine` · `cdc_viral` · `cdc_parasite` · `cdc_injection` · `cdc_pinkbook` · `cdc_acip` · `ukhsa_greenbook` |
| **İlaç güvenliği** | `fda_dsc` · `mhra_dsu` |
| **Onkoloji** | `pdqcis` (etiket **"NCI"** — belgenin kendi şartı: bütün olarak sunulmadıkça PDQ adı taşıyamaz) |
| **İlaç kartı** | `openFDA` *(linkli)* · `TİTCK` · `dailymed` |
| **Kimliksiz rozetler** | `FAERS` · `TİTCK KÜB` · `ICD-11` · `Orphanet` · `RxClass` |
| **Kılavuz/ref (rozet)** | `GeneReviews` · `ClinPGx` · `CIViC` · `ChEMBL` · `OrangeBook` · `BNF` · `NICE` · `SIGN` · `USPSTF` · `EUCAST` · `ClinicalTrials` · `Mondo` · `HPO` |

---

## 6. ÖZEL HATLAR

### 6.1 TİTCK KÜB (SmPC) yapılandırılmış çıkarım — ÇALIŞIYOR

```
TİTCK PDF → pypdf (görsel yedek) → llm.extract (Haiku) → core.kub_extract (onay='pending')
          → --onayla → reference._tr_kub_bul → ilaç kartında ayrı blok:
            "🇹🇷 TÜRKİYE RUHSATLI BİLGİ [TİTCK KÜB]"
```

Script: `scripts/extract_kub_batch.py` · `scripts/kub_url_tazele.py`

⚠ **TİTCK indirmesi YALNIZ YERELDEN çalışır** (bulut-IP engeli).

⚠⚠ **ONAY = KLİNİK YÜZEY.** Onaydan önce `scratchpad/kub_onay_oncesi_kontrol.py` **koş.**

⚠⚠ **DÖRT TUZAK** (birebir `.claude/agents/derya.md`'de): onay · URL kırpma ·
ağ/HTTP hatası ayrımı · `hata=NULL` arafı.

Kapılar: `tr_kub_blok_verify` (SKIP) · `kub_kart_blok_verify` (SKIP) · `etkilesim_kub_kanit_verify` (SKIP).

### 6.2 SGK Ek-4/A liste durumu — LLM'siz

`cds/sgk.py` + `core.sgk_odeme` · script `scripts/sut_ek4a_cek.py` · cron `sut-ek4a.yml`

⚠⚠ **"SGK ÖDER" DEMEZ.** Üç durum ayrı tutulur; **yürürlük tarihi zorunludur.**
Kapı: `sgk_odeme_verify`.

### 6.3 Geri çekilme (retraction) hattı

`core.retracted_pub` · script `scripts/fetch_retractions.py` · cron `retractions.yml`

⚠ **Bilinen tuzak:** `geri_cekilme=1.754` bir kez **%100 yanlış pozitif** verdi — ADI ölçmüştü,
**DEĞERİ** değil. Ad eşleşmesi değer kanıtı **değildir**.
⚠ Durum: yetkili liste + durum kolonları prod'a taşındı (NULL=0), haftalık cron yazıldı;
**rozet/süzgeç hâlâ yok.** Memory: `klivance-kaynak-gecerliligi`.

Kapılar: `geri_cekilme_verify` · `geri_cekilme_suzgec_verify` · `retractions_fetch_verify` ·
`geri_cekilme_kapi_verify` (SKIP).

### 6.4 RxNorm / RxClass — ad çözümleme son çaresi

`cds/rxnorm.py` (TR marka/typo → RxNorm etken madde) · `cds/rxclass.py` (ATC + MoA).
NLM REST, anahtarsız, ticari-serbest, önbellekli.

⚠⚠ **HOT-PATH'TE AĞ ÇAĞRILMAZ.** `resolve_ingredient` (ağlı) yalnız son çaredir ve
**yalnız gerçekten daha zengin sonuç gelirse takas edilir**.

### 6.5 FAERS

`scripts/ingest_faers.py` + `scripts/migrate_faers.py` → `core.adverse_event`.
⚠ Kapı: `faers_kapsam_verify`.

### 6.6 Fenotip (HPO) — nadir hastalık

`cds/fenotip.py` + `cds/fenotip_capa.py` (**üretilmiş dosya** — üreteç `scripts/hpo_capa_uret.py`).
Çapa kümesi: genel klinik literatürde **nadir** geçen HPO terimleri.
⚠ Neden var: **ölçülmüş ölü veri** (2026-08-21 founder kararı "B").
Kapı: `fenotip_verify`.

### 6.7 Orphanet

⚠ Aramada **AÇIK**, monograf CTA'sı **KAPALI** (#379b açılmadan geri açma).

---

## 7. ÖLÇÜM REÇETELERİ (sayı gerektiğinde)

```bash
# Yerel KB — tablo bazında satır sayısı
./.venv/Scripts/python.exe scripts/verify_db.py

# Besleme durumu (watermark + son koşu)
./.venv/Scripts/python.exe scripts/status.py

# Tek kaynağı elle besle (dry-run yok — dikkat)
./.venv/Scripts/python.exe scripts/sync_feed.py openfda --batch 500

# Prod'a taşıma (dry-run VARSAYILAN)
./.venv/Scripts/python.exe scripts/migrate_kb_delta.py        # dry-run
./.venv/Scripts/python.exe scripts/migrate_corpus.py          # dry-run
```

⚠ Prod'a bağlanan her komut **founder onayı** ister.

### Tarihli envanter nerede
**KB satır sayıları, taşıma kayıtları ve taşınmayanların gerekçeleri:**
`.claude/agents/derya.md` (tarihli) ve `docs/derya-veri-defteri.md`.
⚠ **Tarihsiz KB sayısına GÜVENME.**

---

## 8. YENİ KAYNAK EKLEME — adım adım

1. **Lisansı kaynağın KENDİ sayfasından oku.** Belirsizse **indirme** (fail-closed) ve
   `_gorev.txt`'e kalem aç.
2. `sources.yaml`'a giriş ekle (base_url, endpoint, auth, rate_limit, **lisans birebir**, docs).
3. `saglik/connectors/<ad>.py` — `BaseConnector`'ı uygula. `INCREMENTAL` / `DURABLE_CURSOR`
   kararını **bilerek** ver (§3.1).
4. `connectors/__init__.REGISTRY`'ye kaydet.
5. `core.corpus.license` alanına lisans cümlesini **olduğu gibi** yaz.
6. `cds/kaynaklar.KAYNAKLAR`'a **etiket + kimlik + link + gizli** kaydını ekle.
   ⚠ **Ayrı yayın = ayrı etiket** (rozet tekilleştirmesi etikete göredir).
   ⚠ `link` için href **kurucusu yoksa** `None` yaz — izin açmak adres üretmez.
7. Geri-çekilme kolonlarına dokunuyorsan `geri_cekilme_verify._IZINLI`'ye **gerekçeli satır**.
8. Kapıları koş: `atif_zinciri_verify` · `rozet_kapsam_verify` · `geri_cekilme_verify`.
9. Doğrulama scriptini yaz → `test.yml` `SUITES`'e **ekle** (eklemezsen ölü ağ).

---

**Sonraki:** [`07-KIMLIK-ERISIM-ODEME.md`](07-KIMLIK-ERISIM-ODEME.md)
