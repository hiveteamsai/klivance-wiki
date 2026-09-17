# 11 — TEST VE CI KAPILARI

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> **320 kapı** (234 koşan · 70 atlanan · 16 JS). Bu, Klivance'ın en büyük tek varlığıdır:
> devralan ekip kodu bozduğunda **haber alır**.

---

## 1. Model — `tests/` dizini YOKTUR, bu bir tercihtir

Klivance'ta pytest/unittest yapısı yok. Bunun yerine **doğrulama scriptleri** vardır:

```
scratchpad/<ad>_verify.py      →  sys.exit(0) başarı · sys.exit(1) en az bir kontrol kaldı
.github/workflows/test.yml     →  SUITES / SKIP_SUITES / JS_SUITES dizelerine ADI EKLENİR
```

**Neden:** bu kapılar birim değil **BAĞLANTI** ölçer. Depoda en sık hata sınıfı
*"son adım bağlanmamış"*tır: iş yapılmış, üretilen şey tüketiciye bağlanmamış, yüzeysel
bakınca "tamam" görünüyor. Bir kez **26/26 birim testi yeşilken özellik hekime
görünmüyordu.** Bu yüzden kapılar HTML/JSON/canlı URL/DB gibi **tüketici katmanı** sondalar.

### 1.1 Yeni kapı eklerken — dört adım

1. `scratchpad/<ad>_verify.py` yaz. Başarıda `sys.exit(0)`, en az bir kontrol kaldıysa
   `sys.exit(1)`.
   ⚠⚠ **ÇIKIŞ KODU VERMEYEN SCRIPT CI'DA HEP YEŞİLDİR** — `sync-feed`de tam olarak bu
   yaşandı: haftalarca yeşil görünen **ölü bir hat**.
2. **SABİT YOL YAZMA.** Windows mutlak yolu Linux runner'da **çalışmaz**. Bunun yerine
   `ROOT = Path(__file__).resolve().parent.parent`. (3 script bu yüzden düzeltildi.)
3. `test.yml`'deki **`SUITES`** listesine adını **ekle** (uzantısız).
   KB verisi ya da canlı API kimliği gerekiyorsa **`SKIP_SUITES`**'e ekle **+ gerekçeyi
   yorum bloğuna yaz.**
4. ⚠ `SUITES` dizesine **yorum satırı ekleme** — `for s in $SUITES` kelime bölmesi yapar.
   (Bu tuzak 2026-08-31'de **tekrar** yaşandı.)

### 1.2 ⚠⚠ DEĞERİ ÖLÇ, ADI DEĞİL — 2026-08-01'de BEŞ kez çıkan hata sınıfı, biri P0'dı

Bir dizenin/anahtarın **var olduğunu** ölçmek, **doğru olduğunu** ölçmez:

| Ölçülen | Olan | Sonuç |
|---|---|---|
| `gtag('config',…,_kcfg)` dizesi arandı | `_kcfg={}` yapıldı | GA4 tam URL'i gönderdi, kapı **326/326 YEŞİL** kaldı (Meta sağlık kategorisi ihlali riski) |
| `.test(location.pathname)` | `.test(location.href)`'e döndü | Desen `^\/` çapalı, `href` `https://` ile başlar → **koruma tümden öldü**, kapı yine YEŞİL |
| `"_ANALYTICS_PATH_ONLY" in src` | Sabit hiç kullanılmasa da | Geçiyordu |
| `subscribers_paying_ic` sondalandı | Ekrana **basıldığı** ölçülmedi | Alan **ölüydü**, yorum "gösterilir" diyordu |
| İddianın **adı** "bant gerçek sayıları taşıyor" | **İçeriği** yalnız `"admin ·"` | — |

---

## 2. CI iş akışı — `.github/workflows/test.yml`

| | |
|---|---|
| Tetik | `push` → `main` · `pull_request` · `workflow_dispatch` |
| Runner | `ubuntu-latest`, `timeout-minutes: 20` |
| Servis | `postgres:16` (health-check'li — yoksa adımlar DB hazır olmadan başlar → **rastgele kırmızı**) |
| DB | `klivance_ci` — **yalnız ŞEMA, KB verisi YOK** |
| `KLIVANCE_FILE_KEY` | CI'ya ait tek kullanımlık Fernet anahtarı — **gizli değil** |
| `PYTHONIOENCODING=utf-8` | Scriptler ✓/✗ basıyor; C locale'de `UnicodeEncodeError` olmasın |
| Concurrency | `test-<ref>`, `cancel-in-progress: true` |

### 2.1 `paths-ignore` — ekonomi kararı (founder 2026-08-15)

```yaml
paths-ignore:
  - 'docs/**'
  - 'scratchpad/_*_bulgu.md'
  - 'scratchpad/_gorev.txt'
  - 'scratchpad/_oturum.md'
```

**Ölçüldü:** 264 koşunun 37'si (%14) yalnız not dosyası değiştiren push'tu = ~300 dk/ay =
kotanın %15'i. Bu dosyaları **hiçbir süit okumaz** (grep + AST ile bakıldı).

⚠⚠ **LİSTEYİ GENİŞLETME:**
- `reklam/**` → üreteç + süit **okur** (`reklam_v6_kapi`, `_hasan_marka_kapi`)
- `CLAUDE.md` / `.claude/**` → **atıf kapısının girdisi**
- kök dosyalar → `kok_temiz`in girdisi

**Bir dosyayı buraya eklemek = o dosyadaki hatanın CI'da GÖRÜNMEMESİ.** Eklemeden önce
*"hangi süit okuyor"* diye **ölç**. Karışık push (kod + not) yine koşar.

---

## 3. ⚠⚠ SÜRÜKLENME KAPISI — `ci_kapi_verify`

**"Takım yazdım" ile "takım koşuyor" AYNI ŞEY DEĞİLDİR.**

Diskteki her `*_verify.py` · `*_kapisi.py` · `*_kapi.py` · `*_check.py` **bir listede
olmalıdır**; olmayan → `exit 1`.

> **Neden var (2026-07-30 ölçümü):** diskte 46 `*_verify.py` vardı ve **11'i hiçbir
> listede değildi** — ne koşuyorlardı ne atlandıkları yazılıydı. En acısı
> `kayit_kaynak_verify` (22 kontrol) bir **P0**'ı çiviliyordu (tipsiz SQL parametresi +
> `except: pass` → kayıt akışının sessizce veri kaybetmesi) ve CI'da **hiç koşmuyordu.**

### ⚠⚠ KURAL: KAPIYI YENİDEN YAZMA — KAPININ KENDİSİNİ ÇALIŞTIR

İlk sürüm kapıyı Python'da **yeniden yazıp** onu test ediyordu — yani test ettiği şey
CI'da koşan bash **değildi**. Bash kapısı `case` deseniyle iki yanında **boşluk** arıyordu
ama `SUITES` **çok satırlı bir dizedir** (adlar yeni satırla ayrılır) → hiçbir ad tutmadı
ve kapı **47 takımın tamamını "kapısız" sanıp CI'ı kırdı**. Python `.split()` yeni satırı
da böldüğü için süit **13/13 YEŞİL** basıyordu.

Artık **bölüm 5** `test.yml`'deki bash bloğunu **çıkarıp gerçekten koşturur.**

### ⚠ Bölüm 0 öz-testi kritiktir
Kapı **boş bir kümeyle de** "geçti" diyebilir (regex yanlış eşleşirse). Bölüm 0 sentetik
YAML'larla kapının **gerçekten ayırt ettiğini** kanıtlar; silinirse bölüm 1 sessizce hep
yeşil basar.

### 🔴 ÖLÇÜLMÜŞ GÜNCEL DURUM (2026-09-17, bu export sırasında)

```
scratchpad/ci_kapi_verify.py  →  37 geçti · 4 kaldı
KAPISIZ: _firat_782c_kayit_kapisi
```

Bugün eklenen bir kapı (`#782c-a`, commit `1344e26a`) `SUITES`/`SKIP_SUITES`'in
**hiçbirinde değil** → sürüklenme kapısı kırmızı.
**Düzeltme:** adı `SUITES`e ekle (DB/anahtar gerektirmiyorsa) ya da gerekçesiyle
`SKIP_SUITES`e. Bu export bunu **belgeler, düzeltmez.**

---

## 4. Kapı yazarken bilinen KÖRLÜKLER

⚠⚠ **ÖLÇÜM/TARAMA ARACI YAZDIYSAN ARACIN KENDİSİNİ SINA.**

| Körlük | Doğrusu |
|---|---|
| Çok-satırlı yapıyı **regex**'le arama | **AST** kullan |
| **Öz-referans** | Araç kendi dosyasını taramaz |
| **Ada göre desen** | `SUITES=` deseni `SKIP_SUITES=` ile de eşleşir → `(?<![A-Z_])` ile sınırla |
| **Girdiyi TAHMİN etmek** | Alan adlarını **rotanın imzasından OKU**. En tehlikelisi paritenin yine korunması = kusur **SESSİZ** |
| Bozma tatbikatında `git checkout --` | **Tek başına geri almaz** — `__pycache__` de silinmeli (aynı saniye + aynı boyut = bozuk `.pyc` geçerli sayılır) |
| **Platform altından değişir** | Yeşil bir kapı, kodu hiç değişmeden **ölçmeyi bırakabilir** |
| **Kapıyı düzeltince ölçüm aracı da bayatlar** | Her çıkarıcıya *"tarayıcı ölü değil"* koruması yaz |
| Tek yönlü öz-test | **İKİ YÖNLÜ olsun**: kirliyi bulduğunu **VE** temize yanlış alarm vermediğini göster |
| Desen | **ADAY üretir, HÜKÜM üretmez** (#108b) |
| — | **Bir aracın kusurunu, o aracın DENETLEDİĞİ kişiye düzelttirme** |

⚠⚠ **BİR ÖLÇÜTÜ/ARACI DEĞİŞTİRİYORSAN: KABUL TESTİ, ESKİSİNİN KAÇIRDIĞINI GÖSTERMELİ.**
*"Yenisi bu hatayı yakalıyor"* **yetmez** (eskisi de yakalıyor olabilir) → **eski YEŞİL /
yeni KIRMIZI** olduğu ayırt edici durumu kur; ters yön de gerekli.

⚠⚠ **`scripts/*.py`: `__main__` kapısı ŞART.** Çıplak `sys.exit(main())` **import edeni
öldürür** = yeşil basan araç. **+ hata gövdesine ÜYELİKLE bak** (`"HATA" in d`; `.get()`
boş sözlükte fail-open). İkisi de canlıda yakalandı → commit `8c0232e`.

---

## 5. ⚠⚠ YEREL YEŞİL ≠ CI YEŞİL

İşletim sistemi farkı yüzünden **50 koşu kaybedildi.**

| Araç | Ne der | Ne DEMEZ |
|---|---|---|
| `scratchpad/ci_taklit.py` | "veri/anahtar/şema sorunu yok" | **"CI yeşil olur" DEMEZ** |
| `scratchpad/ters_egik_yol_tara.py` | Yol/OS kapısı (öz-testli, kendi dosyasını taramaz, docstring'leri `ast` ile atlar) | |

⚠ **CI log'unu okuma reçetesi** (`gh` **KURULU DEĞİL**, aramaya vakit harcama) →
`docs/tuzaklar-ve-denetim-anlatilari.md`.

⚠⚠ **YEREL KIRMIZI DA KANIT DEĞİL:** yerel PostgreSQL segfault ediyor (açık, kök neden
bulunamadı) ve makine donanım kararsızlığı var. **Nihai kanıt CI'dır.** Uzun ölçümü yarıda
kesen şeyi script'e yüklemeden önce `docker logs saglik-pg`'ye bak.

---

## 6. Bozma tatbikatları (`*_tatbikati.py`) — `SKIP_SUITES`

Bunlar **kaynak dosyayı bozar** ve kapının yakaladığını ölçer → CI'da koşamazlar.
Yerelde elle koşulur. Örnekler: `siralama_bozma_tatbikati` · `cases_bozma_tatbikati` ·
`_derya_lisans_bozma_tatbikati` · `inis_beyani_bozma_tatbikati` · `chat_mod_bozma_tatbikati` ·
`_omer_620b_kapi_bozma`.

⚠ Tatbikattan sonra `git checkout --` **+ `__pycache__` temizliği** şart.

---

## 7. Diğer iş akışları (cron)

| Workflow | Cron | Ne yapar | Secret |
|---|---|---|---|
| `sync-feed.yml` | 12 saatte bir | `openfda` + `clinicaltrials` → **doğrudan PROD'a** | `DATABASE_URL` (Render) |
| `purge-retention.yml` | `17 3 * * *` | KVKK grace (≤30 gün) dolan hesaplar + eski tombstone'lar | `DATABASE_URL` |
| `retractions.yml` | haftalık | Geri çekilme listesi | `DATABASE_URL` |
| `sut-ek4a.yml` | — | SGK Ek-4/A listesi | `DATABASE_URL` |
| `ilk-soru-hatirlatma.yml` | — | Aktivasyon maili | `DATABASE_URL` + SMTP |
| `yenileme-hatirlatma.yml` | — | Abonelik yenileme hatırlatması | `DATABASE_URL` + SMTP |

⚠⚠ Cron'ların `DATABASE_URL` secret'ı **RENDER (prod)** işaret eder — yerel değil.

---

## 8. KAPI ENVANTERİ (320)

> Üretilmiş tablo: kaynak = `test.yml` listeleri + her kapının **kendi docstring'inin ilk
> satırı**. Boş açıklama (`—`) = kapının docstring'i yok (belgelenmemiş kapı).

### A. `SUITES` — CI'da KOŞAN kapılar  (234)

Her push (`main`) ve her PR'da koşar. Kırmızı = birleştirme durur.

| # | Kapı | Ne çiviler |
|---:|---|---|
| 1 | `_cahit_yorum_verify` | `/api/etkilesim/yorum` KAPISI — AI yorum ucunun uçtan uca doğrulaması (Cahit, 2026-08-18). |
| 2 | `_derya_coklu_oztest` | ÇOKLU-İLAÇ ETKİLEŞİM MOTORU — SENTETİK ÖZ-TEST (DB YOK, LLM YOK, AĞ YOK). |
| 3 | `_firat_419_tl_js_kos` | #419b ISTEMCI AYAGI — `_CASE_TIMELINE_JS` `loadTimeline()` taklit DOM'da KOSTURULUR. |
| 4 | `_firat_726b_sira_sonda` | #726b TUR-2: `iz_birak` LLM CAGRISINDAN ONCE MI? — AST SIRA KAPISI (Firat, 04.09). |
| 5 | `_firat_etk_sonda` | FIRAT DENETIM SONDASI — /etkilesim N-ilac + AI yorumu (2026-08-18). |
| 6 | `_firat_konsey_opus_verify` | FIRAT SONDASI — Konsey→Opus kesiminin (9d674d2) KABUL ÖLÇÜTÜ. |
| 7 | `_hasan_geri_kol_verify` | KAPI: `--geri` calisirken `--kol` ZORUNLU mu (RET-3, Firat 04.09). |
| 8 | `_hasan_ig_story_failclosed_verify` | KAPI — `post_instagram_story` FAIL-CLOSED mı? (2026-08-25) |
| 9 | `_hasan_v5_vaat_kapisi` | v5 GÖRSEL-VAAT KAPISI — «görselde verilen söz, metinde ve sayfada karşılanıyor mu?» |
| 10 | `_hasan_v9_yasak_tara` | HASAN — YASAKLI IFADE TARAYICISI (kendi metnime uygulanir). |
| 11 | `_kamil_src_bant_verify` | BANT TIKLAMASI SAYACA 'bant' OLARAK DÜŞÜYOR MU — uçtan uca (2026-08-30). |
| 12 | `_kamil_yanit_js_check` | _kamil_yanit_js_check.py — /chat YANIT SUNUMU degisikliginin i18n kapisi (DB GEREKMEZ). |
| 13 | `_selim_unvan_render_verify` | #127b KAPISI — kısaltılmış unvan GERÇEKTEN BASILIYOR mu (yasal sayfa + makbuz maili). |
| 14 | `_selim_yasal_render` | Selim — yasal metni GERÇEK rotadan render edip iki dilde ÖLÇ. |
| 15 | `_sgk_caption_kapisi` | SGK caption KAPISI — gen_yuzey kapılarını CAPTION metnine uygular. |
| 16 | `acq_server_verify` | SUNUCU-TARAFLI EDİNİM YAKALAMA (_acq_mw) — regresyon testi (2026-07-29). |
| 17 | `ad_cozumleme_kapisi` | AD ÇÖZÜMLEME KAPISI — "bu modüldeki her ad GERÇEKTEN çözülüyor mu?" |
| 18 | `admin_denetim_verify` | ADMİN ERİŞİM / DENETİM LOGU — kapı (#522b, 2026-08-31). Şartname: Selim v2 + Ömer. |
| 19 | `admin_form_deneme_verify` | ADMİN FORMU DENEME GÜNÜ KAPISI (2026-09-01) — para-kritik SESSİZ SIFIRLAMA freni. |
| 20 | `admin_mobil_bar_verify` | ADMIN RAYI = ÜRÜN RAYI — dalın gerçekten kaldırıldığını çivileyen kapı. |
| 21 | `admin_panel_verify` | ADMİN PANELİ — KALICI DOĞRULAMA TAKIMI (2026-07-30, CI'da koşar). |
| 22 | `admin_siralama_verify` | Admin üye listesi — SIRALAMA + ÖDEYEN SATIR (founder 03.09). |
| 23 | `ajan_frontmatter_verify` | Ajan/skill frontmatter'i CRLF ile ayristirilamiyor -> SESSIZCE yuklenmez. (#550b) |
| 24 | `akis_kopma_verify` | AKIŞ KOPMA TOPARLAMA — #748b kapısı (Kamil, 2026-09-06). GERÇEK LLM ÇAĞRISI YOK. |
| 25 | `akis_parite_verify` | AKIŞ PARİTESİ — `/api/chat` (StreamingResponse) davranışını kesim ÖNCESİ/SONRASI kıyaslar. |
| 26 | `aklama_pano_verify` | #673b — ANALİZ-AKLAMA MONİTÖRÜ PANEL KARTI + `store.duvar_say` BEYAZ LİSTESİ (2026-09-03, Kamil). |
| 27 | `amber_token_kapisi` | AMBER METİN TOKEN'I — sürüklenme cırcırı (Deniz, 2026-08-03). |
| 28 | `ana_sayfa_donus_verify` | MOBİLDE ANA SAYFAYA DÖNÜŞ — regresyon testi (2026-07-29). |
| 29 | `analiz_aklama_monitor_verify` | ANALİZ (REDUCE) AKLAMA/YÖNLENDİRME MONİTÖRÜ KAPISI (#665b, 2026-09-03, Cahit).  Koşum: |
| 30 | `analiz_kokpit_verify` | ANALİZ KOKPİTİ KAPISI — spec §3.9 İ1-İ12 (scratchpad/_omer_kokpit_spec.md). |
| 31 | `analiz_ozet_verify` | ANALİZ → TAM KLİNİK ÖZET kapısı (#635b).  Koşum: |
| 32 | `analiz_yorum_verify` | ANALİZ YORUMU kapısı (#636b).  Koşum: |
| 33 | `api_abone_verify` | API/abone PAYDASININ SAYISAL SÖZLEŞMESİ — `admin_reklam.aylik_kz`'nin `abone` alanı. |
| 34 | `api_admin_uclari_verify` | `/api/admin/*` DÖRT UCUNUN REGRESYON AĞI — guard + gerçekten yazıyor mu. |
| 35 | `api_giris_freni_verify` | `/api/query` + `/api/suggest` GİRİŞ FRENLERİ — kapı (Faz 8 bulgusu, 2026-08-01). |
| 36 | `async_db_semantik_verify` | ASYNC GÖVDEDE SENKRON DB (M5) — DAVRANIŞ SÖZLEŞMESİ KAPISI. |
| 37 | `atif_arayuz_kapsam_verify` | ATIF ARAYÜZ KAPSAMI — `srcLabel` / `fmtA`, arka ucun ÜRETEBİLDİĞİ her kaynağı tanıyor mu? |
| 38 | `atif_indeksi_verify` | ATIF ETİKETİ İNDEKSİ KAPISI — kanıt bloğundaki etiketler modele KOPYALANABİLİR mi? |
| 39 | `atif_kimlik_kapisi` | #151b — ATIF KİMLİK DİSİPLİNİ, **ÇIKTIDAN** ölçülür. (Cahit, 2026-08-11) |
| 40 | `atif_zinciri_verify` | ATIF ZİNCİRİ — kayıt → monitör → rota → HTML → ekran, TEK KAYNAKTAN mı doğuyor? |
| 41 | `aw_tag_verify` | Google Ads doğrudan dönüşüm etiketi doğrulaması (calc_cta_click, 2026-07-27). |
| 42 | `aylik_alarm_verify` | AYLIK KULLANIM ALARMI — regresyon ağı (#396b-K / credits.AYLIK_ALARM_*, 2026-08-22). |
| 43 | `bag_cumlesi_kapisi_verify` | #738c PİLOT KAPISI — BAĞ CÜMLESİ TALİMATI YALNIZ 'veri' KOVASINDA PROMPTA GİRER. |
| 44 | `bayat_conn_verify` | BAYAT `conn` KAPISI — `with connect() as conn:` bloğu KAPANDIKTAN SONRA `conn` kullanımı. |
| 45 | `bookshelf_kimlik_kapisi_verify` | KİTAP KİMLİĞİ KAPISI — `scripts/parse_bookshelf_html.py` (2026-08-29). |
| 46 | `bookshelf_lisans_kapisi_verify` | scripts/parse_bookshelf.py LİSANS KAPISI — ayırt edici doğrulama (#41). |
| 47 | `butce_freni_verify` | DÖNEM BÜTÇE KAPISI — KAPI TAKIMI (`butce_freni_verify`, 2026-09-02, Selim · #650b Faz A+B). |
| 48 | `butce_kapisi_verify` | #219b — DÜŞÜNME/METİN BÜTÇE ARİTMETİĞİ KAPISI (ücretsiz: ağ · DB · anahtar İSTEMEZ). |
| 49 | `butce_pano_verify` | DÖNEM BÜTÇESİ — ADMİN GÖRÜNÜRLÜĞÜ KAPISI (§7, 2026-09-02 · Kamil). CI'da koşar (boş şema yeter). |
| 50 | `calc_ask_verify` | KART-İÇİ CTA (.calc-ask) doğrulaması — ajan paneli kararı 2026-07-27. |
| 51 | `calc_cta_verify` | /hesaplayicilar dönüşüm bandı doğrulaması (AG-Hesaplayici'nın ölçülebilirliği). |
| 52 | `calc_derin_baglanti_verify` | `/hesaplayicilar?c=<id>` SUNUCU-TARAFLI DERİN BAĞLANTI — regresyon ağı (Hasan, 2026-08-01). |
| 53 | `calc_sayfa_js_check` | /hesaplayicilar sayfasının inline JS'i TR ve EN'de derleniyor mu (apostrof tuzağı). |
| 54 | `calc_seo_verify` | Sunucu-tarafı hesaplayıcı içeriği doğrulaması (Google kalite skoru düzeltmesi, 2026-07-27). |
| 55 | `calc_v9_duzen_verify` | /hesaplayicilar v9 DÜZEN KAPISI — founder kanvası uygulandıktan sonra (2026-08-19). |
| 56 | `callback_verify` | İYZİCO CALLBACK **HAK-EDİŞ** KAPISI — regresyon ağı (2026-08-01, Selim · görev #66). |
| 57 | `case_file_tekil_verify` | app.case_file SHA-256 TEKİLLİK KAPISI (#619b).  Koşum: |
| 58 | `cases_api_verify` | `/api/cases/*` — 7 UCUN BAŞARI DALI (Faz 7 regresyon ağı). |
| 59 | `cases_govde_verify` | `/cases` + `/cases/{cid}` gövde kesimi (Faz 4) — bayt paritesi ve bölge invariantları. |
| 60 | `chat_bekleme_verify` | SOHBET BEKLEME DURUMU — canlılık + erişilebilirlik (founder isteği 2026-08-10). |
| 61 | `chat_sayfa_js_check` | /chat sayfasının inline JS'i TR ve EN'de derleniyor mu (i18n_mw apostrof tuzağı). |
| 62 | `chat_welcome_verify` | SOHBET KARŞILAMA METNİ + REGÜLASYON SATIRI — regresyon testi (2026-07-28). |
| 63 | `chatgpt_yonlendirme_verify` | `/chatgpt` YÖNLENDİRMESİ + TAŞINAN ARGÜMAN — regresyon ağı (2026-08-24). |
| 64 | `ci_kapi_verify` | CI SÜRÜKLENME KAPISI — "takım yazdım" ile "takım koşuyor" AYNI ŞEY DEĞİL. |
| 65 | `cift_gonderim_verify` | ÇİFT GÖNDERİM FRENİ (#706b) — gerçek ürün JS'i Node'da (chat_body.py'den ÇIKARILIR, kopyalanmaz). |
| 66 | `cihaz_eslesme_verify` | CİHAZ EŞLEŞTİRME KAPISI — kimlik yüzeyinin sözleşmesi (#596b, 2026-09-01). |
| 67 | `claude_md_atif_verify` | K4 · ATIF KAPISI — kılavuzların adını verdiği her script diskte OLMALI. |
| 68 | `css_yorum_verify` | CSS YORUMLARI ÜRETİM HTML'İNE GİTMEZ (#93b, 2026-08-04, Kamil) |
| 69 | `deger_onay_verify` | #362b kapısı — analizden çıkan değerin ONAYLI olarak karta işlenmesi. |
| 70 | `deneme_bir_kez_verify` | ÜCRETSİZ DENEME E-POSTA BAŞINA BİR KEZ (founder kararı, 2026-07-29). |
| 71 | `deneme_fren_kapisi` | DENEME FRENİ KAPISI — «işaretli e-posta deneme günü TAŞIYAMAZ» (#396b-K, 2026-08-22). |
| 72 | `derin_prompt_verify` | DERİN ANALİZ PROMPT KAPISI — `DERIN_SYSTEM` tabandan TÜRETİLDİ mi, BAĞLANDI mı? |
| 73 | `dil_ctop_verify` | — |
| 74 | `dogrulama_donusu_verify` | #206b KAPISI — DOĞRULAMA DÖNÜŞÜ AÇIKLAMASIZ GİRİŞ DUVARINA DÜŞMESİN. |
| 75 | `dosya_boyut_verify` | K3 · BÜYÜME FRENİ — beyan edilen tavanı aşan dosya CI'ı kırmızı yakar. |
| 76 | `duvar_sayac_verify` | ANONİM DUVAR / FİYAT SAYACI — regresyon ağı (#490b, 2026-08-30). |
| 77 | `ek_sunucu_kapisi_verify` | #705b-a KAPISI — /api/chat: EK GELDİ AMA MODELE HİÇ GİTMEYECEKSE TUR ÜRETİLMEZ. |
| 78 | `ek_tur_verify` | EK DOSYA TÜRÜ (#705b istemci ayağı) — gerçek ürün JS'i Node'da + sunucu paritesi. |
| 79 | `ekip_kadro_verify` | EKİP KADRO KAPISI — "ajan yazıldı ama TAKIMA SOKULMADI" hâlini yakalar. |
| 80 | `en_enjeksiyon_verify` | EN ENJEKSİYON AĞI — sunucudan `/chat`e basılan iki global, `i18n_mw`den SAĞLAM geçiyor mu? |
| 81 | `en_sayi_bicim_verify` | EN yüzeyinde SAYI BİÇİMİ — binlik ayracı dile bağlı mı? (2026-08-04, Kamil) |
| 82 | `enabiz_tahlil_paketi_verify` | e-NABIZ TAHLİL PAKETİ KAPISI (#668b) — HTML (akordiyon) → belge → analiz/kokpit. |
| 83 | `esik_cifti_verify` | EŞİK ÇİFTİ KAPISI — birlikte dönmesi gereken kurallar AYNI genişlikte mi? (#164b) |
| 84 | `etkilesim_coklu_verify` | ÇOKLU-İLAÇ ETKİLEŞİM + AI YORUM — TEK REGRESYON KAPISI (Fırat, 2026-08-18). |
| 85 | `etkilesim_dil_sira_verify` | /etkilesim — KANIT PASAJLARININ DİL SIRASI (EN'de Türkçe kanıt SONA). |
| 86 | `etkilesim_govde_verify` | /etkilesim GÖVDE + MOTOR HİZALAMA — 2026-07-30 denetim düzeltmelerinin regresyon ağı. |
| 87 | `etkilesim_yorum_verify` | ENTEGRASYON KAPISI — /etkilesim N ilaç + AI yorum ucunun BAĞLANDIĞINI kanıtlar. |
| 88 | `faers_kapsam_verify` | FAERS KAPSAM + YAZMA SÖZLEŞMESİ KAPISI. |
| 89 | `feed_verify` | SÜREKLİ BESLEME HATTI — imleç davranışı, kalıcı doğrulama. |
| 90 | `fenotip_verify` | FENOTİP → NADİR HASTALIK KAPISI — `cds/fenotip.py` sözleşmesi + bağlantısı. |
| 91 | `fiyat_yuzey_verify` | FİYAT YÜZEYİ KAPISI — ilan edilen fiyat, TAHSİL EDİLEN fiyattan türer mi? (#137b + #139b) |
| 92 | `form_deger_koruma_verify` | KAYIT/ONAY FORMU — REDDEDİLDİĞİNDE GİRİLEN DEĞER KORUNUR + HATA İLK EKRANDA (2026-08-30). |
| 93 | `gclid_verify` | gclid YAKALAMA — regresyon testi (2026-07-28). |
| 94 | `geri_cekilme_suzgec_verify` | GERİ ÇEKİLME SÜZGECİ + KÖPRÜ BOŞLUĞU doğrulaması (2026-08-08) — LLM ÇAĞIRMAZ, PROD'A BAĞLANMAZ. |
| 95 | `geri_cekilme_verify` | GERİ ÇEKİLME VERİ KATMANI doğrulaması (#67, 2026-08-02) — LLM ÇAĞIRMAZ, PROD'A BAĞLANMAZ. |
| 96 | `giris_gecmisi_verify` | Giriş/çıkış geçmişi + admin soru-kimliği — kalıcı doğrulama (founder 2026-08-14). |
| 97 | `giris_kalici_verify` | GİRİŞ KALICI KAYDEDİLİYOR MU (founder bulgusu 2026-07-30). |
| 98 | `goruntu_kaydet_verify` | #781c KAPISI — SOHBETTE ÇOKLU GÖRSEL + "HASTA KARTINA KAYDET" (founder 15.09). |
| 99 | `goruntu_kiyas_verify` | #781c KAPISI — ÇOKLU GÖRSEL KARŞILAŞTIRMA ZİNCİRİ (Cahit, 2026-09-15; founder 15.09). |
| 100 | `goruntu_onokuma_verify` | #620b YOL 2 — GÖRÜNTÜ ÖN-OKUMA KAPISI (KP, Cahit tek yazar, 2026-09-13; sözleşme §5 `goruntu_onokuma_verify`). |
| 101 | `goruntuleme_kapisi_verify` | #620b — GÖRÜNTÜLEME TETKİKİ KAPISI doğrulayıcı (founder 2026-09-01; söylem 2026-09-13 KP ile YENİLENDİ). |
| 102 | `guvence_fold_verify` | GÜVENCE METİNLERİ FOLD İÇİNDE — `/kayit` bandı + landing önizleme uyarısı (2026-08-30). |
| 103 | `guvenlik_tavan_verify` | ÜCRETSİZ GÜVENLİK UÇLARINDA KÖTÜYE KULLANIM TAVANI (güvenlik denetimi P1, 2026-07-29). |
| 104 | `guvenlik_verify` | /guvenlik sayfası doğrulaması — syntax + render (TR/EN) + link/sitemap. |
| 105 | `hasta_baglam_dustu_verify` | #679b-2 KAPISI — SESSİZ HASTA BAĞLAMI KAYBI (hasta güvenliği sınıfı: YANLIŞ GÜVEN). |
| 106 | `hasta_baglam_thread_istemci_verify` | #704b KABUL KAPISI (İSTEMCİ AYAĞI, Kamil) — `openThread` gelen `case_id` ile seçiciyi kurar. |
| 107 | `hasta_baglam_thread_verify` | #704b KAPISI (SUNUCU AYAĞI) — YENİDEN AÇILAN HASTA-BAĞLI SOHBETTE KART YÜKE GİRER. |
| 108 | `hasta_uyari_tetik_verify` | #355b kapısı — hasta kartına RİSK ALANI kaydedilince reçete kontrolünün KENDİLİĞİNDEN |
| 109 | `hastalik_monograf_verify` | HASTALIK MONOGRAFI ("hastalik" modu) — cds kapısı (founder 2026-08-19). |
| 110 | `hastalik_verify` | `/hastaliklar` kapısı (founder 2026-08-19). |
| 111 | `hekim_iz_verify` | HEKİM DAVRANIŞ İZİ — regresyon ağı (#726b, Kamil · 2026-09-04). |
| 112 | `hekimlik_onayi_kaldir_verify` | HEKİMLİK ONAYI YÜZEYLERİ KALDIRILDI (founder kararı 2026-07-30). |
| 113 | `hesap_yasam_dongusu_verify` | HESAP YAŞAM DÖNGÜSÜ — silme gerçekten oluyor mu + panel doğru mu konuşuyor mu? |
| 114 | `i18n_muafiyet_verify` | i18n MUAFİYETİ — `i18n_mw`'den muaf sayfaların ÇEVRİLMEDİĞİ ölçülür. |
| 115 | `ifade_kapisi_verify` | İFADE KAPISI — "ürün kendi altyapısından SÖZ ETMEZ" (founder kararı 2026-08-07). |
| 116 | `ilac_karti_gundelik_verify` | #701b İLAÇ KARTI GÜNDELİK KELİMEYE TAKILIYOR — kapı (Derya, 2026-09-03). LLM ÇAĞIRMAZ. |
| 117 | `inis_fold_verify` | İNİŞ SAYFASI FOLD + RAY doğrulaması (tasarım denetimi 2026-07-29, iki P0). |
| 118 | `inis_kaynak_beyani_verify` | İNİŞ SAYFASI KAYNAK/SAYI BEYANI — regresyon ağı (2026-08-01). |
| 119 | `interactions_alias_verify` | `/interactions` → `/etkilesim` EN TAKMA YOLU (2026-08-04, Kamil) |
| 120 | `iptal_maliyet_verify` | iptal_maliyet_verify — R1: akış iptalinde GERÇEK usage (yan kanal) + erken `calls_partial`. |
| 121 | `iyzico_abonelik_verify` | İYZİCO ABONELİK (recurring) KAPISI — #774c C1 (Selim, 2026-09-12). |
| 122 | `kabuk_ofset_verify` | KAPI 1 · KABUK OFSETİ — ray genişliği ile içerik ofseti AYRIŞTI mı (tarayıcısız). |
| 123 | `kamera_dugme_verify` | Kamera düğmesi KALDIRILDI doğrulaması (chat #filecam) — 2026-08-16. |
| 124 | `kanit_alaka_verify` | KANIT ALAKASI doğrulaması (görev #65, 2026-08-01) — LLM ÇAĞIRMAZ, PROD'A BAĞLANMAZ. |
| 125 | `kanit_ciz_verify` | chat_body RENDER SÖZLEŞMESİ (#223b + #220b) — üretilen veri EKRANA ÇİZİLİYOR MU? |
| 126 | `kapanis_verify` | #769c KAPISI — KAPANIŞ SATIRI İKİ HÂLLİ (founder 2026-09-11): atıf VARSA «Atıflıdır.», |
| 127 | `kart_dili_verify` | «KART GEREKMEZ» AİLESİ — ÜRÜN YÜZEYİ KAPISI (founder 2026-08-23, #K-kartsız). |
| 128 | `kart_dosya_freni_verify` | ANALİZ EDİLMEMİŞ KART DOSYASI FRENİ (#705b-b) — gerçek ürün JS'i Node'da + kablolama. |
| 129 | `kart_form_sozlesme_verify` | HASTA KARTI FORM SÖZLEŞMESİ KAPISI — `/cases/{id}` formu ⊇ `update_case` alan kümesi. |
| 130 | `kaskad_sira_verify` | KASKAD SIRASI KAPISI — mobil `@media` kuralı tabanını GERÇEKTEN eziyor mu? (#162b) |
| 131 | `katman_bag_verify` | KATMAN SATIRI ↔ SORU BAĞI (#715b K1, istemci ayağı) — gerçek ürün JS'i Node'da. |
| 132 | `kayit_kaynak_verify` | KAYIT EDİNİM KAYNAĞI + DOĞRULAMA E-POSTASI — kalıcı doğrulama. |
| 133 | `kayit_masaustu_verify` | /kayit MASAÜSTÜ SAHNESİ kapısı — `.auth-reg` + sol marka paneli (2026-08-15). |
| 134 | `kayit_piksel_tasima_verify` | DÖNÜŞÜM OLAYI ATEŞLENDİĞİ SAYFADA PİKSEL VAR MI? (#84b yan ürünü, 2026-08-04, Kamil) |
| 135 | `kayit_sinyal_verify` | `klv_kayit` (HARİÇ TUTMA, geniş) vs `CompleteRegistration` (OPTİMİZASYON, dar) — ÖLÇÜM. |
| 136 | `kayit_zorunlu_verify` | KAYIT ZORUNLULUĞU + ULUSLARARASI TELEFON (2026-07-30, founder isteği). |
| 137 | `kirpma_vurgu_verify` | #761c — PASAJ VURGUSU = HEKİMİN AYIRT EDİCİ KELİMELERİ (2026-09-11, Derya). |
| 138 | `kisa_link_verify` | SOSYAL KISA LİNKLER (/ig /fb /li) — regresyon testi (2026-07-28). |
| 139 | `kismi_cokme_tara` | KISMİ ÇÖKME MASKESİ TARAYICISI — "süit kırmızı oldu ama KAÇ iddia koştu?" |
| 140 | `kok_temiz_verify` | K1 · KÖK DİZİN KAPISI — kökte allowlist dışında git'e kayıtlı dosya OLAMAZ. |
| 141 | `kombinasyon_urun_verify` | #762c — KOMBİNASYON ÜRÜNÜ TEK ÜRÜNDÜR · ORGANİZMA ADI İLAÇ KARTI DEĞİLDİR (2026-09-11, Derya). |
| 142 | `konu_alakasi_verify` | #702b — KONU ALAKASI KAPISI ağı. |
| 143 | `kritik_goruntu_verify` | #782c KAPISI — GÖRÜNTÜ ÖN-OKUMASINDA KRİTİK BULGU KATMANI (Cahit, 2026-09-15; founder 15.09). |
| 144 | `kritik_serit_verify` | #782c KAPISI — KRİTİK BULGU KADEMESİ, APP AYAĞI (founder 15.09 "aç, daha da geliştir"). |
| 145 | `kunye_rota_verify` | KANIT KÜNYESİ ROTA SINIRI — `source_docs` istemciye ULAŞIYOR mu? (#64 kuyruğu) |
| 146 | `kunye_siniri_verify` | KANIT KÜNYESİ DÖNÜŞ SINIRI (#64) — zengin kaynak bilgisi sınırdan GEÇİYOR mu, düz liste SAĞLAM mı? |
| 147 | `label_date_verify` | #70 — etiket tazeliği (label_date / label_date_kind / label_version) REGRESYON AĞI. |
| 148 | `like_escape_verify` | `_like_escape` — ILIKE JOKER KAPISI (görev #25, 2026-08-01). |
| 149 | `mail_gonderim_kapisi` | MAİL GÖNDERİM KAPISI — «önemli bir mail göndermeden ÖNCE koştur» (founder isteği, 2026-08-17). |
| 150 | `main_fix_verify` | main.py denetim düzeltmeleri (M1..M30) — kalıcı doğrulama. |
| 151 | `maliyet_korlugu_verify` | MALİYET KÖRLÜĞÜ KAPISI — zarar edilirken panel "%100 marj" diyemesin (#138b-3). |
| 152 | `marker_rag_verify` | #726b SUNUCU AYAĞI KAPISI — ÇİP YANITINDA RETRIEVAL GİRDİSİ ASIL SORUYU DA TAŞIR. |
| 153 | `marker_sinyal_verify` | ÇİP/PANEL YANITI SİNYALİ (#726b, istemci ayağı) — gerçek ürün JS'i Node'da. |
| 154 | `meta_signal_js_check` | i18n_mw apostrof tuzağı freni: EN modunda /dogrula ve /chat?welcome=1 sayfalarının TÜM |
| 155 | `meta_signal_verify` | Meta CompleteRegistration sinyal-kalitesi doğrulaması (founder onayı 2026-07-26, ajan paneli P0-3). |
| 156 | `mod_kacirma_verify` | #699b MOD KAÇIRMASI kapısı (Derya, 2026-09-03) — LLM ÇAĞIRMAZ. |
| 157 | `mod_tek_seferlik_verify` | #410b — SOHBET MOD DÜĞMELERİ: tek-seferlik sıfırlama + setMode gerçekten koşuyor. |
| 158 | `modul_kapisi_verify` | K5 · MODÜL KAPISI — yeni bir modül sessizce canlıyı düşürmesin, süitleri körleştirmesin. |
| 159 | `notrack_verify` | klv_notrack iç-trafik temizliği doğrulaması (2026-07-27). |
| 160 | `odeme_sonuc_verify` | `/odeme/sonuc` GÖRÜNÜR METİN — fail-closed regresyon ağı (2026-08-22, Selim · Fırat P1). |
| 161 | `olu_css_kapisi` | ÖLÜ CSS KURALI KAPISI — `PANEL_CSS`'te tanımlı her `.pnl-*` sınıfı render'da GEÇİYOR mu? |
| 162 | `ornek_sorular_verify` | BRANŞA GÖRE ÖRNEK SORULAR — regresyon ağı (2026-08-14). |
| 163 | `panel_ay_verify` | panel_ay_verify — `/panel` "Bu ay" şeridi (#126b-B) + "Branşınıza göre" (#126b-C). |
| 164 | `panel_ic_trafik_verify` | PANELDE İÇ TRAFİK DIŞLAMASI — TEK KURAL OLMALI (2026-08-01, Selim). |
| 165 | `panel_tipo_kapisi` | PANEL TİPOGRAFİ KAPISI — `/panel` masaüstü ölçeği sürüklenmesin (2026-08-03, Deniz). |
| 166 | `panel_verify` | panel_verify — `/panel` (yargım deseni EK sayfa, 2026-08-02) doğrulama takımı. |
| 167 | `parite_maske_verify` | `refactor_parite` ÜYE LİSTESİ MASKESİ — iki yönlü bozma tatbikatı. |
| 168 | `parola_gucu_verify` | PAROLA KURALI — TEK KURAL (UZUNLUK) üç yolda da uygulanıyor mu? (2026-08-05 revizyonu) |
| 169 | `parola_sifirlama_verify` | PAROLA SIFIRLAMA uçtan uca doğrulaması (2026-07-29, denetim P1). |
| 170 | `pazar_yonlendirme_verify` | PAZARA GÖRE ÖDEME YÖNLENDİRMESİ — regresyon ağı (2026-08-09, Selim · #142b). |
| 171 | `pill_rol_kapisi` | PILL ROL-KİLİDİ — "TAM YUVARLAK = BU BİR KAYNAKTIR" (founder kararı 2026-08-08, Deniz) |
| 172 | `plan_sozluk_verify` | PLAN ADI SÖZLÜĞÜ — REGRESYON AĞI (Selim, 2026-08-02). |
| 173 | `prompt_adim_kota_verify` | #710b KAPISI — TAVAN KOTAYA DÖNÜŞMESİN (`DOCTOR_SYSTEM` TARAMA DÜZENİ: madde/adım sayıları). |
| 174 | `prompt_ilac_kimlik_verify` | #703b KAPISI — KARTSIZ İLACA KİMLİK ATFETME YASAĞI (`DOCTOR_SYSTEM` "İLAÇ ADI TANIMA" bloğu). |
| 175 | `purchase_piksel_verify` | `purchase` PİKSELİ — RENDER DÜZEYİ regresyon ağı (2026-08-09, Selim · #142b P1). |
| 176 | `rad_cikar_verify` | IP0 KAPISI — `saglik/cds/rad_cikar.py` + `saglik/cds/isaretler.py` (YOL 1, sözleşme §1.2/§1.6/§1.7; 2026-09-13). |
| 177 | `rad_kablolama_verify` | YOL 1 KABLOLAMA — analiz K-C kapısı · hasta kaydı RAD kesimi · akış olayları · istemci geç notice (KB1-KB4). |
| 178 | `rad_kategori_kapisi_verify` | IP1 KAPISI — K-C `rad_cikar.rad_kategori_kapisi` + sohbet kolu kablolaması (YOL 1, sözleşme §1.2/§1.5/§5; 2026-09-13). |
| 179 | `rad_metin_kablo_verify` | #778c GÖRÜNTÜLEMEDE GÖRÜNTÜ OLMADAN METİN — app KABLO kapısı (Kamil, 2026-09-14). |
| 180 | `rad_metin_verify` | #778c KAPISI — GÖRÜNTÜLEMEDE GÖRÜNTÜ OLMADAN METİN, cds ayağı (`cds/rad_metin.py` + `chat.respond(rad_metin=)`). |
| 181 | `rad_takip_verify` | RAD TAKİP ŞERİDİ + ROTA + KOKPİT 6. KARO — YOL 1 IP2 kapısı (T1-T6; sözleşme §3.2/§5, 2026-09-13). |
| 182 | `ray_olcum_verify` | RAY TIKLAMASI + TAKVİM OLAYLARI — ölçüm altyapısının KAPISI (2026-08-20). |
| 183 | `ray_sira_verify` | #478b · MOBİL ALT ÇUBUK — DİZİLİŞ + VURGU KAPISI (Ömer, 2026-08-29). |
| 184 | `recete_kontrol_motor_verify` | REÇETE KONTROLÜ MOTORU doğrulaması — `saglik/cds/recete_kontrol.py` (2026-08-17, Derya). |
| 185 | `recete_kontrol_verify` | REÇETE KONTROLÜ KAPISI — `/recete-kontrolu` (hub) + `/bobrek-doz` + ray ögesi. |
| 186 | `recete_sekme_verify` | HASTA KARTI · REÇETE KONTROLÜ SEKMESİ — kapı (#336b, 2026-08-17). |
| 187 | `reklam_ay_verify` | AYLIK REKLAM HARCAMASI (yıl+ay girişi) — kalıcı doğrulama (founder 2026-08-19). |
| 188 | `reklam_fiyat_verify` | `antigravity reklam/` reklam metinlerini PARA/İADE tek-doğru-kaynağına karşı çivi. |
| 189 | `reklam_v6_kapi_verify` | v6 REKLAM KAPILARI — CI köprüsü (#87, Deniz 2026-08-03). |
| 190 | `reklam_vaat_kapisi` | REKLAM VAAT KAPISI (#502b) — «kredi / kart / süre» vaadi YAYIN YOLUNDA mı? |
| 191 | `reklam_yasakli_ad_kapisi` | REKLAM YASAKLI-AD KAPISI — `reklam/` altındaki TÜM kreatif kaynaklarını tarar. |
| 192 | `renal_motor_verify` | BÖBREK DOZ KONTROLÜ MOTORU doğrulaması — `saglik/cds/renal.py` (2026-08-17, Derya). |
| 193 | `retention_verify` | KVKK SAKLAMA / SİLME AKIŞI — regresyon ağı (2026-08-01, Selim · görev #24). |
| 194 | `retractions_fetch_verify` | GERİ ÇEKİLME ÇEKİM HATTI kapısı (#306b, 2026-08-15) — AĞ YOK, DB YOK, LLM YOK. |
| 195 | `rol_eskalasyon_verify` | YÖNLENDİRME YASAĞI + PROFESÖR TONU — regresyon ağı (Cahit alanı; yeniden yazıldı 2026-08-15). |
| 196 | `rotate_pagination_verify` | rotate_file_key.py SAYFALAMA doğrulaması (2026-07-29). |
| 197 | `rozet_atif_verify` | ROZET = YANITIN GERÇEĞİ — kalıcı doğrulama (Node ile gerçek JS koşturulur). |
| 198 | `rozet_kapsam_verify` | ROZET KAPSAMI — kanıt bloğuna giren HER kaynak türünün rozeti var mı? (2026-08-01, Cahit) |
| 199 | `sekil_fatura_verify` | ŞEKİL SINIFLANDIRICI ↔ FATURA KURALI — AYRIŞMA KAPISI (Cahit, 2026-08-10) |
| 200 | `sekil_gunluk_kapisi` | ŞEKİL GÜNLÜĞÜ CI KAPISI — `sekil_gunluk.py`nin öz-testini koşturur. |
| 201 | `sgk_odeme_verify` | #354b kapısı — SGK Ek-4/A liste durumu (`saglik/cds/sgk.py` + kart + ekran). |
| 202 | `signup_ghost_verify` | HAYALET sign_up freni doğrulaması (canlı GA4 bulgusu 2026-07-26: GA4'te 1 sign_up |
| 203 | `sinir_deger_verify` | SINIR DEĞER KAPISI (founder 2026-08-27) — "tahlilde referans aralığı BASILI ve değer onun |
| 204 | `sinirsiz_erisim_verify` | SINIRSIZ ERİŞİM KAPISI — kredi sisteminin kaldırılmasını çivileyen regresyon ağı. |
| 205 | `sir_yolu_izsiz_verify` | #210b KAPISI — SIR TAŞIYAN SAYFA ANALYTICS ETİKETİ BASAMAZ. |
| 206 | `sohbet_baslik_verify` | #736c · OTOMATİK SOHBET BAŞLIĞI — üretim kuralları + üzerine yazma sözleşmesi. |
| 207 | `sohbet_sirasi_verify` | SOHBET LISTESI SIRASI — `store.list_threads` son etkinlige gore mi siraliyor? |
| 208 | `soru_analitigi_verify` | #715b K2 — TÜRETME KATMANI ağı (`app.question_insight` / `saglik/app/soru_analiz.py`). |
| 209 | `soru_analiz_panel_verify` | #715b K3 PANELİ — `/admin/soru-analiz` (SPEC: docs/soru-analitigi-spec-2026-09-04.md §2,§4). |
| 210 | `soru_kayit_verify` | #715b K1 KAPISI — katman sonucu + kanıt paketi künyesi GERÇEKTEN yazılıyor mu. |
| 211 | `soru_koruma_verify` | ÜYE SİLİNİNCE SORULAR KAYBOLMASIN (founder kararı 2026-07-29). |
| 212 | `statik_mime_verify` | Statik dosyaların İÇERİK TİPİ doğru mu — `.webp` sessizce octet-stream'e düşmesin. |
| 213 | `sync_feed_exit_verify` | sync_feed.py ÇIKIŞ KODU doğrulaması (2026-07-29). |
| 214 | `takip_kanca_verify` | TAKİP KANCASI KAPISI — `/chat` cevabının altındaki "Takibe al" çipleri (2026-08-21). |
| 215 | `takip_kopru_kapisi_verify` | #700b TAKİP KÖPRÜSÜ YANLIŞ KAPIDA — kapı (Derya, 2026-09-03). LLM ÇAĞIRMAZ. |
| 216 | `takip_kopru_verify` | TAKİP TURU KÖPRÜSÜ KAPISI — `engine._takip_baglami` + `engine.route(gecmis=)` sözleşmesi. |
| 217 | `takvim_verify` | `/takvim` + `/api/calendar*` — TAKVİM REGRESYON AĞI (2026-08-20, plan `_takvim_plan.txt` §1-§12, §14). |
| 218 | `tarayici_ayirtedici_kapisi` | AYIRT EDİCİ TEST — `ters_egik_yol_tara` genişletmesi ESKİSİNİN KAÇIRDIĞINI yakalıyor mu? |
| 219 | `telafi_comp_verify` | COMP (BEDELSİZ) ERİŞİM AYRIMI — regresyon ağı (#396b-K, 2026-08-22). |
| 220 | `terim_ajan_verify` | #591b — TERİM KAPISI: Türkçe hekim yanıtında "ajan" DEĞİL "ilaç" (founder 2026-09-01). |
| 221 | `ters_egik_yol_tara` | TÜM doğrulama scriptlerinde LINUX'TA KIRILACAK yol kullanımı var mı? |
| 222 | `threadpool_kapisi` | AĞIR İŞ EVENT LOOP'TA ÇALIŞMAZ — kapı (M5 invariantı). |
| 223 | `tip_ipucu_kapisi` | TİP İPUCU ↔ GERÇEK DÖNÜŞ AYRIŞMASI TARAYICISI (Selim, 2026-08-02, #17). |
| 224 | `titck_rozet_verify` | TİTCK / TİTCK KÜB rozeti KLİNİK modda da üretiliyor mu (denetim P1, 2026-07-29). |
| 225 | `token_tek_kaynak_kapisi` | TOKEN PALETİ TEK KAYNAK — `ui_css.py` ile `landing.py` AYRIŞAMAZ (#135b, Deniz 2026-08-08) |
| 226 | `uzay_suruklenme_kapisi` | UZAY SÜRÜKLENMESİ KAPISI — ızgara dışı boşluk px'i ARTAMAZ (cırcır). |
| 227 | `worktree_verify` | `_klv_worktree` yardımcısının doğrulaması. |
| 228 | `xfo_muafiyet_verify` | XFO MUAFİYETİ — ödeme dönüş yollarının X-Frame-Options muafiyeti ÖLÇÜLÜR. |
| 229 | `xss_reference_verify` | XSS KAPISI — /reference kartlari dis veriyi HTML olarak mi basiyor? |
| 230 | `yanit_dili_verify` | YANIT DİLİ = SORUNUN DİLİ — `saglik/cds/dil.py` + `chat_routes` bağlantısı (#275b, 2026-08-15). |
| 231 | `yas_asimi_verify` | YAŞ AŞIMI sayacının İKİ YÖNLÜ öz-testi. Tüm yazmalar ROLLBACK edilir. |
| 232 | `yedek_ezme_verify` | YEDEK EZME KAPISI — canlı reklam scriptlerinin GERİ DÖNÜŞ kaydını koruduğunu çivileyen ağ. |
| 233 | `yenileme_hatirlatma_verify` | YENİLEME HATIRLATMASI ("C köprüsü") KAPISI — 2026-09-04, Selim. |
| 234 | `yonlendirme_yasagi_verify` | #467b — YÖNLENDİRME YASAĞI: hangi prompt ÜRETİCİ, hangisi DENETÇİ (kapı). |

### B. `SKIP_SUITES` — bilinçli ATLANAN kapılar  (70)

KB verisi (275K ilaç satırı runner'a taşınmaz) ya da canlı API kimliği ister. ⚠ **Her SKIP takımının gerekçesi `test.yml` yorum bloğunda YAZILI olmalı** — `ci_kapi_verify` bölüm 3 bunu ölçer.

| # | Kapı | Ne çiviler |
|---:|---|---|
| 1 | `_F_ro_kapisi` | FIRAT — `SET default_transaction_read_only = on` GERCEKTEN koruyor mu? |
| 2 | `_deniz_b2_js_check` | (b2) APOSTROF TUZAĞI + `node --check` — /panel TR ve EN. |
| 3 | `_deniz_yanit_kapi` | _deniz_yanit_kapi.py — YANIT SUNUMU KAPISI (20 gercek yanit, gercek fonksiyonlar). |
| 4 | `_derya_lisans_bozma_tatbikati` | BOZMA TATBİKATI — `bookshelf_lisans_kapisi_verify` gerçekten kırmızı yanıyor mu? |
| 5 | `_firat_620b_sonda` | #620b BAGIMSIZ KABUL SONDASI (Firat, 2026-09-01; 13.09 guncelleme: B3/B4 fail-CLOSED beklentisine cevrildi, |
| 6 | `_hasan_caption_kapi` | CAPTION KAPISI — kreatif kapıları HTML tarar; caption AYRI bir yüzeydir ve |
| 7 | `_hasan_hikaye_duzen_kapisi` | HASAN — HİKÂYE (9:16) DÜZEN KAPISI. 4:5 kapısının kapsamadığı sınıf. |
| 8 | `_hasan_marka_kapi` | MARKA KAPISI — kreatifin renk/tipografisi ÜRÜNÜN KENDİ CSS'iyle birebir mi? |
| 9 | `_hasan_psikolog_duzen_kapisi` | HASAN — PSİKOLOG kreatifi DÜZEN KAPISI (kırpılma + kayma). |
| 10 | `_hasan_v7_duzen_kapisi` | HASAN — V7 kreatif DÜZEN KAPISI (kırpılma + kaymayan yerleşim). |
| 11 | `_ilk7gun_kapi` | PROD SALT-OKUNUR — 25 'hic sormayan' /chat'e ULASTI MI? |
| 12 | `_kamil_mutabakat_kapi` | Fırat'ın mutabakat kapısındaki İKİ metin iddiasını gerçeğe hizalar. |
| 13 | `_kamil_yarim_kapi` | [YARIM] kararını CI'DA KOŞAN kapıya bağlar (iki yönlü). |
| 14 | `_o_desen_yok_verify` | #179b KAPISI — yedi Konsül yüzeyinde dekoratif desen YOK + zemin düz `--mist`. |
| 15 | `_o_ikinci_kanal_verify` | #183b KAPISI — zemin `--mist` olunca dolgusu görünmez kalan ögelerin İKİNCİ KANALI var mı? |
| 16 | `_o_prod_kapi` | PROD SALT-OKUNUR — Kamil'in hipotezi: "0 soru" = denemedi mi, DUVARA MI ÇARPTI? |
| 17 | `_o_salt_okunur_kapisi` | SALT-OKUNUR BANDI GERÇEKTEN TUTUYOR MU? — YEREL DB'de sınanır, prod'da ASLA. |
| 18 | `_omer_620b_kapi_bozma` | #620b — KAPININ KENDİSİNİ SINAYAN BOZMA TATBİKATI (Ömer 2026-09-01 · KP 2026-09-13: 8 → 15 sabotaj). |
| 19 | `admin_pano_store_verify` | Admin panosu GÖRÜNÜRLÜK katmanı — store.py tarafı doğrulaması (2026-07-30). |
| 20 | `admin_pano_verify` | ADMİN PANOSU — main.py/ui_css.py katmanının REGRESYON AĞI (2026-07-30). |
| 21 | `admin_sorular_verify` | /admin/sorular — soru + YANIT, HAM gösterim. Kalıcı doğrulama. |
| 22 | `akis_kvkk_tatbikat` | AKIŞ KVKK/THREAD ÇİVİLERİNİN **KASITLI BOZMA TATBİKATI** (#15 kabul kanıtı, 2026-08-01). |
| 23 | `alt_bar_hedef_verify` | #27 · MOBİL ALT ÇUBUK DOKUNMA HEDEFİ KAPISI (Deniz, 2026-08-03). |
| 24 | `anlam_kaniti_olceri` | ANLAM KANITI ÖLÇERİ — "süiti yeniden yazdım, İDDİALARIN ANLAMINA dokundum mu?" |
| 25 | `asi_virus_verify` | #764c-İNDİR kapısı — `scripts/ingest_asi_virus_sayfa.py` (cdc_vaccine · cdc_viral · cdc_parasite · |
| 26 | `cases_bozma_tatbikati` | HASTA DEFTERİ BOZMA TATBİKATI — `cases_api_verify` KIRMIZI basabiliyor mu? (17 mutasyon) |
| 27 | `cases_deger_tatbikati` | `cases_api_verify` BOZMA TATBİKATI — iddia edilen frenler TAŞIYICI mı, SÜS mü? |
| 28 | `cases_maske_tatbikati` | `cases_api_verify` KISMİ ÇÖKME (MASKE) TATBİKATI — istisna altında KAÇ iddia basar? |
| 29 | `cds_fix_verify` | CDS klinik düzeltmeleri — kalıcı doğrulama (2026-07-29). |
| 30 | `chat_mod_bozma_tatbikati` | DENIZ bozma tatbikati: dogrulama kapisi KIRMIZI basabiliyor mu? |
| 31 | `chat_mod_gidis_donus_verify` | DENIZ: mq.change GIDIS-DONUS kapisi (Omer kabul olcutu #3). |
| 32 | `chat_mod_kolon_verify` | DENIZ dogrulama surucusu: terfi + tiklama + rozet + tasma, 4 genislik x TR/EN. |
| 33 | `chat_mod_komsuluk_tatbikati` | DENIZ: 'orijinal komsuluk' iddiasi KIRMIZI basabiliyor mu? |
| 34 | `determinizm_verify` | DETERMİNİZM DOĞRULAMASI (2026-08-02, #67 denetiminde Fırat'ın bulduğu regresyon). |
| 35 | `drug_match_verify` | find_drugs KELİME-SINIRI SIRALAMASI — regresyon testi (2026-07-27 hasta güvenliği düzeltmesi). |
| 36 | `etkilesim_kub_kanit_verify` | /etkilesim TÜRKÇE KANIT — `reference.kub_scan_card` + `interaction_check(kub=)` ağı. |
| 37 | `etkilesim_sinif_verify` | SINIF HARİTASI — etkileşim taramasının TEK yedeği gerçekten çalışıyor mu? |
| 38 | `europepmc_konu_verify` | KAPI — bakteri/mikrobiyom ingest hattı öz-testleri (10.09, Derya). |
| 39 | `gads_offline_verify` | gads_cevrimdisi_donusum.py DOĞRULAMA — hesapta henüz UPLOAD_CLICKS eylemi YOK, o yüzden |
| 40 | `geri_cekilme_kapi_verify` | FAIL-CLOSED KAPI DOĞRULAMASI — `fetch_retractions.backfill` damga DÜŞÜREBİLİR mi? (#18) |
| 41 | `inis_beyani_bozma_tatbikati` | BOZMA TATBİKATI — `inis_kaynak_beyani_verify` KIRMIZI da basabiliyor mu? |
| 42 | `inis_kontrast_verify` | İNİŞ SAYFASI KONTRAST KAPISI — sayfadaki HER metin düğümü WCAG AA eşiğinde. |
| 43 | `kabuk_geometri_verify` | KAPI 2 · KABUK GEOMETRİSİ — sayfayı GERÇEKTEN render edip rayın altında içerik var mı ölçer. |
| 44 | `kamu_kaynak_verify` | KAMU KAYNAK KAPISI — #763c-İNDİR (2026-09-11, Derya): `scripts/ingest_kamu_sayfa.py` |
| 45 | `kimlik_kapisi_verify` | İLAÇ KİMLİĞİ KAPISI — çivileme takımı (saglik/kimlik.py + connector yazma yolu). |
| 46 | `kismi_cokme_tatbikati` | KISMİ ÇÖKME BOZMA TATBİKATI — `etkilesim_govde_verify` istisna altında KAÇ iddia basar? |
| 47 | `kub_kart_blok_verify` | TR KÜB bloğu REFERANS KARTINDA çiziliyor mu? (2026-07-29 düzeltmesinin çivisi) |
| 48 | `marka_kuyruk_verify` | #50b — KISA NİTELİK KUYRUĞU OLAN MARKA: `_drugs_in_query` doğru ürünü seçiyor mu? |
| 49 | `maske_kabul_olceri` | ÇÖKME MASKESİ — KABUL ÖLÇERİ (görev #29 aracı). |
| 50 | `meta_targeting_verify` | Read-only TR-scoped reach measurement via delivery_estimate (no writes, no spend). |
| 51 | `meta_targeting_verify2` | — |
| 52 | `mutabakat_kapisi` | MUTABAKAT KAPISI — /admin/odeme/mutabakat denetim sondasi (Firat, 2026-08-10). |
| 53 | `on_okuma_kalicilik_sonda` | Selim WP-5 — `app.case_file.on_okuma*` KALICILIK SONDASI (#620b YOL 2, sözleşme §2.3 / §5). |
| 54 | `panel_baglanti_verify` | Panel BAGLANTI kapisi (Deniz, 2026-08-03) — rayin "Panel" ogesi GERCEKTEN calisiyor mu. |
| 55 | `panel_renk_kapisi` | /panel RENK + KIRPMA KAPISI — Deniz, 2026-08-04. |
| 56 | `panel_rozet_tatbikat` | PANEL ROZET BANDI — **EKRAN KATMANI** BOZMA TATBİKATI (Fırat P2, 2026-08-01). |
| 57 | `post_parite_verify` | POST PARİTESİ — refactor'ın GET ile ölçülemeyen yüzünü ölçer. |
| 58 | `push_kapisi` | PUSH KAPISI — lead'in push öncesi kontrolünü MEKANİKLEŞTİRİR. |
| 59 | `rag_kopru_verify` | RAG TERİM KÖPRÜSÜ doğrulaması (2026-07-29, denetim P1) — LLM ÇAĞIRMAZ. |
| 60 | `ray_panel_verify` | DENIZ: rayda "Panel" ogesi — BES SAYFA x TR/EN x masaustu/mobil dogrulamasi. |
| 61 | `redact_ilac_verify` | İLAÇ ADI [AD] OLARAK MASKELENMEZ — /admin/sorular ölçüm körlüğü, kalıcı doğrulama. |
| 62 | `reklam_rapor_verify` | reklam_rapor.py DÜZELTMELERİ — regresyon testi (2026-07-29). |
| 63 | `satir_kapisi_tatbikati` | SATIR KAPISI BOZMA TATBİKATI — `parite_maske_verify` bölüm 5 GERÇEKTEN kırmızı basar mı? |
| 64 | `sekil_kapisi_verify` | NETLEŞTİRME ŞEKİL KAPISI — kalıcı doğrulama. |
| 65 | `siralama_bozma_tatbikati` | KASITLI BOZMA TATBİKATI: sıralama kuralını geri alınca drug_match_verify KIRMIZI mı? |
| 66 | `terim_kopru_verify` | TERİM DÜZEYİNDE TR→EN KÖPRÜSÜ — kalıcı doğrulama. |
| 67 | `token_butce_verify` | #62 — DOCTOR_SYSTEM ≥4096 TOKEN TABANI: GERÇEK tokenizer'a bağlandı (2026-08-02). |
| 68 | `tr_kub_blok_verify` | TÜRKİYE RUHSATLI BİLGİ bloğu — kalıcı doğrulama. |
| 69 | `tr_marka_canli_yanit` | UÇTAN UCA: model GERÇEKTEN Türk markasını söylüyor mu? (1 gerçek LLM çağrısı) |
| 70 | `tr_marka_verify` | TR MARKA GÖRÜNÜRLÜĞÜ — regresyon ağı (2026-07-31). |

### C. `JS_SUITES` — Node ile koşan kapılar  (16)

`node scratchpad/<ad>.js`

| # | Kapı | Ne çiviler |
|---:|---|---|
| 1 | `calc_arama_verify` | ============================================================================= |
| 2 | `calc_verify` | ============================================================================= |
| 3 | `chat_liste_verify` | -*- coding: utf-8 -*- |
| 4 | `cihaz_enjekte_js_verify` | ENJEKTE EDİLEN JS DERLENİYOR MU — masaüstü program (#596b, 2026-09-01). |
| 5 | `enabiz_belge_yukleme_verify` | e-Nabız masaüstü — BELGE İNDİRME + ŞİFRELİ YÜKLEME KAPISI (#621b).  Koşum: |
| 6 | `enabiz_cekirdek_tasinabilir_verify` | e-NABIZ ÇEKİRDEĞİ — TAŞINABİLİRLİK KAPISI.  Koşum: node scratchpad/enabiz_cekirdek_tasinabilir_verify.js |
| 7 | `enabiz_derin_baglanti_verify` | DERİN BAĞLANTI KAPISI — `klivance-bridge://aktar?case=<id>` (#602b, 2026-09-01). |
| 8 | `enabiz_eslestir_verify` | Eşleştirme kapısı.  Koşum:  node scratchpad/enabiz_eslestir_verify.js |
| 9 | `enabiz_giris_liste_koprusu_verify` | MASAÜSTÜ — OTO-GİRİŞ · KART LİSTESİ · ÇEKİM→AKTARIM KÖPRÜSÜ (#605b, 2026-09-01). |
| 10 | `enabiz_grup_sizinti_verify` | e-Nabız GRUP (panel) SATIRI SIZINTISI kapısı.  Koşum: |
| 11 | `enabiz_indirme_verify` | e-Nabız eklentisi — BELGE İNDİRME KAPISI.  Koşum: |
| 12 | `enabiz_kopru_arguman_verify` | MASAÜSTÜ PROGRAM — preload KÖPRÜSÜ ARGÜMAN SÖZLEŞMESİ KAPISI. |
| 13 | `enabiz_masaustu_guvenlik_verify` | MASAÜSTÜ PROGRAM — GÜVENLİK TABANI KAPISI. |
| 14 | `enabiz_tahlil_grup_verify` | e-Nabız TAHLİL PANEL (GRUP) SATIRI kapısı.  Koşum: |
| 15 | `enabiz_yol_kapisi_verify` | Yol kapısı + harita tutarlılık kapısı.  Koşum:  node scratchpad/enabiz_yol_kapisi_verify.js |
| 16 | `uzun_is_ilerleme_verify` | ============================================================================= |


---

## 9. Yerel koşum reçeteleri

```bash
# Tek kapı
./.venv/Scripts/python.exe scratchpad/<ad>_verify.py

# JS kapısı
node scratchpad/<ad>_verify.js

# Sürüklenme kapısı (yeni kapı eklediysen ŞART)
./.venv/Scripts/python.exe scratchpad/ci_kapi_verify.py

# Yol/OS kapısı (ters eğik çizgi, sabit yol)
./.venv/Scripts/python.exe scratchpad/ters_egik_yol_tara.py

# Dosya tavanı
./.venv/Scripts/python.exe scratchpad/dosya_boyut_verify.py

# Syntax doğrulama (tek dosya)
./.venv/Scripts/python.exe -c "import ast; ast.parse(open('saglik/app/main.py',encoding='utf-8').read())"

# CI taklidi (⚠ "CI yeşil olur" DEMEZ)
./.venv/Scripts/python.exe scratchpad/ci_taklit.py
```

---

## 10. Değişmez kurallar (özet)

1. **Doğrulama scriptleri COMMIT EDİLİR.**
   ⚠ *"(24/24 geçti)"* iddiası görünce **önce dosyanın VAR olduğunu doğrula** — koşulamayan
   test, **olmayan güvencedir**.
2. **Test iddiası uygulamadan ayrışırsa TESTİ düzelt** — bir kez bir hatayı "beklenen
   davranış" olarak çivilemişti.
3. **COMMIT EDİLMEK YETMEZ, KAPIYA BAĞLANMASI GEREKİR.**
4. **Gizli KB bağımlılığı grep'le bulunmaz → AST.**
5. **SKIP'e koyduğun her takımın gerekçesi yorum bloğunda yazılı olmalı** (`ci_kapi_verify`
   bölüm 3 ölçer).

---

**Sonraki:** [`12-DEPLOY-ORTAM-ENV.md`](12-DEPLOY-ORTAM-ENV.md)
