# 09 — ADMİN PANELİ VE ANALİTİK

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> Bu katmanın tek işi **görünürlük** — ve bu üründe görünürlük katmanının en tehlikeli
> hatası **yanlış pozitif hüküm**'dür ("0 satış var" ≠ "ölçüm çalışmıyor").

---

## 1. ADMİN PANELİ — `/admin*`

### 1.1 Rotalar

| Metot | Yol | Modül |
|---|---|---|
| `GET` | `/admin` | `admin_routes.admin_page` |
| `GET` | `/admin/arsiv` | `admin_routes.admin_archive_page` |
| `GET` | `/admin/odeme` | `admin_routes.admin_payment_diag` |
| `POST` | `/admin/odeme/mutabakat` | `admin_routes.admin_mutabakat` |
| `POST` | `/admin/odeme/aktivasyon` | `abonelik_routes.admin_aktivasyon` |
| `GET` | `/admin/sorular` | `admin_routes.admin_questions` |
| `GET` | `/admin/soru-analiz` | `admin_routes.admin_soru_analiz_page` |
| `POST` | `/admin/soru-analiz/uret` | `admin_routes.admin_soru_analiz_uret` |
| `POST` | `/admin/kur` · `/admin/reklam` | `admin_routes.admin_set_kur` · `admin_set_reklam` |
| `POST` | `/admin/feedback/{fid}` | `admin_routes.admin_feedback_status` |
| `GET` | `/admin/{did}` | `admin_routes.admin_detail_page` |
| `GET` | `/admin/{did}/sorular` | `admin_hekim.admin_doctor_questions_page` |
| `GET` | `/admin/{did}/hastalar` · `/hasta/{cid}` · `/eylemler` | `admin_hasta.*` |
| `POST` | `/api/admin/{did}` · `/delete` · `/grant` · `/profile` | `admin_routes.api_admin_*` |

### 1.2 ⚠⚠ `_admin()` = ROL **VE** DENETİM KAPISI (#522b)

**`_admin()`'i ATLAYAN admin rotası AÇMA.** Kapı iki iş birden yapar:
1. Rol kontrolü (`KLIVANCE_ADMIN_EMAILS`)
2. **Denetim logu** (`app.admin_erisim`, motor `admin_denetim.py`)

⚠ **Loga İÇERİK YAZILMAZ** — metin yok, `?q=` terimi yok.

#### ⚠⚠ İKİ FARKLI "FAIL" — KARIŞTIRMA

| | Davranış | Gerekçe |
|---|---|---|
| **KAPSAM** | **FAIL-CLOSED** | Beyaz liste *"loglanacaklar"* değil — `_MUAF` = **muaflar** listesi. **Yeni admin rotası VARSAYILAN LOGLANIR.** Tanınmayan rota `eylem='siniflandirilmamis'` ile yazılır: satır **kaybolmaz**, sınıflandırılmadığı **görünür** |
| **YAZMA** | **FAIL-OPEN, ama SESSİZ DEĞİL** | DB hıçkırığı founder'ı kendi admin'inden kilitlemesin. Ama *"çalıştırılamadı ≠ temiz"* → yazma başarısızsa **kayıp kaydedilir** (sunucu log'u + `app.setting` kimliksiz sayacı) ⇒ "loglanmamış erişim" sayısı **bilinir** olur, sıfır sanılmaz |

**İki fazlı yazma:** `_admin()` isteğin **başında** koşar; `hedef_kume` ve `kayit_sayisi`
ise sayfa hesaplandıktan **sonra** bilinir.

Kapılar: `admin_denetim_verify` (**SUITES**) · `api_admin_uclari_verify`.

### 1.3 ⚠⚠ ÜÇ-DURUM DİSİPLİNİ — panelin temel sözleşmesi

Veri `store._admin_safe` → `ok` / `error`. Sunum: `_adm_ozet` / `_adm_bos_r` / `_adm_n`.

| Durum | Gösterim |
|---|---|
| Veri geldi | Değer |
| Veri yok (gerçekten 0) | `0` |
| **Hata** | **Amber "Ölçülemedi"** |

⚠⚠ **ASLA yeşil, ASLA sessiz "0".** Hata → pozitif hüküm **YASAK** — *sıfırın sebebini
iddia eden cümle de dahil*.

### 1.4 Panel doğruluğu invariantları (2026-07-30 çürütücü denetimi)

| # | Invariant |
|---|---|
| 1 | **Hata → pozitif hüküm YASAK** (sıfırın sebebini iddia eden cümle dahil) |
| 2 | **YOKLUK en sessiz hâldir** — yalnız çalışan hattı doğrulayan gösterge **hiçbir şey doğrulamaz** |
| 3 | **Sayaçlar AYRIK** |
| 4 | **ROI sütunu PARAYI ölçer** (`app.iyzico_payment.granted`) |
| 5 | `last_active` **geçici kaynağa dayanamaz** |
| 6 | `/health` **sürüm damgası taşır** — **503 dalında DA** |

### 1.5 Panel kartları

5 kart + atıf oranı + **ödeme DENEMELERİ** (redler görünür — başarılar değil).
Kart kart gerekçeler, düzen/taşma iki-ölçü dersi, kapı envanteri →
`docs/kamil-ozellik-defteri.md`.

⚠ Panel **bilerek Türkçe tek-dillidir** — **admin metnini `EN_PAIRS`'e EKLEME.**

### 1.6 Modül bölmesi (tavan borcu)

`admin_routes.py` tavanı 2026-08-22'de **istisna olarak bir kez** yükseltildi (1.760 → 1.806)
ve kayda *"istisna tekrarlanabilir bir hak DEĞİL, bir sonraki artış yine BÖLME ister"*
yazıldı. Sonuç bölmeler:

| Modül | Ne |
|---|---|
| `admin_pano.py` | `/admin` kart gövdeleri (**saf gövde** — `kur(app)` YOK, `app.routes` değişmez) |
| `admin_sunum.py` | Sunum yardımcıları — panelin *"ne söyleyebilir"* disiplini (rota taşımaz) |
| `admin_queries.py` | Sorgular |
| `admin_uyeler.py` | Üye listesi + sıralama **beyaz listesi** |
| `admin_hekim.py` · `admin_hasta.py` | Hekim/hasta detay rotaları |
| `admin_alarm.py` | Aylık kullanım alarmı |
| `admin_grafik.py` · `admin_reklam.py` · `admin_soru_analiz.py` | Grafik · aylık reklam harcaması · soru analizi |
| `admin_denetim.py` | Denetim logu motoru |

⚠ **Dairesel import yönü ölçüldü:** `admin_pano`'yu **yalnız** `admin_routes` ithal eder →
modül düzeyinde ithal edilebilir. **`admin_alarm.py` aynısını YAPAMAZ** çünkü onu `store`
ithal ediyor (`store → admin_alarm → admin_sunum → webutil` zinciri kısmen başlatılmış olur)
→ orada **tembel import** gerekir.

### 1.7 `/admin/sorular` — ham gösterim (founder 2026-08-14)

⚠⚠ Soruyu **ve** yanıtı **HAM gösterir, hekim kimliğiyle**.
⚠⚠ *"Yönetici erişimi ifşa EDİLDİ"* iddiası **2026-08-30'da ÖLÇÜLEREK ÇÜRÜDÜ** → `_gorev.txt` #521b.

### 1.8 Üye yönetimi ve arşiv

| Kural | Detay |
|---|---|
| **Arşiv** | `/admin/arsiv` = **yalnız admin silmeleri** (kalıcı snapshot) |
| **KVKK talebi** | Hekimin kendi talebi (`store.purge_due_accounts`) **kopya ALMAZ** → orada görünmez, yalnız kimliksiz sayaç (`app.setting 'kvkk_purge_n'`) + **"alt sınırdır"** ibaresi |
| Başlık | **SATIR değil KİŞİ** sayar |
| ⚠ Guard | **Bekleyen KVKK talebi olan hesap ARŞİVLENEMEZ** (`api_admin_delete` → `?del=kvkk`) — m.7 talebi işlenirken kalıcı kopya almak hakkı **fiilen bertaraf eder**. Guard **silmeyi** değil **kopya almayı** engeller |
| Snapshot minimizasyonu | `password_hash` / `verify_token` / `reset_token` / `signup_ip` / `gclid(_at)` ve consents içindeki **IP ATILIR**; `email` / `billing` / `phone` / onay kaydı **bilinçli KALIR** (mali eşleşme + ispat) |

⚠⚠ `to_jsonb(d)` her şeyi **otomatik alır** → **yeni `app.doctor` kolonu eklersen arşive
gitmeli mi diye KARAR VER.**

Kapılar: `admin_panel_verify` · `admin_siralama_verify` · `hesap_yasam_dongusu_verify` ·
`retention_verify` · `admin_form_deneme_verify`.

---

## 2. SORU ANALİTİĞİ (#715b) — üç katman

Founder 2026-09-04: *"hangi branşlar neler soruyor. Bizim alameti farikamız bu."*
SPEC: `docs/soru-analitigi-spec-2026-09-04.md`.

| Katman | Modül | Sahip | Ne |
|---|---|---|---|
| **K1 — KAYIT** | `soru_kayit.py` | Cahit | Katman sonucu + **kanıt paketi künyesi** (`app.answer_layer`, `app.evidence_doc`) |
| **K2 — TÜRETME** | `soru_analiz.py` | — | `app.message` ham sorusu → `app.question_insight` |
| **K3 — PANEL** | `admin_soru_analiz.py` | Kamil | `/admin/soru-analiz` |

### ⚠⚠ Kuralları

1. **TÜRETİLMİŞ VERİ ASLA TEK KAYNAK DEĞİLDİR.** `app.question_insight` silinip
   `app.message`tan yeniden üretilebilir ve taksonomi geliştikçe **yeniden üretilir** →
   her satır `uretim_surumu` taşır, batch **idempotenttir** (aynı sürümde iki koşum aynı
   sonucu verir; kapı şartı). Bir sayı burada "kaynak" gibi okunursa **taksonomi borcunu
   dondurmuş** oluruz.

2. **`app.query_stat`'a ve `store._stat_topic`'e DOKUNULMAZ.** O tablo bilinçli **anonim**
   (`doctor_id` ve ham metin YOK) ve bir **regresyon frenidir**.
   `_stat_topic`'in kusuru — **ilk eşleşeni alıp durması**, "gebede warfarin dozu"nu tek
   kutuya düşürmesi — burada **tekrarlanmaz**: `konu` **ÇOK ETİKETLİDİR**, eşleşen **her**
   etiket yazılır. Aynı sebeple o liste **import edilmez**: paylaşılan liste, donmuş olması
   gereken freni hareket ettirirdi.

3. **KLİNİK METİN YAZILMAZ.** Bu tabloya soru/yanıt metni, hasta verisi, alıntı **girmez** —
   yalnız etiketler, sayılar, bayraklar.

Kapılar: `soru_analitigi_verify` · `soru_analiz_panel_verify` · `soru_kayit_verify` ·
`katman_bag_verify` · `soru_koruma_verify`.

---

## 3. HEKİM DAVRANIŞ İZİ (#726b) — `hekim_iz.py`

Founder 2026-09-04: *"hekimin kaç hastası var, hasta analizinde neler yaptı, hastalıklar
sordu mu — ne yaptığını detaylı görmemiz ve KAYIT TUTMAMIZ lazım."*

### Ne çözer (ölçüm: 124 rota AST taraması)

1. **`app.usage.mode` ÖZELLİK DEĞİL SORU TÜRÜDÜR.** `cds/pipeline` çoğu uçta classifier'ın
   `res["mode"]`ini yazar → `/api/chat`, `/api/whatmissed`, `/api/deep-analyze`,
   `/api/council` **hepsi aynı `clinical` satırını** üretir. `app.doctor_event`'te yüzeyi
   classifier değil **çağıran uç** yazar → ayrım **kaynağında** doğar.
2. **İzsiz yüzeyler:** etkileşim taraması, böbrek dozu, SGK, hastalıklar, hesaplayıcılar,
   referans araması, hasta kartı açma, dosya görüntüleme — hiçbiri satır yazmıyordu.

### ⚠⚠ BU BİR FREN DEĞİL, KAYIT

Hiçbir ücretsiz güvenlik ucuna kapı **eklemez**, hiçbir turu **ücretlendirmez**,
`app.usage`'a **dokunmaz**. `credits` / `store.FREE_MODES` / `lifetime_q` mantığı **değişmedi**.

### ⚠⚠ FAIL-OPEN VE ÜRÜN AKIŞINI ASLA BOZMAZ

Yazma `with conn.transaction():` **SAVEPOINT** içinde; hata log'a düşer, çağıran etkilenmez.

⚠⚠ **SAVEPOINT'İN İÇİNDE `commit()` ÇAĞIRMA** — psycopg3 yasaklar ve yazma **sessizce düşer**.
Bu depoda birebir yaşandı: `case_timeline` arşivi **her çağrıda** düşüyordu (2026-08-27).
**Commit SAVEPOINT'ten SONRA atılır.**

Kapı: `hekim_iz_verify`.

---

## 4. ANALİTİK — GA4 + Meta Pixel

### 4.1 Kurulum

| | |
|---|---|
| GA4 | `G-FKK3Z0TWZ9` (public) — property `properties/545762878` |
| Meta Pixel | `META_PIXEL_ID` env |
| Etiket | `main.py` `_ANALYTICS_TAG` + `window.klvTrack(...)` |
| Koşul | **YALNIZ production** (`KLIVANCE_COOKIE_SECURE=1`) |
| Huni | `sign_up` → `first_query`/`Lead` → `begin_checkout` → `purchase` |

### 4.2 ⚠⚠ ÜÇ KAYIT OLAYI, ÜÇ AYRI AMAÇ — yeni kayıt yüzeyi eklersen AYRIMI KORU

| Olay | Amaç | Kapsam | Nerede |
|---|---|---|---|
| **GA4 `sign_up`** | **Google Ads BİRİNCİL DÖNÜŞÜMÜ** — anlamını BOZMA | — | `/chat?welcome=1` |
| **Meta `CompleteRegistration`** | **Optimizasyon sinyali** | **DAR**: yalnız `email_verified` **VE** unvan ≠ `'Tıp Öğrencisi'` | Form kolu `/dogrula/{token}` başarı dalı · Google kolu `/chat?welcome=1` (aynı hekimde **ikisi birden ateşlenmez**) |
| **Meta `klv_kayit`** (`trackCustom`) | **Hariç tutma sinyali** | **GENİŞ**: filtresiz | — |

**KURAL: optimizasyon sinyali DAR, hariç-tutma sinyali GENİŞ.**
İkisini tek olaya koşmak, doğrulanmamış kayıtlara **reklam parası yaktırmıştı.**

⚠ `CompleteRegistration` **filtresi AYNEN KALIR.**

### 4.3 ⚠⚠ HAYALET FRENİ — `?welcome=1` TEK BAŞINA DÖNÜŞÜM KANITI DEĞİL

Giriş yapmış herkes o adresi açınca (sekme geri yükleme, geçmiş, paylaşılan link) olay
ateşleniyor ve **birincil dönüşümü şişiriyordu**.

**İki katman:**
1. Sunucuda: hesap yaşı **< 30 dk**
2. İstemcide: `localStorage` (`klv_su` / `klv_cr`)

`replaceState` URL temizliği **iki dalda da korunur**. **Üç olay da bu kapının içindedir.**

Kapılar: `signup_ghost_verify` · `kayit_piksel_tasima_verify` · `kayit_sinyal_verify`.

### 4.4 ⚠⚠ İÇ TRAFİK — `klv_notrack=1`

Çerez `_ANALYTICS_TAG`'in **TAMAMINI** nihai HTML'den soyar.
- `/admin*` 200 yanıtında **OTOMATİK yazılır**
- Elle: `/iz-kapat` ÷ `/iz-ac`

⇒ **Founder'ın kendi cihazı GA4'e HİÇ düşmez.**
⚠⚠ ***"GA4'te yok" bir şeyin olmadığının KANITI DEĞİLDİR.***

⚠ **Yeni analytics parçası `_ANALYTICS_TAG`'in İÇİNE eklenir** (dışına eklenen soyulmaz).
⚠ Inline dönüşüm çağrıları **`if(window.klvTrack)` korumalı** yazılır.

Kapılar: `notrack_verify` · `panel_ic_trafik_verify` · `meta_signal_verify` ·
`meta_signal_js_check`.

### 4.5 ⚠⚠ HASSAS SORGU PİKSELE GİTMEZ

`_ANALYTICS_PATH_ONLY`: `/reference?q=` · `/cases/*`.
Meta pikselinin **JS kolu yüklenmez**.

⚠⚠ **`<noscript>` kolu MUAFTIR** → *"hiç yüklenmez"* hükmü **kurma** (#105b).
⚠ **Yeni hassas-sorgulu yol eklersen deseni oraya ekle.** Tanım `_ANALYTICS_TAG`'in
**içindedir** = `klv_notrack` onu da soyar.

### 4.6 `purchase` olayı

⚠ **DB'den üretilir, URL'den DEĞİL:** `granted=true` + `doctor_id` + tutar **DB'den**.

`/odeme/sonuc?durum=basarili|iptal|beklemede&tur=sub` → `purchase` **yalnız
`durum=basarili`** dalında. `replaceState` query'yi sildiği için **bilinmeyen/boş durum →
`/account`'a 303** (başarı dalı **catch-all `else` DEĞİL**, `durum` guard'lı) → F5'te sahte
başarı + piksel çift-ateşleme **yok**.

⚠ **Yeni durum eklerken guard listesini güncelle.**

Kapılar: `purchase_piksel_verify` · `odeme_sonuc_verify` · `cift_gonderim_verify`.

### 4.7 Edinim kaynağı — `_acq_mw`

**Sunucu tarafında**, JS'e bağlı olmayan ilk-dokunuş kaydı (2026-07-29).
`auth_routes._acq_cookies` · `signup_source` · `gclid`.
Kapılar: `acq_server_verify` · `gclid_verify` · `aw_tag_verify` · `kisa_link_verify`.

### 4.8 Meta özel dönüşümleri

Kimlikler, `event_name` API tuzağı ve **URL-kural yasağı** → `.claude/agents/hasan.md`
"META DÖNÜŞÜM ALTYAPISI".

### 4.9 GA4 erişimi

Çalışıyor (yetki 2026-07-29'da verildi). Property: `properties/545762878`.
⚠ **4 otomasyon yolu çürütüldü + 2 ölçüm tuzağı** (`countryId`, `fbclid`) →
memory `klivance-ga4-erisim`. Script: `scripts/ga4_report.py`.

---

## 5. SAĞLIK KONTROLÜ — `/health`

```
GET /health   →  UCUZ bir DB dokunuşu (SELECT 1) + SÜRÜM DAMGASI
```

⚠ **Landing render ETMEZ.** Neden: `/` tüm landing'i render edip DB'ye bağlanıyordu;
DB yavaşladığında sağlık probu zaman aşımına uğrar → Render instance'ı yeniden başlatır →
açılışta migrate koşar → yük artar → yine timeout (**kaskad**).

⚠ **Sürüm damgası 503 dalında DA taşınır** (panel invarianti #6).
⚠ `render.yaml` `healthCheckPath: /health` — rota olmadan blueprint uygulanırsa Render
servisi **sürekli yeniden başlatır**.

`/health` ayrıca `credits.butce_mod_bilgi()` görünürlüğünü taşır (#651b).

---

## 6. ADMİN/ANALİTİK DEĞİŞTİRİRKEN KOŞULACAK KAPILAR

| Alan | Kapı |
|---|---|
| Admin paneli | `admin_panel_verify` · `admin_pano_verify` (SKIP) · `admin_pano_store_verify` (SKIP) · `admin_siralama_verify` · `admin_mobil_bar_verify` · `admin_form_deneme_verify` |
| Denetim logu | `admin_denetim_verify` · `api_admin_uclari_verify` |
| Sorular | `admin_sorular_verify` (SKIP) · `soru_koruma_verify` |
| Soru analitiği | `soru_analitigi_verify` · `soru_analiz_panel_verify` · `soru_kayit_verify` · `katman_bag_verify` |
| Hekim izi | `hekim_iz_verify` |
| Analytics | `meta_signal_verify` · `meta_signal_js_check` · `notrack_verify` · `signup_ghost_verify` · `kayit_piksel_tasima_verify` · `purchase_piksel_verify` · `gclid_verify` · `aw_tag_verify` · `acq_server_verify` |
| Panel/alarm | `panel_verify` · `panel_ay_verify` · `aylik_alarm_verify` · `butce_pano_verify` · `aklama_pano_verify` |

---

**Sonraki:** [`10-GUVENLIK-KVKK-UYUM.md`](10-GUVENLIK-KVKK-UYUM.md)
