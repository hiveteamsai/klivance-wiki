# 03 — ROTA ENVANTERİ (HTTP yüzeyi)

> **Üretilmiş dosya — elle düzenleme.** Kaynak: canlı `app.routes` (FastAPI uygulamasının kendisi).
> Ölçüm anı: 2026-09-17 · commit `cb16758d`. Yeniden üretmek için export kökündeki
> `veri/` JSON'larını besleyen çıkarıcıyı koştur (bkz. `18-DEVIR-NOTLARI.md` §6).

**Toplam 142 rota · 27 modül.**

Klivance'ta `include_router` **bilerek kullanılmaz** — her modül `@_rota(metod, yol)` ile
kendi kayıt listesine yazar, `main.py` `kur(app)` çağırınca gerçek kayıt `app.get/post`
public API'siyle yapılır. Sonuç: `app.routes` **düz** bir listedir ve rota envanteri
okuyan araçlar körelmez. Bu dosya o listeden üretildi — desenden değil.

**Rota sırası = eşleşme sırası.** `_rota` çağrıldığı anda listeye yazar; aynı deseni
yakalayan iki rotadan **önce kaydedilen** kazanır.

## Erişim kapıları — ücretli her ucun başında ÜÇLÜ ZİNCİR (sırası sabit)

| # | Kapı | Modül | Davranış | Hata kodu |
|---|------|-------|----------|-----------|
| 1 | `_erisim_402` | `webutil` | **FAIL-CLOSED** — abonelik/deneme yoksa duvar | 402 |
| 2 | `_butce_fren` | `butce_kapisi` | Dönem bütçesi; `gozlem` modunda sayar, bloklamaz | 402 |
| 3 | `_gunluk_fren_429` | `webutil` | **FAIL-OPEN** — günlük tavan | **429** (402 değil) |

Tek karar kaynağı `store.erisim_durumu`. Güvenlik uçları (`/api/dose-check`,
`/api/redflag`) bu zincirin **dışındadır** — güvenlik katmanı kota duvarı arkasına konmaz;
freni ayrı `_gunluk_tavan`dır.

---

## `saglik.app.main`  — 19 rota

*Klivance — FastAPI SaaS uygulaması (Faz 1a).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/` | `root` | `main.py:956` |  |
| `GET` | `/chatgpt` | `chatgpt_landing` | `main.py:1272` | `/chatgpt` → 302 → `/` (sorgu dizesi korunur). Gerekçe: yukarıdaki blok. |
| `GET` | `/favicon.ico` | `_favicon_ico` | `main.py:194` |  |
| `GET` | `/fb` | `sl_facebook` | `main.py:757` |  |
| `GET` | `/guvenlik` | `security_page` | `main.py:1279` | Güvenlik ve veri koruması — herkese açık güven sayfası (2026-07-19 ajan paneli P1). |
| `GET` | `/health` | `health` | `main.py:717` | Render sağlık kontrolü (M29). Landing render etmek yerine UCUZ bir DB dokunuşu: |
| `GET` | `/ig` | `sl_instagram` | `main.py:752` |  |
| `GET` | `/interactions` | `sl_interactions` | `main.py:787` | `/interactions` → 301 → `/etkilesim` (sorgu dizesi korunur). |
| `GET` | `/iz-ac` | `notrack_off_page` | `main.py:547` |  |
| `POST` | `/iz-ac` | `notrack_off` | `main.py:553` |  |
| `GET` | `/iz-kapat` | `notrack_on_page` | `main.py:529` |  |
| `POST` | `/iz-kapat` | `notrack_on` | `main.py:535` | Bu cihazda analitik izlemeyi kapat (founder/QA cihazları — /admin açmayan telefon vb. için). |
| `GET` | `/kilavuz` | `guide_page` | `main.py:1201` | Kullanım kılavuzu — herkese açık (kayıt öncesi de okunabilir). Dil rotada seçilir; |
| `GET` | `/lang/{code}` | `set_lang` | `main.py:301` |  |
| `GET` | `/li` | `sl_linkedin` | `main.py:762` |  |
| `GET` | `/robots.txt` | `robots_txt` | `main.py:794` |  |
| `GET` | `/sitemap.xml` | `sitemap_xml` | `main.py:811` |  |
| `GET` | `/tg` | `sl_telegram` | `main.py:767` |  |
| `GET` | `/yasal/{slug}` | `legal_page` | `main.py:1311` |  |

## `(statik/dahili)`  — 1 rota

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `—` | `/static` | `static` | — |  |

## `saglik.app.abonelik_routes`  — 6 rota

*iyzico ABONELİK (recurring) rotaları — #774c C2 (Selim, 2026-09-12).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/admin/odeme/aktivasyon` | `admin_aktivasyon` | `abonelik_routes.py:468` | PENDING abonelikleri aktive et — YALNIZ ADMİN, POST. Aktivasyon = çekim (para eylemi): |
| `POST` | `/api/abonelik/iptal` | `abonelik_iptal` | `abonelik_routes.py:368` |  |
| `POST` | `/api/abonelik/kart` | `abonelik_kart` | `abonelik_routes.py:391` |  |
| `POST` | `/iyzico/abonelik/callback` | `abonelik_callback` | `abonelik_routes.py:253` |  |
| `POST` | `/iyzico/kart/callback` | `kart_callback` | `abonelik_routes.py:414` |  |
| `POST` | `/iyzico/webhook` | `iyzico_webhook` | `abonelik_routes.py:325` |  |

## `saglik.app.account_routes`  — 14 rota

*HESAP + ÖDEME ROTALARI — /account, abonelik, top-up, fatura profili, iyzico dönüşü*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/abonelik` | `abonelik_page` | `account_routes.py:885` |  |
| `GET` | `/account` | `account_page` | `account_routes.py:274` |  |
| `POST` | `/account/delete` | `account_delete` | `account_routes.py:602` | Hesap silme talebi (KVKK unutulma hakkı). Yazılı onay ("SİL"/"DELETE" ya da hesap e-postası) |
| `POST` | `/account/delete/cancel` | `account_delete_cancel` | `account_routes.py:619` | Silme talebini geri al (grace içinde). deletion_requested_at=NULL. |
| `POST` | `/account/feedback` | `account_feedback` | `account_routes.py:630` | Hekim öneri/geri bildirimi → DB'ye kaydet + founder'a e-posta (best-effort). |
| `POST` | `/api/checkout` | `api_checkout` | `account_routes.py:779` | ⚠⚠ 2026-08-09 (#142b): sağlayıcı artık GLOBAL env değil **HEKİMİN PAZARI**. |
| `GET` | `/api/portal` | `api_portal` | `account_routes.py:844` |  |
| `POST` | `/api/profile` | `api_profile_update` | `account_routes.py:572` |  |
| `POST` | `/api/profile/password` | `api_profile_password` | `account_routes.py:583` |  |
| `POST` | `/api/stripe/webhook` | `stripe_webhook` | `account_routes.py:1000` |  |
| `GET` | `/fatura-bilgileri` | `billing_page` | `account_routes.py:678` |  |
| `POST` | `/fatura-bilgileri` | `billing_submit` | `account_routes.py:724` |  |
| `POST` | `/iyzico/callback` | `iyzico_callback` | `account_routes.py:1064` | iyzico Checkout Form dönüşü — hosted form ödeme sonrası tarayıcıyı buraya `token` ile |
| `GET` | `/odeme/sonuc` | `odeme_sonuc` | `account_routes.py:1211` | Ödeme sonuç sayfası (başarılı/tamam/iptal/beklemede) — iyzico callback buraya yönlendirir. |

## `saglik.app.admin_hasta`  — 3 rota

*ADMİN — HEKİM GÖRÜNÜRLÜĞÜ: hasta defteri + davranış zaman çizgisi (#726b, founder 2026-09-04).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/admin/{did}/eylemler` | `admin_eylemler` | `admin_hasta.py:343` | `app.doctor_event` zaman çizgisi + yüzey dağılımı (founder K2). |
| `GET` | `/admin/{did}/hasta/{cid}` | `admin_hasta_detay` | `admin_hasta.py:243` | TAM İÇERİK (founder K1): anlatı, kronik, ilaçlar, alerji, ziyaretler, sentezler, |
| `GET` | `/admin/{did}/hastalar` | `admin_hasta_listesi` | `admin_hasta.py:191` |  |

## `saglik.app.admin_hekim`  — 1 rota

*HEKİM-ODAKLI ADMİN SAYFALARI — `/admin/{did}/sorular` (founder 2026-08-15: "adminde*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/admin/{did}/sorular` | `admin_doctor_questions_page` | `admin_hekim.py:38` |  |

## `saglik.app.admin_routes`  — 15 rota

*ADMIN ROTALARI — kurucu paneli (Faz 2b, 2026-07-31).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/admin` | `admin_page` | `admin_routes.py:145` |  |
| `GET` | `/admin/arsiv` | `admin_archive_page` | `admin_routes.py:829` | Silinen üyeler arşivi — kayıtlar kaybolmaz (kullanıcı kararı); snapshot özetiyle listelenir. |
| `POST` | `/admin/feedback/{fid}` | `admin_feedback_status` | `admin_routes.py:705` | Geri bildirim durumu: new → read → done (yalnız admin). |
| `POST` | `/admin/kur` | `admin_set_kur` | `admin_routes.py:669` | USD/TRY kurunu kaydet (yalnız admin). Sonuç `/admin?kur=…` ile GÖRÜNÜR (ok · gecersiz · |
| `GET` | `/admin/odeme` | `admin_payment_diag` | `admin_routes.py:1144` | iyzico ödeme teşhisi — son denemeleri listeler; hak-edilmemiş (granted=false) satırların |
| `POST` | `/admin/odeme/mutabakat` | `admin_mutabakat` | `admin_routes.py:1013` | Bekleyen iyzico ödemelerini mutabakata sok (hak-ediş + makbuz). YALNIZ ADMİN. |
| `POST` | `/admin/reklam` | `admin_set_reklam` | `admin_routes.py:689` | SEÇİLEN AYIN toplam reklam harcamasını kaydet (Meta + Google, elle) — birim-ekonomi |
| `GET` | `/admin/soru-analiz` | `admin_soru_analiz_page` | `admin_routes.py:1090` | Soru analizi panosu (#715b K3). Ham soru metni GÖSTERMEZ — o yüzey `/admin/sorular`. |
| `POST` | `/admin/soru-analiz/uret` | `admin_soru_analiz_uret` | `admin_routes.py:1134` | "Şimdi üret" düğmesi — gecelik iş gelene dek TEK çalıştırma yüzeyi (#715b K3). |
| `GET` | `/admin/sorular` | `admin_questions` | `admin_routes.py:715` | Soru görünürlüğü — ⚠ FOUNDER KARARI 2026-08-14: HEKİM KİMLİĞİ GÖSTERİLİR (hekim adı |
| `GET` | `/admin/{did}` | `admin_detail_page` | `admin_routes.py:1222` |  |
| `POST` | `/api/admin/{did}` | `api_admin_update` | `admin_routes.py:1631` |  |
| `POST` | `/api/admin/{did}/delete` | `api_admin_delete` | `admin_routes.py:1600` | Arşivli üye silme. Guard: kendi hesabı / admin silinemez; onay = üyenin e-postası. |
| `POST` | `/api/admin/{did}/grant` | `api_admin_grant_credit` | `admin_routes.py:1646` | Admin comp: hekimin SINIRSIZ ERİŞİM SÜRESİNİ uzatır + bildirim e-postası gönderir. |
| `POST` | `/api/admin/{did}/profile` | `api_admin_profile` | `admin_routes.py:1583` | ⚠ `is_test_sent` hidden alanı olmadan işaretlemeyi KALDIRMAK imkânsızdır: işaretsiz |

## `saglik.app.analiz_routes`  — 1 rota

*ANALİZ ROTASI — dosya kütüphanesi map-reduce → TAM KLİNİK ÖZET (#635b/#642b).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/analyze` | `analyze_case` | `analiz_routes.py:126` | Seçili (veya tüm aktif) dosyaları MAP-REDUCE ile birlikte analiz et → ziyaret olarak kaydet. |

## `saglik.app.auth_routes`  — 19 rota

*KİMLİK / OTURUM ROTALARI — kayıt, giriş, çıkış, doğrulama, parola sıfırlama, onay*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/verify/resend` | `resend_verify` | `auth_routes.py:1760` |  |
| `GET` | `/auth/google` | `google_auth_start` | `auth_routes.py:1534` |  |
| `GET` | `/auth/google/callback` | `google_auth_callback` | `auth_routes.py:1558` |  |
| `GET` | `/cikis` | `logout_get` | `auth_routes.py:1804` | Eski GET bağlantıları (yer imi, e-posta, dış link) için köprü: OTURUMU KAPATMAZ, |
| `POST` | `/cikis` | `logout` | `auth_routes.py:1790` | Çıkış — ⚠ M13: ARTIK POST. GET iken `<img src="…/cikis">` ile CSRF-logout yapılabiliyordu |
| `GET` | `/dogrula/{token}` | `verify_email` | `auth_routes.py:1699` |  |
| `GET` | `/dogrulandi` | `verify_done` | `auth_routes.py:1738` | #230b — doğrulama SONUÇ sayfası. URL'de SIR YOK ⇒ TAM analytics taşır (`izsiz` ALMAZ). |
| `GET` | `/giris` | `login_page` | `auth_routes.py:502` |  |
| `POST` | `/giris` | `login` | `auth_routes.py:533` |  |
| `GET` | `/kayit` | `register_page` | `auth_routes.py:567` |  |
| `POST` | `/kayit` | `register` | `auth_routes.py:1171` |  |
| `GET` | `/mail-tercihi/{token}` | `mail_tercihi_page` | `auth_routes.py:807` | Onay sayfasını GÖSTERİR — HİÇBİR ŞEY UYGULAMAZ (bkz. bölüm başlığı). |
| `POST` | `/mail-tercihi/{token}` | `mail_tercihi_uygula` | `auth_routes.py:829` | Tercihi UYGULAR. İdempotent; oturum/çerez istemez. |
| `GET` | `/onay` | `consent_page` | `auth_routes.py:1331` | `/onay` GET rotası — İNCE SARMALAYICI, gövde `_onay_govde`de. |
| `POST` | `/onay` | `consent_submit` | `auth_routes.py:1444` |  |
| `GET` | `/parola-sifirla` | `reset_request_page` | `auth_routes.py:603` |  |
| `POST` | `/parola-sifirla` | `reset_request` | `auth_routes.py:619` |  |
| `GET` | `/parola-sifirla/{token}` | `reset_form_page` | `auth_routes.py:641` |  |
| `POST` | `/parola-sifirla/{token}` | `reset_submit` | `auth_routes.py:664` |  |

## `saglik.app.cal_routes`  — 6 rota

*TAKVİM — randevu + klinik takip yüzeyi (Faz 9, 2026-08-20).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/api/calendar` | `api_calendar_list` | `cal_routes.py:197` |  |
| `POST` | `/api/calendar` | `api_calendar_create` | `cal_routes.py:222` |  |
| `DELETE` | `/api/calendar/{aid}` | `api_calendar_delete` | `cal_routes.py:321` |  |
| `PATCH` | `/api/calendar/{aid}` | `api_calendar_update` | `cal_routes.py:254` | Etkinliği düzenle (kısmi): {at?, kind?, title?, case_id?}. |
| `POST` | `/api/calendar/{aid}/done` | `api_calendar_done` | `cal_routes.py:301` |  |
| `GET` | `/takvim` | `takvim_page` | `cal_routes.py:178` |  |

## `saglik.app.cases_routes`  — 10 rota

*PSEUDONİM HASTA DEFTERİ + ŞİFRELİ DOSYA KÜTÜPHANESİ (Faz 7, 2026-08-01).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases` | `create_case` | `cases_routes.py:1192` |  |
| `POST` | `/api/cases/{cid}/delete` | `delete_case` | `cases_routes.py:1306` | Hasta kartını KALICI sil — hekimin KENDİ kartı (founder 03.09). |
| `GET` | `/api/cases/{cid}/dosya-durumu` | `case_file_status` | `cases_routes.py:1469` | Kartın ANALİZ DURUMU — yalnız SAYI (#705b-b). LLM yok, ücret yok, 0 yazma. |
| `POST` | `/api/cases/{cid}/files` | `upload_case_files` | `cases_routes.py:1352` | Hastaya kalıcı, ŞİFRELİ dosya yükle (LLM YOK → kota TÜKETMEZ). Dosyalar DB'de bytea. |
| `GET` | `/api/cases/{cid}/files/{fid}` | `download_case_file` | `cases_routes.py:1430` | Şifreli dosyayı çöz + orijinal mime ile inline döndür (önizleme/indirme). |
| `POST` | `/api/cases/{cid}/files/{fid}/delete` | `delete_case_file` | `cases_routes.py:1491` | Dosyayı soft-delete (hekim silme hakkı — KVKK). |
| `POST` | `/api/cases/{cid}/update` | `update_case` | `cases_routes.py:1263` |  |
| `POST` | `/api/cases/{cid}/visit` | `add_visit` | `cases_routes.py:1331` |  |
| `GET` | `/cases` | `cases_page` | `cases_routes.py:331` |  |
| `GET` | `/cases/{cid}` | `case_detail_page` | `cases_routes.py:510` |  |

## `saglik.app.chat_routes`  — 15 rota

*SOHBET + CDS API ROTALARI — /chat, /api/chat (AKIŞ), ikinci-geçiş ajanları*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/timeline` | `api_case_timeline` | `chat_routes.py:1504` | Hasta Zaman-Tüneli — kayıttan kronolojik trend özeti (Sonnet, 1 kota). |
| `POST` | `/api/chat` | `api_chat` | `chat_routes.py:724` |  |
| `POST` | `/api/council` | `api_council` | `chat_routes.py:1426` | Konsey (eski adı Doktor Paneli) — triyaj → 3 branş → moderatör. (~4 kota içeride sayılır.) |
| `POST` | `/api/deep-analyze` | `api_deep_analyze` | `chat_routes.py:1293` | Derin Analiz — max-efor Opus yanıt → critic → koşullu revize. 1 kota (critic/revize içeride, kotasız). |
| `POST` | `/api/dose-check` | `api_dose_check` | `chat_routes.py:1111` | Yanıttaki dozları ayrı dar ajanla denetle (ikinci-göz güvenlik; kotasız). |
| `POST` | `/api/drug-card` | `api_drug_card` | `chat_routes.py:1197` | İlaç kartı — tek ilaç için yapılandırılmış atıflı klinik kart. |
| `GET` | `/api/mycases` | `api_mycases` | `chat_routes.py:1575` | Composer hasta seçici için hafif liste (id, code) + ÇAĞIRAN HEKİMİN KİMLİĞİ. |
| `POST` | `/api/redflag` | `api_redflag` | `chat_routes.py:1151` | Kaçmış aciliyet örüntüsü tara (triyaj rozeti; tanı değil; kotasız). |
| `POST` | `/api/simplify` | `api_simplify` | `chat_routes.py:1385` | Hasta İçin Sadeleştir — yanıtı hasta diline çevir (Haiku, kotasız). |
| `GET` | `/api/threads` | `api_threads` | `chat_routes.py:461` |  |
| `DELETE` | `/api/threads/{tid}` | `api_thread_delete` | `chat_routes.py:498` |  |
| `GET` | `/api/threads/{tid}` | `api_thread_get` | `chat_routes.py:476` |  |
| `PATCH` | `/api/threads/{tid}` | `api_thread_rename` | `chat_routes.py:508` | Konuşmayı yeniden adlandır (sahiplik store'da zorlanır). |
| `POST` | `/api/whatmissed` | `api_whatmissed` | `chat_routes.py:1159` | 'Ne kaçırdım?' — yanıtın atladığı klinik noktaları listeler (Sonnet, 1 kota). |
| `GET` | `/chat` | `chat_page` | `chat_routes.py:285` |  |

## `saglik.app.chat_turn`  — 2 rota

*SOHBET TURU — ÜRETİM BAĞLANTIDAN AYRIK + KOPAN İSTEMCİNİN TOPARLANMASI (#748b, founder 06.09).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/chat/durdur` | `api_chat_durdur` | `chat_turn.py:224` | Durdur tuşu: hekime ait `uretiliyor` satırı `iptal` olur → thread bir sonraki yoklamada |
| `GET` | `/api/chat/turn/{turn}` | `api_chat_turn` | `chat_turn.py:248` |  |

## `saglik.app.cihaz_routes`  — 6 rota

*CİHAZ EŞLEŞTİRME — masaüstü programın Klivance kimliği (#596b, 2026-09-01).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cihaz/bekle` | `cihaz_bekle` | `cihaz_routes.py:253` | Program 2 sn'de bir yoklar. Onaylandıysa CİHAZ SIRRINI **bir kez** döner. |
| `POST` | `/api/cihaz/istek` | `cihaz_istek` | `cihaz_routes.py:188` | Program bir eşleştirme isteği açar; ekranda gösterilecek KODU döner. |
| `POST` | `/api/cihaz/oturum` | `cihaz_oturum` | `cihaz_routes.py:589` | Cihaz sırrı → kısa ömürlü `app.session` çerezi. |
| `GET` | `/cihaz` | `cihaz_sayfa` | `cihaz_routes.py:319` | Onay sayfası (`?k=` ile TEK TIK) + elle kod formu + bağlı cihaz listesi. |
| `POST` | `/cihaz` | `cihaz_onayla` | `cihaz_routes.py:520` | Hekim programdaki kodu buraya yazar; istek ONAYLANIR. |
| `POST` | `/cihaz/sil` | `cihaz_sil` | `cihaz_routes.py:564` | Bağlantıyı kes — cihaz satırı VE onun bastığı oturumlar düşer. |

## `saglik.app.deger_onay`  — 1 rota

*ANALİZDEN ÇIKAN DEĞERİ KARTA İŞLEME — ONAYLI (#362b, founder kararı (a) 2026-08-19).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/deger-onayla` | `deger_onayla` | `deger_onay.py:127` | YALNIZ onaylanan alanı yazar; kartın geri kalanı AYNEN korunur. |

## `saglik.app.enabiz_tahlil`  — 1 rota

*e-NABIZ TAHLİL PAKETİ — HTML (akordiyon) → hasta kartı "belge" → analiz/kokpit (#668b).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/enabiz-tahlil` | `api_enabiz_tahlil` | `enabiz_tahlil.py:413` | Eklentiden gelen tahlil paketi → şifreli belge + önceden dolu çıkarım (LLM YOK). |

## `saglik.app.etkilesim_yorum`  — 1 rota

*`/etkilesim` AI YORUMU — çoklu-ilaç etiket taramasının klinik yorumu (giriş + 1 kredi).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/etkilesim/yorum` | `api_etkilesim_yorum` | `etkilesim_yorum.py:680` | `/etkilesim` taramasının AI yorumu — giriş + 1 kredi (founder kararı 2026-08-18). |

## `saglik.app.hastalik`  — 3 rota

*HASTALIKLAR — `/hastaliklar` (founder 2026-08-19: "ana menüye hastalıklar butonu;*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/api/hastalik-ara` | `api_hastalik_ara` | `hastalik.py:263` | Hastalık adı araması — ücretsiz, LLM yok, kayıt gerekmez. |
| `GET` | `/api/hastalik/{did}` | `api_hastalik` | `hastalik.py:285` | Tek hastalık kartı — ücretsiz, LLM yok. |
| `GET` | `/hastaliklar` | `hastaliklar_page` | `hastalik.py:304` | Arama + kart + soru kutusu. Kart ücretsiz; soru mevcut sohbet yoluna taşınır. |

## `saglik.app.on_okuma`  — 4 rota

*GÖRÜNTÜ ÖN-OKUMA (#620b YOL 2, founder 2026-09-13) — ROTA MODÜLÜ + KART HTML ÜRETİCİLERİ (WP-6, Kamil).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/files/{fid}/belge-oku` | `api_belge_oku` | `on_okuma.py:642` |  |
| `POST` | `/api/cases/{cid}/files/{fid}/on-oku` | `api_on_oku` | `on_okuma.py:514` |  |
| `POST` | `/api/cases/{cid}/on-okuma-onayla` | `api_on_okuma_onayla` | `on_okuma.py:606` | LLM 0 · `record_usage` 0. Beyaz liste dışı `karar` → 303 (yazma yok). `onaylandi` + `onay_reddi` → 400. |
| `POST` | `/api/goruntu-onoku` | `api_goruntu_onoku` | `on_okuma.py:341` | Sohbet 'Görüntü ön-okuma' modu (WP-4 istemcisi). Ağır iş havuzda (M5). |

## `saglik.app.on_okuma_kaydet`  — 1 rota

*SOHBET GÖRSELİNİ HASTA KARTINA KAYDET (#781c, founder 2026-09-15) — TEK UÇ (Kamil).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/goruntu-kaydet` | `api_goruntu_kaydet` | `on_okuma_kaydet.py:83` | Sohbette ön-okunan görselleri hasta kartına ŞİFRELİ kaydet + ön-okumayı `pending` iliştir. |

## `saglik.app.panel_routes`  — 1 rota

*`/panel` — yargım deseninin uyarlaması, EK sayfa (2026-08-02, founder yönü).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/panel` | `panel_page` | `panel_routes.py:180` |  |

## `saglik.app.rad_case`  — 1 rota

*GÖRÜNTÜLEME (YOL 1) — RAPOR METNİNDEN LEZYON ŞERİDİ + TAKİP RANDEVUSU (IP2, 2026-09-13).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `POST` | `/api/cases/{cid}/rad-takip` | `rad_takip` | `rad_case.py:236` | YALNIZ takvime yazar. Gövde `bulgu_idx` + `ay`; başka alan yok sayılır. |

## `saglik.app.recete_case`  — 1 rota

*HASTA KARTI · REÇETE KONTROLÜ SEKMESİ — panel gövdesi + JSON ucu (#336b, 2026-08-17).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/api/cases/{cid}/recete-kontrol` | `api_recete_kontrol` | `recete_case.py:933` | Hasta kartındaki ilaç listesini dört LLM'siz kontrolden geçir. Kredi 0. |

## `saglik.app.recete_routes`  — 3 rota

*REÇETE KONTROLÜ ROTALARI — `/recete-kontrolu` (hub) + `/bobrek-doz` (böbrek doz kontrolü).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/bobrek-doz` | `bobrek_doz_page` | `recete_routes.py:301` | Etiket-temelli böbrek doz kontrolü — kayıt gerekmez, kredi harcamaz, LLM yok. |
| `GET` | `/recete-kontrolu` | `recete_hub_page` | `recete_routes.py:217` | İki ücretsiz kontrol aracının giriş kapısı. Kayıt yok, kredi yok, LLM yok. |
| `GET` | `/sgk-odeme` | `sgk_odeme_page` | `recete_routes.py:254` | Ek-4/A liste araması — hub'ın üçüncü aracı. Kayıt yok, kredi yok, LLM yok. |

## `saglik.app.ref_routes`  — 5 rota

*REFERANS + ÜCRETSİZ ARAÇ YÜZEYİ (Faz 8, 2026-08-01).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/api/query` | `api_query` | `ref_routes.py:269` |  |
| `GET` | `/api/suggest` | `api_suggest` | `ref_routes.py:321` | Referans arama autocomplete — ilaç adı önerileri (aktif segment prefiksi). SALT DB, LLM/kota YOK. |
| `GET` | `/etkilesim` | `etkilesim_page` | `ref_routes.py:133` | Etiket-temelli ilaç etkileşimi taraması — kayıt gerekmez, kredi harcamaz. |
| `GET` | `/hesaplayicilar` | `calculators_page` | `ref_routes.py:157` | Klinik hesaplayıcılar — istemci-tarafı, ücretsiz (kredi harcamaz). HERKESE AÇIK (2026-07-19): |
| `GET` | `/reference` | `reference_page` | `ref_routes.py:117` |  |

## `saglik.app.sgk_ara`  — 1 rota

*SGK BEDELİ ÖDENECEK İLAÇLAR LİSTESİNDE ARAMA (founder 2026-08-19).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/api/sgk-ara` | `api_sgk_ara` | `sgk_ara.py:202` | Ek-4/A araması. Herkese açık, kredi 0, LLM yok. |

## `saglik.app.sgk_case`  — 1 rota

*HASTA KARTI · SGK GERİ ÖDEME KONTROLÜ — AYRI DÜĞME (founder 2026-08-19).*

| Metot | Yol | Endpoint | Kaynak | Açıklama |
|---|---|---|---|---|
| `GET` | `/api/cases/{cid}/sgk-kontrol` | `api_sgk_kontrol` | `sgk_case.py:111` | Hastanın ilaç listesindeki ürünlerin Ek-4/A liste durumu. Kredi 0, LLM yok. |

