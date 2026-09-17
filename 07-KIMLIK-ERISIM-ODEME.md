# 07 — KİMLİK, ERİŞİM VE ÖDEME

> **Para yolu.** Buradaki her değişiklik `firat` (kontrolcü) denetimi gerektirir ve
> founder onayına tabidir. Ölçüm anı: 2026-09-17 · commit `cb16758d`.

---

## 1. KİMLİK — `saglik/app/auth.py`

### 1.1 Model

| Bileşen | Tercih | Gerekçe |
|---|---|---|
| Parola | **Argon2id** (`argon2-cffi`) | |
| Oturum | **Sunucu-taraflı** (`app.session`), çerez `klv_session` | Çerez **yalnız rastgele token** taşır; kimlik DB'de. İptal edilebilir (logout = satır sil), süreli (`expires_at`). **JWT tarayıcıda tutulmaz.** |
| Token entropisi | 256 bit (`_new_token()`) | Kaba kuvvet imkânsız → salt/pepper gerekmez |

### 1.2 ⚠ TOKEN'LAR DB'DE ÖZETLİ (2026-07-29, denetim S4)

Oturum / e-posta doğrulama / parola sıfırlama token'ları DB'ye **SHA-256 özeti** olarak
yazılır; ham değer **yalnız kullanıcıya** gider (çerez / e-posta linki).

> **Neden:** SQL enjeksiyonu, yedek sızıntısı ya da salt-okuma DB erişimi eskiden
> **doğrudan oturum devralma** veriyordu (token = çerez). Özet tek yönlü olduğu için sızan
> satır artık kullanılamaz.

**Geçiş:** eski ham token'lar okunmaya devam eder ve **ilk kullanımda sessizce özete
yükseltilir** → kimse çıkış yapmak zorunda kalmaz. `_tok_lookup()` `[özet, ham]` döner.

### 1.3 İki doğum yolu — BAĞLAYICI

| Yol | Fonksiyon |
|---|---|
| Form kaydı | `auth.create_doctor(conn, email, pw, full_name, …)` |
| Google OAuth | `auth.upsert_google_doctor(conn, email, full_name)` |

⚠⚠ **Yeni kayıt yolu eklersen e-posta Python'da küçültülmeden `app.doctor`'a GİRMESİN.**
Kanonik e-posta = **Python `.strip().lower()`**; HMAC'i **asla SQL'de üretme** (Türkçe İ:
Python ile Postgres `lower()` **farklı hash** üretir).

### 1.4 Giriş güvenliği

| Önlem | Yer |
|---|---|
| **Sabit-zamanlı** kimlik doğrulama — kullanıcı yoksa da bir `verify` koşar | `authenticate()` |
| Giriş deneme freni | `auth_routes.LOGIN_MAX` / `LOGIN_WINDOW` / `_login_blocked` |
| Kayıt freni (saatlik, IP) | `auth_routes.SIGNUP_MAX_PER_HOUR` |
| Tek kullanımlık e-posta engeli | `auth_routes._DISPOSABLE_EMAIL_DOMAINS` |
| Parola gücü | `auth.password_problem(pw, email=, full_name=)` — kapı `parola_gucu_verify` |
| Süresi dolmuş oturum temizliği | `purge_expired_sessions()` (fırsatçı, girişte) |

⚠ Giriş **kalıcı kaydedilir** (`app.doctor.last_login_at`, **SAVEPOINT içinde**).
**Aktiflik gösteren yeni bir yüzey eklersen onu da hesaba kat.**
Kapılar: `giris_kalici_verify` · `giris_gecmisi_verify` · `api_giris_freni_verify`.

### 1.5 PAROLA SIFIRLAMA — `/parola-sifirla[/{token}]`

| Kural | Neden |
|---|---|
| POST **her koşulda AYNI yanıtı** döner | Hesap numaralandırma freni |
| Formu **göstermek** token'ı **tüketmez** (`reset_token_valid`) | |
| **Başarısız deneme** token'ı **yakmaz** | |
| Başarıda **TÜM oturumlar düşer**, otomatik giriş **YAPILMAZ** | |
| Token tüketimi **atomik** (`consume_reset_token`) | Tek kullanımlık |

Kapı: `scratchpad/parola_sifirlama_verify.py`.

### 1.6 Google OAuth (CANLI, 2026-07-15 doğrulandı)

- Rotalar: `/auth/google` → `/auth/google/callback`
- Google Cloud projesi **"Klivance"** (`alfa4n`'den **ayrı** — alfa4n'in OAuth marka adı
  "Alfa4n Mail Sender" idi, tıp ürününde güven kırıcı)
- Consent: marka "Klivance", External, **In production** (email/profile/openid hassas
  olmayan → Google doğrulaması gerekmez)
- Redirect: `https://klivance.com/auth/google/callback`
- Env yoksa **"Google ile devam et" butonu gizlenir**

### 1.7 KAYIT + PROFİL

**`/kayit` zorunlu alanlar:** ad-soyad · e-posta · parola · telefon + ülke kodu ·
doğum tarihi · unvan · branş · ülke + şehir · **2 hukuki onay** → `consents` jsonb (ts + IP, ispat).

| Karar | Durum |
|---|---|
| Diploma no | **İSTENMEZ** |
| E-bülten kutusu | **BİLİNÇLİ YOK** (İYS) |
| 3. onay kutusu (hasta-kimliği-girmeme) | **KALDIRILDI** 2026-08-05 → iki onay |
| Doğum tarihi, ülke/şehir `_profil_eksik`te mi? | **HAYIR — bilerek eklenmedi.** Kapsam yalnız yeni kayıtlar; mevcut hesapta boş olması veri kaybı değil, kapsamın kendisi |
| Unvan/branş | **TR-KANONİK** — etiketine `EN_PAIRS` çifti **EKLEME** |
| `?welcome=1` | **KORUNMALI** — GA4 `sign_up` + Meta `klv_kayit` orada ateşlenir |
| Fatura profili | Kayıtta **DEĞİL**, **ilk ödemede** (`/fatura-bilgileri` → `doctor.billing`) |

**Profil tamamlama kapısı:** `main.profil_kapisi_mw` — girişli ama profili eksik hekimi
`/onay`'a yollar. ⚠⚠ Altı maddelik sözleşme **TEK YERDE** `webutil._profil_eksik(d)`,
ortağı `/onay`ın topladığı küme. Bozmadan önce `.claude/agents/kamil.md` oku.
Kapılar: `kayit_zorunlu_verify` · `kayit_masaustu_verify` · `form_deger_koruma_verify`.

### 1.8 CİHAZ EŞLEŞTİRME (#596b)

Masaüstü programın Klivance kimliği: `POST /api/cihaz/istek` → `POST /api/cihaz/bekle` →
`GET /cihaz`. Tablo `app.cihaz_eslesme`. `auth.revoke_devices(conn, doctor_id)` tümünü düşürür.
Kapılar: `cihaz_eslesme_verify` · `cihaz_enjekte_js_verify` (JS).

---

## 2. ERİŞİM MODELİ — `saglik/app/credits.py`

⚠⚠ **DOSYA ADI TARİHSELDİR — KREDİ YOKTUR.** 2026-08-21 founder kararıyla kredi sistemi
kaldırıldı; ad korundu çünkü **30 dosya import ediyor**.

### 2.1 Model

```
Aktif abonelik          → SINIRSIZ
Yeni kayıt              → DENEME_GUN gün deneme, günde AGIR_GUNLUK_TAVAN_TRIAL soru
İkisi de yok            → DUVAR (402)
```

⚠ **"Sınırsız deneme" dili YASAK** (founder 2026-09-02).

**Tek karar kaynağı:** `store.erisim_durumu`.

### 2.2 ⚠⚠ ÜÇLÜ ZİNCİR — ücretli her ucun başında, SIRASI SABİT

```
_erisim_402 (HAK, fail-closed 402)
    → _butce_fren (PARA, fail-closed 429)
        → _gunluk_fren_429 (HIZ, fail-open 429)
```

| # | Kapı | Modül | Davranış | Kod |
|---|---|---|---|---|
| 1 | `_erisim_402` | `webutil` | **FAIL-CLOSED** | 402 |
| 2 | `_butce_fren` | `butce_kapisi` | Dönem bütçesi; `gozlem` modunda **sayar, bloklamaz** | 429 |
| 3 | `_gunluk_fren_429` | `webutil` | **FAIL-OPEN** | **429, 402 DEĞİL** |

### 2.3 Sabitler — **KODDAN OKU, buraya yazma**

| Sabit | Anlam |
|---|---|
| `DENEME_GUN` | Deneme süresi (gün) |
| `DENEME_TEKRAR_GUN` | Tekrar kayıtta verilen gün |
| `AGIR_GUNLUK_TAVAN` / `_TRIAL` | Günlük soru tavanı (abone / deneme) |
| `AGIR_MODLAR` | Tavana sayılan modlar |
| `MAX_ANALYZE_FILES` · `MAX_ATTACH_COUNT` · `MAX_ONOKUMA_GORSEL` · `MAX_ONOKUMA_B` | Dosya limitleri |
| `SATILABILIR_PLANLAR` · `LEGACY_PLANLAR` | Plan kümeleri |
| `BUTCE_TL` · `BUTCE_KISMI_ESIK` · `FREN_YUZEY` · `REZERV_USD` · `PAHALI_UCLAR` | Bütçe kapısı |
| `KUR_BAYAT_GUN` · `KUR_BAYAT_CARPAN` · `KUR_YOK_CARPAN` · `KUR_YOK_TABAN` | Döviz kuru temkini |
| `AYLIK_ALARM_ESIGI` · `AYLIK_ALARM_GUN` | Admin kullanım alarmı |

### 2.4 ⚠⚠ "SINIRSIZ" İLANI + GİZLİ TAVAN = ADİL KULLANIM İBARESİ ZORUNLU

Yanıltıcı ticari beyan riski. **İki yerde birden** olmalı:
1. Fiyat/landing'de **görünür ibare** → `credits.adil_kullanim_ibaresi(lang)`
2. Yasal metinde **madde**

**İbareyi kaldıran, tavanı da kaldırmalıdır.** Kapı: `sinirsiz_erisim_verify`.

### 2.5 Tekrar-kayıt freni

Deneme **e-posta başına bir kez** (`app.doctor.deneme_gun`, tekrar kayıtta 0).
`auth.trial_quota_for(conn, email)` karar verir; HMAC `store.returned_doctor_ids`.

⚠ **Pencereyi kodda SABİT yazmak freni SESSİZCE kırar.**
⚠⚠ **HMAC TUZU (`app.setting 'trial_salt'`) TEK NOKTA ARIZASIDIR** — kaybolursa deneme freni
düşer (herkes yeniden deneme alır) **ve** panel "hepsi ilk kez geldi" der.
**Tuzu yedeklemeden DB restore etme.**

Kapılar: `deneme_fren_kapisi` · `deneme_bir_kez_verify`.

### 2.6 Ölü kolonlar — birine bakan satır BAYATTIR

Şemada duruyor ama **okunmuyor**: `monthly_quota` · `topup_balance` · `gift_balance` ·
`usage.credits` · `quota_reservation` · `topup_grant`.

⚠ `store.record_usage` **KALDI** ve **iptalde de yazılır** — `app.usage` hem maliyetin hem
günlük frenin **TEK kaynağıdır**; yazılmazsa kaçak **ölçülemez**.
⚠ Admin hediyesi artık **GÜN** (`store.hediye_gun_ver`).

### 2.7 ⚠ "Her abone kârlı" DENMEZ

Tek fren günlük tavandır, o da **seçildi, ölçülmedi** (#396b-A, açık P1).
Abone başına maliyet ölçümü **yoktur** (#396b-B) → admin birim ekonomi kartı marj yerine
**"ölçülmedi"** basar. Bu **dürüstlüktür**, eksik değil.

---

## 3. DÖNEM BÜTÇE KAPISI — `saglik/app/butce_kapisi.py` + `butce_defteri.py`

Founder 2026-09-02: *"zarar etmeyelim."*
Tasarım: `docs/maliyet-tavani-mimari-2026-09-02.md` (§3 akış · §5 sözleşme · §6 metinler ·
**§14 R1–R10 bağlayıcı**).

### Sözleşme

- **Ölçüt** = hekimin dönem içi `sum(app.usage.usd)` (tüm modlar + uçuş-içi rezervler)
  \+ bu ucun **rezerv üst sınırı** ≥ `credits.BUTCE_TL[durum] / kur_eff`.
- ⚠ **Tur SAYMAZ** — tur maliyeti 0,002–3,5 USD arası oynar.

### İki mod — `credits.butce_modu()`, env `KLIVANCE_BUTCE_MOD`

| Mod | Davranış |
|---|---|
| **`gozlem`** (varsayılan) | Hesaplar, `app.wall_stat`'e `butce_gozlem_*` yazar, **HİÇBİR koşulda blok döndürmez (istisna dahil, R8)** |
| **`uygula`** | 429 döndürür |

Aynı env **geri dönüş anahtarıdır**.

⚠⚠ **`uygula` madde yokken KODDA REDDEDİLİR.** `credits.butce_ifsa_var()` yasal metinde
(TR **ve** EN) "dönem toplam kullanım hacmi" maddesini arar; yoksa mod `gozlem`de kalır ve
`_butce_red_logla` yazar. **İfşa ile kural atomiktir.**

### Kademeler

| Kademe | Eşik | Kapsam |
|---|---|---|
| `dolu` | ≥ %100 | Her yüzey |
| `kismi` | ≥ %80 | Yalnız `PAHALI_UCLAR`, yalnız abone |
| `ucusta` | eşzamanlı ≤ 1 | `PAHALI_UCLAR` (R2 — yarış penceresi: `record_usage` tur SONUNDA yazılır) |

### Fail-closed disiplini

Yalnız `uygula`da: defter/kur okunamazsa **429 `butce:"olculemedi"`**.
⚠ Metin **"bütçe doldu" DEMEZ** → **"doğrulanamadı"**. Admin `butce_olculemedi>0` kırmızı görür.

⚠ DB okumaları **`with conn.transaction():`** (SAVEPOINT) içinde — okuma patlarsa `conn`
**zehirlenmez** (R5; havuz autocommit değil, zehirli conn sonraki her uçta 503 / kayıp
`record_usage` üretirdi).

Kapılar: `butce_kapisi_verify` · `butce_freni_verify` · `butce_pano_verify` · `duvar_sayac_verify`.

---

## 4. FİYAT GÖSTERİMİ — `saglik/app/fiyat.py`

**Tutar VE para biriminin TEK KAYNAĞI.** İki ölçülmüş kusurdan doğdu:

| # | Kusur |
|---|---|
| **#137b** | Senkron **AYRAÇ-DUYARLIYDI**: fiyat TR'de `1.450`, EN'de `1,450` yazıldığı için EN'de hiçbir anahtar eşleşmiyordu → env fiyatı değişince **TR düzelir, EN eski fiyatı ilan etmeye devam ederdi** (kalan eski fiyat TR 0 / **EN 8**). `/kilavuz` senkronun **hiç içinde değildi.** |
| **#139b** | **Tutar senkronluydu, para birimi değildi**: `lang=en` sayfası her koşulda "TL" basıyordu. |

### Çözüm — string eşleşmesi YOK

Gövdeler (`landing.py`, `guide.py`) fiyatı **yer tutucu** olarak taşır
(`{{klv.fiyat.<ad>}}`); render anında `fiyat.doldur()` doldurur. Binlik ayracı, sembol,
birim adı **gövdede değil burada**.

⚠ Doldurma **KOŞULSUZDUR** (eski `_apply_try_pricing` yalnız `provider=iyzico`'da koşuyordu).
Doldurulmayan yer tutucu hekime ham metin olarak görünür → kapı **"0 kalıntı"** ölçer.

### ⚠⚠ Para birimi dile bağlıdır (founder 2026-08-09: *"Türkçe hariç herkes dolar görsün"*)

Bu kararın riski **ilan ≠ tahsilat**'tır (bugün herkes iyzico'dan **TL** ödüyor). Risk ortadan
kaldırılmadı, **ifşayla yönetiliyor**: USD gösterimi `fiyat._ifsa` satırından **ayrılamaz** ve
ifşanın koşulu bayrağa değil **gerçeğe** bağlıdır.

⚠⚠ **FİYAT SAYISI HİÇBİR DOKÜMANA YAZILMAZ** — kaynak `iyzico._PLAN_PRICE_DEFAULT`.
⚠⚠ **Render'da kalmış bayat `IYZICO_PRICE_*` env'i yeni fiyatı SESSİZCE EZER** → fiyat
şaşırdığında **önce oraya bak**. (`IYZICO_PRICE_S50/S100/S150` ve `_TOPUP_PRICE_DEFAULT`
2026-08-21'de **öldü**.)

Kapılar: `fiyat_yuzey_verify` · `reklam_fiyat_verify` · `plan_sozluk_verify` · `en_sayi_bicim_verify`.

---

## 5. ÖDEME — iyzico (AKTİF SAĞLAYICI)

### 5.1 Akış — Checkout Form (hosted, 3DS dahil)

```
1) create_checkout(conn, doctor, kind=, plan=, callback_url=, ip=)
      → iyzico CheckoutFormInitialize → paymentPageUrl
      → app.iyzico_payment'e 'pending' satır (token PK)
2) hekim hosted formda öder (kart + 3DS)
      → iyzico tarayıcıyı callbackUrl'e `token` ile POST'lar
3) verify(token) → CheckoutForm.retrieve
      → paymentStatus / fraudStatus SUNUCUDA doğrulanır
      → paidPrice beklenen tutarla karşılaştırılır (KURCALAMA FRENİ)
4) account_routes.iyzico_callback → hak-edişi IDEMPOTENT verir (granted bayrağı atomik)
5) → /odeme/sonuc?durum=basarili|iptal|beklemede&tur=sub
```

⚠⚠ **CALLBACK TOKEN'INA GÜVENME** — durum daima `verify()` ile sunucuda doğrulanır.

### 5.2 Model

**Her dönem için TEK ÇEKİM** (ön-ödemeli). Hekim 1 ay / 1 yıl öder →
`current_period_end = now + interval`.

⚠ **Otomatik yenileme (#774c) KODDA VAR, BAYRAK KAPALI** (`credits.iyz_recurring()`,
env `KLIVANCE_IYZ_RECURRING`). Kapalıyken **tek çekim + eski beyan**.
Açma kararı **founder'ındır**, üç şart: `docs/iyzico-abonelik-acilis-2026-09-12.md`.
Kapı: `iyzico_abonelik_verify`.

### 5.3 Env

| Env | Not |
|---|---|
| `KLIVANCE_PAY_PROVIDER` | Varsayılan `iyzico` (`config.payment_provider()`) |
| `IYZICO_API_KEY` · `IYZICO_SECRET_KEY` | **ŞART** — yoksa `/api/checkout` `IyzicoMissing`, gösterim TL kalır |
| `IYZICO_BASE_URL` | ⚠⚠ **ŞEMASIZ HOST** — `api.iyzipay.com` / `sandbox-api.iyzipay.com`. `https://` öneki → SDK **InvalidURL** |
| `KLIVANCE_IYZ_PAYGROUP` | `_sub_paygroup()` |
| `KLIVANCE_IYZ_RECURRING` | Abonelik bayrağı |

⚠ `config.startup_warnings()`: `IYZICO_BASE_URL` sandbox iken site canlı adresteyse
**açılışta bağırır** (gerçek tahsilat olmaz).

### 5.4 MUTABAKAT — `reconcile_pending`

Hak-edişi verilmemiş son ödemeleri iyzico'dan çekip kapatır.

⚠ **AÇIK VE CANLI KUSUR:** `reconcile_pending` hak-edişi verir ama düzeltmeden önce
**makbuz göndermiyordu** → fraud incelemesinde bekleyip sonra mutabakatla açılan ödemede
hekim plan alır, **onay e-postası almaz**. Düzeltme yeri: `settle_payment` sonrası dal
(`_mutabakat_makbuzu` eklendi — doğrula).

### 5.5 Dürüstlük kuralları

⚠ **verify-exception ve paid-mismatch dallarında "çekim yapılmadı" İDDİA EDİLMEZ** — bilmiyoruz.
⚠⚠ **FİKTİF İŞLEM YASAĞI:** çerçeve sözleşme founder'ın kendi kartıyla kendine satışını
yasaklar (tespitinde **tek taraflı durdurma**) → canlı doğrulama **gerçek müşteriyle** ya da
**iade edilecek tek işlemle** yapılır.

### 5.6 ⚠ AÇIK İŞ — uçtan uca DOĞRULANMADI

Dört maddelik çek listesi: `_gorev.txt` #768c.
**Tek yetkili kaynak `/admin/odeme` (DB)** — **GA4 kanıt DEĞİLDİR** (founder cihazında piksel
soyulur, bkz. `_notrack_mw`).

Teşhis altyapısı: `fail_info` · `/admin/odeme` · `webutil._IYZ_KOD` · prob → `.claude/agents/selim.md`.

Kapılar: `callback_verify` · `odeme_sonuc_verify` · `purchase_piksel_verify` ·
`iptal_maliyet_verify` · `sekil_fatura_verify`.

---

## 6. ÖDEME — Stripe (ABD kolu, PASİF)

`saglik/app/billing.py` — Checkout + Webhook + Müşteri Portalı.

| | |
|---|---|
| Durum | **Pasif** — `KLIVANCE_PAY_PROVIDER=iyzico` |
| Top-up yolu | **2026-08-21'de KALDIRILDI** (`create_topup_checkout` / `sync_topup_session` / `topup_price_usd` silindi) |
| Geç `mode=payment` oturumu | Hak-ediş **verilmez** ama **sessizce yutulmaz** → log'a düşer, founder `/admin/odeme`'den elle çözer |

### ⚠⚠ Pazar yönlendirmesi — `config.pay_routing()` VARSAYILAN KAPALI, bu bir ZORUNLULUK

Açıldığı an yurt dışı hekim Stripe'a yönlenir. Dört engel, **hepsi kod dışı**:

| # | Engel |
|---|---|
| (a) | Stripe **canlı anahtarları yok** (şu an test) |
| (b) | **Payout banka hesabı bağlı değil** |
| (c) | **Yasal metin hâlâ iyzico diyor** — ödeme kuruluşu, hatalı/mükerrer ödeme ve **iade yolu** ("iadeler iyzico aracılığıyla ödemenin yapıldığı karta") iyzico'ya bağlı |
| (d) | **Stripe yolunda makbuz e-postası: durum "ÖLÇÜLEMEDİ", "temiz" DEĞİL.** Ölçülen (AST, async dâhil): `_sub_receipt_mail`/`_topup_receipt_mail` **yalnız** `iyzico_callback`ten çağrılıyor; `billing.py`'de makbuz/`send_email` **0 eşleşme** → **biz** Stripe satın alımında makbuz **göndermiyoruz**. Ölçülemeyen: Stripe'ın kendi e-postası açık mı — **Dashboard hesap ayarıdır**, koddan görünmez |

⚠ **"Stripe zaten gönderiyordur" VARSAYMA** — yanlışsa yurt dışı hekim ödeme yapıp **hiçbir
onay almaz.** Bayrağı açmadan önce panelden **doğrula**.
⚠ Bayrak açılmadan `scratchpad/pazar_yonlendirme_verify.py` yeşil olmalı.

⚠ `=stripe`'a dönmeden **landing fiyatları USD'ye yeniden yazılmalı** (ABD fiyatı ayrı karar).

---

## 7. FATURA VE VERGİ

| Konu | Durum |
|---|---|
| Fiyat | **KDV dahil** |
| Fatura | **E-fatura VEYA e-arşiv** (hekimin vergi durumuna göre), Alfa 4N'den (TR yurt içi, %20 KDV) |
| Kurucu plan | **Ömür boyu ORAN taahhüdü** (legacy; sabit-TL **değil**) |
| Yıllık | Aylık × 10 → **2 ay bedava — TEK avantajı budur.** Kredi bonusu ve top-up paketi **YOK** |
| Fatura profili | `store.set_billing` → `app.doctor.billing` jsonb |
| **TCKN/VKN** | ⚠ **DÜZ METİN saklanır** (founder kararı; at-rest Fernet **önerildi, ERTELENDİ**) |
| Kart bilgisi | Checkout'ta **TOPLANMAZ** (hosted form) |
| Veri sorumlusu | **ALFA 4N SAN. TİC. LTD. ŞTİ.** — VKN 0510246142, Biga/Çanakkale, KEP alfa4n@hs01.kep.tr |

### ABD vergi — kurulduğu AN yürürlükte (kaçırılırsa pahalı)

| Yükümlülük | Tarih | Ceza |
|---|---|---|
| **Form 5472 + pro-forma 1120** | **15 Nisan** (7004 ile 15 Ekim) | **Form başına $25.000** — yabancı sahipli tek-üyeli LLC'de **gelir SIFIR olsa bile ZORUNLU**, ikisi **birlikte** gider |
| **Delaware franchise $300** | **1 Haziran** | Geç: $200 + aylık %1,5 + good standing kaybı (ilk ödeme kuruluştan SONRAKİ yıl) |

Ayrıntı: `docs/abd-vergi-fatura-2026-07-16.md` · memory `klivance-abd-vergi-fatura`.

---

## 8. E-POSTA — `saglik/app/mailer.py`

**Resend** (SMTP, gönderen `info@klivance.com`). SMTP yapılandırılmamışsa gönderim
**sessizce atlanır** ve link log'a yazılır.

### ⚠⚠ FOUNDER KURALI 1 (2026-08-03)
Her **yeni mail tasarımı/metni** önce **founder'a test gönderimiyle** gider; onaylanmadan
**başka hiçbir alıcıya gönderilmez** (onay tasarım başına; mevcut işlemsel mailler
[doğrulama/makbuz/parola] dışında).

### ⚠⚠ FOUNDER KURALI 2 — künye
Bilgilendirme/aktivasyon maillerine **tam Alfa 4N künyesi EKLENMEZ**, ama **tek satır
kimlik** (ticaret unvanı + MERSİS + info@klivance.com) **zorunludur**
(Ticari İleti Yön. **m.8/2**). Künye `mailer._MERSIS`ten üretilir; **boşken betikler
`--apply`ı REDDEDER.**

### ⚠⚠ "Fiyat/plan geçmesin" ARTIK HUKUKİ SINIR, editoryal değil
Bu mailler İYS'siz gidebiliyor çünkü **m.6/1** istisnasındalar ("temin edilen hizmetin
**KULLANIMINA** yönelik"). Maile **plan/indirim/fiyat/satın-alma CTA'sı** giren gün istisna
**düşer** → onay + İYS şart (toplu gönderimde ceza ×10).

⚠ **m.6/2'ye yaslanma** ("hizmet özendirilemez" der; aktivasyon maili tam da özendirir).
⚠ **Ret hakkı:** `List-Unsubscribe` başlığı **tek başına yetmez** — gövdede **görünür iptal
linki** zorunlu (**m.9/4**). İşlemsel mailler ret kapsamı **dışında** (**m.9/5**).
⚠ **Makbuzdan satıcı kimliğini Selim teyidi olmadan KALDIRMA.**

### Makbuz
`/iyzico/callback` **grant SONRASI**, **DB transaction'ının DIŞINDA**, best-effort →
**mail hatası ödemeyi/redirect'i ASLA bozmaz.** Saf işlemsel (pazarlama YOK).

Kapılar: `mail_gonderim_kapisi` · `kunye_siniri_verify` · `kunye_rota_verify` ·
`yenileme_hatirlatma_verify` · `telafi_comp_verify`.
Teslim edilebilirlik: `docs/mail-teslim-edilebilirlik.md`.

---

## 9. Dokunurken koşulacak kapılar (özet)

| Alan | Kapı |
|---|---|
| Kimlik/oturum | `parola_sifirlama_verify` · `parola_gucu_verify` · `giris_kalici_verify` · `api_giris_freni_verify` · `dogrulama_donusu_verify` |
| Kayıt | `kayit_zorunlu_verify` · `kayit_masaustu_verify` · `kayit_kaynak_verify` · `kayit_sinyal_verify` · `signup_ghost_verify` |
| Erişim | `sinirsiz_erisim_verify` · `deneme_fren_kapisi` · `deneme_bir_kez_verify` · `guvenlik_tavan_verify` (SKIP) |
| Bütçe | `butce_kapisi_verify` · `butce_freni_verify` · `butce_pano_verify` · `duvar_sayac_verify` |
| Fiyat | `fiyat_yuzey_verify` · `reklam_fiyat_verify` · `plan_sozluk_verify` |
| Ödeme | `callback_verify` · `odeme_sonuc_verify` · `purchase_piksel_verify` · `iyzico_abonelik_verify` · `pazar_yonlendirme_verify` (SKIP) |
| Mail | `mail_gonderim_kapisi` · `kunye_siniri_verify` · `kunye_rota_verify` |
| Yasal tutarlılık | `_selim_yasal_render` · `retention_verify` · `hesap_yasam_dongusu_verify` |

---

**Sonraki:** [`08-ARAYUZ-I18N-TASARIM.md`](08-ARAYUZ-I18N-TASARIM.md)
