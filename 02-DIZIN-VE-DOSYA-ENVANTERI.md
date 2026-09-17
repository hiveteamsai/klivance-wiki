# 02 — DİZİN VE DOSYA ENVANTERİ

> Ölçüm anı: 2026-09-17 · commit `cb16758d`. Modül tabloları AST ile üretildi;
> "Ne yapar" sütunu **dosyanın kendi başlık docstring'inin ilk satırıdır** — özet değil,
> kaynağın kendi beyanı.

---

## 1. Depo kökü — hangi dizin ne işe yarar

| Yol | İçerik | Devralan için önem |
|---|---|---|
| **`saglik/`** | **ÜRÜN KAYNAK KODU** (140 Python modülü, ~69.000 satır) | ★★★ Tek gerçek ürün ağacı |
| **`db/`** | `schema.sql` (KB yapısı) + `app_schema.sql` (SaaS tabloları) | ★★★ Şemanın kaynağı; `migrate.py` bunları uygular |
| **`scripts/`** | 200+ operasyon/ingest/reklam scripti | ★★☆ Bkz. `13-SCRIPT-ENVANTERI.md` |
| **`scratchpad/`** | **320 doğrulama kapısı** + ölçüm araçları + ajan bulguları | ★★★ CI'ın koştuğu testler BURADA (`tests/` dizini YOK) |
| **`.github/workflows/`** | 7 GitHub Actions iş akışı | ★★★ CI + 4 cron |
| **`docs/`** | 31 karar/ölçüm/sözleşme dokümanı | ★★☆ "Neden böyle" cevapları |
| **`.claude/agents/`** | Alan uzmanı ajan tanımları (alan bilgisi deposu) | ★★☆ Derin alan kuralları burada |
| **`.claude/skills/`** | İş akışı skill'leri (`/ekip`, `/devam` …) | ★☆☆ |
| **`data/`** | İndirilmiş kaynak PDF/HTML (CDC, ECDC, Green Book, EuropePMC…) | ★☆☆ Ingest girdisi |
| **`Klivance_software v101/`** | e-Nabız Chrome eklentisi + masaüstü program (Electron) | ★★☆ Bkz. `14-MASAUSTU-VE-EKLENTI.md` |
| **`Klivance-Marka-Kimligi/`** | Logo, font, marka kılavuzu | ★☆☆ |
| **`reklam/`** | Reklam kreatifleri + vaka külliyatı + performans defteri | ★☆☆ Ürün kodu değil |
| **`social/`, `sosyal-paylasim/`** | Sosyal medya varlıkları | ☆☆☆ |
| **`Resimler/`, `referans resimler/`** | Ekran görüntüleri, referans görseller | ☆☆☆ |
| **`data/`, `raw` şeması** | Ara-staging (~4 GB) — **prod'a ALINMAZ** | ☆☆☆ |

⚠ **`tests/` dizini YOKTUR** ve bu bir eksiklik değil, tercihtir: doğrulama scriptleri
`scratchpad/*_verify.py` olarak yazılır, `sys.exit(0/1)` döner ve `.github/workflows/test.yml`
içindeki `SUITES` dizesine eklenir. **Listede olmayan script sessizce ÖLÜ ağdır.**

### 1.1 Kök dosyalar

| Dosya | Ne |
|---|---|
| `CLAUDE.md` | **Proje kılavuzu** — her oturumun okuduğu değişmez kurallar + dosya haritası (~71 KB) |
| `_gorev.txt` | **Görev/kalem defteri** (~720 KB) — açık işler, ölçümler, founder kararları |
| `AGENTS.md` | Ajan giriş noktası |
| `README.md` · `DEPLOY.md` · `NOTLAR.md` | Kurulum ve deploy notları |
| `sources.yaml` | **Kaynak kayıt defteri** — connector başına erişim/lisans/rate-limit |
| `requirements.txt` | App runtime bağımlılıkları |
| `requirements-ads.txt` | **AYRI** — `google-ads` (ağır grpcio/protobuf); app runtime import ETMEZ |
| `Dockerfile` · `render.yaml` · `docker-compose.yml` | Konteyner + deploy |
| `_kapi_FIRAT_*.log` | Kontrolcü kapı logları |

---

## 2. Ürün kaynak ağacı — `saglik/`

### WEB UYGULAMA KATMANI — `saglik/app/`

*82 modül.*

| Dosya | Satır | Fn | Rota | Ne yapar |
|---|---:|---:|---:|---|
| `__init__.py` | 7 | 0 |  | Klivance SaaS uygulama katmanı (FastAPI) — bilgi tabanı + CDS motorunun üstünde. |
| `abonelik_body.py` | 49 | 0 |  | iyzico ABONELİK (recurring) sayfa gövdeleri — SAF şablon (#774c C2). |
| `abonelik_dili.py` | 126 | 4 |  | ABONELİK DİLİ — tek çekim ↔ otomatik yenileme ibareleri, BAYRAĞA BAĞLI (#774c C3, Selim 2026-09-12). |
| `abonelik_routes.py` | 486 | 26 | 6 | iyzico ABONELİK (recurring) rotaları — #774c C2 (Selim, 2026-09-12). |
| `account_routes.py` | 1398 | 31 | 14 | HESAP + ÖDEME ROTALARI — /account, abonelik, top-up, fatura profili, iyzico dönüşü |
| `admin_alarm.py` | 131 | 3 |  | AYLIK KULLANIM ALARMI — sorgu + `/admin` kartı (kendi evi, 2026-08-22 · #396b-K). |
| `admin_denetim.py` | 390 | 9 |  | ADMİN ERİŞİM / DENETİM LOGU — yazma yolu + okuyucu (#522b, 2026-08-31). |
| `admin_grafik.py` | 130 | 3 |  | AY AY EĞİLİM GRAFİĞİ — satır içi SVG (founder 2026-09-04: "grafik yap düştüğünü |
| `admin_hasta.py` | 394 | 9 | 3 | ADMİN — HEKİM GÖRÜNÜRLÜĞÜ: hasta defteri + davranış zaman çizgisi (#726b, founder 2026-09-04). |
| `admin_hekim.py` | 115 | 2 | 1 | HEKİM-ODAKLI ADMİN SAYFALARI — `/admin/{did}/sorular` (founder 2026-08-15: "adminde |
| `admin_pano.py` | 681 | 17 |  | ADMIN PANOSU — `/admin` KART GÖVDELERİ (bölme, 2026-08-24 · #396b-Q). |
| `admin_queries.py` | 1000 | 21 |  | ADMIN PANELİ SORGULARI — SALT-OKUMA raporlama (para YAZMAZ). |
| `admin_reklam.py` | 561 | 10 |  | AYLIK REKLAM HARCAMASI — yıl+ay seçilerek girilen tutar (founder 2026-08-19: |
| `admin_routes.py` | 1682 | 28 | 15 | ADMIN ROTALARI — kurucu paneli (Faz 2b, 2026-07-31). |
| `admin_soru_analiz.py` | 465 | 16 |  | #715b K3 · PANEL — `/admin/soru-analiz` (SPEC: docs/soru-analitigi-spec-2026-09-04.md §2, §4). |
| `admin_sunum.py` | 488 | 13 |  | ADMIN SUNUM YARDIMCILARI — rota TAŞIMAYAN, panelin "ne söyleyebilir" disiplini. |
| `admin_uyeler.py` | 307 | 3 |  | ADMİN ÜYE LİSTESİ — sorgu · sıralama beyaz listesi · tablo gövdesi (bölme, 2026-09-03 · #670b). |
| `analiz_routes.py` | 316 | 5 | 1 | ANALİZ ROTASI — dosya kütüphanesi map-reduce → TAM KLİNİK ÖZET (#635b/#642b). |
| `auth.py` | 648 | 29 |  | Doktor kimlik doğrulama — Argon2id parola + sunucu-taraflı oturum. |
| `auth_routes.py` | 1818 | 52 | 19 | KİMLİK / OTURUM ROTALARI — kayıt, giriş, çıkış, doğrulama, parola sıfırlama, onay |
| `billing.py` | 337 | 11 |  | Stripe abonelik entegrasyonu — Checkout + Webhook + Müşteri Portalı. |
| `butce_bant.py` | 91 | 2 |  | `/chat` ÜST BANDI — DÖNEM BÜTÇESİ (#650b-EK, 2026-09-03). Kabuk BURADA, karar/metin `butce_kapisi`de. |
| `butce_defteri.py` | 207 | 10 |  | Dönem bütçe kapısı — DEFTER katmanı (2026-09-02, Selim). Tasarım: |
| `butce_kapisi.py` | 135 | 6 |  | DÖNEM BÜTÇE KAPISI — `_butce_fren(d, conn, lang, yuzey)` (founder 2026-09-02: "zarar etmeyelim"). |
| `cal_body.py` | 363 | 0 |  | Takvim sayfa gövdesi (HTML + JS). |
| `cal_routes.py` | 332 | 18 | 6 | TAKVİM — randevu + klinik takip yüzeyi (Faz 9, 2026-08-20). |
| `calc_body.py` | 2341 | 2 |  | Klinik hesaplayıcılar sayfası gövdesi (istemci-tarafı, kredi harcamaz). |
| `cases_body.py` | 410 | 0 |  | /cases sayfa gövdeleri (HTML + CSS + JS şablonları). |
| `cases_routes.py` | 1500 | 21 | 10 | PSEUDONİM HASTA DEFTERİ + ŞİFRELİ DOSYA KÜTÜPHANESİ (Faz 7, 2026-08-01). |
| `chat_body.py` | 1768 | 0 |  | /chat sayfa gövdesi (HTML + JS). |
| `chat_goruntu_js.py` | 294 | 1 |  | /chat — YOL 2 "Görüntü ön-okuma" modunun İSTEMCİ ayağı (#620b, sözleşme |
| `chat_routes.py` | 1593 | 37 | 15 | SOHBET + CDS API ROTALARI — /chat, /api/chat (AKIŞ), ikinci-geçiş ajanları |
| `chat_turn.py` | 337 | 19 | 2 | SOHBET TURU — ÜRETİM BAĞLANTIDAN AYRIK + KOPAN İSTEMCİNİN TOPARLANMASI (#748b, founder 06.09). |
| `cihaz_routes.py` | 634 | 19 | 6 | CİHAZ EŞLEŞTİRME — masaüstü programın Klivance kimliği (#596b, 2026-09-01). |
| `credits.py` | 495 | 12 |  | Erişim modeli — SINIRSIZ ABONELİK (2026-08-21 founder kararı, kredi sistemi kaldırıldı). |
| `deger_onay.py` | 176 | 5 | 1 | ANALİZDEN ÇIKAN DEĞERİ KARTA İŞLEME — ONAYLI (#362b, founder kararı (a) 2026-08-19). |
| `enabiz_tahlil.py` | 493 | 18 | 1 | e-NABIZ TAHLİL PAKETİ — HTML (akordiyon) → hasta kartı "belge" → analiz/kokpit (#668b). |
| `etkilesim_body.py` | 901 | 23 |  | /etkilesim — HERKESE AÇIK ilaç etkileşimi tarama sayfası (giriş yok, kredi yok, LLM yok). |
| `etkilesim_page.py` | 762 | 9 |  | /etkilesim sayfa iskeleti — ilk ekran, form, N İLAÇ (2..8), TR+EN. |
| `etkilesim_yorum.py` | 884 | 21 | 1 | `/etkilesim` AI YORUMU — çoklu-ilaç etiket taramasının klinik yorumu (giriş + 1 kredi). |
| `fiyat.py` | 265 | 9 |  | FİYAT GÖSTERİMİ — tutar VE para biriminin TEK KAYNAĞI (2026-08-09, #137b + #139b). |
| `form_listeler.py` | 83 | 0 |  | Kayıt/onay formunun UNVAN ve BRANŞ listeleri — SAF VERİ. |
| `geo.py` | 204 | 12 |  | Ülke + şehir seçimi — kayıt/onay formlarının SUNUCU tarafı (2026-08-15, founder: |
| `gizli_kaynak.py` | 344 | 8 |  | GİZLİ KAYNAK — gösterimde adı GEÇMEYECEK kaynakların sunucu-taraflı süzgeci (2026-08-19). |
| `guide.py` | 22 | 0 |  | Kullanım kılavuzu (/kilavuz) — TR+EN gövde HTML + CSS. |
| `guvenlik.py` | 235 | 0 |  | Güvenlik ve veri koruması sayfası (/guvenlik) — TR+EN gövde HTML. |
| `hastalik.py` | 331 | 8 | 3 | HASTALIKLAR — `/hastaliklar` (founder 2026-08-19: "ana menüye hastalıklar butonu; |
| `hastalik_body.py` | 824 | 4 |  | HASTALIKLAR SAYFASININ GÖRSEL KATMANI — `/hastaliklar` (founder tasarımı 2026-08-19). |
| `hekim_iz.py` | 244 | 6 |  | HEKİM DAVRANIŞ İZİ — `app.doctor_event` kayıt katmanı (#726b, founder 2026-09-04). |
| `i18n_pairs.py` | 605 | 0 |  | TR→EN çeviri çiftleri (i18n_mw tarafından uzun-önce sırayla uygulanır). |
| `iyzico.py` | 423 | 14 |  | iyzico ödeme entegrasyonu — Checkout Form (hosted, 3DS dahil). TR-önce kredi kartı tahsilatı. |
| `iyzico_abonelik.py` | 553 | 25 |  | iyzico ABONELİK (recurring) — Subscription API istemcisi + hak-ediş defteri (#774c, C1). |
| `kokpit_body.py` | 577 | 0 |  | ANALİZ KOKPİTİ — CSS + JS sabitleri (spec: scratchpad/_omer_kokpit_spec.md §3; |
| `kritik_serit.py` | 216 | 8 |  | KRİTİK BULGU KADEMESİ — TAŞIYICI + ÇÖZÜCÜ (#782c, founder 2026-09-15 "aç, daha da geliştir"). |
| `landing.py` | 1343 | 1 |  | LANDING (herkese açık satış sayfası) — TR + EN gövdeleri ve parçaları. |
| `legal_body.py` | 261 | 0 |  | Yasal metinler — TR (_LEGAL) ve EN (LEGAL_EN). |
| `mailer.py` | 804 | 14 |  | E-posta gönderimi — SMTP (Resend transactional, gönderen info@klivance.com). |
| `main.py` | 1346 | 38 | 19 | Klivance — FastAPI SaaS uygulaması (Faz 1a). |
| `migrate.py` | 374 | 17 |  | Şema migrasyonu — açılışta çalışır (idempotent). |
| `olcum_js.py` | 51 | 0 |  | Ray tıklama ölçümü — SAF JS sabiti (2026-08-20, founder "ölçmek isterim"). |
| `on_okuma.py` | 688 | 27 | 4 | GÖRÜNTÜ ÖN-OKUMA (#620b YOL 2, founder 2026-09-13) — ROTA MODÜLÜ + KART HTML ÜRETİCİLERİ (WP-6, Kamil). |
| `on_okuma_kaydet.py` | 193 | 6 | 1 | SOHBET GÖRSELİNİ HASTA KARTINA KAYDET (#781c, founder 2026-09-15) — TEK UÇ (Kamil). |
| `on_okuma_kayit.py` | 321 | 20 |  | Görüntü ön-okuması (#620b YOL 2, WP-3) — SOHBET/KAYIT tarafındaki SAF yardımcılar. |
| `on_okuma_store.py` | 73 | 4 |  | Görüntü ön-okuması — `app.case_file.on_okuma*` kolonlarının TEK yazıcı/okuyucusu (#620b YOL 2, Selim WP-5). |
| `ornek_sorular.py` | 173 | 2 |  | `/chat` karşılama ekranındaki ÖRNEK SORULAR — branşa göre. SAF VERİ. |
| `panel_body.py` | 684 | 13 |  | `/panel` — yargım deseninin uyarlaması, EK sayfa (2026-08-02, founder yönü). |
| `panel_routes.py` | 679 | 7 | 1 | `/panel` — yargım deseninin uyarlaması, EK sayfa (2026-08-02, founder yönü). |
| `rad_case.py` | 265 | 17 | 1 | GÖRÜNTÜLEME (YOL 1) — RAPOR METNİNDEN LEZYON ŞERİDİ + TAKİP RANDEVUSU (IP2, 2026-09-13). |
| `recete_body.py` | 706 | 14 |  | REÇETE KONTROLÜ — SAF GÖVDE (CSS + metin sözlükleri + render yardımcıları). |
| `recete_case.py` | 981 | 19 | 1 | HASTA KARTI · REÇETE KONTROLÜ SEKMESİ — panel gövdesi + JSON ucu (#336b, 2026-08-17). |
| `recete_routes.py` | 442 | 10 | 3 | REÇETE KONTROLÜ ROTALARI — `/recete-kontrolu` (hub) + `/bobrek-doz` (böbrek doz kontrolü). |
| `ref_body.py` | 427 | 0 |  | /reference sayfa gövdesi (HTML + JS). |
| `ref_routes.py` | 344 | 7 | 5 | REFERANS + ÜCRETSİZ ARAÇ YÜZEYİ (Faz 8, 2026-08-01). |
| `sgk_ara.py` | 393 | 7 | 1 | SGK BEDELİ ÖDENECEK İLAÇLAR LİSTESİNDE ARAMA (founder 2026-08-19). |
| `sgk_case.py` | 641 | 16 | 1 | HASTA KARTI · SGK GERİ ÖDEME KONTROLÜ — AYRI DÜĞME (founder 2026-08-19). |
| `sohbet_baslik.py` | 656 | 20 |  | #736c · OTOMATİK SOHBET BAŞLIĞI — hekimin kendi el yazısındaki gibi klinik kısaltma. |
| `soru_analiz.py` | 531 | 14 |  | #715b K2 · TÜRETME — `app.question_insight` (SPEC: docs/soru-analitigi-spec-2026-09-04.md). |
| `soru_kayit.py` | 357 | 14 |  | #715b K1 · KAYIT — katman sonucu + kanıt paketi künyesi (SPEC: docs/soru-analitigi-spec-2026-09-04.md). |
| `store.py` | 2553 | 101 |  | Kalıcılık + erişim — sohbet thread/mesaj, pseudonim vaka, kullanım kaydı. |
| `tr_saat.py` | 38 | 2 |  | TÜRKİYE GÜN SINIRI — takvim · panel · hasta kartı için TEK "şimdi" (2026-08-20, takvim denetimi kalem 2). |
| `ui_css.py` | 1267 | 0 |  | Uygulama CSS'i — TEK KAYNAK (TR ve EN sayfalar aynı sabiti kullanır). |
| `webutil.py` | 1256 | 34 |  | WEBUTIL — uygulamanın PAYLAŞILAN SÖZLÜĞÜ (Faz 2a, 2026-07-31). |

### KLİNİK KARAR-DESTEK MOTORU — `saglik/cds/`

*36 modül.*

| Dosya | Satır | Fn | Rota | Ne yapar |
|---|---:|---:|---:|---|
| `__init__.py` | 10 | 0 |  | Klivance — hekim klinik karar-destek (CDS) motoru. |
| `acil_toksikoloji.py` | 49 | 2 |  | ACİL TOKSİKOLOJİ VE ANTİDOT PROTOKOLLERİ MOTORU (acil_toksikoloji.py). |
| `aklama.py` | 359 | 4 |  | AKLAMA DİLİ + YÖNLENDİRME YASAĞI — ÜRÜN AĞACINDAKİ TEK KAYNAK. |
| `aklama_monitor.py` | 113 | 3 |  | ANALİZ (REDUCE) ÇIKTISI AKLAMA / YÖNLENDİRME MONİTÖRÜ — SALT-GÖZLEM (#665b, 2026-09-03). |
| `analyze.py` | 400 | 9 |  | Kalıcı hasta dosya kütüphanesi — MAP-REDUCE analiz. |
| `biyoesdegerlik.py` | 50 | 2 |  | BİYOEŞDEĞERLİK VE JENERİK İLAÇ DEĞİŞTİRME REHBERİ (biyoesdegerlik.py). |
| `chat.py` | 2022 | 54 |  | Konuşan doktor-AI orkestratörü. |
| `deger_cikar.py` | 146 | 4 |  | ANALİZ METNİNDEN KART DEĞERİ ÇIKARIMI — `deger_cikar` (#362b, founder 2026-08-19). |
| `dil.py` | 266 | 5 |  | Yanıt dili = SORUNUN dili (founder kararı 2026-08-15, `_gorev.txt` #275b). |
| `engine.py` | 1315 | 34 |  | Anahtarsız Klivance motoru — soruyu yönlendirir, kaynaklı yanıt üretir (LLM YOK). |
| `etkilesim_coklu.py` | 552 | 11 |  | ÇOKLU-İLAÇ ETKİLEŞİM TARAMASI — `/etkilesim`in N-ilaç motoru (founder 2026-08-18: |
| `faithfulness.py` | 387 | 7 |  | Atıf-sadakati kontrolü (QW2) — LLM'siz, deterministik, SALT-GÖZLEM (aşama-1). |
| `fenotip.py` | 404 | 6 |  | FENOTİP → NADİR HASTALIK ADAYLARI (`core.symptom` + `core.symptom_disease`). |
| `fenotip_capa.py` | 1442 | 0 |  | ÜRETİLMİŞ DOSYA — ELLE DÜZENLEME. Üreteç: `scripts/hpo_capa_uret.py`. |
| `global_kb.py` | 310 | 13 |  | KLIVANCE GLOBAL CLINICAL KNOWLEDGE ENGINE (global_kb.py). |
| `hastalik_kanit.py` | 883 | 27 |  | HASTALIK MONOGRAFI — `hastalik` modunun KANIT PAKETİ (founder 2026-08-19). |
| `isaretler.py` | 300 | 2 |  | Görüntüleme (YOL 1 rapor metni · YOL 2 görüntü ön-okuma) — hekime görünen ÖNEK ve METİN sabitleri. |
| `kapanis.py` | 107 | 2 |  | KAPANIŞ SATIRI — İKİ HÂLLİ, SUNUCUDA DETERMİNİSTİK (founder kararı 2026-09-11, #769c). |
| `kaynaklar.py` | 384 | 11 |  | KAYNAK KAYITI — atıf zincirinin TEK DOĞRU KAYNAĞI (2026-08-01, görev #56). |
| `koruyucu_hekimlik.py` | 46 | 2 |  | KORUYUCU HEKİMLİK VE TARAMA PROTOKOLLERİ MOTORU (koruyucu_hekimlik.py). |
| `kritik_goruntu.py` | 370 | 13 |  | #782c KRİTİK BULGU KATMANI — görüntü ön-okumasında zaman-kritik ADAY kademesi. |
| `llm.py` | 674 | 18 |  | Claude istemcisi — model kademesi, prompt caching, maliyet hesabı. |
| `metering.py` | 44 | 1 |  | Doktor başına token/maliyet ölçümü — basit JSONL kaydı (v1). |
| `pgx.py` | 49 | 2 |  | FARMAKOGENOMİK (PGX) VE KİŞİSELLEŞTİRİLMİŞ TIP MOTORU (pgx.py). |
| `pipeline.py` | 1020 | 23 |  | Çok-ajanlı 'ikinci-geçiş' katmanı — yanıt üretildikten sonra ucuz denetim ajanları. |
| `prompts.py` | 1126 | 9 |  | Klivance sistem promptlari — DONUK metinler (prompt-caching icin sabit). |
| `rad_cikar.py` | 555 | 28 |  | RAD satırı — radyoloji RAPORU METNİNDEN yapılandırılmış lezyon satırı (YOL 1, #620b değişimi). |
| `rad_metin.py` | 111 | 5 |  | #778c — GÖRÜNTÜLEMEDE GÖRÜNTÜ OLMADAN METİN (founder 2026-09-14): hekimin yazdığı/yapıştırdığı rapor metni. |
| `recete_kontrol.py` | 929 | 22 |  | HASTA KARTINDAN REÇETE KONTROLÜ — `kontrol()` (Reçete kontrolü ailesi 3. adım; founder |
| `reference.py` | 1866 | 39 |  | İlaç referans + etkileşim tarayıcı — LLM'siz, doğrudan bilgi tabanından. |
| `renal.py` | 593 | 16 |  | BÖBREK DOZ KONTROLÜ MOTORU — `renal_check` (Reçete kontrolü ailesinin ilk aracı; founder |
| `retrieval.py` | 1741 | 28 |  | Kaynaklı kanıt çekimi — hastalık adayları + klinik literatür (LLM'siz). |
| `rxclass.py` | 87 | 5 |  | RxClass ilaç-sınıfı yardımcısı — etken madde → ATC sınıfı + etki mekanizması (MoA). |
| `rxnorm.py` | 137 | 6 |  | RxNorm ad-çözümleme yardımcısı — TR marka/typo → RxNorm etken madde adı (İngilizce). |
| `sgk.py` | 161 | 5 |  | SGK Ek-4/A geri ödeme LİSTE DURUMU (#354b, 2026-08-19). |
| `translate.py` | 91 | 3 |  | Referans/KB içeriği çevirisi — seçili dil kaynak diliyle (İngilizce) aynı değilse. |

### VERİ BAĞLAYICILARI — `saglik/connectors/`

*10 modül.*

| Dosya | Satır | Fn | Rota | Ne yapar |
|---|---:|---:|---:|---|
| `__init__.py` | 32 | 1 |  | Connector kayıt defteri: source_id → connector sınıfı. |
| `base.py` | 77 | 0 |  | Tüm connector'ların ortak arayüzü. |
| `clinicaltrials.py` | 149 | 1 |  | ClinicalTrials.gov v2 connector. |
| `dailymed.py` | 133 | 1 |  | DailyMed SPL connector (NLM) — resmî FDA ilaç etiketi İNDEKSİ + resmî link. |
| `enforcement.py` | 90 | 1 |  | openFDA Drug Enforcement (geri çağırma / FDA RES) connector. |
| `icd11.py` | 194 | 1 |  | WHO ICD-11 connector (hastalıklar — omurga kodlama). |
| `mhra.py` | 159 | 3 |  | MHRA Drug Safety Update (GOV.UK) connector → core.corpus (`mhra_dsu`). #763c-A, 2026-09-11, Derya. |
| `openfda.py` | 130 | 1 |  | openFDA Drug Label connector. |
| `orphanet.py` | 74 | 0 |  | Orphanet nadir hastalık connector (Orphadata product1). |
| `titck.py` | 161 | 2 |  | TİTCK KÜB/KT connector (Türkiye ilaçları). |

### ALTYAPI (kök) — `saglik/*.py`

| Dosya | Satır | Fn | Ne yapar |
|---|---:|---:|---|
| `__init__.py` | 3 | 0 | saglik — açık sağlık bilgi tabanı ingestion pipeline'ı. |
| `config.py` | 263 | 18 | Yapılandırma: .env + sources.yaml yükleme. |
| `db.py` | 162 | 9 | PostgreSQL erişimi ve ham katmana yazma. |
| `etiket_tarih.py` | 143 | 4 | Etiket TAZELİĞİ — ham kaynak payload'ından tarih + sürüm çıkarımı (#70). |
| `filecrypto.py` | 69 | 4 | Hasta dosyası uygulama-katmanı şifrelemesi (Fernet, AES128-CBC + HMAC). |
| `http.py` | 102 | 0 | HTTP istemcisi — tarayıcı TLS parmak izi taklidi + rate limit + retry. |
| `ingest_state.py` | 129 | 6 | Sürekli besleme hattı — watermark (core.sync_state) + koşu logu (core.ingest_log) + |
| `kimlik.py` | 107 | 2 | İLAÇ KİMLİĞİ KAPISI — bir metin gerçekten bir ilaç adı mı? |
| `redact.py` | 114 | 2 | Deterministik kimlik (PII) redaksiyonu — KVKK güvenlik ağı. |
| `ads/__init__.py` | 2 | 0 | Reklam entegrasyonları (Google Ads · Meta Marketing API). Kimlikler env'de; Claude okumaz. |
| `ads/google_ads.py` | 76 | 4 | Google Ads API entegrasyonu — istemci fabrikası + salt-okuma yardımcıları. |
| `web/__init__.py` | 1 | 0 | — |


---

## 3. Dosya adlandırma konvansiyonları

| Desen | Anlamı | Kural |
|---|---|---|
| `*_routes.py` | Rota modülü | `@_rota(metod, yol)` + `def kur(app)`. `main.py`'den import ETMEZ. |
| `*_body.py` | **SAF** sayfa gövdesi (HTML/CSS/JS sabitleri) | f-string yok, modül durumu yok, import ≈ 0 |
| `*_case.py` | Hasta kartı sekmesi/ucu | `/api/cases/{cid}/...` — **statik son segment** (indeks kaymasın) |
| `admin_*.py` | Admin yüzeyi parçası | `_admin()` kapısından geçer (rol **VE** denetim) |
| `scratchpad/*_verify.py` | Doğrulama kapısı | `sys.exit(0/1)` + `test.yml` `SUITES`'e eklenir |
| `scratchpad/*_tatbikati.py` | **Bozma tatbikatı** | Kaynağı bozar, kapının yakaladığını ölçer → `SKIP_SUITES` |
| `scratchpad/_<ajan>_*.py` | Bir ajanın ölçüm aracı | Ürün ağacı değil |
| `scripts/meta_*.py` · `gads_*.py` | Reklam operasyon scripti | Çoğu `--apply` gerektirir; dry-run varsayılan |
| `scripts/ingest_*.py` · `migrate_*.py` | Veri hattı | |

### ⚠ Üretilmiş dosyalar — elle düzenleme
| Dosya | Üreteç |
|---|---|
| `saglik/cds/fenotip_capa.py` | `scripts/hpo_capa_uret.py` |
| `saglik/app/geo.py` verisi | `scripts/geo_veri_uret.py` |

---

## 4. Bir dosyaya dokunmadan önce

1. **`git status` ile KİRLİ Mİ bak** — `git log` yetmez (commit'e bakmak çalışma ağacındaki
   işi göstermez). Bu depoda eş zamanlı çalışma olur.
2. **Dosyanın kendi başlık docstring'ini oku.** Bu depoda invariantlar dosya başlıklarında
   yazılıdır (toplam ~278.000 karakter docstring). `⚠⚠` ile başlayan her satır bir
   sözleşmedir.
3. **Hangi kapı bu dosyayı koruyor?** — `grep -l "<dosya adı>" scratchpad/*_verify.py`
4. **Dosya tavanı** — `python scratchpad/dosya_boyut_verify.py` (tavan aşılıyorsa böl).

---

**Sonraki:** [`03-ROTA-ENVANTERI.md`](03-ROTA-ENVANTERI.md)
