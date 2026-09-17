# WEB UYGULAMA KATMANI — fonksiyon referansı


> **Üretilmiş dosya — elle düzenleme.** Kaynak: Python `ast` ile kaynak ağacın tamamı.
> Ölçüm anı: 2026-09-17 · commit `cb16758d`. Yeniden üretmek için export kökündeki
> `veri/` JSON'larını besleyen çıkarıcıyı koştur (bkz. `18-DEVIR-NOTLARI.md` §6).

**82 modül · 1039 fonksiyon/metot.** Kapsam: WEB UYGULAMA KATMANI (`saglik/app/`)

Okuma anahtarı: `_` ile başlayan ad = modül-içi (dışarıdan çağırma); `async def`
işaretlidir. **Modül başlığındaki `⚠⚠` satırları sözleşmedir** — bu referans yalnız
imzayı ve ilk satırı taşır, bir fonksiyona dokunmadan önce dosyanın kendi başlığını oku.

---

## `saglik/app/__init__.py`

`7 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Klivance SaaS uygulama katmanı (FastAPI) — bilgi tabanı + CDS motorunun üstünde.

auth   → doktor hesabı, Argon2 parola, sunucu-taraflı oturum
store  → sohbet/vaka/kullanım kalıcılığı + kota
main   → FastAPI rotaları (giriş, sohbet, referans, vaka, hesap)
```

</details>

## `saglik/app/abonelik_body.py`

`49 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
iyzico ABONELİK (recurring) sayfa gövdeleri — SAF şablon (#774c C2).

⚠ Buraya YALNIZ saf sabit konur: f-string yok, modül durumu yok, import yok (Kamil'in
`cases_body.py` kalıbı). Dinamik parça `abonelik_routes.py`de `.replace("{{X}}", …)` ile girer.

⚠ iyzico abonelik formu `checkoutFormContent` = iyzico'nun ürettiği <script>/<div> parçası
(hosted `paymentPageUrl` YOK — belge, B-774-5). Parça `{{FORM}}` yerine OLDUĞU GİBİ basılır
(escape EDİLMEZ: iyzico'nun script'i). `{{FORM}}` dışındaki her dinamik metin `esc()` ile gelir.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `FORM_CSS` | `'<style>\n.ab-wrap{max-width:560px;margin:0 auto;padding:40px 22px 56px}\n.ab-title{font-family:var(--font-head);font-si` | 12 |
| `FORM_BODY` | `'<div class="ab-wrap">\n<h1 class="ab-title">{{TITLE}}</h1>\n<p class="ab-sub">{{SUB}}</p>\n<div class="ab-box">{{ROWS}}` | 26 |
| `YONETIM` | `'<p class="muted" style="margin-top:10px;font-size:12.5px">{{MSG}}</p>{{BTN}}'` | 38 |
| `IPTAL_FORM` | `'<form method="post" action="/api/abonelik/iptal" style="margin-top:12px" onsubmit="return confirm(\'{{CONFIRM}}\')"><bu` | 40 |
| `KART_FORM` | `'<form method="post" action="/api/abonelik/kart" style="margin-top:12px"><button type="submit" class="btn btn-primary">{` | 44 |
| `DONUSUM_KART` | `'<div class="card" id="donusum" style="margin-top:18px;border-left:4px solid var(--verify)"><h2>{{TITLE}}</h2><p class="` | 47 |

## `saglik/app/abonelik_dili.py`

`126 satır` · `4 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ABONELİK DİLİ — tek çekim ↔ otomatik yenileme ibareleri, BAYRAĞA BAĞLI (#774c C3, Selim 2026-09-12).

⚠⚠ NEDEN BAYRAĞA BAĞLI (Ömer kararı, 12.09): yasal metni ve landing'i doğrudan değiştirseydik
   herhangi bir push (başkası da atabilir) "otomatik yenilenir" beyanını canlıya taşırdı — oysa
   iyzico'da imza başlığı + `IYZICO_MERCHANT_ID` + sandbox e2e henüz yok → webhook 503/401 →
   alınan abonelikler yenilenmez. Beyan ile davranış AYNI anahtara bağlı: `credits.iyz_recurring()`.
   Bayrak KAPALIYKEN bu modül KİMLİK fonksiyonudur (html olduğu gibi döner) → kaynak metin
   (`legal_body.py`, `landing.py`) BİREBİR korunur; kapı bunu diff ile ölçer.

⚠⚠ İFŞA KURALLA ATOMİK: bayrağı açan env aynı anda (a) davranışı (recurring checkout/webhook)
   ve (b) beyanı (mesafeli satış Madde 3 · iade "Abonelik" · koşullar · landing garanti/liste
   satırları · ödeme güvence notu) açar. İkisi ayrı anahtarda olsaydı ilan ≠ tahsilat olurdu
   (#137b/#139b sınıfı).

⚠ ÇİFTLERİN SOL TARAFI KAYNAKTAKİ CÜMLENİN BİREBİR ALT-DİZESİDİR. Kaynak cümle değişirse
   `str.replace` SESSİZCE no-op olur → kapı bölüm 16 her sol tarafın kaynakta bulunduğunu ve
   bayrak açıkken rotada 0 kez kaldığını ölçer ("son adım bağlanmamış" freni).
⚠ Hukuki yorum ÜRETİLMEDİ: mevcut metnin biçimi/tonu korunarak yalnız yenileme/iptal/kart/
   başarısız çekim olguları yazıldı; founder metni okuyacak.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_YASAL_TR` | `[('kartıyla <b>peşin ve tek çekim</b> tahsil edilir.', 'kartıyla <b>dönem başında peşin</b> tahsil edilir; abonelik döne` | 27 |
| `_YASAL_EN` | `[('Payment is collected <b>in advance, as a single charge</b>, by Visa/Mastercard via the licensed payment institution <` | 46 |
| `_LANDING_TR` | `[('Aylık faturalama · taahhüt yok · e-fatura · otomatik yenileme yok', 'Aylık faturalama · istediğiniz an iptal · e-fatu` | 65 |
| `_LANDING_EN` | `[('Monthly billing · no commitment · e-invoice · no auto-renewal', 'Monthly billing · cancel anytime · e-invoice · autom` | 80 |
| `CIFTLER` | `{'yasal_tr': _YASAL_TR, 'yasal_en': _YASAL_EN, 'landing_tr': _LANDING_TR, 'landing_en': _LANDING_EN}` | 95 |
| `NOT_EK_TR` | `' · Dönem sonunda kayıtlı karttan otomatik yenilenir; hesabınızdan istediğiniz an iptal edebilirsiniz.'` | 98 |
| `NOT_EK_EN` | `' · Renews automatically from your saved card at the end of each period; cancel anytime from your account.'` | 99 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `acik` | `(zorla: bool \| None = None) -> bool` | 102 |  |
| `uygula` | `(html: str, *, yuzey: str, en: bool = False, zorla: bool \| None = None) -> str` | 106 | `yuzey` ∈ {'yasal','landing'}. Bayrak KAPALI → html OLDUĞU GİBİ (kimlik). Açık → çiftler uygulanır. |
| `not_ek` | `(en: bool = False, zorla: bool \| None = None) -> str` | 115 | Ödeme güvence notu eki — bayrak kapalıyken BOŞ dize (yüzeye tek bayt eklenmez). |
| `eski_ibareler` | `(en: bool = False) -> list[str]` | 122 | Bayrak açıkken hiçbir yüzeyde KALMAMASI gereken sol taraflar (kapı bunu rotadan ölçer). |

## `saglik/app/abonelik_routes.py`

`486 satır` · `26 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
iyzico ABONELİK (recurring) rotaları — #774c C2 (Selim, 2026-09-12).

Rotalar (hepsi `@_rota` + `kur(app)`; `include_router` YOK — main.py `_abonelik_kur(app)`):
  POST /iyzico/webhook              iyzico abonelik bildirimi (imza V3, FAIL-CLOSED)
  POST /iyzico/abonelik/callback    abonelik checkout formu dönüşü (token) → doğrula + ilk sipariş
  POST /api/abonelik/iptal          hekim kendi aboneliğini iptal eder (erişim dönem sonuna kadar)
  POST /api/abonelik/kart           kart güncelleme formu (başarısız çekim sonrası)
  POST /iyzico/kart/callback        kart formu dönüşü → başarısız sipariş yeniden denenir
  POST /admin/odeme/aktivasyon      PENDING abonelikleri dönem bitince aktive et (admin butonu)
Yardımcılar: `recurring_checkout` (`/api/checkout` recurring dalı, account_routes çağırır) ·
`yonetim_html` / `donusum_karti` (hesap ve abonelik sayfası parçaları) · `aktivasyon_karti` (/admin/odeme).

PARA/GÜVENLİK SÖZLEŞMESİ:
· Webhook: `IYZICO_MERCHANT_ID` yoksa **503 + log** (yapılandırma eksik, iyzico tekrar dener);
  başlık yok / imza yanlış → **401 + log** — sessiz 200 YOK. Uygulama `iyzico_abonelik.webhook_uygula`
  (gövdeye güvenmez, siparişi iyzico'dan okur). Detay okunamazsa 500 → iyzico tekrar gönderir.
· Hepsi POST (para/durum eylemi; GET ön-yükleme/geçmişle tetiklenmez). CSRF = oturum çerezi
  `samesite=lax` + rotanın kendi `_doctor`/`_admin` guard'ı (`profil_kapisi_mw` POST'a bakmaz).
· Rotalar `def` (senkron) → threadpool; webhook `async` gövdesi yalnız body okur, iş havuzda.
· Bayrak (`credits.iyz_recurring()`) KAPALIYKEN: `/api/checkout` eski tek çekim yoluna gider;
  webhook/iptal/kart rotaları recurring hesap ister → kapalıyken fiilen ulaşılamaz.
· Mailler best-effort, DB transaction'ının DIŞINDA; mail hatası hak-edişi/yanıtı ASLA bozmaz.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 42 |
| `_WEBHOOK_IZ` | `'iyz_webhook_son'` | 43 |
| `_AKT_IZ` | `'iyz_aktivasyon_son'` | 44 |
| `AKTIVASYON_TAVAN` | `25` | 45 |
| `_GSM` | `re.compile('^\\+[1-9]\\d{9,14}$')` | 125 |
| `_GSM_YER_TUTUCU` | `'+905000000000'` | 126 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 48 |  |
| `kur` | `(app) -> None` | 55 |  |
| `_iz_yaz` | `(conn, anahtar: str, deger: str) -> None` | 60 |  |
| `_mail_gonder` | `(paket) -> bool` | 67 | (to, subj, html, text) → send_email; best-effort. ⚠ unsub_url YOK (işlemsel). |
| `_hekim_mail` | `(conn, doctor_id: int)` | 79 |  |
| `_dil` | `(phone: str \| None) -> str` | 85 | Mail dili — `request` yok (webhook): pazar ölçütü telefon (+90 → tr), yoksa tr. |
| `_basarisiz_mail` | `(conn, doctor_id: int) -> bool` | 91 |  |
| `_iptal_mail` | `(conn, doctor_id: int, lang: str) -> bool` | 101 |  |
| `_makbuz` | `(conn, info: dict, lang: str = 'tr')` | 111 | Başarılı sipariş makbuzu — tek çekim yoluyla AYNI şablon (`_sub_receipt_mail`). |
| `telefon_gecerli` | `(phone: str \| None) -> bool` | 129 | Abonelik ucu için GSM — E.164 (+ ve 10-15 rakam) ve yer tutucu DEĞİL. |
| `recurring_checkout` | `(request: Request, conn, d: dict, plan: str)` | 141 | Recurring abonelik başlat → gömülü form sayfası (HTMLResponse) ya da 303. |
| `form_sayfasi` | `(request: Request, d: dict, plan: str, form_html: str, initial: str) -> str` | 173 |  |
| `yonetim_html` | `(hd: dict, en: bool) -> str` | 209 | Recurring hesabın 'Abonelik' kartı yönetim parçası (durum + düğme). |
| `donusum_karti` | `(en: bool, formlar: str) -> str` | 240 | Ön-ödemeli hekim, dönem bitişine ≤ pencere gün: otomatik yenilemeye geçiş kartı. |
| `async` `abonelik_callback` 🌐 | `(request: Request)` | 254 |  |
| `_abonelik_callback_sync` | `(request: Request, token: str)` | 260 |  |
| `_not` | `(conn, token: str, notu: str) -> None` | 314 |  |
| `async` `iyzico_webhook` 🌐 | `(request: Request)` | 326 |  |
| `_webhook_sync` | `(body: bytes, imza: str \| None)` | 332 |  |
| `abonelik_iptal` 🌐 | `(request: Request)` | 369 |  |
| `abonelik_kart` 🌐 | `(request: Request)` | 392 |  |
| `async` `kart_callback` 🌐 | `(request: Request)` | 415 |  |
| `_kart_callback_sync` | `(request: Request, token: str)` | 421 | Kart güncellendi → `unpaid` ise son başarısız sipariş yeniden denenir (sonuç webhook'la). |
| `bekleyen_aktivasyon` | `(conn) -> dict` | 443 | ÜÇ DURUM: {ok, n} — sorgu patlarsa ok=False ('?' basılır, '0' DEĞİL). |
| `aktivasyon_karti` | `(conn) -> str` | 452 |  |
| `admin_aktivasyon` 🌐 | `(request: Request)` | 469 | PENDING abonelikleri aktive et — YALNIZ ADMİN, POST. Aktivasyon = çekim (para eylemi): |

## `saglik/app/account_routes.py`

`1398 satır` · `31 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HESAP + ÖDEME ROTALARI — /account, abonelik, top-up, fatura profili, iyzico dönüşü
(Faz 5, 2026-08-01).

`main.py`'den ayrıldı. Kapsam: 15 rota + bu bölgeye ÖZEL yardımcılar —
`/account` · `/account/delete[/cancel]` · `/account/feedback` · `/api/profile[/password]` ·
`/fatura-bilgileri` (GET+POST) · `/api/checkout` · `/api/portal` · `/api/topup` ·
`/abonelik` · `/api/stripe/webhook` · `/iyzico/callback` · `/odeme/sonuc`.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ).
  Ölçüldü: taşınan kodun main.py'de kalan tek bağımlılıkları `app` nesnesi ve
  `_pay_provider` idi. `app` → `kur(app)`; `_pay_provider` → **webutil**'e taşındı
  (main.py'nin kendi `_apply_try_pricing`i de onu kullanıyor, iki yerde tanımlamak
  tek-doğru-kaynağı bozardı).

⚠⚠ NEDEN `@app.get` DEĞİL `@_rota` — VE NEDEN `include_router` DEĞİL:
  `app` burada yok, o yüzden dekoratörler rotayı hemen kaydetmez; `_rota(...)` onu
  **KAYIT** listesine yazar, main.py `kur(app, blok)` çağırınca gerçek kayıt
  `app.get/post` PUBLIC API'siyle yapılır → `app.routes` DÜZ kalır. `APIRouter` +
  `include_router` Faz 2b'de DENENDİ ve ÖLÇÜLEREK geri alındı: bu FastAPI sürümünde
  tembeldir, rotaları `app.routes`a düzleştirmez → rota envanterini oradan okuyan
  güvenlik ağları (`refactor_parite`, `admin_panel_verify`) sessizce körelir.

⚠⚠ İKİ BLOK — `kur(app, 0)` ve `kur(app, 1)`. BU BİR SÜS DEĞİL:
  kesilen bölgenin ORTASINDA `main.py`'nin `_admin_kur(app)` çağrısı duruyordu
  (`account_page`in hemen altında). Tek bir `kur(app)` ile taşımak 14 rotayı admin
  bloğunun ÜSTÜNE çıkarır ve `app.routes` SIRASI değişirdi. Bugün bu gölgeleme
  ÜRETMEZ — taşınan 15 yolun hepsi statik, `/admin/{did}` ve `/api/admin/{did}`
  onları yakalayamaz — ama güvenceyi "bugünkü rota listesine" bahis koyarak almak
  bu deponun açıkça yasakladığı şey. İki blokla sıra BİREBİR korunur.
  ⚠ Yeni rota eklerken `blok` VARSAYILANI 1'dir (admin çağrısının altı) — doğru
  varsayılan. `blok=0` YALNIZ `/account` için ve ELLE konmuştur; oraya yeni bir şey
  eklemeden önce `scratchpad/faz2b_rota_sira_olc.py` ile ölç.

⚠ PARA İNVARİANTLARI (bu dosyanın var oluş sebebi — BOZMA):
  · **M13: `/api/checkout`, `/api/topup`, `/iyzico/callback` POST'TUR.** GET iken
    tarayıcı/üçüncü-taraf link ön-yükleyicisi hekim HİÇ TIKLAMADAN iyzico'da işlem
    yaratabiliyordu. `@_rota("get", …)` yazmak tek harflik bir para hatasıdır;
    `scratchpad/post_parite_verify.py` bunu 3 sonda ile çiviler.
  · **`_XFO_EXEMPT` (main.py) YOL ADINA GÖRE eşleşir** — `/iyzico/callback` ve
    `/odeme/sonuc` X-Frame-Options muafiyeti buradaki yol string'ine BAĞLI. Yolu
    değiştirirsen muafiyet sessizce kırılır ve hiçbir araç görmez.
  · **İKİ AYRI CÜZDAN:** `gift_balance` (admin hediyesi) kilitsiz — her zaman
    harcanır; `topup_balance` (satın alınan) YALNIZ aktif abonelikte harcanır.
    `/api/topup` satın alımı da yalnız aktif aboneye açıktır (deneme+top-up arbitraj
    freni, sunucuda zorlanır). ✅ **AĞI VAR:** `topup_satinalma_verify.py` (73, CI'da).
    ⚠ Bu satır 2026-08-09'a dek "ağ YOKTUR, önce test yaz" diyordu; ağ 01.08'de
    yazılmıştı → bayat uyarı okuyanı İKİNCİ bir süit yazmaya itiyordu.
    ⚠⚠ ARBİTRAJ FRENİ ARTIK FİYATTA DA VAR (2026-08-09): top-up 15,00 > abonelik
    14,50 TL/kredi. Önceden top-up UCUZDU (10,90-12,90 vs 18,00) ve tek koruma bu
    kilitti. Kilit KALIR; fiyat sırasını bozan biri freni tek bacağa indirir.
  · **rid başına TEK commit VEYA TEK cancel** — ikisi birden çift tahsil eder.
  · **`/odeme/sonuc` bilinmeyen `durum` → `/account`'a 303.** Bu bilinçli bir
    guard: başarı dalı catch-all `else` DEĞİL. Çıplak `/odeme/sonuc` F5'te SAHTE
    başarı + çift `purchase` pikseli ateşlerdi. Yeni durum eklerken guard'ı güncelle.
  · **`iyzico_callback` grant'ı yalnız `fraud==1`** (fraud==0 = inceleme → `pending`,
    hak-ediş ERTELENİR) + kurucu kontenjanı advisory-kilitli re-check.
  · **`purchase` olayı DB'den üretilir**, URL'den DEĞİL (`?v=999999` ile sahte
    dönüşüm yazdırılabiliyordu) — `_purchase_script` çağıranı ödemeyi
    `app.iyzico_payment`ten doğrular.

⚠ Panelin aksine bu yüzey İKİ DİLLİ: metinler `EN_PAIRS` ile çevrilir → yeni
  İngilizce çeviri yazarken **kıvrık kesme `’` (U+2019)** kullan; düz `'` bir JS
  string'ini kapatır ve o sayfanın TÜM JS'i sessizce çöker.

⚠ main.py bu adları `# noqa: F401` ile YENİDEN DIŞA VERİR (fasad). Ölçüldü: bugün
  hiçbiri main.py dışından okunmuyor — fasad zorunlu değil, ucuz sigorta.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 90 |
| `_CARD_MARKS` | `' <span style="display:inline-flex;align-items:center;gap:5px;vertical-align:middle;margin-left:5px"><span style="displa` | 202 |
| `_SN_IKON` | `{'acc': '<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>', 'plan': '<rect x="2" y="5` | 235 |
| `_PR_STYLE` | `'<style>\n.pr-wrap{max-width:460px;margin:0 auto;padding:56px 26px 48px;text-align:center}\n.pr-brand{font-family:var(--` | 1186 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, blok: int = 1, **kw)` | 93 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app, blok: int = 1) -> None` | 106 | Bu bloğun rotalarını uygulamaya bağlar. main.py İKİ KEZ çağırır (0, sonra 1). |
| `_purchase_script` | `(item_name: str, value: float, currency: str, txid: str) -> str` | 118 | Ödeme başarısı sayfasında `purchase` olayını BİR KEZ ateşle + URL'i temizle. |
| `_lang_switch` | `(lang, cls = 'langsw')` | 152 |  |
| `_plan_price_disp` | `(plan: str, en: bool = False, d = None, birim = None)` | 169 | (fiyat_metni, birim) — sağlayıcıya göre. iyzico→TL, stripe→USD. |
| `_plan_label` | `(plan: str, en: bool = False, d = None) -> str` | 184 | Plan etiketi ('Kurucu Hekim (2.900 TL/ay)' / 'Standard (4.500 TL/mo)') — sağlayıcıya göre. |
| `_pay_secure_note` | `(en: bool = False, d = None) -> str` | 210 | Ödeme güvence satırı — sağlayıcıya göre + kabul edilen kart şemaları (Visa/Mastercard). |
| `_alt_nav` | `(aktif: str, lang: str) -> str` | 243 | `/account` · `/abonelik` · `/fatura-bilgileri` ALT-NAVİGASYONU. |
| `account_page` 🌐 | `(request: Request)` | 275 |  |
| `api_profile_update` 🌐 | `(request: Request, full_name: str = Form(''), unvan: str = Form(''), specialty: str = Form(''), kurum: str = Form(''), phone: str = Form(''))` | 573 |  |
| `api_profile_password` 🌐 | `(request: Request, old_pw: str = Form(...), new_pw: str = Form(...), new_pw2: str = Form(...))` | 584 |  |
| `account_delete` 🌐 | `(request: Request, confirm: str = Form(''))` | 603 | Hesap silme talebi (KVKK unutulma hakkı). Yazılı onay ("SİL"/"DELETE" ya da hesap e-postası) |
| `account_delete_cancel` 🌐 | `(request: Request)` | 620 | Silme talebini geri al (grace içinde). deletion_requested_at=NULL. |
| `account_feedback` 🌐 | `(request: Request, category: str = Form(''), message: str = Form(''))` | 631 | Hekim öneri/geri bildirimi → DB'ye kaydet + founder'a e-posta (best-effort). |
| `_fatura_js` | `(en = False)` | 661 | Fatura formu JS'i (dil-duyarlı hint). i18n_mw'den geçse de İngilizce metin EN_PAIRS'te yok → no-op; |
| `billing_page` 🌐 | `(request: Request, plan: str = '', pkg: str = '', err: str = '')` | 679 |  |
| `billing_submit` 🌐 | `(request: Request, plan: str = '', pkg: str = '', btype: str = Form('bireysel'), bname: str = Form(''), tckn_vkn: str = Form(''), tax_office: str = Form(''), address: str = Form(''), city: str = Form(''))` | 725 |  |
| `_buy_form` | `(action: str, field: str, value: str, label: str, cls: str = 'btn btn-ink btn-block') -> str` | 772 | Satın-alma düğmesi = tek alanlı POST formu (görsel olarak eski <a class="btn"> ile aynı). |
| `api_checkout` 🌐 | `(request: Request, plan: str = Form('aylik'))` | 780 | ⚠⚠ 2026-08-09 (#142b): sağlayıcı artık GLOBAL env değil **HEKİMİN PAZARI**. |
| `api_portal` 🌐 | `(request: Request)` | 845 |  |
| `_plans_cards_html` | `(lang: str, kurucu_left: int = 0, d = None) -> str` | 861 | Plan kartları — ⚠⚠ 2026-08-09 founder kararı: TEK PLAN (Aylık 1.450/100 + Yıllık |
| `abonelik_page` 🌐 | `(request: Request, ok: str = '', iptal: str = '', session_id: str = '')` | 886 |  |
| `async` `stripe_webhook` 🌐 | `(request: Request)` | 1001 |  |
| `_stripe_webhook_sync` | `(payload, sig)` | 1008 | Webhook işleme — DB havuzda (M5). Hata dalı `with` İÇİNDE kalır: eski davranışla |
| `_fmt_try_amount` | `(amount) -> str` | 1018 | 4500 → '4.500,00 TL' (TR binlik '.' / ondalık ','). Geçersizse ''. |
| `_sub_receipt_mail` | `(conn, doctor_id, plan, interval, amount, txn, lang, bonus: int = 0)` | 1027 | Abonelik makbuzu mail tuple'ı (to,subj,html,text) \| None — best-effort (hata/e-posta yok → None). |
| `_iyz_fail_note` | `(conn, token: str, note: str)` | 1054 | Red nedenini iyzico_payment satırına yaz (best-effort) — token süresi dolsa da |
| `async` `iyzico_callback` 🌐 | `(request: Request)` | 1065 | iyzico Checkout Form dönüşü — hosted form ödeme sonrası tarayıcıyı buraya `token` ile |
| `_iyzico_callback_sync` | `(request: Request, token: str)` | 1078 | Doğrulama + hak-ediş — `iyzico_callback`ın senkron gövdesi (davranış BİREBİR; yalnız |
| `_billing_err` | `(msg: str) -> str` | 1182 |  |
| `odeme_sonuc` 🌐 | `(request: Request, durum: str = '', tur: str = '')` | 1212 | Ödeme sonuç sayfası (başarılı/tamam/iptal/beklemede) — iyzico callback buraya yönlendirir. |

## `saglik/app/admin_alarm.py`

`131 satır` · `3 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
AYLIK KULLANIM ALARMI — sorgu + `/admin` kartı (kendi evi, 2026-08-22 · #396b-K).

⚠⚠ NEDEN AYRI MODÜL: alarm ilk yazımda `admin_queries.py` (sorgu) ve `admin_routes.py`
(kart) arasına dağıtılmıştı ve İKİ dosyayı da tavana dayadı — `admin_queries.py`
826/831 (tavanı yükselten kayıt taşmanın 49 satırını AÇIKÇA bu sorguya yazmış ve
"bir sonraki artış BÖLME ister" demişti), `admin_routes.py` 1.821/1.806 = **AŞTI**
ve o dosyanın kuralı "bölmesiz yükseltme YASAK". Depo deseni: yeni yüzey şişkin
dosyaya değil KENDİ EVİNE (`admin_reklam.py` / `admin_hekim.py`).

⚠ Buraya ROTA GİRMEZ: `/admin` sayfası `admin_routes.py`de kalır, bu modül yalnız
  (a) salt-okuma sorgusunu ve (b) o sorgunun kart gövdesini taşır — `admin_reklam.py`
  ile aynı sözleşme. Yazan hiçbir yol eklenmez.
⚠ Dairesel import YOK: `store`u modül düzeyinde İTHAL ETMEZ (`_sq()` ile geç bağlanır);
  `store` bu modülü ithal edip `aylik_alarm`ı yeniden dışa verir → çağıranlar DEĞİŞMEDİ.
⚠ Sunum yardımcıları `admin_sunum.py`den gelir (üç-durum disiplini orada tanımlı).

⚠⚠ DAVRANIŞ DEĞİŞMEDİ — bu bir TAŞIMADIR: `aylik_alarm_verify` (28 iddia) taşımadan
ÖNCE ve SONRA aynı sonucu vermek zorundadır; kartın `/admin` grid'indeki yeri de aynı.
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_sq` | `()` | 33 | `store`a GEÇ bağlanma — modül düzeyinde ithal DAİRESEL olurdu (`admin_queries` |
| `aylik_alarm` | `(conn, esik: int \| None = None, gun: int \| None = None) -> dict` | 40 | AYLIK ANOMALİ ALARMI — sınırsız abonelikte marj kuyruğunu GÖRÜNÜR kılar. |
| `kart` | `(alarm: dict) -> str` | 92 | `/admin` "Aylık kullanım alarmı" kartı. Girdi: `aylik_alarm()` çıktısı. |

## `saglik/app/admin_denetim.py`

`390 satır` · `9 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMİN ERİŞİM / DENETİM LOGU — yazma yolu + okuyucu (#522b, 2026-08-31).

Şartname: Selim v2 (birleşik) + Ömer'in üç hükmü. Şema gerekçeleri `db/app_schema.sql`
`app.admin_erisim` bloğunda — BURAYA KOPYALAMA (kopyalanan kural bayatlar).

⚠⚠ NEDEN AYRI MODÜL: kod `store.py` (+75) ve `admin_routes.py` (+70) içine yazıldığında
  İKİSİ DE `dosya_boyut_verify` tavanını aştı → tavan yükseltmek yerine BÖLÜNDÜ
  (`admin_pano`/`admin_sunum`/`admin_hekim`/`admin_alarm` deseniyle aynı).

⚠⚠ İKİ FARKLI "FAIL" — KARIŞTIRMA (Ömer hükmü):
  · **KAPSAM = FAIL-CLOSED.** Beyaz liste "loglanacaklar" DEĞİL, `_MUAF` = muaflar listesi.
    Yeni bir admin rotası eklendiğinde VARSAYILAN LOGLANIR. Sebep: ileride eklenen klinik
    içerikli bir yüzey sessizce eksik loglanmasın ("son adım bağlanmamış" bu deponun en
    sık hata sınıfı). Tanınmayan rota `eylem='siniflandirilmamis'` ile yazılır — satır
    KAYBOLMAZ, ama sınıflandırılmadığı GÖRÜNÜR.
  · **YAZMA = FAIL-OPEN, AMA SESSİZ DEĞİL.** DB hıçkırığı founder'ı kendi admin'inden
    kilitlemesin. AMA depo invarianti geçerli: *"çalıştırılamadı ≠ temiz"* → yazma
    başarısız olursa KAYIP KAYDEDİLİR (sunucu log'u + `app.setting` kimliksiz sayacı),
    böylece "loglanmamış erişim" sayısı BİLİNİR olur, sıfır sanılmaz.

⚠⚠ İKİ FAZLI YAZMA — NEDEN: `_admin()` isteğin BAŞINDA koşar, `hedef_kume` ve
  `kayit_sayisi` ise sayfa hesaplandıktan SONRA bilinir.
    faz 1  `erisim_kaydi()`  — `_admin()` içinde, KAPSAMI garanti eder (satır her hâlükârda
                               yazılır; `hedef_id` path_params'tan gelir).
    faz 2  `kapsam_yaz()`    — yüzey kendi kümesini/sayısını EKLER (aynı satırı UPDATE).
  ⚠ Faz 2 ATLANIRSA satır yine vardır (yalnız kümesiz) — fail-closed budur. Kapı hangi
    yüzeyin faz 2 borcu olduğunu ayrı ayrı ölçer.

⚠⚠ İÇERİK VE HAM QUERY STRING GEÇMEZ: `?q=` terimi pratikte bir hekimin ADI ya da
  E-POSTASIDIR → terimin kendisi kişisel veridir. İmzada öyle bir parametre YOKTUR ve
  eklenirse kapı KIRMIZI olur.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_MUAF` | `frozenset({'/admin/kur', '/admin/reklam'})` | 50 |
| `_EYLEM` | `{'/admin': 'gor', '/admin/sorular': 'gor', '/admin/arsiv': 'gor', '/admin/odeme': 'gor', '/admin/{did}': 'gor', '/admin/` | 66 |
| `KUME_YUZEY` | `frozenset({'/admin/sorular', '/admin/odeme/mutabakat'})` | 118 |
| `SAYI_YUZEY` | `frozenset({'/admin', '/admin/arsiv', '/admin/odeme'})` | 119 |
| `_KAYIP_ANAHTAR` | `'admin_erisim_kayip'` | 121 |
| `ALANLAR` | `('id', 'admin_id', 'hedef_id', 'hedef_kume', 'kayit_sayisi', 'yol', 'eylem', 'at', 'ip', 'hedef_kayit')` | 237 |
| `SAKLAMA_AY` | `24` | 351 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_inet_ya_da_none` | `(ip: str)` | 124 | IP'yi `inet` kolonuna girmeden ÖNCE doğrula; geçersizse None. |
| `_kayip_say` | `(conn, sebep: str) -> None` | 144 | Yazılamayan denetim kaydını SAY (kimliksiz) — "çalıştırılamadı ≠ temiz". |
| `erisim_yaz` | `(conn, *, admin_id, yol: str, eylem: str, hedef_id = None, ip: str = '', hedef_kayit = None)` | 172 | Denetim satırını yaz; log id döner (faz 2 için), yazılamazsa None. |
| `kapsam_yaz` | `(conn, request: Request, *, kume = None, sayi = None) -> bool` | 207 | FAZ 2 — yüzey kendi kapsamını EKLER (DISTINCT hedef kümesi ya da kayıt sayısı). |
| `erisim_listesi` | `(conn, limit: int = 200, hedef: int \| None = None) -> list[dict]` | 241 | Denetim logunu oku — m.11'in gerçek sorusu: "BENİM verime kim baktı". |
| `yol_sablonu` | `(request: Request) -> str` | 261 | Rota ŞABLONU (`/admin/{did}/sorular`) — ham URL DEĞİL. |
| `erisim_kaydi` | `(request: Request, conn, d, adm) -> None` | 274 | FAZ 1 — admin yüzeyine erişimi yaz. `_admin()` içinden çağrılır, FAIL-OPEN. |
| `_feedback_hekimi` | `(conn, fid)` | 326 | `/admin/feedback/{fid}` → geri bildirimin sahibi hekim (best-effort). |
| `purge` | `(conn) -> int` | 354 | Saklama süresi dolan erişim kayıtlarını KALICI sil. Döner: silinen satır sayısı. |

## `saglik/app/admin_grafik.py`

`130 satır` · `3 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
AY AY EĞİLİM GRAFİĞİ — satır içi SVG (founder 2026-09-04: "grafik yap düştüğünü
arttığını görelim").

⚠⚠ **HARİCİ KÜTÜPHANE YOK, JS YOK.** Panel sunucuda üretilir; bir grafik kütüphanesi
  eklemek (a) admin'e CDN bağımlılığı sokar, (b) `i18n_mw`nin script bloklarını çevirme
  tuzağına yeni bir yüzey açar (CLAUDE.md "apostrof tuzağı"), (c) `_ANALYTICS_PATH_ONLY`
  ile korunan admin yüzeyine dış istek ekler. SVG bunların üçünü de doğurmaz.

⚠⚠ **GRAFİK, TABLONUN YERİNE GEÇMEZ — AYNI SAYIYI GÖSTERİR.** Seriler `kz_tablosu`nun
  bastığı hücrelerle AYNI kaynaktan (`aylik_kz`) gelir; grafik ayrı bir hesap YAPMAZ.
  Ayrı hesap yapsaydı iki yüzey sessizce ayrışırdı (bu panelin en pahalı hata sınıfı:
  "panel kendi kendisiyle çelişiyor").

⚠⚠ **ÖLÇÜLEMEYEN NOKTA ÇİZİLMEZ — 0 UYDURULMAZ.** Bölen yokken (abone-ay/deneme-ay = 0)
  oran hesaplanamaz; o ay için nokta ATLANIR ve çizgi orada KESİLİR. Sıfır çizmek,
  "maliyet düştü" diye okunan bir YALAN üretirdi — tablodaki "—" hücresinin grafik
  karşılığı budur.

⚠ Eksen etiketi: her ay değil, sığdığı kadarı (ilk/son + aralar) — üst üste binen etiket
  okunmaz ve "grafik var" ile "grafik okunuyor" farkı tam burada başlar.
⚠ Renkler token'lardan: `--ink` (abone) · `--verify`/amber (deneme). Kontrast için METİN
  yeşili `--ink-2` (CLAUDE.md tasarım kuralı).
⚠ Panel Türkçe tek-dilli → buradaki metni `EN_PAIRS`e EKLEME.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_RENK` | `{'abone': '#0C3B3E', 'deneme': '#8A6D1F', 'toplam': '#12B886'}` | 33 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_tr` | `(v: float, dec: int = 0) -> str` | 36 |  |
| `_ay_kisa` | `(ym: str) -> str` | 41 | '2026-08' → '08.26' (dar eksende ay adı sığmaz; sayı biçimi TR). |
| `cizgi_grafik` | `(aylar: list[str], seriler: list[tuple[str, str, dict[str, float]]], baslik: str = '') -> str` | 46 | Çok serili çizgi grafiği. |

## `saglik/app/admin_hasta.py`

`394 satır` · `9 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMİN — HEKİM GÖRÜNÜRLÜĞÜ: hasta defteri + davranış zaman çizgisi (#726b, founder 2026-09-04).

Founder isteği (birebir): "admin sayfasında hekim özel sayfasına girince hekimin kaç hastası
var. hasta analizinde neler yaptı. hastalıklar sordu mu vs. ne yaptığını detaylı görmemiz
lazım ve kayıt tutmamız lazım."

ÜÇ ROTA (hepsi `admin_routes._admin()` kapısından geçer = ROL **VE** DENETİM):
  · `/admin/{did}/hastalar`      — hekimin hasta listesi
  · `/admin/{did}/hasta/{cid}`   — TAM İÇERİK (founder K1)
  · `/admin/{did}/eylemler`      — `app.doctor_event` davranış zaman çizgisi (founder K2)
+ `/admin/{did}` detay sayfasına İKİ KOMPAKT KART (`defter_karti`, `ne_yapti_karti`).

⚠⚠ NEDEN AYRI MODÜL: 04.09 ölçümünde `admin_routes` 5 · `admin_hekim` 4 · `admin_queries` 3
  satırlık paya sahipti (`dosya_boyut_verify`) — bu iş HİÇBİRİNE sığmaz. `admin_hekim.py`
  başlığındaki "ileride hekim-odaklı başka tam sayfa BURAYA gelir" notu, o dosyanın kendi
  tavanı yüzünden uygulanamadı; kayıt burada bırakılıyor ki bir sonraki oturum aynı soruyu
  yeniden sormasın.
⚠ `include_router` DEĞİL — `kur(app)` `app.get` ile kaydeder (`app.routes` DÜZ kalmalı;
  gerekçe ölçümüyle `admin_routes.py` başlığında).
⚠ Rota sırası: `/admin/{did}/hastalar` ve `/admin/{did}/eylemler` İKİ segmentlidir,
  `/admin/{did}` ile ÇAKIŞMAZ. `/admin/{did}/hasta/{cid}` üç segmentli — o da ayrık.

⚠⚠ FOUNDER KARARI K1 — TAM İÇERİK, VE BEDELİ EKRANDA YAZILI: 2026-08-05 kararıyla
  `case_note.code` GERÇEK HASTA ADI taşıyabilir. Yani bu sayfa yöneticiye KVKK m.6 özel
  nitelikli veri gösterir. Founder bunu bilerek seçti; sayfa bunu SAKLAMAZ, amber bantla
  söyler (`_HAM_UYARI`). ⚠ Bu bir "gereksiz uyarı" değildir: uyarıyı kaldıran, sayfanın
  destekleyemediği bir masumiyet ima etmiş olur.
⚠⚠ DENETİM LOGUNA İÇERİK YAZILMAZ — yalnız rota ŞABLONU (`/admin/{did}/hasta/{cid}`),
  `hedef_id` (HEKİM) ve `hedef_kayit` (HASTA KAYDININ id'si). Hasta adı, anlatı, dosya adı
  ve arama terimi loga GİRMEZ (`admin_denetim` sözleşmesi).
  ⚠⚠ `hedef_kayit` 04.09'DA EKLENDİ (#726b/P1-3) ve bu satır ÖNCE YANLIŞTI: "(+ `cid` yol
    parametresi olarak)" diyerek hasta kimliğinin kaydedildiğini İMA EDİYORDU, oysa şablonda
    `{cid}` yer tutucu olarak kalıyor ve gerçek değer HİÇBİR KOLONDA yoktu (Fırat ölçtü).
    Yani docstring, kodun yapmadığı bir güvenceyi anlatıyordu — bu üründeki en pahalı hata
    sınıfı ("yanlış cevap" değil YANLIŞ GÜVEN). Şimdi ikisi de ölçümle uyumlu.
⚠⚠ ŞİFRELİ DOSYANIN KENDİSİNİ İNDİRME UCU BU TURDA AÇILMADI (ayrı risk, ayrı karar).
  Dosya adı + `extracted_text` (modelin çıkarımı) gösterilir; `content_enc`e DOKUNULMAZ.

⚠⚠ İKİ SAYI İKİ KÜME (`admin_sunum._SORU_KAPSAM` dersi): `app.usage` turları ≠ `app.message`
  kayıtları ≠ `app.doctor_event` satırları. Üçü FARKLI kümedir, eşit çıkacakları İDDİA
  EDİLMEZ ve kapsam EKRANDA yazılıdır (`_KAPSAM`).
⚠ ÜÇ DURUM: `ok=False` → amber "Ölçülemedi", ASLA yeşil, ASLA sessiz "0".
⚠ Panel BİLEREK Türkçe tek-dilli → buradaki metni `EN_PAIRS`e EKLEME.

Kapılar: `admin_panel_verify` (erişim küme eşitliği) · `admin_denetim_verify` (`_EYLEM`
haritası + loga içerik sızmıyor) · `hekim_iz_verify` (kayıt katmanı).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_LIMIT_HASTA` | `300` | 61 |
| `_LIMIT_EYLEM` | `500` | 62 |
| `_EYLEM_GUN` | `90` | 63 |
| `_HAM_UYARI` | `f'<div style="margin:-4px 0 14px;font-size:13px;{_ADM_AMBER};border-radius:10px;padding:10px 12px"><b>⚠ HAM KLİNİK İÇERİ` | 65 |
| `_KAPSAM` | `"Bu kart <b>davranış izi</b> sayar (<code>app.doctor_event</code>; hekimin hesabı açık olduğu sürece saklanır, hesap kap` | 74 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_defter_sayilari` | `(conn, did: int) -> dict` | 90 | Hekimin hasta defteri künyesi TEK SORGUDA. |
| `defter_karti` | `(conn, did: int) -> str` | 123 | "Hasta defteri" kartı — N hasta / ziyaret / dosya / çıkarım / sentez + son işlem. |
| `ne_yapti_karti` | `(conn, did: int) -> str` | 153 | "Ne yaptı" kartı — `app.doctor_event` yüzey dağılımı (son `_EYLEM_GUN` gün). |
| `_kabuk` | `(adm, request, baslik: str, ic: str) -> str` | 176 |  |
| `_hekim_ya_da_yonlendir` | `(conn, did: int)` | 186 |  |
| `admin_hasta_listesi` | `(did: int, request: Request)` | 191 |  |
| `admin_hasta_detay` | `(did: int, cid: int, request: Request)` | 243 | TAM İÇERİK (founder K1): anlatı, kronik, ilaçlar, alerji, ziyaretler, sentezler, |
| `admin_eylemler` | `(did: int, request: Request)` | 343 | `app.doctor_event` zaman çizgisi + yüzey dağılımı (founder K2). |
| `kur` | `(app) -> None` | 386 | Rotaları bağlar (main.py, `admin_hekim.kur(app)` ile aynı noktadan sonra). |

## `saglik/app/admin_hekim.py`

`115 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HEKİM-ODAKLI ADMİN SAYFALARI — `/admin/{did}/sorular` (founder 2026-08-15: "adminde
doktora basınca hangi soruları sorduğunu gösteren bir sayfa açsın").

⚠⚠ NEDEN AYRI MODÜL: `admin_routes.py` tavanına 10 satır kalmıştı ve bir sonraki
  yükseltme **`admin_pano.py` bölmesi yapılmadan kabul edilmiyor** (`dosya_boyut_verify`
  girdisi, iki kez ertelenmiş şart). Yeni admin yüzeyi şişkin dosyaya değil kendi evine
  yazılır; ileride hekim-odaklı başka tam sayfa (giriş geçmişinin tamamı vb.) BURAYA gelir.
  Detay sayfasındaki "Sorduğu sorular (son 20)" kartı `admin_routes.py`de KALDI — bu sayfa
  onun "tümü, yanıtlarıyla" hâlidir; kart ve liste satırı buraya link verir.

⚠ `include_router` DEĞİL — `kur(app)` `app.get` ile kaydeder (gerekçe `admin_routes.py`
  başlığında ölçümüyle: tembel router `app.routes`ı düzleştirmez, iki ağ körelir).
  main.py `_panel_kur(app)`den SONRA çağırır: `/admin/{did}/sorular` İKİ segmentli,
  tek-segmentli `/admin/{did}` ile ÇAKIŞMAZ → sıra serbest, mevcut rota indeksleri kaymaz.
⚠ Veri: `store.doctor_questions` (mevcut `app.message`, YENİ saklama yok). ÜÇ DURUM:
  ok=False → amber "Ölçülemedi" (`_adm_hata`), sessiz boş tablo BASILMAZ. Soru+yanıt HAM
  (2026-07-29 kararı) · hekim kimliği gösterilir (2026-08-14 kararı) · saatler TR duvar saati.
⚠ TAVAN EKRANDA SÖYLENİR: `_LIMIT`+1 çekilir; taşıyorsa "en az N" yazılır (toplam iddiası
  kurulmaz — `/admin/sorular` ile aynı disiplin).
⚠ Panel Türkçe tek-dilli → buradaki metni `EN_PAIRS`e EKLEME.
Kapılar: `giris_gecmisi_verify` (sayfa + üç durum + link zinciri) · `admin_panel_verify`
  (erişim küme eşitliği: rota `GET_YOL`da beyanlı, admin olmayan 303).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_LIMIT` | `300` | 35 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `admin_doctor_questions_page` | `(did: int, request: Request)` | 38 |  |
| `kur` | `(app) -> None` | 112 | Rotayı uygulamaya bağlar (main.py, `_panel_kur(app)`den sonra). |

## `saglik/app/admin_pano.py`

`681 satır` · `17 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMIN PANOSU — `/admin` KART GÖVDELERİ (bölme, 2026-08-24 · #396b-Q).

⚠⚠ NEDEN AYRI MODÜL — BORÇ ÖDEMESİ, TERCİH DEĞİL: `admin_routes.py` tavanı
  2026-08-22'de **istisna olarak** bir kez yükseltildi (1.760 → 1.806) ve kayda
  "istisna tekrarlanabilir bir hak DEĞİL, bir sonraki artış yine BÖLME ister"
  diye yazıldı (`dosya_boyut_verify` girdisi + `_gorev.txt` #396b-Q). Bölme adayı
  o kayıtta ADIYLA duruyordu: `admin_page`in kart kurma bloğu.

⚠⚠ NEDEN `kur(app)` YOK — BU BİR **SAF GÖVDE** TAŞIMASI (`landing.py` / `cases_body.py`
  / `admin_sunum.py` sınıfı), Faz 2b rota taşıması DEĞİL. Ölçüldü (AST, taşımadan önce):
  taşınan bloğun serbest değişkenleri yalnız store çıktısı sözlükleri
  (`dash insights cite atif_k feed adopt leak pay_att`) ve sunum yardımcılarıydı —
  `request` GÖRÜLMÜYOR, `conn` GÖRÜLMÜYOR, hiçbir rota tanımlanmıyor.
  ⇒ `@_rota`/`kur(app)` GEREKMEZ ve **`app.routes` HİÇ DEĞİŞMEZ**. `include_router`
  sorusu burada hiç doğmaz; o karar rota TAŞIYAN bölmeler içindir ve ölçülmüş gerekçesi
  `admin_routes.py` başlığındadır.

⚠ DAİRESEL İMPORT YOK — İTHAL ZİNCİRİNİN YÖNÜ ÖLÇÜLDÜ: bu modülü YALNIZ `admin_routes`
  ithal eder; `store`/`webutil`/`admin_sunum` onu ithal ETMEZ. Bu yüzden ithaller
  MODÜL DÜZEYİNDE yapılabilir. `admin_alarm.py` aynısını YAPAMIYOR çünkü onu `store`
  ithal ediyor (`store → admin_alarm → admin_sunum → webutil` zinciri kısmen
  başlatılmış `webutil`e çarpıyordu) — fark komşunun stili değil, ZİNCİRİN YÖNÜ.

⚠⚠ DAVRANIŞ DEĞİŞMEDİ — BU BİR TAŞIMADIR. Kabul ölçütü "sayfa 200 dönüyor mu" DEĞİL
  (o ölçüt bir kartın tümden düşmesini de yeşil geçirir = "son adım bağlanmamış"):
  aynı DB · aynı hesap ile ÖNCE/SONRA **bayt paritesi** ölçüldü
  (`scratchpad/_kamil_pano_parite.py`).

⚠ PANEL DİSİPLİNİ BURADA DA GEÇERLİ (gerekçesi `admin_routes.py` başlığında, İKİ YERE
  YAZILMAZ): hata durumunda POZİTİF HÜKÜM YASAK · sessiz sıfır YOK · sıfırın SEBEBİ
  iddia edilmez · sayaçlar AYRIK. Yardımcılar `admin_sunum`dan gelir; buraya eklenecek
  yeni kart da onları KULLANMAK ZORUNDA.
⚠ PANEL BİLEREK TÜRKÇE TEK-DİLLİ — buradaki metinleri `EN_PAIRS`e EKLEME.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_IC_PILL` | `'<span class="pill" style="background:var(--mist);color:var(--slate);margin-left:5px" title="İç trafik: admin ya da is_t` | 49 |
| `_KIND_ARSIV` | `{'sub': 'Abonelik', 'topup': 'Top-up (geçmiş)', 'sub_recur': 'Abonelik (otomatik)'}` | 60 |
| `_KIRMIZI` | `'color:var(--alert-ink)'` | 324 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `gunluk_karti` | `(dash: dict) -> str` | 66 | "Son 14 gün" kartı — günlük yeni kayıt · soru · API maliyeti (mini bar). |
| `mod_karti` | `(dash: dict) -> str` | 84 | "Bu ay — mod dağılımı" kartı. |
| `soru_istatistik_karti` | `(insights: dict, cite: dict, atif_k: dict) -> str` | 95 | "Soru istatistiği" kartı + `Atıf oranı` ve `Sıfır-atıf` bölümleri. |
| `ret_orani_karti` | `(ret: dict) -> str` | 190 | "Ret oranı" kartı (#C4) — kapsam kilidi kaç soruyu geri çevirdi. |
| `duvar_karti` | `(duvar: dict) -> str` | 247 | "Erişim duvarı → fiyat" kartı (#490b) — huninin PAYDASI. |
| `_butce_sayac_satirlari` | `(duvar: dict) -> str` | 327 | Duvar kartının DÖNEM BÜTÇE KAPISI satırları (2026-09-02, §7). |
| `butce_blogu` | `(b: dict) -> str` | 360 | "Dönem bütçesi" bloğu — Birim ekonomi kartının içinde (§7; founder 02.09, 7 gün GÖZLEM). |
| `_tr_kur` | `(v) -> str` | 440 | Kur değeri TR yazımıyla (41.5 → '41,5'); None → '?'. |
| `edinim_karti` | `(dash: dict) -> str` | 445 | "Edinim kaynağı (bu ay)" kartı — kayıt → doğruladı → sordu → ÖDEDİ hunisi. |
| `besleme_karti` | `(feed: dict) -> str` | 482 | "Bilgi tabanı beslemesi" kartı — hat bayatlarsa ürünün TEK değer vaadi çürür. |
| `benimseme_karti` | `(adopt: dict) -> str` | 520 | "Özellik benimseme" kartı — bakım yükü taşıyan ama kullanılmayan özellikler. |
| `sizinti_karti` | `(leak: dict) -> str` | 545 | "Aktivasyon sızıntısı" kartı — kaç KİŞİ sıfırda oturuyor. |
| `dogrulama_bekleyen_karti` | `(dash: dict) -> str` | 570 | "E-posta doğrulaması bekleyen (deneme)" kartı — hunide TAKILANLAR. |
| `silme_talebi_karti` | `(dash: dict) -> str` | 583 | "Hesap silme talepleri" kartı (KVKK m.7 kuyruğu). |
| `en_aktif_karti` | `(dash: dict) -> str` | 593 | "En aktif hekimler (bu ay)" kartı. |
| `odeme_deneme_karti` | `(dash: dict, pay_att: dict) -> str` | 603 | "Ödeme denemeleri" kartı — REDLER DE GÖRÜNÜR. |
| `aklama_karti` | `(duvar: dict) -> str` | 641 | "Analiz çıktısı — aklama/yönlendirme monitörü" kartı (#673b; sayaç #665b `cds/aklama_monitor`). |

## `saglik/app/admin_queries.py`

`1000 satır` · `21 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMIN PANELİ SORGULARI — SALT-OKUMA raporlama (para YAZMAZ).

⚠⚠ NEDEN AYRI DOSYA (2026-08-10, Ömer kararı · `_gorev.txt` #141b'de adıyla bekliyordu):
  `store.py` beşinci kez tavan yükseltmesi isteyecekti. ⚠ Gerekçe "tavanın altına inmek"
  DEĞİL — o çerçeve, bölme sayıyı yeterince düşürmediğinde bölmeyi gereksiz gösterir.
  **Amaç SAYIYI değil YÜZEYİ küçültmek:** buradaki hiçbir fonksiyon para YAZMAZ, rezervasyon
  / hak-ediş / cüzdan yoluna DOKUNMAZ. `store.py` para yolunun evi olarak KALIR.

⚠⚠ BURAYA YAZMA KURALI: bu dosyaya **yalnız salt-okuma panel sorgusu** girer. `settle_payment`
  · `grant_*` · `hediye_gun_ver` · `set_setting` gibi YAZAN hiçbir şey buraya taşınmaz;
  şüphedeysen `store.py`de bırak.
  ⚠ 2026-08-22: bu satır bir zamanlar kredi rezervasyon fonksiyonlarını da sayıyordu;
    o zincir 2026-08-21'de SİLİNDİ (kredi sistemi kaldırıldı) → **artık aranmaz.** Kural
    (yazan hiçbir şey buraya girmez) değişmedi, yalnız örnek listesi güncellendi.

⚠⚠ ÜÇ DURUM SÖZLEŞMESİ BURADA YAŞIYOR (`_admin_safe`): her sorgu `ok=True` + veri ya da
  `ok=False` + `error` döner. Panelin **"hatada amber ölçülemedi, ASLA sessiz sıfır"**
  invariantı buna dayanır — `ok` anahtarını kaldıran/atlayan bir değişiklik, panelin
  yanlış güven üretmesi demektir (2026-07-30 çürütücü kararı).

⚠⚠ `_IC_SQL` FONKSİYON BAŞINDA YEREL TAKMA ADA ALINIR (`_IC_SQL = _sq()._IC_SQL`) ve
  f-string'lerde `{_IC_SQL}` YAZIMI KORUNUR. Sebep ölçüldü: bölmede `{_sq()._IC_SQL}` diye
  yazınca İKİ kapı birden kırıldı — `panel_ic_trafik_verify` bu dizgeyi SAYIYOR ("her para
  sorgusu predikatı interpole etmeli", 7 yer) ve MD mutasyonu hedef metni olarak kullanıyor
  (bulamayınca mutasyon SESSİZCE etkisiz kaldı). Yazımı değiştirmek invariantı denetlenemez
  yapar; geç bağlama korunur ama METİN aynı kalır.

⚠ DAİRESEL İMPORT YOK: bu modül `store`u **modül düzeyinde İTHAL ETMEZ.** Ortak SQL
  parçaları (`_IC_SQL`, `_TR_BUGUN`, `_tr_gun`) ve `trial_email_hash` `store`da KALIR
  (para yolu sorguları da kullanıyor) ve `_sq()` ile **çağrı anında** bağlanır.
  `store` bu modülü ithal edip adları yeniden dışa verir → eski `store.feed_health(...)`
  çağıranların hiçbiri değişmez.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_FEED_CRON` | `('openfda', 'clinicaltrials', 'dailymed')` | 624 |
| `_FEED_CADENCE_H` | `24` | 632 |
| `_FEED_AMBER_H` | `2 * _FEED_CADENCE_H` | 634 |
| `_FEED_RED_H` | `3 * _FEED_CADENCE_H` | 636 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_sq` | `()` | 44 | `store` modülüne GEÇ bağlanma — modül düzeyinde ithal DAİRESEL olurdu. |
| `recent_questions` | `(conn, limit: int = 200, only_gaps: bool = False)` | 52 | Soru listesi (admin görünürlüğü): SORU VE YANIT METNİ HAM gösterilir ve |
| `doctor_questions` | `(conn, doctor_id: int, limit: int = 20) -> dict` | 113 | Tek hekimin sorduğu SON sorular (detay kartı 2026-08-14 · `/admin/{did}/sorular` tam sayfa 2026-08-15, yanıt `a` ile). |
| `doctor_login_history` | `(conn, doctor_id: int, limit: int = 15) -> dict` | 152 | Tek hekimin giriş/çıkış geçmişi (app.login_event — KALICI; founder 2026-08-14: |
| `query_stat_summary` | `(conn, days: int = 30) -> dict` | 184 | Anonim soru istatistiği özeti (admin panosu): konu dağılımı + KB açıkları + kaynak-tipi kullanımı. |
| `ret_orani_ozet` | `(conn, days: int = 30) -> dict` | 214 | ANONİM ret oranı — kapsam kilidi kaç soruyu geri çevirdi (#C4, 2026-08-30). |
| `duvar_stat_ozet` | `(conn, days: int = 14) -> dict` | 262 | ANONİM duvar/fiyat sayacı özeti (#490b) — huninin PAYDASI. |
| `_butce_olaylari` | `() -> tuple` | 344 | `wall_stat`teki bütçe olay adları — `store._DUVAR_OLAYLAR`dan süzülür (TEK kaynak). |
| `butce_donem_ozet` | `(conn) -> dict` | 349 | DÖNEM BÜTÇESİ — hekim başına dönem içi API maliyeti, bütçeye oranı ve kademe (§7). |
| `returned_doctor_ids` | `(conn, hekimler) -> dict` | 436 | Bu hekimlerden hangileri AYNI ADRESLE DAHA ÖNCE gelmiş? (ÜÇ DURUMLU) |
| `_admin_safe` | `(conn, ad: str, bos: dict, sorgu)` | 486 | `query_stat_summary` (V9) hata sözleşmesinin ORTAK hâli — panel '0' ile 'ölçülemedi'yi |
| `panel_ay_ozeti` | `(conn, doctor_id: int) -> dict` | 510 | `/panel` "Bu ay" kartı — HEKİMİN KENDİ kullanımı. {soru, hasta, kredi, ok, error} |
| `acquisition_breakdown` | `(conn) -> dict` | 558 | Bu ayın kayıtları KAYNAK BAZINDA + her kaynağın hunide nereye kadar gittiği. |
| `feed_health` | `(conn) -> dict` | 638 | Bilgi tabanı besleme hattının TAZELİĞİ (core.sync_state + son core.ingest_log koşusu). |
| `feature_adoption` | `(conn) -> dict` | 715 | Hangi özellik GERÇEKTEN kullanılıyor: kayıt sayısı + kaç HEKİM + SON kullanım tarihi. |
| `activation_leak` | `(conn) -> dict` | 746 | Huninin nerede KAÇ KİŞİ kaybettiği: hiç soru sormamış hekim + açık oturumu olup sormayan. |
| `payment_attempts` | `(conn, days: int = 30) -> dict` | 814 | Son `days` gündeki iyzico provizyon DENEMELERİ — başarılı/başarısız, iç trafik AYRI. |
| `queue_counters` | `(conn) -> dict` | 863 | KUYRUK SAYAÇLARI — panelde hiç yüzeyi olmayan "bekleyen elle iş" ve suistimal-freni sayıları. |
| `citation_rate` | `(conn, days: int = 30) -> dict` | 906 | ATIF ORANI: kaç yanıt gerçekten kaynak taşıdı (mod bazında), app.message'dan ÖLÇÜLEREK. |
| `atif_kapsam` | `(conn, pencere: int = 300) -> dict` | 947 | SIFIR-ATIF: son N yanıtın kaçı **hiç** `[kaynak:kimlik]` atfı taşımıyor. |
| `_ortam_etiketi` | `() -> str` | 991 | 'prod' \| 'yerel' — kart hangi ortamda ÖLÇTÜĞÜNÜ söylesin. |

## `saglik/app/admin_reklam.py`

`561 satır` · `10 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
AYLIK REKLAM HARCAMASI — yıl+ay seçilerek girilen tutar (founder 2026-08-19:
"aylık reklam harcamalarını gireceğim bir yer olsun, yıl ve ay seçip fiyat yazabileceğim").

Birim ekonomi kartındaki **"bu ay GERÇEK nakit = tahsilat − API − reklam"** satırının
girdisi. ⚠ Reklam harcaması hâlâ ELLE girilir: otomatik toplayıcı (Meta Graph / Google Ads)
İSTEK YOLUNDA çağrılamaz — 90 sn'lik bir çağrı paneli kilitler ve hata hâlinde sessizce
"harcama 0"a düşer, yani kart yine KÂR gösterir (fail-open). Bu modül o kararı değiştirmez,
yalnız girdiyi **aya bağlar**.

⚠⚠ ANAHTAR BİÇİMİ `app.setting['ad_spend_YYYY-MM']`. LEGACY `ad_spend_month` AYRI BİR
  ŞEYDİR ve SİLİNMEDİ: 2026-08-19'a dek TEK alan vardı ve girilen tutarın HANGİ AYA ait
  olduğu kayıtlı DEĞİLDİ (panel onu her ay "bu ay" diye okuyordu). Bu yüzden:
    · yeni yazma DAİMA aylı anahtara gider,
    · cari ay yazılırken legacy anahtar da AYNALANIR (türev — kaynak değil),
    · cari ayın aylı kaydı YOKKEN legacy değer kullanılır ama ekranda **ay etiketsiz eski
      kayıt** diye İŞARETLENİR. İşareti kaldıran biri, hangi aya ait olduğu bilinmeyen bir
      sayıyı "bu ayın reklamı" diye ilan etmiş olur.
  ⚠ Aynalamayı kaldıracak olan `ad_spend_month` okuyan HER yeri taşımalı: panelin geçiş
    dalı + `admin_panel_verify` + `admin_pano_verify`.

⚠⚠ AY SINIRI TEK İFADEDEN (`ay_simdi`): "bu ay tahsilat" (`admin_dashboard`) ve "bu ay API"
  (`admin_stats`) sorguları `date_trunc('month', now())` kullanır — yani SUNUCU saat dilimi
  (prod'da UTC). Ay sınırını Python'da `date.today()` ile hesaplasaydık ayın ilk saatlerinde
  reklam gideri BİR AY KAYAR, nakit satırı sessizce yanlış çıkardı. Aynı ifadeden okumak
  ayrışmayı YAPISAL olarak imkânsız kılar.

⚠ İKİ SALT-OKUMA SORGUSU NEDEN BURADA, `store.py`de DEĞİL: `store.py` tavanında pay 16 satır
  ve **bir sonraki yükseltmesi panel sorgularının `admin_queries.py`ye taşınmasına bağlı**
  (`dosya_boyut_verify` girdisi); `admin_queries.py` de 773/773 (pay 0) ve zaten YAZAN
  fonksiyon kabul etmiyor. Yazma yolu yine `store.set_setting` üzerinden gider — yani para
  benzeri tek yazıcı korunur. O bölme yapıldığında bu iki okuma oraya taşınmalı.

⚠ Panel Türkçe tek-dilli → buradaki metni `EN_PAIRS`e EKLEME. Ay adları zaten orada bir
  çifte sahip (`>Ocak</option>` → `>Jan</option>`, doğum tarihi seçimi için) ve EN modunda
  yalnız ETİKET değişir; `value` sayısaldır, veri etkilenmez.
⚠ JS içine TÜRKÇE METİN YAZMA: `i18n_mw` EN modunda script bloklarını da çevirir ve düz
  kesme içeren bir EN karşılığı JS string'ini kapatıp sayfanın TÜM JS'ini çökertir
  (CLAUDE.md "apostrof tuzağı"). Aşağıdaki script yalnız sunucuda üretilmiş DEĞERLERİ taşır.

Kapı: `scratchpad/reklam_ay_verify.py` (biçim/aralık/ayna/geçiş dalı + inline `node --check`).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_ONEK` | `'ad_spend_'` | 53 |
| `_SQL_DESEN` | `'^ad_spend_[0-9]{4}-[0-9]{2}$'` | 57 |
| `_AYLAR_TR` | `('Ocak', 'Şubat', 'Mart', 'Nisan', 'Mayıs', 'Haziran', 'Temmuz', 'Ağustos', 'Eylül', 'Ekim', 'Kasım', 'Aralık')` | 58 |
| `_TAVAN` | `10000000` | 60 |
| `_LISTE` | `18` | 61 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_ay_gecerli` | `(ym: str) -> bool` | 64 | 'YYYY-MM' biçim + aralık (ay 01-12, yıl 2000-2100). |
| `_ay_adi` | `(ym: str) -> str` | 72 | '2026-08' → 'Ağustos 2026' (geçersizse ham değeri döndürür, uydurmaz). |
| `ay_simdi` | `(conn) -> str` | 80 | Cari ay ('YYYY-MM') — panelin 'bu ay' penceresiyle AYNI ifadeden (başlıktaki gerekçe). |
| `aylar` | `(conn) -> dict[str, float]` | 85 | Kayıtlı tüm aylar → TL. ⚠ Çözülemeyen değer ATLANIR (0 UYDURULMAZ: 'girilmedi' ile |
| `aylik_kz` | `(conn) -> dict` | 103 | AY AY KÂR/ZARAR girdisi: |
| `cari_tutar` | `(harita: dict[str, float], ym: str, legacy: str \| None) -> tuple[float, str]` | 288 | (tutar, kaynak) — kaynak: 'ay' (aylı kayıt) · 'legacy' (ay etiketsiz eski kayıt) · '' (yok). |
| `kaydet` | `(conn, yil: str, ay: str, tutar: str) -> str` | 300 | Formu işle → durum kodu ('ok:YYYY-MM' \| 'hata:ay' \| 'hata:tutar'). |
| `_durum_satiri` | `(mesaj: str) -> str` | 322 | `?reklam=` kodunu ekrana çevir. ⚠ Kod BEYAZ LİSTEDEN geçer — URL'den gelen serbest |
| `kz_tablosu` | `(harita: dict[str, float], kz: dict \| None, fx: float, simdi: str) -> str` | 341 | Ay ay + TOPLAM kâr/zarar tablosu (founder 2026-08-19: "toplam kâr zarar kısmını da ekle"). |
| `form_blogu` | `(harita: dict[str, float], simdi: str, mesaj: str = '', kz: dict \| None = None, fx: float = 0.0) -> str` | 515 | Yıl+ay seçimli giriş formu + AY AY KÂR/ZARAR tablosu (saf sunum; DB'ye gitmez). |

## `saglik/app/admin_routes.py`

`1682 satır` · `28 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMIN ROTALARI — kurucu paneli (Faz 2b, 2026-07-31).

`main.py`'den ayrıldı: 1.279 satır çıktı (6.595 → 5.316), bu dosya 1.388 satır.
Panelin TAMAMI burada: `/admin`, `/admin/kur`,
`/admin/reklam`, `/admin/feedback/{fid}`, `/admin/sorular`, `/admin/arsiv`,
`/admin/odeme`, `/admin/{did}` + 4 `/api/admin/*` ucu.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ. Etmesi de gerekmiyor: ölçüldü,
  admin kodunun main.py'de kalan TEK bağımlılığı `app` nesnesiydi ve o `kur(app)`
  ile karşılandı. main.py'den import etmek dairesel olur → uygulama AÇILMAZ.
  Paylaşılan sözlük (`esc`/`head`/`nav`/`_doctor`/token'lar) **webutil**'den gelir.

⚠⚠ NEDEN `@app.get` DEĞİL `@_rota` — VE NEDEN `include_router` DEĞİL:
  `app` burada yok (dairesel olurdu), o yüzden dekoratörler rotayı hemen kaydetmez;
  `_rota(...)` onu **KAYIT** listesine yazar, main.py `kur(app)` çağırınca gerçek
  kayıt `app.get/post` PUBLIC API'siyle yapılır.
  Doğal alternatif `APIRouter` + `app.include_router()` idi; DENENDİ ve GERİ ALINDI:
  **FastAPI 0.139.0'da `include_router` tembeldir** — rotaları `app.routes`'a
  düzleştirmez, tek bir `_IncludedRouter` sarmalayıcı bırakır (ölçüldü: `app.routes`
  yolları `['/docs','/redoc','_IncludedRouter']`, istekler yine 200). Ürün çalışır
  ama iki güvenlik ağı KÖRELİR: `refactor_parite.py` 12 rotayı "kayboldu" sanar ve
  `admin_panel_verify.py`'nin "admin rota kümesini UYGULAMADAN oku" freni boş kümeyle
  sessizce ölür. `kur(app)` bu ağların ikisini de olduğu gibi bırakır.
  ⚠ Bir üst FastAPI sürümüne geçilirse bu kararı yeniden ölçmeden `include_router`'a
  DÖNME — ölçüt "istek 200 dönüyor mu" değil, "`app.routes` düz mü".

⚠⚠ ROTA SIRASI DEĞİŞTİRİLEMEZ — `/admin/{did}` statik `/admin/<ad>` sayfalarından
  SONRA gelmeli. `did: int` converter'ı yüzünden `/admin/sorular`, `/admin/arsiv`,
  `/admin/odeme` bu rotanın ÜSTÜNDE tanımlı DEĞİLSE 422 döner. Sıra artık
  **KAYIT listesinin sırası** = bu dosyadaki tanım sırasıdır; yeni bir statik
  `/admin/<ad>` sayfasını `admin_detail_page`'in ÜSTÜNE koy.
  main.py `kur(app)`'ı bloğun ESKİ YERİNDE çağırır → app.routes sırası bölünmeden
  önceki hâliyle birebir aynıdır.

⚠ PANEL DİSİPLİNİ (bu dosyanın var oluş sebebi olan invariantlar — BOZMA):
  · **Hata durumunda POZİTİF HÜKÜM YASAK.** Sorgu patlarsa amber “Ölçülemedi”,
    asla yeşil “👍”. Yardımcılar: `_adm_ozet` / `_adm_hata` / `_adm_bos_r` / `_adm_n`.
  · **Sessiz sıfır basma** — “0” ile “ölçülemedi” ayrı şeylerdir.
  · **Sıfırın SEBEBİNİ iddia etme** — sorgu patlakken sebep bilinemez ve o cümle
    gerçek nedeni (kırık sorgu) aktif olarak eler.
  · **Sayaçlar AYRIK olmalı** — örtüşen iki rozet iş yükünü şişirip gerçek kuyruğu gizler.
  · **Gün sınırı Türkiye saati** (`store._TR_BUGUN`); `now()::date` Render'da UTC.
  · Panel destekleyemediği bir şey SÖYLEMEZ.

⚠ PANEL BİLEREK TÜRKÇE TEK-DİLLİ — buradaki metinleri `EN_PAIRS`'e EKLEME
  (global tablo + apostrof tuzağı riski; tek Türkçe kullanıcı için karşılığı yok).

⚠ Analytics: `/admin*` yolları `_ANALYTICS_PATH_ONLY` deseninde ve `_notrack_mw`
  admin sayfalarında etiketi tümden soyar (P0 kapandı). Yeni admin yolu eklersen
  desen onu zaten kapsar — deseni DARALTMA.

⚠ main.py bu 21 adı `# noqa: F401` ile YENİDEN DIŞA VERİR (fasad):
  `scratchpad/admin_panel_verify.py` `M._adm_hata` / `M._adm_n` / `M._adm_bos`
  üzerinden 8 iddia koşuyor.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KUR_BANT_ORAN` | `0.3` | 77 |
| `KAYIT` | `[]` | 90 |
| `MUTABAKAT_TAVAN` | `25` | 935 |
| `_MUT_ANAHTAR` | `'iyz_mutabakat_son'` | 939 |
| `_MUT_ALANLAR` | `('incelendi', 'verildi', 'bekliyor', 'basarisiz', 'hata', 'kalan', 'admin')` | 964 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_kur_bant_uygun` | `(onceki: float \| None, yeni: float, oran: float = KUR_BANT_ORAN) -> bool` | 80 | Saf: `yeni`, `onceki`nin ±oran bandında mı (onceki yoksa True). Kapı testi bunu sınar. |
| `_rota` | `(metod: str, yol: str, **kw)` | 93 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 104 | Admin rotalarını uygulamaya bağlar. main.py bunu bloğun ESKİ YERİNDE çağırır. |
| `_admin` | `(request: Request, conn)` | 115 | Yalnız role='admin' hekim döndürür (kurucu paneli koruması). Değilse None. |
| `admin_page` 🌐 | `(request: Request)` | 146 |  |
| `_kur_satiri` | `(kb: dict) -> str` | 614 | Birim ekonomi kartındaki KUR SATIRI (§7): değer + yaş → yeşil TAZE / amber BAYAT / KIRMIZI YOK. |
| `_kur_mesaj_html` | `(m: str) -> str` | 648 | `POST /admin/kur` sonucu (query `kur=`) → GÖRÜNÜR satır; boşsa "". Eski sürüm geçersiz |
| `admin_set_kur` 🌐 | `(request: Request, kur: str = Form(''))` | 670 | USD/TRY kurunu kaydet (yalnız admin). Sonuç `/admin?kur=…` ile GÖRÜNÜR (ok · gecersiz · |
| `admin_set_reklam` 🌐 | `(request: Request, tutar: str = Form(''), yil: str = Form(''), ay: str = Form(''))` | 690 | SEÇİLEN AYIN toplam reklam harcamasını kaydet (Meta + Google, elle) — birim-ekonomi |
| `admin_feedback_status` 🌐 | `(fid: int, request: Request, st: str = 'read')` | 706 | Geri bildirim durumu: new → read → done (yalnız admin). |
| `admin_questions` 🌐 | `(request: Request)` | 716 | Soru görünürlüğü — ⚠ FOUNDER KARARI 2026-08-14: HEKİM KİMLİĞİ GÖSTERİLİR (hekim adı |
| `admin_archive_page` 🌐 | `(request: Request)` | 830 | Silinen üyeler arşivi — kayıtlar kaybolmaz (kullanıcı kararı); snapshot özetiyle listelenir. |
| `_mutabakat_bekleyen` | `(conn) -> dict` | 942 | Hak-edilmemiş (granted=false) ödeme sayısı — ÜÇ DURUM sözleşmesiyle. |
| `_mutabakat_iz_yaz` | `(conn, ozet: dict) -> None` | 967 | Özeti `app.setting`e kompakt yaz (120 krk sınırı — yukarı bkz.). |
| `_mutabakat_son` | `(conn) -> dict` | 988 | Son çalıştırmanın özeti (app.setting) — yoksa boş sözlük, bozuksa ok=False. |
| `admin_mutabakat` 🌐 | `(request: Request)` | 1014 | Bekleyen iyzico ödemelerini mutabakata sok (hak-ediş + makbuz). YALNIZ ADMİN. |
| `admin_soru_analiz_page` 🌐 | `(request: Request)` | 1091 | Soru analizi panosu (#715b K3). Ham soru metni GÖSTERMEZ — o yüzey `/admin/sorular`. |
| `_soru_analiz_uret_sync` | `(request: Request)` | 1110 | Türetme batch'i — SENKRON gövde (threadpool'dan çağrılır). |
| `async` `admin_soru_analiz_uret` 🌐 | `(request: Request)` | 1135 | "Şimdi üret" düğmesi — gecelik iş gelene dek TEK çalıştırma yüzeyi (#715b K3). |
| `admin_payment_diag` 🌐 | `(request: Request)` | 1145 | iyzico ödeme teşhisi — son denemeleri listeler; hak-edilmemiş (granted=false) satırların |
| `admin_detail_page` 🌐 | `(did: int, request: Request)` | 1223 |  |
| `_admin_credit_card` | `(did: int, m: dict) -> str` | 1480 | Admin HEDİYE GÜN yükleme (comp) — erişim süresini UZATIR + hekime bildirim e-postası. |
| `_admin_edit_card` | `(did: int, m: dict) -> str` | 1523 | Üye bilgisi düzenleme kartı — e-posta LOGIN kimliği (benzersizlik sunucuda doğrulanır). |
| `_admin_delete_card` | `(did: int, m: dict, adm: dict) -> str` | 1555 | Arşivli silme kartı — kendi hesabı ve adminler için kilitli (guard sunucuda da zorlanır). |
| `api_admin_profile` 🌐 | `(did: int, request: Request, full_name: str = Form(''), email: str = Form(''), unvan: str = Form(''), specialty: str = Form(''), kurum: str = Form(''), phone: str = Form(''), is_test: str = Form(''), is_test_sent: str = Form(''))` | 1584 | ⚠ `is_test_sent` hidden alanı olmadan işaretlemeyi KALDIRMAK imkânsızdır: işaretsiz |
| `api_admin_delete` 🌐 | `(did: int, request: Request, reason: str = Form(''), confirm: str = Form(''))` | 1601 | Arşivli üye silme. Guard: kendi hesabı / admin silinemez; onay = üyenin e-postası. |
| `api_admin_update` 🌐 | `(did: int, request: Request, plan: str = Form(''), monthly_quota: str = Form(''), verification_status: str = Form(''))` | 1632 |  |
| `api_admin_grant_credit` 🌐 | `(did: int, request: Request, credits: str = Form(''), note: str = Form(''), nonce: str = Form(''))` | 1647 | Admin comp: hekimin SINIRSIZ ERİŞİM SÜRESİNİ uzatır + bildirim e-postası gönderir. |

## `saglik/app/admin_soru_analiz.py`

`465 satır` · `16 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
#715b K3 · PANEL — `/admin/soru-analiz` (SPEC: docs/soru-analitigi-spec-2026-09-04.md §2, §4).

Founder 2026-09-04: *"hangi branşlar neler soruyor. Bizim alameti farikamız bu."*
K1 (`app.answer_layer` · Cahit) ve K2 (`app.question_insight` · Derya, üretici
`saglik/app/soru_analiz.py`) katmanlarının ÜSTÜNDE duran SAYI YÜZEYİdir. Bu modül
SORGU + SAF SUNUM taşır; **rota tanımlamaz** (`admin_uyeler.py` deseni) — rota
`admin_routes.py`de ve `_admin()` kapısının ARKASINDADIR.

⚠⚠ **ADMİN/TEST HESAPLARI VARSAYILAN OLARAK HARİÇ — BU BİR TERCİH DEĞİL, ÖLÇÜLMÜŞ BİR
ÇARPIKLIĞIN FRENİDİR.** 2026-09-04'te prod'dan salt-okunur ölçüldü: 535 sorunun **%39,9'u
TEK BİR admin hesabından** (founder'ın kendi test hesabı, 214 soru), ilk 5 hesap %57.
Ayrımı yapmayan bir panel bize **kendi test sorularımızı** "Türk hekimi şunu soruyor" diye
okutur. Admin DAHİL görünüm ayrı ve ETİKETLİDİR (`?admin=1`), varsayılan değildir.

⚠⚠ **EŞİK ALTI HÜCREDE ORAN YAZILMAZ** (`ESIK_N`, tek yerde tanımlı ve panel metninde de
GÖRÜNÜR). Gerekçe ölçülebilir: aynı ölçümde ortanca hekim başına **2 soru** düştü; n<12'de
%50 dolayındaki bir oranın Wilson %95 aralığı **±25 puandan geniştir** — yani iki branşı
karşılaştıran her cümle gürültüyü okur. Oran basılabilen yerlerde **Wilson alt sınırı** da
yazılır (#116b dersi: kapsam hükmü nokta tahminle değil alt sınırla kurulur).

⚠⚠ **ÜÇ DURUM** (panel disiplini): sorgu patlarsa amber "Ölçülemedi" — ASLA yeşil, ASLA
sessiz "0". `question_insight` boşsa panel **"henüz üretilmedi"** der, "0 soru" DEMEZ:
ikisi bambaşka şeylerdir ve ikincisi ürün hakkında YANLIŞ bir olgu beyanıdır.
⚠ `kanit_n IS NULL` = "künye yok, ÖLÇEMEDİK" · `kanit_n = 0` = "paket BOŞTU". KB açığı oranı
YALNIZ ölçülmüş satırlar üzerinden hesaplanır ve paydası ekranda YAZILIR.

⚠ **HAM SORU METNİ BU SAYFADA GÖSTERİLMEZ.** O yüzey `/admin/sorular` ve ayrı bir founder
kararıdır (14.08); burası sayı yüzeyidir. Bu modülde `app.message.content` okuyan bir sorgu
YOKTUR ve eklenmemelidir (kapı `soru_analiz_panel_verify` HTML'de ham metin arar).

⚠ **DÜRÜSTLÜK SINIRI PANELİN KENDİ METNİNDE** (SPEC §4): iddia "Türk hekimi" DEĞİL
**"Klivance kullanıcısı"**; branş **beyandır**, doğrulanmadı (diploma no istenmiyor).
⚠ Gün sınırı TÜRKİYE saati (`store._tr_gun`) — `now()::date` Render'da UTC'dir.
⚠ Panel Türkçe TEK DİLLİ: buradaki metinler `EN_PAIRS`e EKLENMEZ.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ESIK_N` | `12` | 45 |
| `_TOP_KONU` | `8` | 46 |
| `_TOP_BRANS` | `12` | 47 |
| `_GUN` | `30` | 48 |
| `_FROM` | `'FROM app.question_insight qi JOIN app.message m ON m.id = qi.message_id JOIN app.thread t ON t.id = m.thread_id LEFT JO` | 56 |
| `_HARIC` | `"WHERE d.role IS DISTINCT FROM 'admin' "` | 60 |
| `_TUMU` | `'WHERE true '` | 61 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_nerede` | `(admin_dahil: bool) -> str` | 64 |  |
| `_wilson_alt` | `(k: int, n: int, z: float = 1.96) -> float \| None` | 68 | Oranın Wilson %95 ALT sınırı — nokta tahmin tek başına hüküm taşımaz (#116b). |
| `oran_hucre` | `(k, n) -> str` | 82 | ORAN HÜCRESİ — eşik altında SAYI BASILMAZ, "yetersiz" yazılır. |
| `veri` | `(conn, admin_dahil: bool = False) -> dict` | 100 | Panelin bütün sayıları. Her anahtar `ok`/`error` taşır (kart kendi hatasını basar). |
| `_kart` | `(baslik: str, ic: str, alt: str = '') -> str` | 190 |  |
| `_uretim_karti` | `(oz: dict) -> str` | 196 | ÜRETİM KARTI — "son adım bağlanmamış" frenidir: türetmeyi ÇAĞIRAN bir yüzey yoksa |
| `_uret_formu` | `() -> str` | 234 |  |
| `_kb_karti` | `(kb: dict) -> str` | 241 |  |
| `_konu_karti` | `(k: dict) -> str` | 259 |  |
| `_firsat_karti` | `(k: dict) -> str` | 273 | ÇOK SORULAN + AZ KAYNAKLI = Derya'nın iş sırası. |
| `_matris_karti` | `(m: dict, kb: dict) -> str` | 306 |  |
| `_zaman_karti` | `(z: dict) -> str` | 343 |  |
| `_katman_karti` | `(kat: dict) -> str` | 367 | GÜVENLİK KATMANI ÖZETİ — hangi katman kaç kez çalıştırılamadı. |
| `_sinir_bandi` | `(admin_dahil: bool, oz: dict) -> str` | 399 | DÜRÜSTLÜK SINIRI — panelin kendi metninde (SPEC §4), süs değil kabul şartı. |
| `_uret_bandi` | `(durum: str) -> str` | 422 | "Şimdi üret" düğmesinin SONUCU — sessiz başarısızlık YASAK. |
| `sayfa` | `(v: dict, admin_dahil: bool, uret_durum: str = '') -> str` | 440 | Sayfa gövdesi — SAF: `request` görmez, DB'ye gitmez, `_admin()` kapısı çağıranda. |

## `saglik/app/admin_sunum.py`

`488 satır` · `13 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMIN SUNUM YARDIMCILARI — rota TAŞIMAYAN, panelin "ne söyleyebilir" disiplini.

⚠⚠ NEDEN AYRI DOSYA (2026-08-09, Ömer'in ŞARTI · `_gorev.txt` #143b): `admin_routes.py`
  üçüncü kez tavan yükseltmesi istedi (1.420 → 1.455 → 1.606) ve dördüncüsü **bölme
  yapılmadan kabul edilmiyor**. ⚠ Gerekçe "tavanın altına inmek" DEĞİL — o çerçeve, bölme
  sayıyı yeterince düşürmediğinde bölmeyi GEREKSİZ gösterir. **Amaç sayıyı değil YÜZEYİ
  küçültmek:** buradakilerin hiçbiri rota tanımlamaz, DB'ye gitmez, istek nesnesi görmez.

BURADAKİ HER ŞEY TEK BİR KURALIN PARÇASI — **panel destekleyemediği bir şey SÖYLEMEZ**:
  · `_adm_hata` / `_adm_n` / `_adm_bos_r` / `_adm_ozet` — ÜÇ DURUM (değer / boş / ölçülemedi).
    Hata hâlinde ASLA olumlu hüküm; "0" ile "ölçülemedi" AYRI şeylerdir.
  · `_odeme_modu` / `_odeme_modu_bant` — "bu gerçek para mı?" sorusunun panel karşılığı.
    CANLI hükmü **BEYAZ LİSTEYLE** kurulur; boş/tanınmayan yapılandırma BİLİNMEYENDİR.
  · `_KAYDIRMA_IPUCU` — gizli içerik ancak ÖLÇÜLEREK işaretlenir (statik metin yanlış beyan).
  · `_tr_tutar` — TL yazımı; aynı panelde iki sayı biçimi olmasın diye TEK yerde.

⚠ Panel BİLEREK Türkçe tek-dillidir → buradaki metinleri `EN_PAIRS`e EKLEME.
⚠ Buraya ROTA ya da DB sorgusu EKLEME; eklenecekse `admin_routes.py`ye ait.
⚠ `admin_routes.py` bunların hepsini yeniden dışa aktarır (fasad) — mevcut çağıranlar
  (`main.py` fasadı, `admin_panel_verify`) DEĞİŞMEDEN çalışır.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_ADMIN_MOBIL_CSS` | `'<style>@media(max-width:640px){.content.admin .apx-table td[data-label="E-posta"]{flex-wrap:wrap;row-gap:6px;justify-co` | 54 |
| `_KAYDIRMA_IPUCU` | `'<script>(function(){function tara(){[].forEach.call(document.querySelectorAll(".klv-kaydir"),function(x){x.remove();});` | 64 |
| `_ADMIN_KUYRUK` | `_ADMIN_MOBIL_CSS + _KAYDIRMA_IPUCU` | 94 |
| `_IYZ_CANLI_UC` | `frozenset({'api.iyzipay.com'})` | 100 |
| `_SORU_KAPSAM` | `'Bu liste <b>sohbet kaydı üreten</b> turları gösterir (<code>app.message</code>). “Toplam soru” <b>başka bir kümeyi</b> ` | 322 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_odeme_modu` | `() -> dict` | 103 | Ödeme sağlayıcısının GERÇEK modu — `{ok, saglayici, canli, sebep}`. |
| `_odeme_modu_bant` | `() -> str` | 165 | Ödeme modu bandı — panonun en üstünde, para rozetlerinin YANINDA. |
| `_tr_tutar` | `(v: float, dec: int = 0, isaret: bool = False) -> str` | 185 | TL tutarı TÜRK yazımıyla: 2152 → '2.152' · -740 → '-740' · isaret=True → '+2.152'. |
| `_adm_hata` | `(res: dict, ne: str = '') -> str` | 196 | `ok=False` ise kartın başına amber 'ölçülemedi' bandı; aksi hâlde boş string. |
| `_adm_n` | `(res: dict, key: str, bos = '—')` | 207 | Sayıyı üç durumla bas: ölçülemedi→'?', 0/None→`bos`, aksi hâlde değer. |
| `_kesik_not` | `(gosterilen: int, toplam: int) -> str` | 217 | Liste LIMIT'e çarptıysa SÖYLE; çarpmadıysa (ve eşitlikte) SUS. |
| `_adm_bos` | `(mesaj: str, kolon: int = 3) -> str` | 231 | Boş tablo BIRAKMA: anlamlı boş-durum satırı (neyin yokluğu olduğunu söyler). |
| `_adm_bos_r` | `(res: dict, mesaj: str, kolon: int = 3) -> str` | 236 | `_adm_bos`'un HATA-FARKINDA hâli. |
| `_erisim_rozeti` | `(er: dict) -> str` | 257 | Hekimin GERÇEK erişimi — rozet. Kaynak YALNIZ `store.erisim_durumu` (üç durumlu). |
| `_soru_bos_satiri` | `(lifetime_q, kolon: int = 4) -> str` | 342 | Boş liste satırı — `lifetime_q > 0` iken "Henüz soru sormamış" ASLA basılmaz. |
| `_hekim_soru_karti` | `(did: int, res: dict, lifetime_q: int) -> str` | 361 | `/admin/{did}` "Sorduğu sorular (son 20)" kartı (2026-08-30'da buraya taşındı). |
| `_adm_ozet` | `(res: dict, renk: str, metin: str) -> str` | 393 | Kart başındaki VERDİKT satırı — hata durumunda ASLA olumlu/yeşil basmaz. |
| `_mutabakat_karti` | `(bek: dict, son: dict, tavan: int) -> str` | 413 | Mutabakat kartı: bekleyen sayısı + buton + son çalıştırma özeti (ÜÇ DURUM). |

## `saglik/app/admin_uyeler.py`

`307 satır` · `3 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ADMİN ÜYE LİSTESİ — sorgu · sıralama beyaz listesi · tablo gövdesi (bölme, 2026-09-03 · #670b).

⚠⚠ NEDEN AYRI MODÜL — BORÇ ÖDEMESİ, TERCİH DEĞİL: #669b (ödeyen satır + `?sirala=`) aynı
  özelliği ÜÇ dosyaya yaydı ve iki tavanı birden aşırdı (`admin_routes.py` 1.681/1.661,
  `store.py` 2.536/2.527). İki girdinin de kuralı "bölmesiz yükseltme YASAK" der; tavanı
  "CI kırmızıydı" diye yükseltmek gerekçe değildir. Üye listesi TEK özelliktir (sorgu +
  sıralama anahtarları + satır/başlık sunumu) → tek eve toplandı; iki dosya tavan ALTINA indi
  ve tavanları AŞAĞI çekildi (cırcır ters yöne). `admin_reklam.py`/`admin_alarm.py` sınıfı:
  özellik modülü, sorgu + sunum bir arada.

⚠ ÇAĞIRANLAR DEĞİŞMEDİ: `store.admin_list_doctors(...)` ve `store.ADMIN_SIRALAMA` **yeniden dışa
  verilir** (`admin_queries` / `admin_alarm` deseni; `store.py`nin SONUNDA). `admin_siralama_verify`,
  `admin_pano_verify`, `admin_pano_store_verify` aynı adla okumaya devam eder.

⚠ DAİRESEL İMPORT YOK — ZİNCİRİN YÖNÜ: bu modülü `store` ithal eder (yeniden dışa verme için)
  VE `admin_routes` ithal eder. `store`u modül düzeyinde ithal EDEMEZ (`_sq()` ile geç bağlanır,
  `admin_queries` deseni); `webutil`i de edemez — `webutil → store → admin_uyeler → webutil`
  kısmen başlatılmış modüle çarpar (`admin_alarm` vakasında ÖLÇÜLDÜ). Sunum yardımcıları bu
  yüzden `uye_tablosu` İÇİNDE ithal edilir. Bu bir stil tercihi değil, ölçülmüş bir kısıt.

⚠ `kur(app)` YOK ve OLMAMALI: rota tanımlamaz, `request`/`conn` görmez (`uye_tablosu` SAF SUNUM,
  sorgu `conn`u parametre olarak alır). Kabul ölçütü "200 döndü" DEĞİL — aynı DB + aynı hesapla
  ÖNCE/SONRA **bayt paritesi** (`scratchpad/_kamil_pano_parite.py`).

⚠ PANEL BİLEREK TÜRKÇE TEK-DİLLİ — buradaki metinleri `EN_PAIRS`e EKLEME.
⚠ ÜYE LİSTESİNDE ERİŞİM ROZETİ YOK (ölçülmüş karar) — gerekçe `.claude/agents/kamil.md`
  "ÜYE LİSTESİNDE ERİŞİM ROZETİ"; rozet hekim DETAYINDA (`admin_sunum._erisim_rozeti`).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ADMIN_SIRALAMA` | `{'kayit': 'd.created_at DESC', 'soru': 'lifetime_q DESC, d.created_at DESC', 'giris': 'last_active DESC NULLS LAST, d.cr` | 44 |
| `_SIRA_ETIKET` | `{'kayit': 'Kayıt tarihi', 'soru': 'Soru sayısı', 'giris': 'Son giriş', 'tur': 'Tur', 'hasta': 'Hasta sayısı', 'odedi': '` | 54 |
| `_ODEYEN_CSS` | `'<style>.content.admin .apx-table tr.odeyen td{font-family:Sora,Inter,sans-serif;color:#0B5F52;background:#F2FBF7}.conte` | 59 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_sq` | `()` | 36 | `store`a GEÇ bağlanma — modül düzeyinde ithal DAİRESEL olurdu (bkz. başlık). |
| `admin_list_doctors` | `(conn, q: str = '', limit: int = 300, sirala: str = 'kayit')` | 65 | Tüm hekimler + retention/aktivasyon sinyalleri. q: e-posta/ad arama. |
| `uye_tablosu` | `(rows, q: str, sirala: str, geri_gelen: dict) -> str` | 166 | `/admin` üye tablosu: köprü amber bandı + sıralama çubuğu + ödeyen stili + `#uyeler` tablo |

## `saglik/app/analiz_routes.py`

`316 satır` · `5 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ANALİZ ROTASI — dosya kütüphanesi map-reduce → TAM KLİNİK ÖZET (#635b/#642b).

ROTALAR (tek uç): POST /api/cases/{cid}/analyze — MAP (dosya başı Haiku, kotasız,
`extracted_text` cache) → REDUCE (ANALIZ_OZET_SYSTEM, tek ücretli sentez) → TAM özet
`app.case_timeline` arşivine (modele DÖNMEZ), ÇEKİRDEK ziyarete (modele bütün gider).

⚠⚠ FAZ KESİMİ (#642b, 02.09): `cases_routes.py` tavanının 132 üstündeydi; analiz bloğu
  buraya taşındı — tavan YÜKSELTİLMEDİ (aşımı gizlemek borcu görünmez yapar; bedeli
  7 günlük kör CI olarak bir kez ödendi). Desen diğer rota modülleriyle AYNI:
  `@_rota` + `kur(app)`; **`include_router` ÖLÇÜLEREK REDDEDİLDİ** (CLAUDE.md).
⚠ `cases_routes` bu modülden YALNIZ `_ANA_TAG_*` alır (Özet sekmesi kartının süzgeci);
  bu modül `cases_routes`u import ETMEZ — tek yön, dairesellik yok.
⚠ İNVARİANTLAR (taşınırken DEĞİŞMEDİ; kapılar: `analiz_ozet_verify` · `cases_api_verify`
  · `threadpool_kapisi`): ağır iş event loop'ta KOŞMAZ (run_in_threadpool) · sessiz
  kırpma YASAK (_atlama_notu: sayı+ad+sebep) · YanitKesildi'de usage YAZILIR ·
  `_erisim_402` → `_gunluk_fren_429` sırası · fail-closed bölünmüş akış.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 35 |
| `_ANA_TAG_TR` | `'[Dosya analizi'` | 54 |
| `_ANA_TAG_EN` | `'[File analysis'` | 55 |
| `MAX_ANALYZE_TOTAL_B` | `30 * 1024 * 1024` | 56 |
| `_ATLAMA_AD_LIMIT` | `10` | 58 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 38 | `cases_routes` ile AYNI desen: dekorasyon anında yalnız KAYDEDER. |
| `kur` | `(app) -> None` | 46 | Rotaları uygulamaya bağlar; main.py TEK KEZ çağırır (include_router YASAK). |
| `_atlama_notu` | `(lang: str, sayi: list, boyut: list, okunamayan: list, kisalan: list \| None = None, goruntuleme: list \| None = None) -> str` | 61 | Analize DAHİL EDİLMEYEN dosyaları SAYISI, ADI ve SEBEBİYLE hekime bildirir. |
| `async` `analyze_case` 🌐 | `(cid: int, request: Request)` | 127 | Seçili (veya tüm aktif) dosyaları MAP-REDUCE ile birlikte analiz et → ziyaret olarak kaydet. |
| `_analyze_case_sync` | `(cid, request, body, lang)` | 141 |  |

## `saglik/app/auth.py`

`648 satır` · `29 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Doktor kimlik doğrulama — Argon2id parola + sunucu-taraflı oturum.

Oturum cookie'si yalnız rastgele bir token taşır; doktor kimliği DB'de (app.session).
İptal edilebilir (logout = satırı sil), süreli (expires_at). JWT tarayıcıda tutulmaz.

⚠ TOKEN'LAR DB'DE ÖZETLİ (2026-07-29, denetim S4): oturum / e-posta doğrulama / parola
sıfırlama token'ları artık DB'ye SHA-256 ÖZETİ olarak yazılır; ham değer yalnız
kullanıcıya (cookie / e-posta linki) gider. Neden: SQL enjeksiyonu, yedek sızıntısı ya
da salt-okuma DB erişimi eskiden DOĞRUDAN oturum devralma veriyordu (token=cookie).
Özet tek yönlü olduğu için sızan satır artık kullanılamaz. Entropi zaten 256 bit
olduğundan salt/pepper gerekmez (kaba kuvvet imkânsız) — hızlı özet doğru seçim.
GEÇİŞ: eski ham token'lar okunmaya devam eder ve İLK kullanımda sessizce özete
yükseltilir → kimse çıkış yapmak zorunda kalmaz; 7 gün sonra ham token kalmaz.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `SESSION_TTL_DAYS` | `7` | 26 |
| `COOKIE_NAME` | `'klv_session'` | 27 |
| `RESET_TTL_HOURS` | `1` | 28 |
| `RESET_THROTTLE_SECONDS` | `60` | 29 |
| `COOKIE_SECURE` | `os.getenv('KLIVANCE_COOKIE_SECURE', '0') == '1'` | 31 |
| `_DUMMY_HASH` | `_ph.hash('klivance-sabit-zaman-kukla')` | 33 |
| `PW_MIN` | `6` | 57 |
| `_LE_UYARILDI` | `False` | 482 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `password_problem` | `(pw: str, *, email: str \| None = None, full_name: str \| None = None) -> str \| None` | 80 | Parola kabul edilebilir mi? Döner: None (kabul) ya da hata KODU. |
| `_tok_hash` | `(token: str) -> str` | 108 | Token'ın DB'de saklanan biçimi (SHA-256 hex). Ham token asla yazılmaz. |
| `_tok_lookup` | `(token: str) -> list[str]` | 113 | Aranacak değerler: [özet, ham]. Ham değer YALNIZ geçiş dönemi içindir (eski satırlar); |
| `_new_token` | `() -> str` | 119 | 256-bit rastgele token (URL-güvenli). |
| `hash_pw` | `(pw: str) -> str` | 125 |  |
| `verify_pw` | `(pw_hash: str, pw: str) -> bool` | 129 |  |
| `authenticate` | `(conn, email: str, password: str)` | 136 | Giriş — sabit-zamanlı. Kullanıcı yoksa da bir verify çalıştırır ki zamanlama |
| `set_session_cookie` | `(resp, token: str, *, gun: int \| None = None) -> None` | 147 |  |
| `clear_session_cookie` | `(resp) -> None` | 154 |  |
| `revoke_devices` | `(conn, doctor_id: int) -> int` | 159 | Bu hekime bağlı TÜM cihaz eşleşmelerini düşürür. Döner: silinen satır sayısı. |
| `purge_expired_sessions` | `(conn) -> None` | 192 | Süresi dolmuş oturumları toplu sil (fırsatçı temizlik; girişte çağrılır). |
| `trial_quota_for` | `(conn, email: str) -> int` | 199 | Bu e-postaya verilecek ÜCRETSİZ DENEME **GÜNÜ** (2026-07-29 founder kararı; birim |
| `create_doctor` | `(conn, email: str, pw: str, full_name: str, specialty: str = '', diploma_no: str = '') -> int` | 217 |  |
| `upsert_google_doctor` | `(conn, email: str, full_name: str = '') -> int` | 235 | Google ile giriş: e-posta ile eşleşen hekim varsa onu döndür, yoksa OLUŞTUR. |
| `get_doctor_by_email` | `(conn, email: str)` | 283 |  |
| `email_exists` | `(conn, email: str) -> bool` | 291 |  |
| `new_verify_token` | `(conn, doctor_id: int) -> str` | 295 | E-posta doğrulama token'ı üret + kaydet (verify_sent_at damgalar). Döner: HAM token |
| `verify_email_token` | `(conn, token: str) -> bool` | 306 | Token'ı doğrula → email_verified=true + token temizle. Döner: True/False. |
| `verify_email_token_ex` | `(conn, token: str) -> tuple[bool, str, str]` | 312 | verify_email_token'ın unvan+e-postayı da döndüren sürümü. |
| `new_reset_token` | `(conn, email: str) -> tuple[str \| None, int \| None]` | 339 | Parola sıfırlama token'ı üret. Döner: (ham_token, doctor_id) \| (None, None). |
| `reset_token_valid` | `(conn, token: str) -> bool` | 368 | Token hâlâ geçerli mi? (formu göstermeden önce; TÜKETMEZ.) |
| `consume_reset_token` | `(conn, token: str) -> int \| None` | 378 | Token'ı ATOMİK tüket (tek kullanımlık) → doctor_id \| None (geçersiz/süresi dolmuş). |
| `set_password_by_token` | `(conn, token: str, new_pw: str) -> tuple[bool, str \| None]` | 392 | Token ile parola belirle. Döner: (ok, hata_kodu). |
| `get_profile` | `(conn, doctor_id: int)` | 428 | Profil sayfası için tam hekim bilgisi. Döner: dict \| None. |
| `update_profile` | `(conn, doctor_id: int, full_name: str, unvan: str, specialty: str, kurum: str, phone: str) -> bool` | 445 | Hekim profil bilgilerini güncelle (parola/e-posta HARİÇ — onlar ayrı akış). |
| `change_password` | `(conn, doctor_id: int, old_pw: str, new_pw: str)` | 459 | Parola değiştir — mevcut parolayı doğrular. Döner: (ok, hata_kodu). |
| `create_session` | `(conn, doctor_id: int, *, denetim: bool = True, cihaz_id: int \| None = None, gun: int \| None = None) -> str` | 485 | Yeni oturum → HAM token döner (cookie'ye yazılır). DB'ye SHA-256 ÖZETİ yazılır (S4). |
| `doctor_for_token` | `(conn, token: str \| None)` | 540 | Geçerli oturumdaki doktoru döndürür (süresi dolmuşsa None). last_seen günceller. |
| `delete_session` | `(conn, token: str) -> None` | 632 | Çıkış — özet VE (geçiş dönemi) ham token satırını sil. |

## `saglik/app/auth_routes.py`

`1818 satır` · `52 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
KİMLİK / OTURUM ROTALARI — kayıt, giriş, çıkış, doğrulama, parola sıfırlama, onay
(Faz 3, 2026-07-31).

`main.py`'den ayrıldı. Kapsam: 16 rota + bu bölgeye ÖZEL yardımcılar —
`/giris` · `/kayit` · `/cikis` · `/dogrula/{token}` · `/api/verify/resend` ·
`/parola-sifirla[/{token}]` · `/onay` · `/auth/google[/callback]`.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ. Ölçüldü: bu kodun main.py'de
  kalan TEK bağımlılığı `app` nesnesiydi, o da `kur(app)` ile karşılandı. Paylaşılan
  sözlük (`esc`/`head`/`en`/`get_lang`/`_doctor`/`_rate_ok`/`_client_ip`/
  `_profil_eksik`) **webutil**'den gelir. main.py'den import etmek dairesel olur →
  uygulama AÇILMAZ.

⚠⚠ NEDEN `@app.get` DEĞİL `@_rota` — VE NEDEN `include_router` DEĞİL:
  `app` burada yok (dairesel olurdu), o yüzden dekoratörler rotayı hemen kaydetmez;
  `_rota(...)` onu **KAYIT** listesine yazar, main.py `kur(app)` çağırınca gerçek
  kayıt `app.get/post` PUBLIC API'siyle yapılır. `APIRouter` + `app.include_router()`
  Faz 2b'de DENENDİ ve ÖLÇÜLEREK geri alındı: bu FastAPI sürümünde tembeldir,
  rotaları `app.routes`a düzleştirmez (tek bir `_IncludedRouter` sarmalayıcı kalır) →
  ürün çalışır ama rota envanterini `app.routes`tan okuyan iki güvenlik ağı
  (`refactor_parite`, `admin_panel_verify`) sessizce körelir.
  ⚠ FastAPI yükseltilirse ÖLÇMEDEN dönme; ölçüt "istek 200 dönüyor mu" DEĞİL,
  `[r.path for r in app.routes]` listesinin DÜZ ve TAM olması.

⚠⚠ ROTA SIRASI = bu dosyadaki TANIM SIRASI (`_rota` çağrıldığı anda KAYIT'a yazar).
  `/parola-sifirla` statik rotası `/parola-sifirla/{token}`den ÖNCE kalmalı.
  main.py `kur(app)`'ı bloğun ESKİ YERİNDE (`root()`ün hemen altında) çağırır →
  `app.routes` sırası bölünmeden önceki hâliyle birebir aynıdır.

⚠ GÜVENLİK İNVARİANTLARI (bu dosyanın var oluş sebebi — BOZMA):
  · **Parola sıfırlama POST'u HER KOŞULDA AYNI yanıtı döner** (hesap numaralandırma
    freni: "hesap yok" ile "throttle" bilerek ayırt ettirilmez) · formu GÖSTERMEK
    token'ı tüketmez, yalnız kaydetmek tüketir · başarısız deneme token'ı YAKMAZ ·
    başarıda TÜM oturumlar düşer, bu yüzden **otomatik giriş YAPILMAZ** ·
    IP başına ek `_rate_ok('pwreset:<ip>', 5)`.
  · **`auth.create_session` `last_login_at`'i SAVEPOINT içinde yazar** — düz
    `try/except` Postgres'te transaction'ı zehirler ve oturum INSERT'i de düşer
    (yaşanmış P0: giriş sessizce çalışmaz olur).
  · **Ücretsiz deneme e-posta başına BİR KEZ** — kota `auth.trial_quota_for()` ile
    hesaplanır, `15` sabiti ELLE YAZILMAZ; Google kolu da aynı frene tabidir.
  · **Tipsiz SQL parametresi YASAK** (`%s::text`) — `CASE WHEN %s IS NOT NULL` biçimi
    `IndeterminateDatatype` verir, `except: pass` onu yutar ve AYNI transaction'daki
    doğrulama e-postası da sessizce düşer (yaşanmış P0).
  · **Üretim kodunda `consents` OTOMATİK DOLDURULMAZ** — hukuki onayı uydurmak olur.
  · ⚠⚠ **ZORUNLU ALAN KURALI `webutil._profil_eksik`TE; ORTAĞI `/kayit` DEĞİL `/onay`DIR
    (2026-08-05'te değişti — `/kayit`la kıyaslamak artık YANLIŞ ölçüt).** `/kayit` yalnız
    ad-soyad + e-posta + parola + 3 hukuki onay toplar. Kural ile `/onay`ın TOPLADIĞI alan
    kümesi ayrışırsa hesap kapıya çarpıp ÇIKAMAZ (sonsuz yönlendirme) ya da kapı hiç
    bloklamaz. ⚠ Formdan çıkmak alanı OPSİYONEL YAPMAZ, SORULDUĞU ANI değiştirir.
  · **`/onay` onay kutularını yalnız `consents` YOKKEN sorar** (GET ve POST AYNI koşulla
    dallanır). Form kaydından gelende consents zaten vardır; `save_consents`i ikinci kez
    çağırmak ts+IP damgasının ÜZERİNE yazar = onayın alındığı ANI kaybetmek.

⚠ ANALYTICS = ÖLÇÜM SÖZLEŞMESİ, üç olayın ayrımı BİLEREK kuruldu (BOZMA):
  · GA4 `sign_up` → `/chat?welcome=1` (Google Ads BİRİNCİL dönüşümü). Hayalet freni:
    hesap yaşı <30 dk + `localStorage klv_su`.
  · Meta `CompleteRegistration` → `email_verified` VE unvan ≠ 'Tıp Öğrencisi' iken,
    ⚠ İKİ yerde: `/dogrula/{token}` VE `chat_routes` `/chat?welcome=1`. (Burası
    "YALNIZ /dogrula" diyordu; #5'teki sızıntıyı üreten yanlış anlama buydu.)
  · Meta `klv_kayit` → `trackCustom`, FİLTRESİZ; CR'nin ateşlendiği HER yerde
    ateşlenmeli (KAPSAMA: `klv_kayit` ⊇ CR). Yeni kayıt yüzeyinde bu ayrımı KORU.

⚠ main.py bu adları `# noqa: F401` ile YENİDEN DIŞA VERİR (fasad): `_header_right`
  ve 10 doğrulama script'i (`gclid_verify`, `kayit_zorunlu_verify`, `_klv_profil`,
  `deneme_maliyet_olc` …) onları `main.<ad>` üzerinden okuyor.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_ML` | `f'minlength="{auth.PW_MIN}"'` | 83 |
| `KAYIT` | `[]` | 94 |
| `SIGNUP_MAX_PER_HOUR` | `5` | 120 |
| `_LOGIN_FAILS` | `{}` | 128 |
| `LOGIN_MAX` | `8` | 129 |
| `LOGIN_WINDOW` | `300` | 130 |
| `_HATA_BANDI_JS` | `'<script>(function(){var b=document.getElementById("form-hata");if(b&&b.focus)b.focus();})();</script>'` | 264 |
| `_STUDENT_JS` | `'<script>(function(){var u=document.getElementById("rg-unvan"),b=document.getElementById("rg-brans");if(!u\|\|!b)return;` | 317 |
| `_PW_MATCH_JS` | `'<script>(function(){var p=document.getElementById("rg-pw"),q=document.getElementById("rg-pw2");if(!p\|\|!q)return;funct` | 326 |
| `_MT_METIN` | `{'tr': {'baslik': 'E-posta tercihiniz', 'alt_cik': 'Bu tür bilgilendirme e-postalarını almayı durdurabilirsiniz.', 'buto` | 716 |
| `_DISPOSABLE_EMAIL_DOMAINS` | `{'mailinator.com', 'guerrillamail.com', 'guerrillamailblock.com', 'sharklasers.com', '10minutemail.com', 'temp-mail.org'` | 888 |
| `_ULKE_KODLARI` | `[('+90', 'Türkiye'), ('+1', 'ABD / Kanada'), ('+44', 'Birleşik Krallık'), ('+49', 'Almanya'), ('+971', 'BAE'), ('+966', ` | 921 |
| `_ULKE_KOD_KUMESI` | `{k for k, _ in _ULKE_KODLARI}` | 942 |
| `DOGUM_MIN_YAS` | `18` | 997 |
| `DOGUM_MAX_YAS` | `100` | 998 |
| `_AYLAR` | `[(1, 'Ocak'), (2, 'Şubat'), (3, 'Mart'), (4, 'Nisan'), (5, 'Mayıs'), (6, 'Haziran'), (7, 'Temmuz'), (8, 'Ağustos'), (9, ` | 1005 |
| `_DOGUM_JS` | `'<script>(function(){var g=document.getElementById("PFX-gun"),a=document.getElementById("PFX-ay"),y=document.getElementB` | 1098 |
| `_ULKE_EN_BILESIK` | `{'ABD / Kanada': 'USA / Canada', 'BAE': 'UAE', 'Rusya / Kazakistan': 'Russia / Kazakhstan', 'K. Makedonya': 'North Maced` | 1120 |
| `_ULKE_EN_ONBELLEK` | `None` | 1126 |
| `_PHONE_FMT_JS` | `'<script>function fmtPhone(el){var f=el.form;if(!f)return;var cc=(f.querySelector("[name=phone_cc]")\|\|{}).value\|\|"+9` | 1162 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 97 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 108 | Kimlik/oturum rotalarını uygulamaya bağlar. main.py bunu bloğun ESKİ YERİNDE çağırır. |
| `_login_keys` | `(ip: str, email: str)` | 132 |  |
| `_login_blocked` | `(ip: str, email: str) -> bool` | 135 |  |
| `_login_fail` | `(ip: str, email: str) -> None` | 144 |  |
| `_login_ok` | `(ip: str, email: str) -> None` | 149 |  |
| `_acq_cookies` | `(request)` | 153 | Kayıt anında saklanacak edinim verisi: (kaynak, gclid, gclid_zamanı). |
| `_marka_paneli` | `(lang = 'tr')` | 169 | `/kayit` masaüstü sahnesinin sol marka paneli (Deniz spec K-6/K-7, 2026-08-15). |
| `_consent_boxes` | `(terms: bool = False, kvkk: bool = False)` | 225 | 2 zorunlu kayıt onayı (koşullar+sorumluluk reddi, KVKK aydınlatma) — ts+IP ile |
| `_hata_bandi` | `(err: str) -> str` | 268 | Formun ÜSTÜNE basılan, odaklanabilir hata bandı. `err` boşsa HİÇBİR ŞEY basmaz. |
| `_kayitli_cikis` | `(lang: str) -> str` | 279 | «Bu e-posta zaten kayıtlı.» mesajının ÇIKIŞ YOLU eki — giriş + parola sıfırlama. |
| `esc_attr_eposta` | `(v: str) -> str` | 331 | #206b — ön-dolgu varsa `value="..."`, yoksa `autofocus`. Değer KAÇIRILIR. |
| `_auth_page` | `(kind, err = '', eposta = '', lang = 'tr', v = None, nxt = '', prg = False)` | 336 |  |
| `login_page` 🌐 | `(request: Request)` | 503 |  |
| `login` 🌐 | `(request: Request, email: str = Form(...), password: str = Form(...), next: str = Form(''), prg: str = Form(''))` | 534 |  |
| `register_page` 🌐 | `(request: Request)` | 568 |  |
| `_send_verify_email` | `(conn, doctor_id: int, email: str, lang: str) -> bool` | 571 | Doğrulama token'ı üret + maili gönder (best-effort; SMTP yoksa link log'a düşer). |
| `_reset_card` | `(title: str, sub: str, inner: str, msg: str = '', err: str = '') -> str` | 581 | Parola sıfırlama sayfalarının kabuğu — `_auth_page` ile AYNI görsel dil (.auth/.card). |
| `reset_request_page` 🌐 | `(request: Request)` | 604 |  |
| `reset_request` 🌐 | `(request: Request, email: str = Form(...))` | 620 |  |
| `reset_form_page` 🌐 | `(token: str, request: Request)` | 642 |  |
| `reset_submit` 🌐 | `(token: str, request: Request, password: str = Form(...), password2: str = Form(''))` | 665 |  |
| `_mt_dil` | `(request: Request) -> str` | 758 | Sayfa dili — `?l=` maildeki dili TAŞIR, yoksa `get_lang`. |
| `_mail_karti` | `(baslik: str, alt: str, inner: str, *, msg: str = '', err: str = '') -> str` | 768 | Mail tercihi sayfalarının kabuğu — `_reset_card` ile AYNI görsel dil. |
| `_mt_hesap_baglantisi` | `(t: dict) -> str` | 789 |  |
| `_mt_form` | `(token: str, lang: str, *, geri: bool) -> str` | 794 | Tek düğmelik POST formu. `geri=True` → yeniden abone ol dalı. |
| `mail_tercihi_page` 🌐 | `(token: str, request: Request)` | 808 | Onay sayfasını GÖSTERİR — HİÇBİR ŞEY UYGULAMAZ (bkz. bölüm başlığı). |
| `async` `mail_tercihi_uygula` 🌐 | `(token: str, request: Request)` | 830 | Tercihi UYGULAR. İdempotent; oturum/çerez istemez. |
| `_mail_tercihi_sync` | `(token: str, cik: bool)` | 869 | Jeton çöz + tercihi yaz — havuzda (M5). (did, yazildi) döner; istisna çağırana çıkar. |
| `_send_welcome_email` | `(email: str, name: str, lang: str) -> bool` | 877 | Yeni Google kaydına 'hesabın hazır' hoş-geldin maili (best-effort). Google akışı doğrulama |
| `_email_ok` | `(email: str) -> bool` | 898 | Kayıt e-postası: temel format + tek-kullanımlık domain reddi (farming freni). |
| `_norm_phone` | `(cc: str, national: str)` | 945 | (ülke kodu, ulusal numara) → kanonik E.164 (+<rakamlar>); geçersizse None. |
| `_norm_tr_phone` | `(phone: str)` | 981 | GERİYE DÖNÜK SARMALAYICI — tek alanlı eski çağrılar için (admin profil düzenleme). |
| `_yas` | `(d: _date, bugun: _date) -> int` | 1001 |  |
| `_dogum_yil_araligi` | `() -> tuple[int, int]` | 1009 | (en_eski_yıl, en_yeni_yıl) — `DOGUM_*_YAS`tan ÜRETİLİR, elle yazılmaz (bayatlar). |
| `_norm_dogum` | `(gun, ay, yil)` | 1015 | (gün, ay, yıl) → `date`; geçersiz/aralık dışıysa None. |
| `_dogum_alan` | `(pfx: str, secili = None, ham = None) -> str` | 1042 | Doğum tarihi — üç açılır liste (Gün / Ay / Yıl). |
| `_ulke_en_ad` | `(tr_ad: str) -> str` | 1129 | TR ülke adı → EN karşılığı; bulunamazsa TR adın KENDİSİ (asla boş/None). |
| `_ulke_opts` | `(secili: str = '+90', lang: str = 'tr') -> str` | 1142 |  |
| `register` 🌐 | `(request: Request, first_name: str = Form(''), last_name: str = Form(''), email: str = Form(...), password: str = Form(...), password2: str = Form(''), phone: str = Form(''), phone_cc: str = Form('+90'), dogum_gun: str = Form(''), dogum_ay: str = Form(''), dogum_yil: str = Form(''), ulke: str = Form(''), sehir: str = Form(''), sehir_diger: str = Form(''), unvan: str = Form(''), specialty: str = Form(''), c_terms: str = Form(''), c_kvkk: str = Form(''))` | 1172 |  |
| `_next_path` | `(nxt: str) -> str` | 1315 | Açık-yönlendirme freni: yalnız site-içi path kabul et. |
| `consent_page` 🌐 | `(request: Request, next: str = '/chat', err: str = '')` | 1332 | `/onay` GET rotası — İNCE SARMALAYICI, gövde `_onay_govde`de. |
| `_onay_govde` | `(request: Request, next: str = '/chat', err: str = '', v = None)` | 1344 | `/onay` gövdesi. `v` = 400 dönüşünde hekimin GÖNDERDİĞİ ham değerler (yoksa None). |
| `consent_submit` 🌐 | `(request: Request, next: str = '/chat', next_f: str = Form(''), phone: str = Form(''), phone_cc: str = Form('+90'), unvan: str = Form(''), specialty: str = Form(''), dogum_gun: str = Form(''), dogum_ay: str = Form(''), dogum_yil: str = Form(''), ulke: str = Form(''), sehir: str = Form(''), sehir_diger: str = Form(''), c_terms: str = Form(''), c_kvkk: str = Form(''))` | 1445 |  |
| `google_auth_start` 🌐 | `(request: Request)` | 1535 |  |
| `google_auth_callback` 🌐 | `(request: Request, code: str = '', state: str = '', error: str = '')` | 1559 |  |
| `_verify_done_html` | `(ok: bool, already: bool, en: bool, ogrenci: bool = False) -> str` | 1629 | #230b — doğrulama sonuç sayfasının GÖVDESİ. Sırsız `/dogrulandi`den çağrılır. |
| `verify_email` 🌐 | `(token: str, request: Request, response: Response)` | 1700 |  |
| `verify_done` 🌐 | `(request: Request, response: Response)` | 1739 | #230b — doğrulama SONUÇ sayfası. URL'de SIR YOK ⇒ TAM analytics taşır (`izsiz` ALMAZ). |
| `resend_verify` 🌐 | `(request: Request)` | 1761 |  |
| `logout` 🌐 | `(request: Request)` | 1791 | Çıkış — ⚠ M13: ARTIK POST. GET iken `<img src="…/cikis">` ile CSRF-logout yapılabiliyordu |
| `logout_get` 🌐 | `(request: Request)` | 1805 | Eski GET bağlantıları (yer imi, e-posta, dış link) için köprü: OTURUMU KAPATMAZ, |

## `saglik/app/billing.py`

`337 satır` · `11 fonksiyon` · `2 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Stripe abonelik entegrasyonu — Checkout + Webhook + Müşteri Portalı.

Doktor abone olur; fiyat `PLANS`ten okunur (⚠ 2026-08-09'a dek burada "$49/$490" YAZIYORDU
ve fiyat kararıyla bayatladı — modül docstring'i de bir gösterim yüzeyidir, SAYI YAZMA).
Klivance token maliyetini kendi Anthropic hesabından öder; doktordan API anahtarı
istenmez. Abonelik durumu app.doctor'da tutulur.

Anahtar .env'de: STRIPE_SECRET_KEY (zorunlu), STRIPE_WEBHOOK_SECRET (webhook için).
Fiyatlar Stripe'ta lookup_key ile idempotent oluşturulur (ensure_prices).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `PLANS` | `{'aylik': {'amount': 3500, 'interval': 'month', 'lookup': 'klivance_aylik_usd_v2', 'product': 'Klivance Hekim — Aylık', ` | 44 |

### `class StripeMissing(RuntimeError)`  <sub>satır 17</sub>

STRIPE_SECRET_KEY .env'de yok — arayüz zarifçe uyarır.

### `class WebhookRejected(RuntimeError)`  <sub>satır 21</sub>

Webhook doğrulanamadı (imza yok/secret yok/sağlayıcı Stripe değil) → 400.

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `plan_price_usd` | `(plan: str) -> str \| None` | 52 | Plan için GÖSTERİM fiyatı ('$35') — `PLANS`ten türetilir. Bilinmeyen/legacy plan |
| `_stripe` | `()` | 68 |  |
| `ensure_prices` | `() -> dict[str, str]` | 83 | Her plan için Stripe fiyatını bul (lookup_key) ya da oluştur. {plan: price_id}. |
| `ensure_customer` | `(conn, doctor: dict) -> str` | 102 | Doktorun Stripe müşteri kimliği (yoksa oluştur, DB'ye yaz). |
| `create_checkout` | `(conn, doctor: dict, plan: str) -> str` | 119 | Abonelik Checkout oturumu → ödeme sayfası URL'si döndürür. |
| `billing_portal` | `(conn, doctor: dict) -> str` | 161 | Aboneliği yönetme (iptal/kart) için Stripe Müşteri Portalı URL'si. |
| `purchase_facts` | `(sess, doctor_id, item_name: str) -> dict \| None` | 177 | Bir Stripe session'ından **DOĞRULANMIŞ** satın alım olgusu \| None. |
| `_field` | `(obj, key, default = None)` | 212 | StripeObject VEYA dict'ten güvenli alan okuma (stripe 15'te StripeObject.get() yok). |
| `_apply_subscription` | `(conn, sub) -> None` | 223 | Bir Stripe subscription nesnesini app.doctor'a uygula (durum + kota + plan). |
| `sync_checkout_session` | `(conn, session_id: str, doctor_id = None) -> dict \| None` | 262 | Başarı dönüşünde oturumu çekip aboneliği eşitle (webhook olmadan da çalışsın). |
| `handle_event` | `(conn, payload: bytes, sig_header: str \| None) -> dict` | 280 | Webhook olayını DOĞRULA ve işle. İmza doğrulaması ZORUNLU. |

## `saglik/app/butce_bant.py`

`91 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
`/chat` ÜST BANDI — DÖNEM BÜTÇESİ (#650b-EK, 2026-09-03). Kabuk BURADA, karar/metin `butce_kapisi`de.

Neden ayrı dosya: `butce_kapisi` kapı testi bölüm 7'de "string sabitlerinde rakam/TL/USD/% YOK"
(hekime giden metin) ve bölüm 8'de "tam 3 defter okuması, hepsi SAVEPOINT'te" diye taranır; CSS
(`#FCF3E4`, `44px`) ve dördüncü okuma o kapıyı kirletirdi. `chat_routes` ve `webutil` tavanlı.

SÖZLEŞME
· `butce_bant_js(conn, doctor_id, lang)` → `<script>` ya da "". Gözlemde (`credits.butce_modu()
  != "uygula"`) defter HİÇ OKUNMAZ (sayfa başına iki boş sorgu yakılmaz) ve "" döner — kesmeyen
  kapının bandı da olmaz ("kapalı" yazıp geçirmek yanlış beyandır, R8 yönü).
· Defter okuması `with conn.transaction():` (SAVEPOINT, R5): okuma patlarsa `chat_page`in
  bağlantısı zehirlenmez. Her istisna → "" (pazarlama/uyarı yüzeyi, KAPI DEĞİL — kapı `_butce_fren`).
· Kademe `bant_durumu` (burada) (rezervsiz; `ucusta`/`olculemedi` bandı yok), cümle
  `butce_kapisi._metin` (429 gövdesiyle birebir; iki yerde tutulsaydı biri bayatlardı).
· deneme `dolu` → CTA `/abonelik?src=butce` (`store._FIYAT_KAYNAK` beyaz listesinde VAR, 'diger'e
  düşmez); abone → CTA YOK (satılacak şey yok). `upgrade` anahtarı/diyaloğu YOK.
· Kabuk `chat_routes._erisim_banner_js` ile aynı: `#log` önüne, `textContent` (XSS yok),
  `btn btn-primary`, CTA min-height 44px (mobil dokunma tabanı). `id="butce-bant"` ölçüm çapası —
  kapı `butce_freni_verify` bölüm 12 gerçek sayfada bunu arar.
· Bant zinciri sırası `chat_page`te: tekrar-kayıt → duvar → BÜTÇE → e-posta doğrulama → geri sayım.
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `bant_durumu` | `(h) -> tuple[str, str, str \| None] \| None` | 31 | `/chat` üst bandı için kademe — `(kademe, durum, acilis)` ya da None (bant yok). |
| `butce_bant_js` | `(conn, doctor_id: int, lang: str) -> str` | 58 |  |

## `saglik/app/butce_defteri.py`

`207 satır` · `10 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Dönem bütçe kapısı — DEFTER katmanı (2026-09-02, Selim). Tasarım:
docs/maliyet-tavani-mimari-2026-09-02.md (§4 veri modeli, §14 R2/R5/R6/R10 bağlayıcı).

⚠ NEDEN AYRI MODÜL: `store.py` TAVANLI (`dosya_boyut_verify`, pay 1 satırdı) → bölme, deponun
`admin_queries`/`admin_alarm` deseni. ÇAĞIRAN DEĞİŞMEZ: `store.donem_harcama(conn, id)` diye okur
(store sonunda yeniden dışa verir). Bu modül `store`u ÇAĞRI ANINDA okur (`_st()`) — tepede import
dairesel olurdu; `credits` tepede güvenli (credits kimseyi import etmez).

SÖZLEŞME — bu katman SALT DEFTERDİR: karar vermez, TL bilmez, blok döndürmez, `webutil`i tanımaz.
· `donem_harcama(conn, doctor_id)` → {"durum","since","biter","usd"}  (TÜM modlar + uçuş-içi rezerv)
· `kur_bilgisi(conn)`             → {"kur","yas_gun","kaynak"}        (etkin kur: `credits.kur_eff`)
· `rezerv_koy / rezerv_kaldir / rezerv_temizle / ucusta_sayisi`  (R2 uçuş-içi rezerv, `app.usage`)
⚠ `admin_queries`nin "salt-okuma" kuralı BURADA GEÇERLİ DEĞİL — rezerv fonksiyonları YAZAR ve
  kendi commit'ini atar (satır başka bağlantıdan HEMEN görünmeli). SAVEPOINT içinde yazma, commit
  SAVEPOINT'ten SONRA (CLAUDE.md); bir `with conn.transaction():` bloğunun İÇİNDEN ÇAĞRILMAZ.
⚠ Sabitler `credits.py`de; sayı buraya KOPYALANMAZ.
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_st` | `()` | 26 | `store` geç bağlama (dairesel import freni) — `admin_queries._sq()` ile aynı desen. |
| `_rezerv_disi` | `(kolon: str = 'mode') -> str` | 32 | `rezerv_*` (uçuş-içi geçici) satırlarını DIŞLAYAN SQL predikatı — panel/gösterim |
| `donem_harcama` | `(conn, doctor_id: int) -> dict` | 45 | Hekimin BU DÖNEM (bütçe penceresi) içindeki toplam API maliyeti — bütçe kapısının defteri. |
| `_kur_parse` | `(cand) -> float \| None` | 108 | `webutil._usd_try` ile AYNI ayrıştırma kuralı (virgül → nokta; 1 ≤ v ≤ 1000), buraya |
| `kur_bilgisi` | `(conn) -> dict` | 121 | USD/TRY kuru ve YAŞI — bütçe kapısı için (`credits.kur_eff` etkin kura çevirir). |
| `_rezerv_mod` | `(yuzey: str) -> str` | 145 |  |
| `rezerv_koy` | `(conn, doctor_id: int, yuzey: str, usd: float) -> int \| None` | 150 | UÇUŞ-İÇİ REZERV (R2): kapı geçildiği an `app.usage`a `mode='rezerv_<yüzey>', usd=REZERV` |
| `rezerv_kaldir` | `(conn, rezerv_id: int \| None) -> bool` | 171 | Rezerv satırını sil (tur bitti; gerçek satır `record_usage` ile yazılır). Yalnız `rezerv_*` |
| `rezerv_temizle` | `(conn, dakika: int = REZERV_OMUR_DK) -> int` | 188 | `dakika`dan eski rezerv satırlarını sil (açılış + backstop). Döner: silinen sayısı. |
| `ucusta_sayisi` | `(conn, doctor_id: int, yuzey: str) -> int` | 200 | Hekimin bu yüzeyde şu an UÇUŞTA olan (ömrü dolmamış) rezerv sayısı — PAHALI_UCLAR için |

## `saglik/app/butce_kapisi.py`

`135 satır` · `6 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
DÖNEM BÜTÇE KAPISI — `_butce_fren(d, conn, lang, yuzey)` (founder 2026-09-02: "zarar etmeyelim").

Mimari ve gerekçe: `docs/maliyet-tavani-mimari-2026-09-02.md` (§3 akış · §5 sözleşme · §6 metinler ·
**§14 R1–R10 bağlayıcı revizyonlar**). Bu dosya webutil'in TAVANI yüzünden ayrı; çağıran
`webutil._butce_fren` diye okur (yeniden-dışa-verme), zincir:

    _erisim_402 (HAK, fail-closed 402) → _butce_fren (PARA, fail-closed 429) → _gunluk_fren_429 (HIZ, fail-open 429)

SÖZLEŞME
· Ölçüt = hekimin DÖNEM içi `sum(app.usage.usd)` (tüm modlar + uçuş-içi rezervler) + bu ucun REZERV
  üst sınırı ≥ `credits.BUTCE_TL[durum] / kur_eff`. Tur SAYMAZ (0,002–3,5 USD arası oynar).
· İki mod (`credits.butce_modu()`, env `KLIVANCE_BUTCE_MOD`): **gozlem** (varsayılan, ilk 7 gün) →
  hesaplar, `wall_stat`e `butce_gozlem_*` yazar, **HİÇBİR koşulda blok döndürmez (istisna dahil,
  R8)**; **uygula** → 429. Aynı env geri dönüş anahtarıdır.
· FAIL-CLOSED yalnız `uygula`da: defter/kur okunamazsa 429 `butce:"olculemedi"` — metin "bütçe doldu"
  DEMEZ ("doğrulanamadı"), admin `butce_olculemedi>0` kırmızı görür. Gerekçe: para yolunda hata = dur;
  `_erisim_402` zaten aynı bağlantıda fail-closed, ek maruziyet yalnız `app.usage`/`app.setting` arızası.
· DB okumaları `with conn.transaction():` (SAVEPOINT) içinde — okuma patlarsa conn ZEHİRLENMEZ (R5;
  havuz autocommit değil, zehirli conn sonraki her uçta 503/kayıp `record_usage` üretirdi).
· Kademe: `dolu` (≥%100, her yüzey) · `kismi` (≥%80, yalnız `PAHALI_UCLAR`, yalnız abone) · `ucusta`
  (PAHALI_UCLAR'da eşzamanlı ≤1 — R2; yarış penceresi: `record_usage` tur SONUNDA yazılır).
· Geçince (İKİ modda da, ölçüm için) `store.rezerv_koy` ile uçuş-içi rezerv satırı konur; kaldıran
  `store.record_usage` (billable satır yazılınca en eski rezervi siler) + `rezerv_temizle` backstop
  (açılış, `REZERV_OMUR_DK`). Rezerv kendi commit'iyle yazılır → bu kapı bir `conn.transaction()`
  bloğunun İÇİNDEN ÇAĞRILMAZ (kapı testi AST ile çiviler).
· Dönüş 429 gövdesi: `{"error", "butce": "dolu"|"kismi"|"olculemedi"|"ucusta", "acilis": ISO|None,
  "cta": "abonelik"|None}` — **`upgrade` anahtarı YAZILMAZ** (istemci onu satın-alma diyaloğuna
  dallar; abonede satılacak şey yok). Deneme `dolu` → `cta:"abonelik"` (istemci `butcePrompt`).
· Metinlerde TL/USD/yüzde/kur/sayı YOK (bayatlar, sayaç saydırır); yalnız `acilis` tarihi (DB'den).
  `dolu`/`olculemedi` metinleri "güvenlik araçları açık kalır" DEMEZ (R4: doz/kırmızı bayrak
  tamamlanmış bir Sor yanıtının ikinci gözüdür; Sor kapalıyken denetlenecek yanıt yoktur).
· Güvenlik uçları (dose_check/redflag/simplify/reçete kartı) bu kapıdan GEÇMEZ; maliyetleri
  toplama GİRER (founder: masraf = tüm API maliyeti).
```

</details>

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_say` | `(conn, olay: str, yuzey: str) -> None` | 43 |  |
| `_acilis` | `(h: dict) -> str \| None` | 50 |  |
| `_tarih` | `(acilis: str \| None, lang: str) -> str` | 58 |  |
| `_metin` | `(kademe: str, durum: str, lang: str, acilis: str \| None) -> tuple[str, str \| None]` | 64 | (metin, cta). Sayı/para birimi YOK; TR/EN sunucuda (EN_PAIRS'e girmez, kıvrık kesme). |
| `_429` | `(kademe: str, durum: str, lang: str, acilis: str \| None) -> JSONResponse` | 91 |  |
| `_butce_fren` | `(d, conn, lang, yuzey: str = '')` | 96 | Dönem bütçe kapısı — aşıldıysa 429 (yalnız `uygula`), aksi None. Sözleşme modül başlığında. |

## `saglik/app/cal_body.py`

`363 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Takvim sayfa gövdesi (HTML + JS).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-07-29, mimari denetimi). İçerik
AYNEN taşındı — tek karakter değişmedi; render çıktısı bayt bayt doğrulandı.
main.py bu adları yeniden dışa aktarır, yani mevcut importlar (`from saglik.app.main
import ...`) ve doğrulama scriptleri KIRILMAZ.

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_CAL_BODY` | `'\n<style>\n .cal-wrap{max-width:1120px;margin:0 auto}\n .cal-head{display:flex;align-items:center;justify-content:space` | 13 |

## `saglik/app/cal_routes.py`

`332 satır` · `18 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
TAKVİM — randevu + klinik takip yüzeyi (Faz 9, 2026-08-20).

`main.py`den ayrıldı. Kapsam: 6 rota —
`/takvim` · `GET/POST /api/calendar` · `PATCH/DELETE /api/calendar/{aid}` ·
`POST /api/calendar/{aid}/done`.

⚠⚠ BU MODÜL `main.py`DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ).
  Paylaşılan sözlük `webutil`den DOĞRUDAN okunur (main fasadından DEĞİL), gövde
  `cal_body`den. main.py adları FASAD olarak yeniden dışa verir (dış okuyucu ölçüldü: 0).

⚠⚠ `include_router` KULLANILMAZ — bu FastAPI sürümünde TEMBEL: rotaları `app.routes`a
  düzleştirmiyor (ölçüldü Faz 2b: 95 rota → 83). Rota envanterini `app.routes`tan okuyan
  güvenlik ağları (`refactor_parite`, `admin_panel_verify`) sessizce körelir. Bunun yerine
  `@_rota` KAYIT listesi + `kur(app)` → `app.get/post/patch/delete` public API'si.

⚠ `kur(app)` ÇAĞRISININ YERİ SÖZLEŞMENİN PARÇASI: main.py'de blok tam nerede duruyorsa
  oraya konur — hemen ardından `_ref_kur(app)` gelir. Taşıma paritesi HEAD ile
  (methods, path, endpoint adı) SIRALI birebir ölçüldü (sayı `takvim_verify`/rapor).

⚠ NEDEN AYRILDI: main.py tavana (`dosya_boyut_verify`) 8 satır kala takvim denetiminin
  düzeltmeleri sığmıyordu. Taşıma + aynı turda uygulanan düzeltmeler aşağıda.

────────────────────────────────────────────────────────────────────────────────
BU DOSYADAKİ İNVARİANTLAR — ölçülmüş hatalardan doğdu, BOZMA
────────────────────────────────────────────────────────────────────────────────
⚠⚠ DUVAR-SAATİ (memory `klivance-takvim-saat-modeli`): `_appt_json` OFSETSİZ ISO döner,
  `_parse_dt` NAİVE döner, karşılaştırmalar naive TR duvar-saatiyle yapılır. `+03` sabiti
  YOK; instant (`astimezone`) çevrimi YOK. Bozulursa ızgara 09:00 / yan panel 12:00 der.

⚠⚠ "BUGÜN" SINIRI TEK KAYNAKTAN: `tr_saat.tr_gun_basi()` (Türkiye gün başı, naive; SAF modül).
  Eskiden burada çıplak sunucu-saati (`now()`) vardı → Render UTC'de TR 00:00–03:00 arası
  /api/calendar DÜNÜ "bugün" sayıyor, /panel doğru sayıyordu (aynı takibe iki hüküm).
  ⚠ Çağrılar `_ts.tr_simdi()` / `_ts.tr_gun_basi()` biçiminde MODÜL ÜZERİNDEN yapılır
    ki testin TEK monkeypatch noktası (`tr_saat.tr_simdi`) buraya da işlesin — ada bağlı
    `from .tr_saat import tr_simdi` patch'i GÖRMEZDİ.

⚠⚠ 401 GÖVDESİ BOŞ LİSTEDEN AYRIŞIR: `{"events": [], "error": "auth"}`. Eski gövde
  `{"events": []}` idi → oturum düşünce takvim "boş" görünüyor, gecikmiş INR kutusu
  sessizce gizleniyordu (fail-open). İstemci (`klvCalFetch`, cal_body) 401'i /giris'e çevirir.

⚠⚠ SAHTE "ok" YOK: done/DELETE satır bulamazsa (başkasının id'si = IDOR, ya da silinmiş)
  **404**, `{"ok": true}` DEĞİL. IDOR WHERE'leri store'da (`AND doctor_id=%s`) — DOKUNMA.

⚠ KİMLİK ÖNCE, GÖVDE SONRA: anonim bozuk gövde 401 alır (500 değil); gövde `_json_body`
  ile okunur (JSON değilse / sözlük değilse `{}`) → girişli bozuk gövde 400 alır.

⚠ YAZMA TARİH KAPISI: `2000-01-01 ≤ at ≤ TR şimdi + 5 yıl`, YALNIZ POST/PATCH. '0226'
  yazım hatası ızgarada asla görünmeyen kayıt üretiyordu. OKUMA etkilenmez, mevcut satıra
  dokunulmaz (şema/CHECK EKLENMEZ — prod verisi ölçülmeden additive değil).

⚠ SAHİPSİZ `case_id` → 400 'Hasta bulunamadı.' — store `ValueError('case')` fırlatır,
  INSERT/UPDATE YAPILMAZ. Eskiden sessizce `case_id=None` ile yazılıyordu (hasta bağı
  hekime söylenmeden düşüyordu). Sayıya çevrilemeyen ya da ≤0 (`0`/`"0"`) `case_id` de aynı 400'dür.

⚠ `kind` beyaz liste (`randevu`/`takip`) → 400; `title` str değilse 400; `done` yalnız
  bool → 400. store'daki sessiz 'randevu' düşüşü İKİNCİ savunma olarak durur.
  `kind='randevu'`ya çevrilen tamamlanmış takip store'da `done=false`a döner (çizik kalmasın).

⚠ `has_more`: liste `limit+1` çekilip `limit`e kırpılır; yanıt ADDITIVE alan taşır
  (`{"events": [...], "has_more": bool}`; ızgara dalında `false`). Eskiden 60/30'da sessiz
  kesiliyordu ve hekim "hepsi bu" sanıyordu. Tavanlar `store.UPCOMING_LIMIT/OVERDUE_LIMIT`.

⚠ Hata metinleri SUNUCUDA `lang` ile (API JSON'u `i18n_mw`den GEÇMEZ; EN_PAIRS'e girmez).
⚠ Rotalar `async def` kalır (threadpool 'sonra' kalemi). LLM/kredi YOK.
Ağ: `scratchpad/takvim_verify.py` (kapı ajanı, aynı tur).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 84 |
| `_KINDS` | `('randevu', 'takip')` | 108 |
| `_AT_ALT` | `_dt.datetime(2000, 1, 1)` | 109 |
| `_AT_ILERI_YIL` | `5` | 110 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 87 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 98 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_parse_dt` | `(s)` | 113 | ISO 'YYYY-MM-DDTHH:MM' → naive datetime. Geçersiz/str-değil → None. |
| `_at_ust` | `()` | 123 |  |
| `_at_araliginda` | `(at) -> bool` | 131 | YAZMA tarih kapısı (okuma etkilenmez): 2000-01-01 ≤ at ≤ TR şimdi + 5 yıl. |
| `_hata` | `(lang, tr, en_, kod = 400)` | 136 |  |
| `_auth_401` | `()` | 140 |  |
| `_case_id_coz` | `(v)` | 146 | `case_id` gövde değeri → int \| None. Boş/None → None; çevrilemeyen ya da ≤0 → ValueError('case'). |
| `_appt_json` | `(r)` | 163 |  |
| `takvim_page` 🌐 | `(request: Request)` | 179 |  |
| `api_calendar_list` 🌐 | `(request: Request, start: str = '', end: str = '', upcoming: str = '', overdue: str = '')` | 198 |  |
| `async` `api_calendar_create` 🌐 | `(request: Request)` | 223 |  |
| `_calendar_create_sync` | `(request: Request, body: dict, lang: str)` | 230 |  |
| `async` `api_calendar_update` 🌐 | `(aid: int, request: Request)` | 255 | Etkinliği düzenle (kısmi): {at?, kind?, title?, case_id?}. |
| `_calendar_update_sync` | `(request: Request, aid: int, body: dict, lang: str)` | 262 |  |
| `async` `api_calendar_done` 🌐 | `(aid: int, request: Request)` | 302 |  |
| `_calendar_done_sync` | `(request: Request, aid: int, body: dict, lang: str)` | 307 |  |
| `api_calendar_delete` 🌐 | `(aid: int, request: Request)` | 322 |  |

## `saglik/app/calc_body.py`

`2341 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Klinik hesaplayıcılar sayfası gövdesi (istemci-tarafı, kredi harcamaz).

Tüm UI CALC_LANG ile JS tarafında iki dilli render edilir → i18n_mw'den MUAF tutulur
(main.py i18n_mw path atlar), böylece EN_PAIRS JS'i bozamaz (apostrof tuzağı yok).
Formüller scratchpad/calc_verify.js ile bilinen referans değerlere karşı DOĞRULANDI.
⚠ Tıbbi güvenlik: her hesaplayıcıda formül adı+kaynak + "karar-destek, tanı koymaz" notu var.

⚠⚠ SUNUCU-TARAFI İÇERİK (2026-07-27, ölçülmüş Google kalite skoru sorunu):
Sayfa 207 KB HTML üretiyordu ama <script> blokları çıkarılınca GÖRÜNÜR METİN 86 KARAKTERDİ
(yalnız menü) — h1 boş, 'GFR/kreatinin/QTc/MELD' kelimelerinin HİÇBİRİ yok. Google
"gfr hesaplama" sorgusuyla inen sayfada o kelimeyi bulamadığı için `post_click_quality_score
= BELOW_AVERAGE` verdi (QS=1) → CPC şişti (17,57 TL). Ayrıca sayfa organik aramaya da görünmez.
ÇÖZÜM: `calc_static_index(lang)` CALCS/CATS'ı BU DOSYANIN KENDİ KAYNAĞINDAN parse edip
sunucu-tarafı bir dizin HTML'i üretir (tek-doğru-kaynak korunur: yeni hesaplayıcı eklenince
otomatik yansır). JS yüklenince `render()` `#calc-root`'u zaten EZER → kullanıcı deneyimi
DEĞİŞMEZ, tarayıcı/robot ise gerçek içerik görür (progressive enhancement).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `CALC_BODY` | `'\n<style>\n /* ══ v9 ARAÇLAR TASARIMI — founder kanvası `scratchpad/_dc_6-hesaplayicilar.html`\n    (uygulama 2026-08-1` | 63 |
| `_ADIL_TR` | `_adil('tr').replace("'", '’')` | 2329 |
| `_ADIL_EN` | `_adil('en').replace("'", '’')` | 2330 |
| `CALC_BODY` | `CALC_BODY.replace('Kayıttan sonra {DILAN}.', f'Kayıttan sonra {_ilan('tr')}. {_ADIL_TR}').replace('After sign-up: {DILAN` | 2333 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_parse_calcs` | `()` | 22 | CALC_BODY kaynağından (id, cat, tr_ad, en_ad) üçlülerini ve kategori adlarını çıkar. |
| `calc_static_index` | `(lang: str = 'tr') -> str` | 37 | Sunucu-tarafı hesaplayıcı dizini (JS yüklenince ezilir). Boş dönerse sayfa eskisi gibi çalışır. |

## `saglik/app/cases_body.py`

`410 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
/cases sayfa gövdeleri (HTML + CSS + JS şablonları).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-08-01, Faz 4). İçerik AYNEN taşındı —
tek karakter değişmedi; `/cases` ve `/cases/{cid}` render çıktısı TR+EN bayt bayt
doğrulandı (`scratchpad/cases_govde_verify.py`). main.py bu adları yeniden dışa aktarır.

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı.

⚠ NEDEN yalnız 5 blok taşındı (704 satırın tamamı değil): `case_detail_page` gövdesi
71 düz string ile 70 f-string'i İÇ İÇE örer. Python `'A' f'B{x}'` yazımını tek bir
`JoinedStr`e katlar → o parçalar buraya ALINAMAZ (kaynak aralığını isimle değiştirmek
f-string'i ortadan böler). Ölçüldü (`scratchpad/faz4_blok_olc.py`): f-string DIŞI ve
≥10 satırlık bloklar = 5 blok / 219 satır. Eşiği 5'e indirmek 2 blok karşılığı yalnız
12 satır getiriyordu (f-string'lerle örülü, okunabilirlik zararı kazançtan büyük);
15'e çıkarmak zaman tüneli JS'ini geride bırakıyordu.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_CASES_NEW_FORM_A` | `'<form method="post" action="/api/cases"><div class="field-grid" style="grid-template-columns:2fr 1fr 1fr;gap:12px"><div` | 43 |
| `_CASES_NEW_FORM_B` | `'<details class="newacc"><summary>Klinik profil <span class="subhint">— doz &amp; etkileşim (kronik · ilaç · lab)</span>` | 56 |
| `_CASE_LIB_JS` | `'\n(function(){\n  var LIB_TXT = __LIB_JSON__;\n  var LIB_MAX_FILES = __MAX_FILES__;\n  var LIB_MAX_B = __MAX_B__;\n  va` | 89 |
| `_CASE_TIMELINE_JS` | `'\nfunction _mdLite(t){var d=document.createElement(\'div\');d.textContent=t\|\|\'\';var h=d.innerHTML;var G=[];try{G=__` | 234 |
| `_CASE_TABS_CSS` | `'<style>.tabs-ready .tabp{display:none}.tabs-ready .tabp.on{display:block}.subnav.tabs a{cursor:pointer}.tab-note{font-s` | 259 |
| `_CASE_TABS_JS` | `'\n(function(){\n var w=document.getElementById("case-tabs");\n var wrap=document.getElementById("case-tabwrap");\n var ` | 277 |
| `_CASE_ACC_JS` | `'<script>(function(){var K="klv_acc___ID__";var st={};\ntry{st=JSON.parse(localStorage.getItem(K)\|\|"{}")}catch(e){}\nv` | 321 |
| `_CASE_VISIT_JS` | `'<script>function saveAndAnalyze(btn){\nvar ta=document.getElementById("visitNote");var note=(ta.value\|\|"").trim();\ni` | 337 |
| `_CASE_ACC_CSS` | `'<style>.acc{border:1px solid var(--line);border-radius:var(--r-card);background:#fff;margin-top:14px;overflow:hidden}.a` | 355 |
| `_CASE_ZEMIN_CSS` | `'<style>.content{background-color:var(--mist)}</style>'` | 380 |
| `_ENABIZ_AKTAR_KARTI` | `'<div class="card" style="margin-top:14px"><h2 style=\'font-size:15px;margin:0 0 6px\'>e-Nabız\'dan aktar</h2><p class="` | 397 |

## `saglik/app/cases_routes.py`

`1500 satır` · `21 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
PSEUDONİM HASTA DEFTERİ + ŞİFRELİ DOSYA KÜTÜPHANESİ (Faz 7, 2026-08-01).

`main.py`den ayrıldı. Kapsam: 9 rota —
`/cases` · `/cases/{cid}` · `/api/cases` · `/api/cases/{cid}/update` ·
`/api/cases/{cid}/delete` · `/api/cases/{cid}/visit` · `/api/cases/{cid}/files` ·
`/api/cases/{cid}/files/{fid}` · `/api/cases/{cid}/files/{fid}/delete`.
⚠ `/api/cases/{cid}/analyze` BU DOSYADA DEĞİL — #642b faz kesiminde `analiz_routes.py`ye
  taşındı; bu satır o taşımadan sonra da eski listeyi tekrarlıyordu (03.09'da düzeltildi).
⚠⚠ `/api/cases/{cid}/delete` (03.09) HASTA KARTINI KALICI SİLER — CASCADE ziyaret/dosya/
  zaman tünelini de götürür, randevular elle silinir, sohbetler SİLİNMEZ (bağı kopar).
  Sözleşmenin tamamı `store.delete_case` başlığında; kapı `cases_api_verify` bölüm 15.

⚠⚠ BU MODÜL `main.py`DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ).
  Ölçüldü (`scratchpad/faz7_kapsam_olc.py`): taşınan kodun main.py'de kalan tek
  bağımlılığı `app` nesnesiydi → `kur(app)`. webutil'e YENİ ad eklenmedi; bölgeye özel
  16 yardımcının tamamı buraya geldi (`_sniff_mime`, `_human_size`, `_sex_select`,
  `_build_allergies`, `_allergy_widget_new`, `_analyze_case_sync`…). ⚠ O 16'dan biri olan
  yaş bandı <select> yardımcısı 2026-09-05'te silindi (giriş kaldırıldı) — ad artık YOK.

⚠⚠ NEDEN `@app.post` DEĞİL `@_rota` — VE NEDEN `include_router` DEĞİL:
  `app` burada yok; `_rota(...)` rotayı KAYIT listesine yazar, main.py `kur(app)`
  çağırınca gerçek kayıt `app.get/post` PUBLIC API'siyle yapılır → `app.routes` DÜZ
  kalır. `APIRouter` + `include_router` Faz 2b'de DENENDİ ve ÖLÇÜLEREK geri alındı:
  bu FastAPI sürümünde tembeldir, rotaları `app.routes`a düzleştirmez → rota
  envanterini oradan okuyan güvenlik ağları sessizce körelir.
⚠ `kur(app)` çağrısının YERİ sözleşmenin parçası: bloğun main.py'deki ESKİ yerinde
  durur, böylece `app.routes` SIRASI kesim öncesiyle birebir aynı kalır (ölçüldü:
  hasta rotaları 56-64, `/api/cases/{cid}/timeline` 43'te ve chat_routes'tan gelir).

⚠⚠ KVKK + HASTA VERİSİ İNVARİANTLARI (bu dosyanın en pahalı bölgesi — BOZMA):
  · ⚠⚠ **PSEUDONİM ZORUNLULUĞU KALKTI (2026-08-05, founder kararı + avukat onayı).**
    Hekim hasta defterine **GERÇEK AD GİREBİLİR**; takma kod İSTEĞE BAĞLI. Bu
    dosyadaki "kimlik yazMAyın / kimlik içermez" dili ve `cases_body`deki ad-soyad
    uyarı script'i aynı gün kaldırıldı (ürünün yaptığının TERSİNİ söylüyorlardı).
    Metinler yanlış değildi, **altındaki KURAL değişti** — kural dönerse dönmeli.
    ⚠⚠ TEKNİK GERÇEK (kullanıcıya söylenen metinden AYRI — ikisini karıştırma): gerçek ad
    girilen kayıt KVKK m.6 **özel nitelikli veri**dir ve `analyze`/`/chat` yolundan
    **yurt dışındaki model sağlayıcısına aktarılır**. Bu OLGU duruyor; **ifşası founder
    kararıyla metinlerden çıkarıldı** (2026-08-05, `_gorev.txt` #119b) → yeni bir dışa
    aktarım eklerken olguyu VARSAY, ifşa kararını Selim'e/founder'a sor, kendi başına
    METNE YAZMA. ⚠ `name="code"`/`case_note.code` DEĞİŞMEDİ; ETİKET ise 2026-08-05'te HASTA ADI SORAR oldu (founder) — sütun başlığı/arama ipucu/hata mesajı da "kod" demeyi bıraktı, ama alan SERBEST METİN kaldı: kod yazmak hâlâ mümkün ve yasal metin bunu bir TERCİH diye anlatıyor, alanı ada KİLİTLEMEK o cümleyi yalanlar. Yer tutucu örneği founder kararıyla KALDIRILDI.
  · **Dosya içeriği `Fernet(KLIVANCE_FILE_KEY)` ile ŞİFRELİ ve DB bytea'da** —
    Render diski ephemeral, DİSKE YAZMA. Anahtar yoksa `?err=svc` (fail-safe).
  · **Soft-delete içeriği ANINDA sıfırlar** (`content_enc` boş, `extracted_text` NULL);
    tombstone META satırını purge job ≥30 gün sonra fiziksel siler.
  · **MIME İSTEMCİDEN GELMEZ** (`_sniff_mime`, M11): `image/svg+xml` REDDEDİLİR
    (SVG script çalıştırır → indirme yolunda saklı XSS). İndirmede mime YENİDEN
    baytlardan doğrulanır; tanınmayan içerik `attachment` + octet-stream olur.
  · **İSTEK BAŞINA BELLEK TAVANI** (M6): dosya başına ~3 kopya tutulur; toplam tavan
    olmadan 50×20 MB tek istekte OOM ediyordu ve OOM-kill o worker'daki BAŞKA
    hekimlerin uçuştaki sohbetlerini de koparıyordu.
  · **KISMİ YÜKLEME SESSİZ OLMAZ** (M6b): kabul/ret sayısı `?y=<kabul>&r=<ret>`.
  · **AĞIR İŞ EVENT LOOP'TA ÇALIŞMAZ** (M5): `analyze_case` `async def`tir ama ağır
    kısım `run_in_threadpool(_analyze_case_sync, …)` ile havuza gider. İki eşzamanlı
    analiz siteyi HERKES için (landing/giriş dahil) donduruyordu.
    Kapı: `scratchpad/threadpool_kapisi.py` — doğrudan çağrıya çevirirsen KIRMIZI.
  · **Analiz ücretsiz DEĞİL ama KREDİSİZ** (2026-08-21): abonelik sınırsız; tek fren
    `MAX_ANALYZE_FILES` dosya tavanı + günlük yoğunluk tavanı. Eski kademeli tarife
    (`credits_for_analyze`) KALDIRILDI; onunla birlikte rezervasyon/iade zinciri de.
    MAP çağrıları `mode='map'` (FREE_MODES) → günlük frene SAYILMAZ.
  · **`store.get_case()` TUPLE döndürür (dict DEĞİL), 18-geniş.** Depodaki TEK
    pozisyonel unpack `case_detail_page` içindedir; yeni alan SONA eklenir.
    Diğer okuyucular `webutil._case_ctx_base` üzerinden index-tabanlıdır.
  · **Hasta detay `<title>` dil-duyarlı** — kod içerdiği için dinamiktir ve `i18n_mw`
    onu çeviremez, dil ELLE uygulanır.

⚠ REGRESYON AĞI (kesim öncesi ÖLÇÜLDÜ: 9 ucun **7'sine hiçbir süit dokunmuyordu**):
  · `scratchpad/cases_api_verify.py` — 7 `/api/cases/*` ucunun BAŞARI dalı; şifreleme, IDOR,
    soft-delete, kredi muhasebesi. GERÇEK LLM ÇAĞIRMAZ (analiz sahtelenir).
  · `cases_govde_verify.py` — HTML paritesi · `threadpool_kapisi.py` — ağır iş loop'ta değil.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_CKPT_H_TR` | `('Kısa özet', 'Kimlik ve bağlam', 'Alerji', 'Kronik durumlar ve kayıtlı tanılar', 'Düzenli ilaçlar', 'Laboratuvar seyri'` | 115 |
| `_CKPT_H_EN` | `('Brief summary', 'Identity and context', 'Allergies', 'Chronic conditions and recorded diagnoses', 'Regular medications` | 120 |
| `_CKPT_NITELIK` | `['düşük', 'yüksek', 'normal', 'düşük-normal', 'yüksek-normal', 'pozitif', 'negatif', 'sınırda', 'low', 'high', 'borderli` | 133 |
| `_CKPT_ESDEGER` | `{'demir (fe)': 'demir (serum)', 'serum demir': 'demir (serum)', 'demir': 'demir (serum)', 'akş-g': 'glukoz (açlık)', 'ak` | 140 |
| `KAYIT` | `[]` | 158 |
| `MAX_CASE_FILE_B` | `20 * 1024 * 1024` | 182 |
| `MAX_CASE_FILES_PER_CASE` | `50` | 183 |
| `MAX_CASE_UPLOAD_TOTAL_B` | `40 * 1024 * 1024` | 188 |
| `_ALLERGY_CHIPS` | `['Penisilin/Beta-laktam', 'Sülfonamid', 'NSAİİ', 'Opioid', 'Kontrast madde', 'Lokal anestezik']` | 262 |
| `_ALLERGY_SEV` | `['Döküntü', 'Anjiyoödem', 'Anafilaksi']` | 263 |
| `_RISK_ALANLARI` | `('meds', 'egfr', 'allergy_status', 'allergies', 'gebe_olabilir', 'emziriyor')` | 1241 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 161 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 172 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_sniff_mime` | `(raw: bytes, claimed: str \| None) -> str \| None` | 190 | Dosya TİPİNİ BAYTLARDAN belirle (M11). İstemcinin `content_type` beyanı bağlayıcı DEĞİL. |
| `_human_size` | `(n) -> str` | 223 | Bayt → okunur boyut (dosya kütüphanesi satırları için). |
| `_yas_or_none` | `(s)` | 238 | Tam yaş (#125b) → `(deger, aralik_disi)`. Aralık dışı = **KAYDEDİLMEZ** (None). |
| `_build_allergies` | `(status, chips, other, severity)` | 266 | Alerji çip seçimi + serbest 'diğer' + şiddeti tek metne birleştir (yalnız status='var'). |
| `_sex_select` | `(current = '', name = 'sex')` | 282 | Cinsiyet <select> — TR-kanonik (Kadın/Erkek/Belirsiz); eski serbest değer (E/K) korunur. |
| `_allergy_widget_new` | `(lang: str = 'tr')` | 293 | Yeni-hasta formu alerji alanı — 3 durum + 'Var'da çip çoklu-seçim + diğer + şiddet. |
| `cases_page` 🌐 | `(request: Request)` | 332 |  |
| `case_detail_page` 🌐 | `(cid: int, request: Request)` | 511 |  |
| `create_case` 🌐 | `(request: Request, code: str = Form(...), age_band: str = Form(''), sex: str = Form(''), narrative: str = Form(''), chronic: str = Form(''), meds: str = Form(''), egfr: str = Form(''), kreatinin: str = Form(''), kilo_kg: str = Form(''), gebelik_haftasi: str = Form(''), boy_cm: str = Form(''), allergy_status: str = Form(''), allergy_chip: list[str] = Form([]), allergy_other: str = Form(''), allergy_severity: str = Form(''), gebe_olabilir: str = Form(''), emziriyor: str = Form(''), yas: str = Form(''))` | 1193 |  |
| `_risk_alani_degisti` | `(onceki, **yeni) -> bool` | 1244 | Öncesi yoksa (yeni kart) False — ilk kayıtta 'değişti' demek anlamsız. |
| `update_case` 🌐 | `(cid: int, request: Request, age_band: str = Form(''), sex: str = Form(''), narrative: str = Form(''), chronic: str = Form(''), meds: str = Form(''), egfr: str = Form(''), kreatinin: str = Form(''), kilo_kg: str = Form(''), gebelik_haftasi: str = Form(''), boy_cm: str = Form(''), allergy_status: str = Form(''), allergies: str = Form(''), gebe_olabilir: str = Form(''), emziriyor: str = Form(''), yas: str = Form(''))` | 1264 |  |
| `delete_case` 🌐 | `(cid: int, request: Request, onay: str = Form(''))` | 1307 | Hasta kartını KALICI sil — hekimin KENDİ kartı (founder 03.09). |
| `add_visit` 🌐 | `(cid: int, request: Request, note: str = Form(...), analyze: str = Form(''))` | 1332 |  |
| `async` `upload_case_files` 🌐 | `(cid: int, request: Request, files: list[UploadFile] = File(...))` | 1353 | Hastaya kalıcı, ŞİFRELİ dosya yükle (LLM YOK → kota TÜKETMEZ). Dosyalar DB'de bytea. |
| `_upload_yetki_sync` | `(request: Request, cid: int)` | 1406 | Oturum + kart sahipliği — havuzda. Hekim sözlüğü ya da 303 `Response` döner. |
| `_case_file_kaydet_sync` | `(did: int, cid: int, name: str, mt: str, raw: bytes) -> bool` | 1417 | Şifrele (Fernet) + SHA-256 + DB bytea — dosya başına kendi bağlantısı/commit'i. |
| `download_case_file` 🌐 | `(cid: int, fid: int, request: Request)` | 1431 | Şifreli dosyayı çöz + orijinal mime ile inline döndür (önizleme/indirme). |
| `case_file_status` 🌐 | `(cid: int, request: Request)` | 1470 | Kartın ANALİZ DURUMU — yalnız SAYI (#705b-b). LLM yok, ücret yok, 0 yazma. |
| `delete_case_file` 🌐 | `(cid: int, fid: int, request: Request)` | 1492 | Dosyayı soft-delete (hekim silme hakkı — KVKK). |

## `saglik/app/chat_body.py`

`1768 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
/chat sayfa gövdesi (HTML + JS).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-07-29, mimari denetimi). İçerik
AYNEN taşındı — tek karakter değişmedi; render çıktısı bayt bayt doğrulandı.
main.py bu adları yeniden dışa aktarır, yani mevcut importlar (`from saglik.app.main
import ...`) ve doğrulama scriptleri KIRILMAZ.

⚠⚠ #620b-C: `gorNotuCiz` gerekçesi BURADA, `_CHAT_BODY` İÇİNDE DEĞİL — o sabit HAM
DİZEDİR, TAMAMI tarayıcıya iner, içine yazılan yorum da her hekime GÖNDERİLİR (#553b'de
ölçüldü). Sözleşme + bozma tatbikatı: `scratchpad/goruntuleme_kapisi_verify.py` §6.
⚠ 2026-09-13 (#620b YOL 2, sözleşme §2.4): "görüntü tetkiki okumaz" söylemi TARİHSEL — görsel artık `Görüntü ön-okuma` moduyla (env açıkken `#goruntubtn`, `chat_goruntu_js.py`) ÖN-OKUNUR; notice metni sunucudan (`isaretler.METIN`), aşağıdaki eski alıntılar ölçüm anlatısıdır.

⚠⚠ #698b (2026-09-03) — DÜZ METİN ROZET GEREKÇESİ DEĞİLDİR. `citedSources`in eski "değer
gövdede geçiyor" dalı, kimliği İLAÇ ADI olan türlerde (livertox · openFDA · TİTCK) adın yanıt
METNİNDE geçmesiyle rozet basıyordu — "metformin" yazan her yanıtta metformin geçer; bu atıf
değil KONUdur. Canlı külliyatta (Fırat sondası) rozetli gerçek-hekim yanıtlarının %17,6'sında
metinde HİÇ kaynak-türü atfı yoktu (tek parantez "[genel bilgi — kanıt dışı]", ekranda LiverTox).
Sunucu ikizi `faithfulness.check` yalnız köşeli parantezi sayar → dal ikizleri AYRIŞTIRIYORDU
(#56/#58). Kaldırılınca sahte 26→0, gerçek atıflı yanıtlarda rozet kaybı 0 (düşen rozetlerin
tümü türü hiç atıflanmamış ya da aynı türde BAŞKA kimlik atıflanmış karttı). Hoşgörü KORUNUR:
köşeli parantez İÇİNDE kısaltılmış ad (`ortusuyor`) geçer. Kapı `rozet_atif_verify` #698b bölümü
(eski-kod tatbikatı + sunucu paritesi).

⚠⚠ #706b (2026-09-03) — ÇİFT GÖNDERİM FRENİ (`gonderimKilidi` + `tekrarFreni`). Canlı
külliyatta ardışık BİREBİR aynı soru 7 çift; ≤48 sn aralıklılar KAZA (çift yanıt + çift maliyet),
≥71 sn olanlar KASITLI (aynı soru 8 kez; beğenilmeyen yanıtı yeniden isteyen hekim). Eşik 60 sn
= ölçülen boşluğun (48–71) içi. İki ayak: (1) akış sürerken Enter GÖNDERMEZ ve DURDURMAZ
(eskiden Enter = belgesiz "durdur"); ⏹ ilk 700 ms yok sayılır (çift tıklama = gönder+durdur
tuzağı), sonrası durdurur. (2) aynı parmak izi (metin+dosya+hasta+mod) ≤60 sn içinde ikinci
kez gelirse uyarı basar ve GÖNDERMEZ; bir kez daha basılırsa GÖNDERİR — kasıtlı tekrar
ENGELLENMEZ. İz localStorage'da (iki sekme aynı freni görür); hata/iptal yolları izi SİLER ki
başarısız denemenin ardından yeniden gönderme sürtünmesiz kalsın. ⚠ Sunucuda ikinci ücretlendirme
freni YOK ve bilerek yok. Kapı `scratchpad/cift_gonderim_verify.py`.

⚠⚠ #704b-b (2026-09-03, P0 hasta güvenliği) — `openThread` SEÇİCİYİ SUNUCUNUN `case_id`SİYLE
KURAR (`caseSelKur`). Eskiden seçici yalnız URL `?case=` ile kuruluyordu (`loadCases`); geçmişten
açılan hasta-bağlı sohbette `caseSel` boş kalıyor, sonraki tur `/api/chat` gövdesinde `case_id`
taşımıyor ve yanıt HASTA BAĞLAMI OLMADAN üretiliyordu (canlı: metformin+tirzepatid/eGFR 58
hastasına "hasta bağlamı verilmedi"; alerji/gebelik/renal talimatları prompta hiç girmedi = yanlış
güven). Sözleşme (Cahit, `store.thread_case_id`): `GET /api/threads/{tid}` üst düzey `case_id`
int (hekimin kendi, var olan kartı) ya da null. null → seçici BOŞ; kart listede yoksa `loadCases`
ile yeniden çekilir; hâlâ yoksa seçici BOŞ + görünür uyarı — ekranda "hasta yok" görünürken yanıt
hastaya özel OLMAZ (sessiz sunucu fallback'i yok, Ömer reddetti). #679b-2 "kart silinmiş" bayrağı
bu yoldan doğal çalışır. Kapılar `hasta_baglam_thread_istemci_verify` (istemci) · `hasta_baglam_thread_verify` (sunucu).

⚠⚠ #705b-istemci (2026-09-03) — EK TÜRÜ UZANTIDAN TÜRETİLİR, DESTEKLENMEYEN TÜR ÜCRET ÖDEMEDEN
UYARIR (`_ekTuru` · `ekTurUyar`). Eskiden `readFile` türü FileReader data-URL'inden okuyordu:
tarayıcı `type=""`/`application/octet-stream` verdiğinde (Android paylaşım/indirme sınıfı) o da
aynen gidiyor, sunucu `chat._attachment_block` None dönüyor, model dosyayı HİÇ görmüyor, hekime
uyarı çıkmıyor ve tur TAM ÜCRETLİ oluyordu (canlı 533→535 adayı; ölçüm `scratchpad/_kamil_705b_govde.js`).
Şimdi tür ÖNCE tarayıcıdan (`application/pdf` / `image/*`), boş ya da generic ise UZANTIDAN
(`_EK_UZANTI`: pdf · jpg/jpeg · png · webp · gif); ikisi de tutmazsa `send()` fetch'ten ÖNCE
görünür uyarı basar ve GÖNDERMEZ (iz yazılmaz, ücret yok). Küme sunucunun `_attachment_block`
kümesiyle aynı; parite `ek_tur_verify`de ölçülür. ⚠ Sunucu ayağı ("ek var, blok 0" → 400) ve
kart-dosyası çıkarım kararı AYRI kalemler (`_gorev.txt` #705b-a / #705b-b).

⚠⚠ #726b (2026-09-04) — ÇİP/PANEL YANITI SUNUCUYA AÇIKÇA BİLDİRİLİR (`marker_yaniti`,
`marker_mid`). Prod ölçümü (Cahit): takip turlarının **%26'sı** ürünün kendi çip yanıtı
("🚫 Gebelik: DIŞLANDI" gibi) ve o kovanın kaynaksızlık oranı **%41** — bütün kovaların en
kötüsü. Kök yapısal: `addChips.fire()` çipleri `q.value`ya yazıp normal `send()` çağırıyor,
RAG thread'in ASIL sorusu yerine çipin üç kelimesinden koşuyor; yani hekime kolaylık diye
konan çip kanıt paketini öldürüp tam ücreti alıyor. Sunucu bunu METİNDEN çıkaramaz →
istemci SÖYLER; retrieval girdisini kurmak Cahit/Derya'nın ayağı.
  · ⚠⚠ SİNYAL ÇİPİN BASILDIĞI YERDEN DOĞAR, METİNDEN DEĞİL: `fire()` bir kerelik
    `window._markerTur = {mid, q}` bırakır. **Emoji önekinden çıkarım YASAK** — emoji bir
    ürün garantisi değil, hekim de yazabilir ("desen ADAY üretir, HÜKÜM üretmez").
  · ⚠⚠ TÜKETİM METNE BAĞLI VE TEK SEFERLİK: `send()` bayrağı yalnız `q` birebir aynıysa
    kullanır ve her hâlde SİLER. Sebebi ölçülebilir bir kaza: fren (`dosyaFreni`/
    `tekrarFreni`) turu durdurup hekim metni DEĞİŞTİRİRSE bayrak eski turdan kalırdı ve
    elle yazılmış bir soru "çip yanıtı" diye etiketlenirdi. Yanlış sinyal, sinyalsizlikten
    kötüdür (aynı gün `message_id` bağında öğrenilen ders). Fren durdurduğunda bayrak
    TÜKETİLMEZ → onaylayan ikinci basış sinyali korur.
  · ⚠ ANAHTARLAR YALNIZ ÇİP YOLUNDA GİDER (`hastalik_id` deseni): elle yazılan turda gövdede
    HİÇ bulunmazlar → sunucu "değil" ile "bilinmiyor"u ayırt eder, eski istemci kırılmaz ve
    alan yokken davranış BUGÜNKÜ gibi kalır (fail-safe).
  · ⚠ `marker_mid` = markerları ÜRETEN yanıtın kimliği; kaynağı #715b-K1'in sözleşmesi
    (`done.message_id`), yeni tesisat YOK. Geçmişten açılan sohbette yük kimlik taşımadığı
    için `lastMid` null'dır — kimlik ÜRETİLMEZ, `marker_yaniti` yine true gider.
  · Kapı `scratchpad/marker_sinyal_verify.py` (node davranış + eski kod KIRMIZI + ters yön).

⚠⚠ #715b-K1 (2026-09-04) — İKİNCİ-GEÇİŞ KATMAN SATIRLARI TURA BAĞLANIR (`message_id`).
`/api/chat` `done` yükünde `message_id` döner (sunucu ayağı Cahit, `soru_kayit`); istemci onu
`addSources`/`secondPass`/`simplifyAns` üzerinden `dose-check` · `redflag` · `simplify`
gövdelerine GERİ GÖNDERİR. Öncesinde `app.answer_layer` satırları BAĞSIZ yazılıyordu: "kaç kez
çalıştırılamadı" cevaplanıyor, "HANGİ SORUDA" cevaplanmıyordu — #715b K3 panosunun bütün değeri
o bağda. Kimlik BUBBLE'A BAĞLI taşınır (modül düzeyinde "son mesaj" değişkeni DEĞİL): hekim
eski bir yanıttaki "Hastaya anlat"a bastığında son turun kimliği yazılsaydı, kusur BAŞKA bir
soruya yazılırdı — yanlış bağ, bağsız satırdan daha kötüdür.
  · ⚠ KOLAYLIK, KAPI DEĞİL: `mid` yoksa (`||null`) uçlar AYNEN çalışır, satır bağsız yazılır.
    `message_id` yokluğu hekimin güvenlik katmanını ASLA engellemez.
  · ⚠ TAHMİN YOK: geçmişten açılan sohbette (`openThread`) `/api/threads/{tid}` yükü mesaj
    kimliği TAŞIMIYOR → `m.message_id` undefined → null gider. Sunucu yükü bir gün id verirse
    bu satır kendiliğinden çalışır; istemci id ÜRETMEZ.
  · ⚠ Konsey/derin analiz/ilaç kartı çağrıları da `j.message_id||null` okur — o uçlar bugün
    alanı DÖNDÜRMÜYOR (null gider); sözleşme tek yerde, ikinci bir kural yazılmadı.
  · ⚠ Sunucuda `soru_kayit.mesaj_sahibi_mi` FAIL-CLOSED: istemciden gelen kimlik BAŞKASININ
    mesajınaysa satır BAĞSIZ yazılır (istemci tarafında ayrıca doğrulama yapılmaz, yapılamaz).
  · Kapı `scratchpad/katman_bag_verify.py` (node davranış + eski kod KIRMIZI + yabancı id).

⚠⚠ #705b-b (2026-09-04, founder onayı) — ANALİZ EDİLMEMİŞ KART DOSYASI FRENİ
(`dosyaFreni` · `_bekleyenDosya`). Hekim hasta KARTINA dosya yüklüyor, sohbette "ekteki
tahlile bak" deyip soruyordu; karta yüklenen dosya otomatik çıkarılmadığı için ücretli tur
yalnız "önce analiz et" talimatı üretiyordu (canlı 429→431 ve 533→535, founder hesabı dahil)
= hekim para ödeyip hiçbir şey almıyor. Artık hasta seçiliyken Gönder'e basıldığında, fetch
GİTMEDEN ÖNCE kaç dosyanın analiz edilmediği + "içerikleri bu turda modele GİTMEZ" + iki yol
(kartta ‘Seçilenleri analiz et’ / yine de gönder) yazan görünür uyarı çıkar; ücret ancak hekim
bilerek ikinci kez bastığında alınır.
  · ⚠⚠ SIRA `tekrarFreni`den ÖNCE ve bu ölçüldü, tercih değil: sonraya konsaydı ilk basış
    izi localStorage'a YAZAR (`_gonderimKaydet`), onaylayan ikinci basış bu kez ÇİFT GÖNDERİM
    uyarısına çarpardı → hekim aynı soruyu göndermek için ÜÇ kez basmak zorunda kalırdı.
    Bant `_ekTuru` freniyle aynı (#705b-istemci): iz yazılmadan, ücret ödenmeden uyar.
  · ⚠⚠ VERİ SUNUCUDAN, TAHMİNLE DEĞİL: `GET /api/cases/{cid}/dosya-durumu` (`cases_routes`,
    salt-okuma, LLM YOK, 0 ücret) ve kaynağı `chat_routes._build_case_record`in bekleyen-dosya
    notuyla AYNI (`store.list_case_files` → `extracted_text IS NOT NULL`). İkinci doğru-kaynak
    kurulmaz; ayrışırsa uyarı ile turun gerçeği çelişir.
  · ⚠ FAIL-OPEN (uç patlar/401 → 0 → uyarı yok, davranış bugünküyle birebir): bu bir güvenlik
    kapısı değil ücret şeffaflığı freni; ağ hıçkırığı yüzünden hekimi gönderemez hâle
    getirmek daha pahalı olurdu. Hasta seçili DEĞİLKEN uç HİÇ çağrılmaz.
  · ⚠ Onay anahtarı `cid:n` — kart değişirse ya da analiz edilmemiş YENİ dosya eklenirse onay
    DÜŞER ve hekim yeniden uyarılır (tek `_dosyaOnay=true` bayrağı olsaydı ikinci kart sessiz
    geçerdi). Uçuştaki kontrol sırasında ikinci basış `_dosyaKontrol` ile yutulur.
  · ⚠ Uyarı metnindeki buton adı GERÇEK: `/cases` sayfasında düğme "Seçilenleri analiz et" /
    "Analyze selected" (hiçbirini seçmezsen tümü). Sunucunun bekleyen notundaki
    "Tümünü analiz et" ifadesi UI'da böyle bir düğme OLMADIĞI için kopyalanmadı.
  · Kapı `scratchpad/kart_dosya_freni_verify.py` (statik + kablolama + node davranış: eski
    kod KIRMIZI = analiz edilmemiş dosyayla fetch gidiyor; yeni kod YEŞİL; ters yön =
    analizli kartta uyarı YOK, hasta seçili değilken uç hiç çağrılmıyor).

⚠⚠ #679b-2 `hastaDustuCiz` — SESSİZ HASTA BAĞLAMI KAYBI (gerekçe BURADA, aynı #553b kuralı).
Sunucu seçili hasta kartını bulamazsa (silinmiş ya da başkasına ait) turu hasta bağlamı
OLMADAN üretir. Eskiden bunu SÖYLEMİYORDU ve seçici seçili kaldığı için hekim yanıtı hastaya
özel sanıyordu — alerji/çapraz-reaktivite, gebelik-emzirme ve renal doz talimatları hiç
girmemişken. Sınıf "yanlış cevap" değil **YANLIŞ GÜVEN**; tetik, silme sırasında AÇIK KALAN
İKİNCİ SEKME. Sunucu sözleşmesi `chat_routes._hasta_baglami`; akış dalında bayrak İLK NDJSON
satırıdır (sonda gelse hekim uyarıyı yanıtı okuduktan SONRA görür).
  · AMBER `.gor-notu`, aklama/yeşil dil YOK · `role="alert"` (`gorNotuCiz`in `note`u DEĞİL).
  · ⚠⚠ SEÇİCİ DE BOŞALTILIR — kozmetik değil: seçim kalırsa hekimin SONRAKİ turu da sessizce
    bağlamsız gider ve uyarı bir daha basılmaz.
  · ⚠⚠ İKİ UYARI BİRBİRİNİ YUTMAMALI — VE İLK SÜRÜM TAM BUNU YAPIYORDU (Fırat P1, 03.09).
    Hasta uyarısı stili paylaşsın diye `class="gor-notu hasta-dustu"` yazıyor; `gorNotuCiz`in
    dedupe seçicisi `.gor-notu` ise o sınıfla **DA** eşleşiyordu. Akışın GERÇEK sırası
    `hasta_dustu` (1. satır) → `notice` (ilk delta'dan hemen önce, `gizli_kaynak.py`) olduğu
    için görüntüleme notu HİÇ BASILMIYORDU: hekim "Klivance görüntü tetkiki okumaz, bu
    dosyanın içeriği DEĞERLENDİRİLMEDİ" uyarısını göremiyor, yüklediği görüntünün okunduğunu
    sanıyordu — yani bu uyarının kapatmak için yazıldığı YANLIŞ GÜVEN sınıfı, başka bir
    yerden yeniden açılıyordu. Çözüm: `gorNotuCiz` dedupe'u `.gor-notu:not(.hasta-dustu)`.
    ⚠ İlk sürümün buradaki iddiası ("takılmaz") ÖLÇÜLMEMİŞTİ ve yalnız TERS sırada doğruydu;
    tek yön ölçen bir iddia, çift-yönlü bir sözleşmeyi çiviliyormuş gibi görünür.
    ⚠ Dedupe'lar AYRIK: hasta uyarısı `.hasta-dustu`, görüntü notu `.gor-notu:not(...)`.
    ⚠⚠ #782c AYNI SINIF ÜÇÜNCÜ KEZ: kritik kademe şeridinin amber kolu da stil paylaşsın diye
    `class="gor-notu krt-serit"` yazıyor → seçiciye `:not(.krt-serit)` EKLENDİ. Çağrı sırasına
    ("bugün `gorNotuCiz` önce koşuyor") güvenmek yetmez; sıra bir gün değişirse notice metni
    şeridin metnini SESSİZCE ezer. Gerekçe + kapı: `chat_goruntu_js.py` docstring "KRİTİK
    BULGU KADEMESİ" · `scratchpad/kritik_serit_verify.py`.
    Kapı `hasta_baglam_dustu_verify` bölüm 6 **İKİ YÖNÜ DE** ölçer (jsdom); sonda
    `scratchpad/_firat_679b_dom_sonda.py` (A/B/C/D).
  · ⚠ İngilizce metinde KESME İŞARETİ YOK ("was not found"): `en()` script bloklarını da
    çevirir, düz kesme tek-tırnaklı JS string'ini kapatıp sayfanın TÜM JS'ini çökertir.
  · Kapı `scratchpad/hasta_baglam_dustu_verify.py` (CI SUITES; DB/ağ/LLM istemez) ·
    tatbikat `scratchpad/_kamil_679b_tatbikat.py` (ürün ağacına yazar, yerelde koşulur).

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı. (Tek istisna: `chat_goruntu_js._GORUNTU_JS` — o da SAF sabit; YOL 2 script'i sona eklenir, dinamik `#goruntubtn` ise chat_routes'ta yer tutucuya yerleşir.)

⚠⚠ GİZLİ KAYNAK — GEREKÇE BURADA DURUR, `_CHAT_BODY` İÇİNDE DEĞİL (#553b, 2026-08-31).
`_CHAT_BODY` bir HAM DİZE ve TAMAMI tarayıcıya iner: içindeki JS/HTML yorumları da iner.
Ölçüldü (`_firat_statpearls_yuzey_sonda.py`): gizlenmesi gereken kaynağın ADI `/chat`
HTML'inde 13 kez geçiyordu ve 10'u YORUMDU — yani ürünün kendi gizleme kararının gerekçesi,
kararın ihlal ettiği şeyi sayfaya yazıyordu. Bu modül docstring'i `_CHAT_BODY`nin DIŞINDADIR
ve render'a girmez, gerekçe bu yüzden buraya taşındı.

  · KARAR (founder 2026-08-19, birebir): "sitede HİÇBİR yerde statpearls geçmesin";
    31.08 teyidi: "kaynakları kullanalım, bilgileri alalım, ama hekim karşısına StatPearls
    diye çıkmasın" → GÖSTERİM yasağı, KANIT yasağı DEĞİL.
  · LİSANS: StatPearls gerçek lisansı CC BY-NC-ND 4.0 (ticari kullanım + türev YASAK);
    `core.corpus.license` alanında 'CC BY (StatPearls Publishing)' diye YANLIŞ etiketli —
    açık P0, tam kayıt `CLAUDE.md` "P0 LİSANS" bölümü.
  · TEK KARAR NOKTASI `cds/kaynaklar.py` → `"gizli": True`; üç tüketici oradan doğar
    (`app/gizli_kaynak.py` sunucu · `hastalik._GIZLI_KAYNAK` SQL · bu dosyadaki
    `_KANIT_GIZLI` istemci sabiti). Sabit BİLEREK literal: yer tutucuya çevirmek 5 kapıyı
    kırar, eşitliği `rozet_atif_verify` bölüm 2c çiviler.
  · ⚠ Bu süzgeç adı GÖSTERİMDEN siler; MODELE giden kanıt paketine DOKUNMAZ (kaynağı
    kanıttan çıkarmak `faithfulness`ı meşru atıflara ⚠UNSUPPORTED bastırırdı).
  · ⚠ AÇIK SINIR: model adı DÜZ METİNDE (etiketsiz, cümle içinde) yazarsa hiçbir süzgeç
    onu yakalamaz — kök katman `prompts.DOCTOR_SYSTEM`, ayrı kalem (#545b).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_CHAT_BODY` | `'\n<style>\n /* Chat kabuğu — paylaşılan CSS\'ten SONRA gelir; .cwrap ve mobil .sbar KAYNAK düzeltmesi burada.\n    Toke` | 190 |

## `saglik/app/chat_goruntu_js.py`

`294 satır` · `1 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
/chat — YOL 2 "Görüntü ön-okuma" modunun İSTEMCİ ayağı (#620b, sözleşme
`docs/goruntuleme-sozlesme-2026-09-13.md` §2.4 · WP-4, Kamil).

⚠⚠ GEREKÇE BURADA, `_GORUNTU_JS` İÇİNDE DEĞİL (#553b kuralı): script tarayıcıya iner, içine
yazılan yorum her hekime gönderilir. `_GORUNTU_JS` yalnız kod + tek satırlık işaret taşır.

NE YAPAR
· `goruntu_dugmesi(lang)` — `.ctool` grubuna giren `#goruntubtn` HTML'i. **Env kapalıyken
  (`credits.goruntu_onokuma_acik()` False) BOŞ DİZE döner → sunucu düğmeyi HİÇ BASMAZ**
  (kararlar C.Q12; P10-b "env kapalı → goruntubtn 0"). `chat_routes.chat_page` bunu
  `_CHAT_BODY`deki `<!--__GORUNTUBTN__-->` yer tutucusuna render anında yerleştirir.
  ⚠ Neden yer tutucu: `_CHAT_BODY` SAF SABİTTİR (chat_body docstring'i: f-string yok, modül
  durumu yok) — env'e göre "basmama" yalnız render anında mümkün. Yer tutucu HTML YORUMUDUR:
  bir gün yerleştirilmeden servis edilse bile ekranda görünmez (metin yer tutucusu görünürdü).
· `_GORUNTU_JS` — `_CHAT_BODY`nin SONUNA eklenen `<script>`; ana script'teki DÖRT global
  fonksiyonu SARAR (yeniden tanımlamaz, kopyalamaz): `setMode` (goruntu dalı: `#goruntubtn`
  `.active`, ipucu `onokuma_mod_hint`, Gönder başlığı) · `send` (mod `goruntu` → `goruntuGonder`;
  başka modda ORİJİNAL çağrılır, tek satırı değişmez) · `modRozet` (`goruntu` → `.deepbadge
  deepbadge-warn` + `onokuma_rozet`; başka mod orijinale düşer) · `addSources` (`goruntu`
  balonunda YALNIZ Kopyala + Takibe al kalır — `saveToPatient`/`whatMissed`/`simplifyAns`
  render edilmez, kararlar C.Q9/K14: ön-okuma karta ve ikincil uçlara ÇERÇEVESİZ gitmez).
  Sarmalama ana script'ten SONRA koştuğu için (`_CHAT_BODY` sonu) dört ad zaten tanımlıdır;
  klasik script'te `function` bildirimi yazılabilir global bağdır, atama geçerlidir.
  ⚠ Geçmiş yüklemede (`openThread`) aynı `addSources`/`modRozet` çağrıldığı için `mode=goruntu`
  kayıtlı tur da rozetli ve iki-eylemli çizilir — canlı ile geçmiş TEK yol.

METİNLER — TEK SÖZLÜK
· JS'te TR literal YOK; hekime görünen her metin `window.KLV_GORUNTU_M` (sunucu enjekte:
  `isaretler.METIN[lang]`, `chat_routes.chat_page`) üzerinden `M(anahtar)` ile okunur.
  ⚠ Enjeksiyon `_CHAT_BODY`den SONRA gelir (chat_routes yorumu: tembel okunanlar sonda) →
  `M()` yalnız olay anında çağrılır, top-level'da OKUNMAZ (yoksa `undefined`).
· Kopyala/Takibe al etiketleri YENİ metin değildir: orijinal `addSources` üretir, sarmalayıcı
  yalnız fazlalarını (ilk ve son dışındakileri) DÜŞÜRÜR. Sıra sözleşmesi ORİJİNALDE: ilk
  düğme Kopyala, son düğme Takibe al (chat_body `addSources`, `a.append(cp)…a.append(tk)`).
· `onokuma_notice` sunucudan "⚠ " ile gelir; `gorNotuCiz` de "⚠ " ekler → çift işaret
  olmasın diye baştaki işaret düşürülür (yalnız görsel, metin sözlükten).

UÇ SÖZLEŞMESİ (`on_okuma._goruntu_onoku_sync`, sözleşme §1.5/§2.4)
· `POST /api/goruntu-onoku` `{question, attachment:[…], thread_id, case_id, zorla:0|1}`.
· 200 `{answer, notice, status:"ok"|"unavailable", error, sources, mode:"goruntu", thread_id,
  message_id}`; `error==="taninmadi"` → görsel radyolojik sayılmadı: yanıt yok, `notice` amber
  basılır + "Yine de ön-oku" (`onokuma_zorla`) düğmesi `zorla:1` ile AYNI dosyayı yeniden
  gönderir (tespit atlanır). `status==="unavailable"` → `answer` zaten sabit amber cümle
  (FAIL-OPEN YASAK: yeşil/sessiz yok), rozet yine basılır. ⚠ 200'de `error` DOLU gelebilir (`istisna`/`kalite`)
  → hata dalı `r.status!==200||j.mode!=='goruntu'` (`j.error` ile dallanan ilk sürüm amber yanıtı yutuyordu, S6) · #779c: 200 gövdesi `_heartbeat_stream` NDJSON (10 sn ping; `done`=bu yük, `error`=`mode`suz) → `readJob`+`jobProgress`; `catch` YALNIZ JSON olmayan gövde (proxy 502/524 HTML) ya da ağ → `console.warn(http|network)` + TEK ⚠ (14.09 canlı vaka).
· 4xx/5xx `{error, …bayrak}` → mevcut `hataDallandir` (verify/upgrade/butce/kapali/mime/…).
· ⚠⚠ #781c ÇOKLU GÖRSEL (founder 15.09 "İki grafiği kıyasla"): sunucu `MAX_ONOKUMA_GORSEL` görsel
  kabul EDİYORDU, istemci YAPISAL olarak 1 gönderiyordu (`attachment:[att]`) → ikinci görsel modele
  HİÇ ULAŞMIYORDU. `goruntuGonder(q,fs,zorla)` artık LİSTE alır; `attachment` N elemanlı gider.
  ⚠ Tavan JS'te SABİT DEĞİL: `#goruntubtn` `data-max` (sunucuda `credits.MAX_ONOKUMA_GORSEL`ten
  üretilir) → tek kaynak sunucuda kalır. Öznitelik yoksa/parse edilemezse **1** (bugünkü davranış,
  fail-closed). Aşımda amber + DUR; sunucu 413'ü ZATEN yetkili kapı, bu yalnız boşa istek önler.
  ⚠ Kullanıcı balonunda dosya ADLARI ` · ` ile birleşik basılır (`bubble`nın `file` argümanı, escH'li).
  ⚠⚠ FIRAT P1 (15.09) — red dalı SEÇİM BİLGİSİ metnini (`onokuma_cok_gorsel`) HATA metni olarak
  kullanıyordu; ÜÇ kusur bir aradaydı: (1) ham `{n}` ekrana sızıyordu (`String.replace` YALNIZ İLK
  geçişi değiştirir, metinde İKİ `{n}` var) · (2) sayı tavanı gösteriyordu, hekimin seçtiğini değil
  · (3) "birlikte DEĞERLENDİRİLİR" diyordu ama istek HİÇ GİTMEMİŞTİ (fetch 0), hekim sonuç bekledi.
  Çözüm: TÜM yer tutucular `F(s,v)` (split/join) ile — sınıf TEK YERDE kapandı, `onokuma_kaydedildi`
  de ondan geçer · red metni AYRI anahtar (`onokuma_cok_gorsel_red`, `{secilen}`/`{tavan}`), yoksa
  `data-red` yedeği · SEÇİM metni YALNIZ gönderilen turda ve gerçek sayıyla (`.gor-notu`, kullanıcı
  balonunun üstünde) → cümlenin "değerlendirilir" iddiası artık DOĞRU.
· ⚠⚠ #781c HASTA KARTINA KAYDET: ön-okuma balonunun ALTINDA (`.onoku-kaydet` satırı) ayrı düğme —
  `.bubacts` İÇİNE KONMAZ, çünkü K14/C.Q9 sözleşmesi o kutuda YALNIZ iki eylem bırakıyor ve
  sarmalayıcı fazlasını siliyor (üçüncü düğme oraya konsa SESSİZCE silinirdi).
  ⚠ FAIL-CLOSED: `isaretler.METIN`de `onokuma_karta_kaydet` anahtarı YOKSA düğme HİÇ BASILMAZ.
  ⚠⚠ BAYTLAR NEREDE: `File` NESNE REFERANSI o mesajın kapanışında tutulur, base64 DEĞİL.
  `clearFiles()` yalnız `<input>`u boşaltır — `File`/`Blob` referansı geçerli kalır ve baytlar
  DİSKTE durur (tarayıcı belleğinde değil). Kaydet tıklanınca `readFile` YENİDEN okur. Bellekte
  base64 tutmak 3×5 MB için ~20 MB UTF-16 dize demekti; bu yol sıfır. Bedeli: dosya tıktan önce
  diskten silinirse `readFile` patlar → amber (sessiz başarı YOK).
  ⚠ Geçmiş turda (`openThread`) düğme YOKTUR ve olmamalı: baytlar yalnız CANLI turda elde.
  Uç: `POST /api/cases/{cid}/goruntu-kaydet` `{attachment, message_id, thread_id}` → `{ok, fid[], durum}`.
  Hasta kartı seçili değilse hekime `onokuma_kart_sec` amber'ı basılır, istek GİTMEZ.
· #778c GÖRSELSİZ METİN (founder 14.09): görsel yoksa ön-okuma ucuna GİDİLMEZ, ORİJİNAL `send`e düşülür.
  Dosyasız + metin → tek seferlik `window.KLV_RAD_METIN=true` (chat_body `send` BAŞINDA okur ve sıfırlar,
  erken dönüşte bayat kalmaz) → `/api/chat` gövdesine `rad_metin:1` → sunucu YOL 1 MAP'ini metin üzerinde
  koşar. PDF varsa bayrak GİTMEZ (ek zaten YOL 1). Boş metin + dosyasız → amber ipucu. Rozet basılmaz
  (yanıt ön-okuma değil). Kapı `scratchpad/rad_metin_kablo_verify.py`.

KRİTİK BULGU KADEMESİ (#782c, founder 15.09 "aç, daha da geliştir")
· ⚠⚠ ÇEKİRDEK KURAL: **kırmızının yanmaması bulgunun YOKLUĞU DEĞİLDİR.** Nötr şerit bunu
  metninde AÇIKÇA söyler (`isaretler.METIN.kritik_yokluk`, `kritik_goruntu.blok` her kademede
  KOŞULSUZ basar); "temiz/normal/sorun yok" dili YASAK. Tarama hiç koşmadıysa (yük yok ya da
  bozuk) **nötr de GÖSTERİLMEZ** — şerit basılmaz, yanıt yine çizilir (fail-closed).
· TAŞIYICI: kademe, ön-okuma METNİNİN İÇİNDE bir blokla taşınır (`on_okuma_kayit.kademe_ekle`,
  ÖNEK satırından SONRA). ⚠ Neden metin: `app.case_file`de serbest/JSON kolon YOK ve metin zaten
  dört yüzeyden geçiyor (canlı sohbet · `openThread` geçmişi · karta-kaydet ucu · kart kolu
  `<details>`) → kademe hepsinde EK KOD OLMADAN korunur.
· ⚠⚠ GRAMER JS'TE YAZILI DEĞİL — SUNUCUDAN ENJEKTE (`window.KLV_KRITIK`, `chat_routes.chat_page`):
  işaret öneki ve tanınan kademe kümesi `cds/kritik_goruntu`nun KENDİ sabitlerinden gelir
  (`kademe_isareti`/`kademe_oku` ile aynı kaynak). İki yerde elle desen yazılsaydı biri ötekini
  sessizce kaçırırdı (Ömer/Cahit kuralı, 15.09). Yük SAF ASCII → `EN_PAIRS` çakışmaz.
· ⚠⚠ ÜÇ DURUM, İKİ DEĞİL: işaret HİÇ YOK → kayıt hiç taranmamış, ŞERİT ÇİZİLMEZ (bu katmandan
  önceki ön-okumalar; hepsine amber basmak geçmişi alarma boğardı) · işaret VAR ama blok bozuk →
  AMBER (bozuk işaret "temiz" değildir; metin `kritik_calistirilamadi`) · tanınan → kendi kademesi.
· ⚠ `kritik is None` "TEMİZ" DEĞİLDİR: sunucu `ok` yolunda yük gelmese bile `hepsi_olculemedi()`
  AMBER'ini yazar — istemciye "kademesiz ön-okuma" ULAŞMAZ.
· ÇİZİM `addSources` SARMALAYICISINDA, çünkü `text` argümanı HEM canlı HEM geçmiş yolunda
  geliyor (`openThread` `pch.clean` ile çağırır) → canlı/geçmiş paritesi bedava. Sarmalayıcı
  bloğu metinden SÖKER, `bubEl`i temiz metinle yeniden render eder ve şeridi `wrap` içine,
  balonun HEMEN ÜSTÜNE koyar. ⚠ `_addSources`e TEMİZ metin verilir → Kopyala düğmesi ve
  `citedSources` markerı görmez. ⚠ Re-render güvenli: orijinal `addSources` çıktılarının
  hepsi `wrap`a eklenir, `bubEl`in çocuğu DEĞİLDİR (ölçüldü, chat_body:1155).
· ⚠⚠ ŞERİT `.bubacts` İÇİNE KONMAZ: sarmalayıcı o kutuda ilk ve son dışındaki HER çocuğu
  siler (#781c dersi — kırpma ORTADAKİ çocuğu yer). `wrap` çocuğu olduğu için kırpmadan
  SONRA da durur; kapı bunu ayrıca ölçer.
· KADEME → STİL (hepsi MEVCUT sınıflar, yeni palet uydurulmadı): kırmızı `redflag` (ikinci-göz
  deseni) · amber `gor-notu` (görüntü notu ailesi) · nötr `rffail` (aynı amber ailesinin DÜŞÜK
  VURGULU üyesi: 12px, `#8a5a00` on `#FDF6E3`). Hepsine `krt-serit` işaret sınıfı.
  ⚠⚠ NÖTR GRİ/YEŞİL DEĞİL, AMBER AİLESİNDE (Ömer kararı 15.09): nötrün anlamı "tarandı, aday
  ayırt edilmedi"dir ve bu bir AKLAMA DEĞİLDİR; sakin bir gri onu aklama gibi okutur. Amber ile
  nötrü ayıran RENK değil BAŞLIK METNİ + ağırlık.
· ⚠ İKİ AĞIRLIK (Ömer 15.09): şeridin SON satırı (kaynaksızlık ibaresi) `.krt-hint` ile ikincil
  stilde basılır; geri kalanı (yokluk uyarısı dahil) tam ağırlıkta. Metin KIRPILMAZ, yalnız
  ağırlık ayrışır — ikisi aynı ağırlıkta okunursa hekim İKİSİNİ BİRDEN atlıyor. Bölme KONUMA
  göre (son satır), metni tanıyan dizgeye göre DEĞİL; sonun gerçekten `kritik_kaynaksiz`
  olduğunu kapı çiviler (`kritik_serit_verify` C10) → üretici sırayı değiştirirse KIRMIZI.
  ⚠⚠ `gorNotuCiz` dedupe seçicisi `.gor-notu:not(.hasta-dustu)` idi ve amber şeridi de
  YAKALARDI (#679b-2 ile BİREBİR aynı sınıf hata: paylaşılan sınıf, ayrı anlam) → seçiciye
  `:not(.krt-serit)` eklendi. Çağrı SIRASINA güvenmek yetmezdi: bugün `gorNotuCiz` önce
  koşuyor, ama sıra bir gün değişirse notice şeridin metnini SESSİZCE ezerdi.
· Şerit metni `textContent` ile basılır (innerHTML = XSS yüzeyi) ve çok satırlıdır →
  `.krt-serit{white-space:pre-wrap}`.

i18n
· `en()` script bloklarını da çevirir → `_GORUNTU_JS` içinde TR sözcük yok; EN metinler sunucudan
  JSON (`ensure_ascii`) geldiği için apostrof tuzağına kapalı. Kapı: P15 (`head()+nav()+
  _CHAT_BODY → en() → <script> → node --check`) + `en(_GORUNTU_JS) == _GORUNTU_JS` bayt paritesi.
· Düğme HTML'i `lang` ile sunucuda üretilir (statik EN_PAIRS çifti DEĞİL — metin `isaretler`
  sözlüğünde, kopya açılmaz).

ÖLÇÜM: `scratchpad/_kamil_wp4_sonda.py` (DOM-stub node harness: gerçek `setMode`/`modRozet`/
`addSources` kaynağı + bu sarmalayıcı; env açık/kapalı; zorla yolu; balonda 2 eylem).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_YER_TUTUCU` | `'<!--__GORUNTUBTN__-->'` | 140 |
| `_RED_YEDEK` | `{'tr': '⚠ {secilen} görsel seçtiniz; tek ön-okumada en fazla {tavan} görsel gönderilebilir. İstek GÖNDERİLMEDİ, ücret al` | 147 |
| `_GORUNTU_JS` | `'<script>\n/* YOL 2 sohbet ayagi — gerekce: saglik/app/chat_goruntu_js.py docstring */\n(function(){\n function M(k){var` | 170 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `goruntu_dugmesi` | `(lang) -> str` | 155 | `#goruntubtn` HTML'i — env kapalıysa '' (sunucu basmaz). Metinler `isaretler.METIN[lang]`. |

## `saglik/app/chat_routes.py`

`1593 satır` · `37 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
SOHBET + CDS API ROTALARI — /chat, /api/chat (AKIŞ), ikinci-geçiş ajanları
(Faz 6, 2026-08-01).

`main.py`'den ayrıldı. Kapsam: 15 rota + bu bölgeye ÖZEL yardımcılar —
`/chat` · `/api/chat` · `/api/threads` (GET/GET-tid/DELETE/PATCH) · `/api/dose-check` ·
`/api/redflag` · `/api/whatmissed` · `/api/drug-card` · `/api/deep-analyze` ·
`/api/simplify` · `/api/council` · `/api/cases/{cid}/timeline` · `/api/mycases`.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ).
  Ölçüldü: taşınan kodun main.py'de kalan bağımlılıkları `app` nesnesi + 4 paylaşık
  yardımcıydı (`_json_body`, `_case_ctx_base`, `_email_gate`, `_erisim_402`). `app` →
  `kur(app)`; dördü → **webutil**'e taşındı, çünkü main.py'de KALAN dosya-kütüphanesi
  analizi (`analyze_case` / `_analyze_case_sync`) da onları kullanıyor.

⚠⚠ NEDEN `@app.post` DEĞİL `@_rota` — VE NEDEN `include_router` DEĞİL:
  `app` burada yok; `_rota(...)` rotayı KAYIT listesine yazar, main.py `kur(app)`
  çağırınca gerçek kayıt `app.get/post` PUBLIC API'siyle yapılır → `app.routes` DÜZ
  kalır. `APIRouter` + `include_router` Faz 2b'de DENENDİ ve ÖLÇÜLEREK geri alındı:
  bu FastAPI sürümünde tembeldir, rotaları `app.routes`a düzleştirmez → rota
  envanterini oradan okuyan güvenlik ağları sessizce körelir.

⚠⚠ ERİŞİM + AKIŞ İNVARİANTLARI (bu dosyanın en pahalı bölgesi — BOZMA):
  · ⚠⚠ **KREDİ SİSTEMİ KALDIRILDI (2026-08-21, founder).** Rezervasyon/iade/commit
    zinciri (`try_reserve`/`commit_reservation`/`cancel_reservation`/`reserved_portions`)
    bu dosyadan TÜMDEN çıktı. Yerine ücretli her ucun başında **İKİ SATIRLIK TEK KAPI**:
    `_erisim_402` (hakkı var mı: 15 gün deneme / aktif abonelik) → `_gunluk_fren_429`
    (bugünlük yoğunluk tavanı). ⚠ SIRA BÖYLE KALIR: hakkı olmayana "yarın tekrar
    deneyin" demek yanlış bilgidir.
  · **İPTALDE DE `record_usage` YAZILIR** — kredi kalksa da geçerli: `app.usage` artık
    hem maliyetin hem günlük frenin TEK kaynağı, yazılmazsa kaçak ÖLÇÜLEMEZ.
    `_settle_abort` yalnız bunu yapar (ücret kararı vermez).
    ⚠⚠ #748b (06.09): BAĞLANTI KOPMASI ≠ İPTAL — üretim ayrı thread'de sürer (`chat_turn.py`);
      `GeneratorExit` üretimi DURDURMAZ, `_settle_abort` yalnız turn `iptal` (Durdur tuşu) görünce koşar.
  · **`refused` ve `_is_clarify` FRENE SAYILMAZ** (FREE_MODES) — hekim yanıt almadıysa
    günlük sayacı yanmaz; teslim edilmiş tam yanıt sayılır.
  · **`log_query_stat` ANONİMDİR** — ham soru metni ve doctor_id YAZILMAZ. Bu satıra
    kimlik/ham metin eklenirse KVKK ifşası AYNI commit'te zorunlu olur.
  · **Güvenlik katmanları (dose-check/redflag/simplify) `_email_gate`in ve ERİŞİM
    KAPISININ DIŞINDA** — güvenlik hiçbir zaman duvarın arkasına konmaz; frenleri
    kendi `_gunluk_tavan`larıdır. FAIL-OPEN YASAK: hata durumunda
    `status="unavailable"` döner, ASLA `clean`/yeşil tik.
  · **`api_suggest` bu modülde DEĞİL** (referans yüzeyi, Faz 7) — hot-path'te ağ
    çağırmama kuralı orada geçerlidir.
  · **Ağır iş event loop'ta çalışmaz**: rotalar `async def` kalır, yalnız gövde okuması
    `await`lenir, ağır kısım `run_in_threadpool(_..._sync, …)` ile havuza gider.
  · **Model çıktısı ekrana `textContent` ile basılır** (innerHTML = XSS yüzeyi).

⚠ REGRESYON AĞI: `scratchpad/akis_parite_verify.py` (26 senaryo — red dalları + akış +
  iptal eşiği 599/600/800). GERÇEK LLM ÇAĞIRMAZ, DB'ye YAZMAZ. Bu uca dokunan her
  değişiklikte kesim öncesi/sonrası koşturulur.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 84 |
| `MAX_QUESTION_CHARS` | `16000` | 108 |
| `MAX_ATTACH_B64` | `8 * 1024 * 1024` | 111 |
| `MAX_HISTORY_MSGS` | `20` | 114 |
| `STREAM_ABORT_FREE_CHARS` | `600` | 119 |
| `SIMPLIFY_DAILY_CAP` | `120` | 136 |
| `SAFETY_DAILY_CAP` | `400` | 1037 |
| `SAFETY_DAILY_CAP_TRIAL` | `60` | 1052 |
| `SIMPLIFY_DAILY_CAP_TRIAL` | `40` | 1055 |

### `class _AbortUsage(—)`  <sub>satır 122</sub>

Akış iptalinde maliyet logu için asgari `usage` benzeri nesne (llm.cost_of getattr okur).

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, delivered_chars: int, usd_taban: float = 0.0)` | 131 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 87 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 98 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_faith` | `(mode, answer, sources) -> None` | 139 | M32 — atıf sadakati gözlemi (denetim bulgusu D1: `cds/faithfulness.py` yazılmıştı ama |
| `_verify_banner_js` | `(email: str, en: bool) -> str` | 155 | Doğrulanmamış DENEME hekimine chat üstünde kalıcı 'e-postanı doğrula' bandı |
| `_erisim_banner_js` | `(er, lang: str = 'tr', comp: bool = False) -> str` | 202 | `/chat` erişim bandı: duvar → "planları gör", erişim bitiyor → geri sayım. |
| `chat_page` 🌐 | `(request: Request)` | 286 |  |
| `api_threads` 🌐 | `(request: Request)` | 462 |  |
| `api_thread_get` 🌐 | `(tid: int, request: Request)` | 477 |  |
| `api_thread_delete` 🌐 | `(tid: int, request: Request)` | 499 |  |
| `async` `api_thread_rename` 🌐 | `(tid: int, request: Request)` | 509 | Konuşmayı yeniden adlandır (sahiplik store'da zorlanır). |
| `_thread_rename_sync` | `(request: Request, tid: int, title: str)` | 522 |  |
| `_clip` | `(text: str, n: int, lang: str) -> str` | 531 | Klinik metni kırp ve KIRPILDIĞINI İŞARETLE (M27). |
| `_build_case_record` | `(conn, doctor_id: int, case_id: int, lang: str, labs: bool = True, bekleyen_notu: bool = True) -> str` | 548 | Hastaya-özel chat için klinik kayıt metni: ziyaret notları (analiz sonuçları dahil) |
| `_chat_yuzeyi` | `(hastalik_id, attachment) -> str` | 597 | `/api/chat` turunun bütçe yüzeyi — tek kural, hazırlık ve iptal aynı adı kullansın. |
| `_hasta_baglami` | `(conn, doctor_id, case_id, lang)` | 602 | Seçili hasta kartının bağlamını kurar — DÖRT ucun TEK kaynağı. |
| `_chat_hazirlik_sync` | `(request, lang, question, attachment, thread_id, case_id, hastalik_id, _hst_bozuk, body = None)` | 632 | `/api/chat` akış ÖNCESİ DB ön koşulları — kimlik · hastalık id · hız · sahiplik · |
| `async` `api_chat` 🌐 | `(request: Request)` | 725 |  |
| `_second_pass_ctx` | `(conn, doctor_id, case_id, lang = 'tr')` | 1006 | İkinci-geçiş ajanları (doz/kırmızı-bayrak/ne-kaçırdım) için hasta bağlamı: |
| `_gunluk_tavan` | `(d, normal: int, dar: int) -> int` | 1058 | Doğrulanmamış deneme hesabı → dar tavan. (`_email_gate` ile AYNI ölçüt.) |
| `_safety_unavailable` | `(kind: str)` | 1063 | Güvenlik kontrolü koşmadı → dürüst 'unavailable' yanıtı (asla yeşil ✓ değil). |
| `_dose_check_sync` | `(request, body, lang)` | 1070 |  |
| `async` `api_dose_check` 🌐 | `(request: Request)` | 1112 | Yanıttaki dozları ayrı dar ajanla denetle (ikinci-göz güvenlik; kotasız). |
| `_redflag_sync` | `(request, body, lang)` | 1119 |  |
| `async` `api_redflag` 🌐 | `(request: Request)` | 1152 | Kaçmış aciliyet örüntüsü tara (triyaj rozeti; tanı değil; kotasız). |
| `async` `api_whatmissed` 🌐 | `(request: Request)` | 1160 | 'Ne kaçırdım?' — yanıtın atladığı klinik noktaları listeler (Sonnet, 1 kota). |
| `_whatmissed_sync` | `(request, body, lang)` | 1167 |  |
| `async` `api_drug_card` 🌐 | `(request: Request)` | 1198 | İlaç kartı — tek ilaç için yapılandırılmış atıflı klinik kart. |
| `_drug_card_sync` | `(request, body, lang)` | 1205 |  |
| `_heartbeat_stream` | `(work, err_msg: str)` | 1264 | Uzun senkron LLM işini arka-plan thread'inde koştur; 10 sn'de bir {'ping':1} NDJSON kalp |
| `async` `api_deep_analyze` 🌐 | `(request: Request)` | 1294 | Derin Analiz — max-efor Opus yanıt → critic → koşullu revize. 1 kota (critic/revize içeride, kotasız). |
| `_deep_analyze_sync` | `(request, body, lang)` | 1301 |  |
| `async` `api_simplify` 🌐 | `(request: Request)` | 1386 | Hasta İçin Sadeleştir — yanıtı hasta diline çevir (Haiku, kotasız). |
| `_simplify_sync` | `(request, body, lang)` | 1393 |  |
| `async` `api_council` 🌐 | `(request: Request)` | 1427 | Konsey (eski adı Doktor Paneli) — triyaj → 3 branş → moderatör. (~4 kota içeride sayılır.) |
| `_council_sync` | `(request, body, lang)` | 1434 |  |
| `api_case_timeline` 🌐 | `(cid: int, request: Request)` | 1505 | Hasta Zaman-Tüneli — kayıttan kronolojik trend özeti (Sonnet, 1 kota). |
| `api_mycases` 🌐 | `(request: Request)` | 1576 | Composer hasta seçici için hafif liste (id, code) + ÇAĞIRAN HEKİMİN KİMLİĞİ. |

## `saglik/app/chat_turn.py`

`337 satır` · `19 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
SOHBET TURU — ÜRETİM BAĞLANTIDAN AYRIK + KOPAN İSTEMCİNİN TOPARLANMASI (#748b, founder 06.09).

⚠⚠ NEDEN VAR (ölçülen mekanizma, Ömer): hekim telefonda soruyu sorup yanıtı beklerken sekmeyi
  arka plana alıyor; iOS Safari birkaç saniyede, Android sekme donunca `fetch('/api/chat')`
  KOPUYOR. Eski kodda kopma = iptal sayılıyordu: `GeneratorExit` → `_settle_abort` → üretim
  BIRAKILIYOR, yanıt HİÇ kaydedilmiyor, hekim aynı soruyu ikinci kez soruyor → para İKİ KEZ
  yanıyor (yarım + tam), üstelik `tekrarFreni` bir de "aynı soruyu mu gönderiyorsunuz" diyordu.
  KARAR: **bağlantı kopması ≠ iptal.** Hekimin AÇIK "Durdur"u iptaldir, sekme arka planı değil.

NE YAPAR (üç parça, hepsi bu dosyada — chat_routes yalnız BAĞLAR):
  1. `ayrik_akis(uret, turn, …)` — üretimi ayrı bir THREAD'de koşturur (`uret(emit)`), HTTP
     üreteci bir `queue.Queue`dan okuyup `yield` eder. İstemci koparsa (`GeneratorExit`) üreteç
     yalnız LOG basıp döner; **thread devam eder, `done`a ulaşır, `add_message` + `record_usage`
     GERÇEK usage ile yazılır.** Thread KENDİ `connect()`ini açar (chat_routes `_uret` içinde).
  2. `app.chat_turn` satırı (şema `db/app_schema.sql`): `uretiliyor → bitti | hata | iptal`.
     ⚠ İÇERİK YOK — yalnız kimlikler. ⚠ Çok worker → durum YALNIZ DB'de, süreç-içi sözlük YASAK
     (Durdur isteği başka worker'a düşebilir). ⚠ CASCADE AÇIK (hesap yaşam döngüsü kuralı).
  3. İki ücretsiz uç (`_doctor` ister, `_erisim_402`e GİRMEZ — LLM'siz, kotasız):
     `POST /api/chat/durdur {turn}` → hekime ait satırı `iptal` yapar (yabancı → 404);
     `GET /api/chat/turn/{turn}` → `{durum, thread_id, message_id}` (yabancı → 404).
     + istemci JS (`toparla_js()`): `/chat` sayfasına `_CHAT_BODY`den SONRA enjekte edilir.

⚠⚠ İNVARİANTLAR — ölçülmüş hatalardan doğdu, BOZMA:
  · **`_settle_abort` + "iptalde de `record_usage`" KORUNDU:** artık `GeneratorExit`te değil,
    thread `iptal` durumunu görünce çağrılır (`IptalYoklayici`, ~2 sn'de bir, delta aralarında).
    Yoklama `respond_stream`in `ping`lerinde de koşar → model düşünürken de Durdur işler.
  · **`{"turn": …}` satırı ilk `delta`dan ÖNCE gelir** — `hasta_dustu` varsa O İLK kalır (#679b-2
    founder kökenli, `hasta_baglam_dustu_verify` "İLK satır" der; Ömer kararı 06.09), `turn` hemen
    ardından. İstemci `turn`ü SIRAYLA değil ANAHTARLA arar; bilinmeyen anahtar yok sayılır (ping
    deseni) → eski istemci kırılmaz, yeni istemci `localStorage klv_turn` yazar.
  · **`bitti` işareti `done` satırından ÖNCE yazılır** — istemci `done`u görmeden `bitti`yi
    görebilir ama tersi olmaz (toparlama `bitti` bekler). ⚠ `store.add_message` kendi commit'ini
    attığı için işaret AYNI transaction'da DEĞİL, hemen sonraki commit'te; aradaki ms'lik pencerede
    çökme = satır `uretiliyor` kalır, istemci 5 dk sonra `chatFail` basar, mesaj yine de threadde.
  · **24 saatten eski satır okumada `hata`** (`oku`) ve açılışta best-effort silinir (`ac`).
    Sabit +03 ofset YAZILMAZ, `now()` ile karşılaştırılır.
  · **Durdur = önce `POST /api/chat/durdur` (keepalive), SONRA `ctl.abort()`** (`gonderimKilidi`).
    Sıra ters olsaydı abort'tan sonra sayfa kapanınca durdur isteği hiç gitmeyebilirdi.
  · **Toparlama yalnız `visibilityState==='visible'` iken sorar** (gizliyken bekler) — arka
    planda 2 sn'de bir istek atmak tam da kaçındığımız pil/ağ yüküdür.
  · **Toparlamada yanıt `openThread(thread_id)` ile çizilir** — kaynak/çip/rozet mevcut yol,
    ikinci bir çizim yolu YOK (tek kopya). `refused` turunda thread yok → hekime açıkça söylenir.

⚠ NE ÖLÇMEZ / AÇIK UÇLAR: worker'ın kendisi ölürse (deploy, OOM) thread de ölür, satır
  `uretiliyor` kalır → istemci 5 dk bekler. Kapı `scratchpad/akis_kopma_verify.py`.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 62 |
| `DURUMLAR` | `('uretiliyor', 'bitti', 'hata', 'iptal')` | 78 |
| `IPTAL_YOKLAMA_SN` | `2.0` | 79 |
| `ESKI_SATIR` | `'1 day'` | 80 |
| `THREAD_ON_EK` | `'klv-turn-'` | 170 |
| `_TOPARLA_JS` | `"<script>\n/* #748b AKIŞ KOPMA TOPARLAMA — gerekçe ve invariantlar saglik/app/chat_turn.py başlığında. */\nvar TOPARLA_M` | 267 |

### `class IptalYoklayici(—)`  <sub>satır 149</sub>

Thread içinde, delta aralarında çağrılır: en sık `IPTAL_YOKLAMA_SN`de bir DB okur.

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self, conn, turn: str, aralik: float = IPTAL_YOKLAMA_SN)` | 154 |  |
| `iptal_mi` | `(self) -> bool` | 158 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 65 | `@app.get` yerini tutar — `app` bu modülde YOK (chat_routes ile aynı desen; include_router YASAK). |
| `kur` | `(app) -> None` | 73 |  |
| `yeni_kimlik` | `() -> str` | 83 |  |
| `ac` | `(conn, doctor_id: int, turn: str) -> None` | 88 | Satırı `uretiliyor` olarak açar; önce 1 günden eski satırları best-effort temizler |
| `durum` | `(conn, turn: str) -> str \| None` | 100 |  |
| `isaretle` | `(conn, turn: str, yeni: str, thread_id = None, message_id = None) -> None` | 105 |  |
| `isaretle_sessiz` | `(conn, turn: str, yeni: str, thread_id = None, message_id = None) -> bool` | 113 | Hata dallarında transaction zehirli olabilir → önce rollback, sonra işaret; patlarsa |
| `durdur` | `(conn, doctor_id: int, turn: str) -> str \| None` | 128 | Hekime ait satırı `iptal` yapar (yalnız `uretiliyor` ise). Döner: güncel durum \| None (yabancı/yok). |
| `oku` | `(conn, doctor_id: int, turn: str) -> dict \| None` | 138 | `{durum, thread_id, message_id}` \| None (yabancı/yok). 1 günden eski satır `hata` okunur. |
| `ayrik_akis` | `(uret, turn: str, doctor_id, hata_satiri: str)` | 173 | `uret(emit)`i ayrı thread'de koşturur; HTTP üreteci kuyruktan okur. |
| `thread_bekle` | `(zaman_asimi: float = 15.0) -> int` | 207 | Kapılar için: `klv-turn-*` thread'lerini join eder, sayısını döner. Üründe çağrılmaz. |
| `_hata` | `(lang: str, kod: int) -> JSONResponse` | 218 |  |
| `async` `api_chat_durdur` 🌐 | `(request: Request)` | 225 | Durdur tuşu: hekime ait `uretiliyor` satırı `iptal` olur → thread bir sonraki yoklamada |
| `_durdur_sync` | `(request, turn: str, lang: str)` | 235 |  |
| `api_chat_turn` 🌐 | `(turn: str, request: Request)` | 249 |  |
| `toparla_js` | `() -> str` | 313 | `/chat` sayfasına `_CHAT_BODY`den SONRA eklenir (chat_body fonksiyonlarını çağırır). |
| `turn_satiri` | `(turn: str) -> str` | 318 | `turn` satırı (ilk delta'dan ÖNCE; `hasta_dustu` varsa ondan SONRA) — tek kaynak. |
| `hata_satiri` | `(lang: str) -> str` | 323 | Thread beklenmedik patlarsa istemciye giden satır (UI dili). |
| `hata_yazici` | `(conn, turn: str, emit, line)` | 331 | `hata(obj)` üretir: turn `hata` + hata satırı — toparlayan istemci `bitti` beklemesin. |

## `saglik/app/cihaz_routes.py`

`634 satır` · `19 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
CİHAZ EŞLEŞTİRME — masaüstü programın Klivance kimliği (#596b, 2026-09-01).

Rotalar (6): `POST /api/cihaz/istek` · `POST /api/cihaz/bekle` · `GET /cihaz` ·
`POST /cihaz` · `POST /cihaz/sil` · `POST /api/cihaz/oturum`.

⚠⚠ ÇÖZDÜĞÜ İKİ SORUN (founder 2026-09-01):
  1. **Google ile kaydolan hekimin BİLDİĞİ bir parolası yoktur.** `upsert_google_doctor`
     `password_hash`e rastgele bir değer yazar — NULL değil, ama hekim onu bilmez. Masaüstü
     program e-posta+parola istiyor ⇒ o hekim için ÇIKMAZ. (Programda "Google ile devam et"
     de çalışmıyor: yerelden giden `redirect_uri` Google Cloud istemcisine kayıtlı değil.)
  2. **`SESSION_TTL_DAYS` MUTLAK ömürdür** — `create_session` `expires_at`i BİR KEZ yazar,
     `doctor_for_token` yalnız `last_seen` günceller, UZATMAZ. ⇒ parolası olan hekim bile
     programa 7 günde bir yeniden girer. Asıl kazanç burada: cihaz sırrı varken program
     401 gördüğü an sessizce yeni oturum alır, hekim hiçbir şey yapmaz.

⚠⚠ NEDEN "MAGIC LINK" DEĞİL — ÖLÇÜLDÜ, MİMARİ §4'TEN AYRILDI:
  Maildeki link **varsayılan tarayıcıda** açılır; oluşan çerez sistem tarayıcısının
  kavanozuna düşer, programın `persist:klivance` kabinine DEĞİL. Yani link tıklanır ve
  program hâlâ dışarıda kalır — mail ayağı hiçbir işi çözmez, yalnız yeni yüzey açar
  (mail tasarımı + founder onay döngüsü + hesap numaralandırma ucu + kurumsal posta
  tarayıcısının linki GET'leyip SESSİZCE yetkilendirmesi).
  ⇒ YÖN TERSİNE ÇEVRİLDİ: kod **programda** doğar, hekim **tarayıcıya** yazar.
  Posta kutusu saldırganı sınıfının TAMAMI böylece konusuz kalır.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur). `@_rota` + `kur(app)`;
  **`include_router` KULLANILMAZ** — bu FastAPI sürümünde rotaları `app.routes`a
  düzleştirmez ve rota envanterini oradan okuyan güvenlik ağları sessizce körelir.

⚠ İPTAL AUTH.PY'DE: `auth.revoke_devices()` — gerekçesi orada yazılı (dairesel import
  + iptali kimliği düşüren kodun yanında tutmak). Bu modül onu ÇAĞIRMAZ, auth çağırır.

Kapı: `scratchpad/cihaz_eslesme_verify.py`.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ISTEK_DK` | `10` | 56 |
| `CIHAZ_GUN` | `90` | 57 |
| `CIHAZ_OTURUM_GUN` | `1` | 58 |
| `MAX_CIHAZ` | `5` | 59 |
| `MAX_BEKLEYEN` | `30` | 68 |
| `CIHAZ_UI` | `os.getenv('KLIVANCE_CIHAZ_UI', '').strip() == '1'` | 78 |
| `_CIHAZ_UI_HEKIM` | `{e.strip().lower() for e in os.getenv('KLIVANCE_CIHAZ_UI_HEKIM', '').split(',') if e.strip()}` | 88 |
| `_ALFABE` | `'23456789ABCDEFGHJKMNPQRSTVWXYZ'` | 114 |
| `KOD_UZUNLUK` | `8` | 115 |
| `KAYIT` | `[]` | 143 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `cihaz_ui_gorunur` | `(d) -> bool` | 92 | Bu hekim cihaz eşleştirme yüzeylerini GÖRSÜN mü. |
| `_kod_uret` | `() -> str` | 118 |  |
| `kod_normalize` | `(s: str) -> str` | 122 | Hekimin yazdığını kanonikleştir: boşluk/tire at, büyüt, karıştırılan harfleri eşle. |
| `_ozet` | `(s: str) -> str` | 131 |  |
| `_hex_mi` | `(s: str) -> bool` | 135 | İstemcinin gönderdiği `istek_ozet` gerçekten bir SHA-256 hex özeti mi. |
| `_rota` | `(metod: str, yol: str, **kw)` | 146 |  |
| `kur` | `(app) -> None` | 153 |  |
| `temizle` | `(conn) -> int` | 159 | Fırsatçı temizlik: süresi geçmiş bekleyen istekler + ömrü dolmuş cihazlar. |
| `_cihaz_listesi` | `(conn, doctor_id: int) -> list[tuple]` | 179 |  |
| `async` `cihaz_istek` 🌐 | `(request: Request)` | 189 | Program bir eşleştirme isteği açar; ekranda gösterilecek KODU döner. |
| `_istek_sync` | `(ozet: str, ad: str, ip: str) -> tuple[int, dict]` | 215 |  |
| `async` `cihaz_bekle` 🌐 | `(request: Request)` | 254 | Program 2 sn'de bir yoklar. Onaylandıysa CİHAZ SIRRINI **bir kez** döner. |
| `_bekle_sync` | `(oz: str) -> dict` | 275 |  |
| `_sayfa` | `(doctor, lang: str, govde: str) -> str` | 303 | Sayfa kabuğu — `/account` ile AYNI görsel dil (`.stage/.content narrow`). |
| `cihaz_sayfa` 🌐 | `(request: Request, h: str = '', g: str = '', k: str = '')` | 320 | Onay sayfası (`?k=` ile TEK TIK) + elle kod formu + bağlı cihaz listesi. |
| `cihaz_onayla` 🌐 | `(request: Request, kod: str = Form(''))` | 521 | Hekim programdaki kodu buraya yazar; istek ONAYLANIR. |
| `cihaz_sil` 🌐 | `(request: Request, cid: str = Form(''))` | 565 | Bağlantıyı kes — cihaz satırı VE onun bastığı oturumlar düşer. |
| `async` `cihaz_oturum` 🌐 | `(request: Request)` | 590 | Cihaz sırrı → kısa ömürlü `app.session` çerezi. |
| `_oturum_sync` | `(oz: str) -> str \| None` | 621 |  |

## `saglik/app/credits.py`

`495 satır` · `12 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Erişim modeli — SINIRSIZ ABONELİK (2026-08-21 founder kararı, kredi sistemi kaldırıldı).

⚠⚠ BU DOSYA 2026-08-21'e KADAR **KREDİ TARİFESİYDİ** (özellik başına ağırlık, iki cüzdan,
top-up paketleri, yıllık bonus). Founder kararıyla kredi sayacının TAMAMI kaldırıldı:

    aktif abonelik  → SINIRSIZ kullanım
    yeni kayıt      → DENEME_GUN gün deneme, günde AGIR_GUNLUK_TAVAN_TRIAL soru
                      (founder 2026-09-02: "deneme üye olanlara günde 12 soru ücretsiz";
                      2026-08-21..09-02 arası deneme de sınırsızdı, tavan yalnız e-postası
                      doğrulanmamış hesaba dardı)
    ikisi de yoksa  → duvar (402)

Adı `credits.py` KALDI çünkü 30 dosya buradan import ediyor ve yeniden adlandırmanın
ölçülmüş getirisi yok; içeriği erişim modelidir, kredi tarifesi değil.

⚠ **KALDIRILAN ÜRÜN KARARLARI** (founder'a bildirildi, sessizce düşürülmedi):
· **Yıllık bonus +100 kredi** (#140b, 2026-08-09) — kredi kalmayınca karşılığı yok.
  Yıllığın avantajı yine FİYAT (12 ay kullan 10 ay öde). `donem_yillik_mi` KALDI:
  bonusu değil `current_period_end`e eklenecek ARALIĞI (1 ay / 1 yıl) belirliyor.
· **Top-up paket satışı** — sınırsızda satın alınacak kredi yoktur.
· **Özellik başına ağırlık** (`ENDPOINT_CREDITS`) — konsey/derin analiz artık "3 kredi"
  değil, aboneliğin içinde. Maliyet kuyruğunu KREDİ değil GÜNLÜK TAVAN frenler (aşağı).

⚠⚠ **ŞEMADAN HİÇBİR KOLON DÜŞÜRÜLMEDİ** (`topup_balance`, `gift_balance`,
`monthly_quota`, `app.quota_reservation`, `app.topup_grant`, `app.usage.credits`).
Yeni kod bunları okumaz/yazmaz; DROP etmemek geri dönüş maliyetini sıfırda tutar ve
`app.usage` maliyet ölçümünü bozmaz. **`record_usage` AYNEN sürüyor** — kredi kalksa da
"iptalde de usage yaz, yoksa kaçak ÖLÇÜLEMEZ" invariantı geçerli.

⚠ MARJ NOTU (devralındı, çürümedi): `credits.py`nin eski başlığı p95 chat maliyetini
0,292 USD/kredi ölçmüştü — yani 100 kredilik planda ağır bir abone zarar ettiriyordu.
Sınırsızda o kuyruk **KREDİYLE SINIRLANMIYOR**, tek fren aşağıdaki günlük tavandır.
"Her abone kârlı" DENMEZ; tavan sayısını değiştiren `app.usage`tan gerçek dağılımı ölçsün.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `DENEME_GUN` | `15` | 41 |
| `DENEME_TEKRAR_GUN` | `0` | 48 |
| `AGIR_GUNLUK_TAVAN` | `40` | 68 |
| `AGIR_GUNLUK_TAVAN_TRIAL` | `12` | 69 |
| `DENEME_GUNLUK_SORU` | `AGIR_GUNLUK_TAVAN_TRIAL` | 72 |
| `AYLIK_ALARM_ESIGI` | `400` | 108 |
| `AYLIK_ALARM_GUN` | `30` | 109 |
| `AGIR_MODLAR` | `('clinical', 'interaction', 'drug', 'hastalik', 'timeline', 'etkilesim_yorum', 'aborted', 'goruntu')` | 132 |
| `MAX_ANALYZE_FILES` | `25` | 143 |
| `MAX_ATTACH_COUNT` | `5` | 149 |
| `MAX_ONOKUMA_GORSEL` | `3` | 156 |
| `MAX_ONOKUMA_B` | `5 * 1024 * 1024` | 157 |
| `BUTCE_TL` | `{'abone': 800, 'deneme': 400}` | 178 |
| `BUTCE_KISMI_ESIK` | `0.8` | 183 |
| `FREN_YUZEY` | `frozenset({'chat_hazirlik', 'whatmissed', 'drug_card', 'deep_analyze', 'council', 'api_case_timeline', 'analyze_case', '` | 201 |
| `REZERV_USD` | `{'chat_hazirlik': 0.75, 'hastalik': 1.0, 'chat_ek': 1.75, 'whatmissed': 0.3, 'drug_card': 0.5, 'deep_analyze': 1.0, 'cou` | 219 |
| `REZERV_BILINMEYEN` | `max(REZERV_USD.values())` | 233 |
| `PAHALI_UCLAR` | `frozenset({'council', 'deep_analyze', 'analyze_case', 'api_case_timeline', 'etk_yorum'})` | 238 |
| `REZERV_MODLAR` | `tuple((f'rezerv_{y}' for y in sorted(FREN_YUZEY)))` | 244 |
| `REZERV_OMUR_DK` | `10` | 249 |
| `KUR_BAYAT_GUN` | `7` | 262 |
| `KUR_BAYAT_CARPAN` | `1.1` | 263 |
| `KUR_YOK_CARPAN` | `1.5` | 264 |
| `KUR_YOK_TABAN` | `40` | 265 |
| `SATILABILIR_PLANLAR` | `('aylik', 'yillik')` | 272 |
| `KURUCU_LIMIT` | `0` | 274 |
| `LEGACY_PLANLAR` | `('kurucu', 'kurucu_yillik')` | 283 |
| `IYZ_BASARISIZ_EK_GUN` | `0` | 295 |
| `IYZ_DONUSUM_PENCERE_GUN` | `3` | 296 |
| `IYZ_WEBHOOK_TOLERANS_SAAT` | `24` | 303 |
| `BUTCE_MOD_GOZLEM` | `'gozlem'` | 379 |
| `BUTCE_MOD_UYGULA` | `'uygula'` | 380 |
| `IFSA_MADDE_TR` | `('toplam kullanım hacmi', 'dönem')` | 413 |
| `IFSA_MADDE_EN` | `('usage volume', 'period')` | 414 |
| `_BUTCE_RED_SON` | `None` | 437 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `iyz_recurring` | `() -> bool` | 306 | iyzico recurring abonelik yolu AÇIK mı? Env `KLIVANCE_IYZ_RECURRING` (1/true/on). |
| `goruntu_onokuma_acik` | `() -> bool` | 312 | Görüntü ön-okuma (#620b YOL 2) AÇIK mı? Env `KLIVANCE_GORUNTU_ONOKUMA` (1/true/on). |
| `donem_yillik_mi` | `(interval: str \| None, plan: str = '') -> bool` | 323 | Bu satın alım YILLIK bir dönem mi? — **TEK DOĞRU KAYNAK**. |
| `deneme_ilan` | `(lang: str = 'tr') -> str` | 341 | Denemenin hekime GÖSTERİLEN ifadesi — "N gün ücretsiz, günde M soru". |
| `adil_kullanim_ibaresi` | `(lang: str = 'tr') -> str` | 364 | "Sınırsız" ilanının YANINDA görünmesi ZORUNLU olan ibare. |
| `butce_modu` | `() -> str` | 383 | Bütçe kapısının modu: `"gozlem"` \| `"uygula"` — env `KLIVANCE_BUTCE_MOD`. |
| `_duz_metin` | `(d) -> str` | 417 | `_LEGAL`/`LEGAL_EN` SÖZLÜKTÜR ve değerleri (başlık, gövde) TUPLE — `'x' in d` anahtarda |
| `butce_ifsa_var` | `() -> tuple[bool, str]` | 424 | `legal_body` TR+EN'de dönem toplam kullanım hacmi maddesi var mı → (var, eksik). |
| `_butce_red_logla` | `(eksik: str) -> None` | 440 |  |
| `butce_mod_bilgi` | `() -> dict` | 449 | Görünürlük: {"istenen": env değeri, "etkin": butce_modu(), "red": eksik\|None}. |
| `kur_eff` | `(bilgi: dict \| None) -> tuple[float, str]` | 460 | `store.kur_bilgisi()` çıktısından ETKİN kur → `(kur_eff, etiket)`; etiket ∈ |
| `rezerv_usd` | `(yuzey: str \| None) -> float` | 479 | Yüzeyin rezervi; boş/bilinmeyen/`diger` → REZERV_BILINMEYEN (en pahalı, #138b deseni). |

## `saglik/app/deger_onay.py`

`176 satır` · `5 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ANALİZDEN ÇIKAN DEĞERİ KARTA İŞLEME — ONAYLI (#362b, founder kararı (a) 2026-08-19).

Zincir bugüne dek KIRIKTI: tahlil → analiz metni → **⛔** → hasta kartı → tetik → uyarı.
Bu modül eksik oku kapatır ve kapatırken founder'ın seçtiği yolu uygular: **onaylı işleme**.

  1. `/cases/{cid}` yüklenince dosya çıkarımlarında (`app.case_file.extracted_text`)
     kart alanına yazılabilir sayısal değer aranır (`cds.deger_cikar` — SAF, yazmaz).
  2. Karttakinden FARKLI olanlar bir ŞERİT olarak sunulur: "Analiz eGFR 22 buldu —
     karta işlensin mi?" + değerin geçtiği SATIR (kanıt).
  3. Hekim onaylarsa `POST /api/cases/{cid}/deger-onayla` YALNIZ o alanı yazar ve
     `?kontrol=1#recete` ile döner → **mevcut #355b tetiği** devreye girer, reçete
     kontrolü kendiliğinden koşar. Zincir böylece uçtan uca kapanır.

⚠⚠ NEDEN ANALİZ UCUNA DEĞİL SAYFA YÜKÜNE BAĞLI: analiz bitince istemci zaten
   `location.href="/cases/{id}"` yapıyor (bkz. `cases_body.libAnalyze`). Şeridi sayfa
   yükünde üretmek "yeni tahlil düştüğünde"yi KARŞILAR ve analiz yolunun (kredi rezervi,
   threadpool, kısmi-atlama notu) hiçbir satırına dokunmaz — para yolu korunur.

⚠⚠ OTOMATİK YAZMA YOK. Hekimin girmediği bir sayıyı klinik kayda işlemek onun kararıdır;
   yanlış çıkarım (yanlış birim, eski tarihli tahlil, başka hastanın belgesi) sessizce
   kalıcı olurdu. Öneri DAİMA kanıt satırıyla gelir.

⚠ ÇELİŞKİDE ÖNERİ YOK: aynı parametre için farklı değerler varsa `deger_cikar` hiçbirini
  önermez (`belirsiz`) — şerit o alanı hiç göstermez. Tahmin, yanlış sayı demektir.

⚠ Bu modül `main.py`den ve `cases_routes`tan HİÇBİR ŞEY import ETMEZ; `cases_routes`
  BENİ import eder (tek yön).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 39 |
| `_M` | `{'tr': {'h': 'Analiz bu tahlillerde kart alanına yazılabilir değer buldu', 'not': 'Bunlar hekimin girdiği veri DEĞİLDİR;` | 41 |
| `_ETIKET` | `{'egfr': ('eGFR', 'eGFR'), 'kreatinin': ('Kreatinin', 'Creatinine'), 'kilo_kg': ('Kilo', 'Weight'), 'boy_cm': ('Boy', 'H` | 56 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(yontem: str, yol: str, **kw)` | 60 |  |
| `kur` | `(app) -> None` | 67 |  |
| `oneriler` | `(conn, doctor_id: int, case_id: int, case_ctx: dict) -> list[dict]` | 72 | Bu hastanın dosya çıkarımlarından gelen, karttakinden FARKLI değerler. |
| `serit` | `(oneri: list[dict], cid: int, lang: str) -> str` | 96 | Onay şeridi. Öneri yoksa BOŞ dize (şerit hiç çizilmez). |
| `deger_onayla` 🌐 | `(cid: int, request: Request, alan: str = Form(''), deger: str = Form(''))` | 128 | YALNIZ onaylanan alanı yazar; kartın geri kalanı AYNEN korunur. |

## `saglik/app/enabiz_tahlil.py`

`493 satır` · `18 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
e-NABIZ TAHLİL PAKETİ — HTML (akordiyon) → hasta kartı "belge" → analiz/kokpit (#668b).

⚠⚠ FOUNDER KURALI (03.09.2026, kendi e-Nabız hesabında gösterdi): e-Nabız'ın ürettiği
   tahlil PDF'inde BAZI DEĞERLER YOK; akordiyon açılınca satırda (Sonuç/Birim/Referans)
   YAZIYOR. PDF = eksik kopya, sayfa HTML'i = tam veri. Tahlil DEĞERLERİ daima HTML'den
   alınır; PDF yalnız belge olarak kalır; PDF'te olmayan parametre "yok" SAYILMAZ.

NE YAPAR: eklenti (`klivance.js`) `cikarici.tahlil` çıktısını (#621b ile sayfa
sayacıyla birebir) `POST /api/cases/{cid}/enabiz-tahlil` ucuna JSON olarak gönderir.
Sunucu paketi DOĞRULAR/normalize eder, `app.case_file`e ŞİFRELİ bir "belge" olarak
yazar (Fernet, sha256 tekilliği #619b, soft-delete, CASCADE — hepsi MEVCUT desen),
AYNI hastanın ESKİ e-Nabız paketlerini düşürür (#668b(c), `_eski_paketleri_dusur`:
iki paket = REDUCE'a çift sayım; yeni paket tek kaynak) ve
`extracted_text`i ÖNCEDEN doldurur → MAP (Sonnet) ATLANIR (0 kredi, 0 çağrı), REDUCE
paketi `<dosya_ozetleri>`nde görür, "Laboratuvar seyri" oradan kurulur, kokpit onu
çizer. Chat kolu da aynı metni `list_case_lab_summaries` ile alır.

⚠⚠ NEDEN BU TASARIM (üç seçenek ÖLÇÜLDÜ, 03.09):
  · narrative'e yazmak — `narrative` `saglik/cds/` altında 0 çağrı (eslestir.js kendi
    yorumunda da ölçülü): modele HİÇ ulaşmaz → elendi.
  · yeni jsonb kolon/tablo — şema + arşiv (`to_jsonb(d)`) kararı + `_case_ctx_block`
    yeni tüketici + KVKK ifşa metni (Selim) + düz metin at-rest; 5+ dosya.
  · ÖNCEDEN DOLU "belge" (bu) — sıfır şema, şifreli at-rest (PDF ile aynı sınıf → ifşa
    metni yeni kategori AÇMAZ: "hastaya bağlı tahlil dosyaları kalıcı+şifreli"), mevcut
    silme/KVKK/arşiv yolları, analiz+chat tüketicileri ZATEN bağlı. `_sniff_mime` JSON'u
    reddettiği için `/files` ucu kullanılamaz → bu uç.

⚠ BÜTÇE ÖLÇÜLDÜ (gerçek paket, founder'ın 17.08 çekimi, panel alt parametreleri
  #621b desenine göre açılmış): sayılar `scratchpad/enabiz_tahlil_paketi_verify.py`
  başlığında ve `_kamil_bulgu.md`de. `OZET_TAVAN_KRK` o ölçümden: tam gerçek paket
  tavanın ALTINDA kalır (kart kırpılmaz); daha büyük paketlerde EN YENİ kartlar tam,
  eskiler yalnız referans dışı (+ "N parametre belgede" sayısı) — SESSİZ kırpma yok,
  başlık satırı kaç kartın tam olduğunu söyler.

⚠ `kisaltildi_mi` (analyze.py) `len>=1200` ile "kısaltıldı" der; bu metin bilerek uzun →
  `extracted_model == ENABIZ_MODEL` ile MUAF (sahte "özet kısaltıldı" etiketi basılmasın).
  ⚠ Muafiyet ÖNEKE DEĞİL MODEL KOLONUNA bağlı (Fırat P2): önek belgeye gömülü metinle
  taklit edilebilirdi; model kolonunu yalnız bu uç yazar.

⚠ HÜKÜM ÜRETİLMEZ: bayrak yalnız e-Nabız'ın kendi sınıfından ya da cikarici'nin
  sayfanın kendi referans aralığından türettiği (#621b) `normal=false`tan gelir;
  `normal=null` → bayrak YOK ("hüküm yok" ≠ "normal"). Sunucu aralık HESAPLAMAZ.

⚠ Rota kaydı `@_rota` + `kur(app)` (include_router ölçülerek reddedildi, bkz. main.py).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 62 |
| `KART_TAVAN` | `100` | 79 |
| `PARAM_TAVAN` | `3000` | 80 |
| `GOVDE_TAVAN_B` | `1000000` | 81 |
| `OZET_TAVAN_KRK` | `24000` | 82 |
| `REF_DISI_LISTE_TAVAN` | `60` | 83 |
| `_REF_BLOK_TAVAN` | `4000` | 84 |
| `_BASLIK_PAYI` | `500` | 85 |
| `_KUYRUK_PAYI` | `120` | 86 |
| `CHAT_TAVAN_KRK` | `4200` | 87 |
| `_KLIP` | `{'ad': 90, 'sonuc': 40, 'birim': 30, 'referans': 48, 'grup': 60, 'kurum': 80}` | 88 |
| `_TARIH` | `re.compile('\\b(\\d{2})\\.(\\d{2})\\.(\\d{4})\\b')` | 90 |
| `_BASLIK_ON` | `re.compile('^\\s*\\d{1,2}\\s+\\S+\\s+\\d{4}\\s+\\d{2}\\.\\d{2}\\.\\d{4}\\s*')` | 91 |
| `_BASLIK_SON` | `re.compile('\\s+(\\d+\\s+Normal\\b\|\\d+\\s+Referans\\s+D\|PDF\\s*\\().*$', re.S)` | 92 |
| `_TEMIZ_ARALIK` | `re.compile('^-?(?:\\d+(?:[.,]\\d+)?\|[.,]\\d+)\\s*-\\s*-?(?:\\d+(?:[.,]\\d+)?\|[.,]\\d+)$')` | 93 |
| `_FE_ADLARI` | `frozenset({'demir (serum)', 'demir (fe)', 'demir', 'serum demir', 'fe', 'iron'})` | 193 |
| `_TEK_SAYI` | `re.compile('^-?(?:\\d+(?:[.,]\\d+)?\|[.,]\\d+)$')` | 194 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 65 |  |
| `kur` | `(app) -> None` | 72 | main.py TEK KEZ çağırır (`_cases_kur(app)`in hemen altında). |
| `_s` | `(v, n: int) -> str` | 96 |  |
| `_tarih` | `(baslik: str) -> str` | 101 |  |
| `_tarih_anahtar` | `(t: str)` | 106 |  |
| `_kurum` | `(baslik: str) -> str` | 111 |  |
| `paket_dogrula` | `(govde) -> tuple[dict \| None, str]` | 117 | İstemci JSON'u → normalize paket ya da (None, hata_kodu). |
| `_tr_kucuk` | `(s: str) -> str` | 197 |  |
| `_sayi` | `(s: str)` | 203 |  |
| `_demir_sinifi` | `(ad: str)` | 213 |  |
| `_etiket_kontrol` | `(satirlar: list) -> list[str]` | 222 | Bir kartın satırlarında Fe/UIBC/TIBC üçlüsü varsa aritmetiği sına. Tutarsızsa UIBC/TIBC |
| `_deger` | `(p: dict) -> str` | 248 |  |
| `_kart_satirlari` | `(k: dict) -> list[str]` | 264 | Bir kart → '- TARİH \| a: v u (ref); b: …' satırları (grup başına bir satır). |
| `ozet_metni` | `(paket: dict) -> tuple[str, dict]` | 274 | Normalize paket → extracted_text. Döner (metin, sayim). |
| `chat_metni` | `(metin: str, lang: str = 'tr') -> str` | 385 | Chat kolu (`chat_routes._build_case_record`) için KOMPAKT kesim (Fırat P1-b). |
| `async` `api_enabiz_tahlil` 🌐 | `(cid: int, request: Request)` | 414 | Eklentiden gelen tahlil paketi → şifreli belge + önceden dolu çıkarım (LLM YOK). |
| `_kaydet_sync` | `(request: Request, cid: int, ham_govde: bytes, lang: str)` | 429 |  |
| `_eski_paketleri_dusur` | `(conn, doctor_id: int, cid: int, yeni_fid: int) -> int` | 467 | #668b(c): AYNI hastada yeni paket gelince ESKİ e-Nabız paketleri soft-delete edilir. |

## `saglik/app/etkilesim_body.py`

`901 satır` · `23 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
/etkilesim — HERKESE AÇIK ilaç etkileşimi tarama sayfası (giriş yok, kredi yok, LLM yok).

⚠⚠ NEDEN VAR (2026-07-30, ölçüldü): Google'da ilaç-etkileşimi kelimeleri 7 günde 334,80 TL
   yaktı (Google bütçesinin %42'si), kalite skorları 1/10 ve GA4'te o kelimeden gelen 6
   oturumun 0'ı etkileşimliydi (0,0 sn). Sebep: AG-Etkilesim'in 10 kelimesi de '/' anasayfaya
   iniyordu ve asıl araç `/reference` GİRİŞ DUVARININ arkasındaydı (ölçüldü: 303 → /giris).
   Kanıt ki desen çalışıyor: `/hesaplayicilar` (giriş istemeyen, işini yapan araç) google/cpc
   trafiğinde %33,3 etkileşimli ve 18,0 sn/kişi; '/' ise %10,5 ve 4,1 sn/kişi.

⚠⚠⚠ BU SAYFANIN MODAL SONUCU "İŞARET YOK"TUR — TASARIM BUNUN ÜZERİNE KURULUDUR.
   Ölçüldü: iki tarafı da klinik metinli 60 rastgele sık-TR-marka çiftinde sinyal %8;
   küratörlü (şüphe-güdümlü) çiftlerde bile 19/23. Yani ziyaretçilerin ÇOĞUNLUĞU "bulunamadı"
   ekranını görecek. Bu ekran bir YOKLUK olarak render edilirse hekim 3,8 saniyede çıkar ve
   sayfa reklam parasını anasayfadan DAHA HIZLI yakar. Bu yüzden "işaret yok" bir SONUÇ olarak
   basılır: NE tarandığı (taraf başına karakter + bölüm adları) ve resmî belge linkleri AYNI
   blokta verilir. Bu hem FDA Kriter-4 malzemesidir hem de doğrulanabilir bir NEGATİF üretir.

⚠⚠ MOTORUN `scanned` BAYRAĞI KULLANILMAZ — SAHTEDİR. `reference.py` içinde alan boş değilse
   `scanned=True` oluyor; DailyMed satırlarının `warnings` alanı yalnız bir URL taşıyor.
   Ölçüldü: coumadin kartı warnings=101 karakter (sadece link), diğer klinik alanlar 0 →
   `coumadin + majezik` (varfarin + NSAİİ = MAJOR KANAMA) "tarandı, bulgu yok" veriyordu.
   Bu sayfa `_taranan_klinik()` ile TARAF BAŞINA gerçek klinik karakteri sayar ve SAYIYI
   EKRANA BASAR. Durum GLOBAL DEĞİL TARAF BAŞINADIR: kör taraf bandı, bulunmuş bir sinyalle
   AYNI ANDA basılabilir (ölçüldü: coumadin+ibuprofen hit=2 iken coumadin tarafı 0 karakter —
   "bir taraf körse koşulsuz kontrol edilemedi" kuralı o iki kanama pasajını GİZLERDİ).

⚠ ALTI DURUM, HİÇBİRİ BİRLEŞTİRİLEMEZ ve hiçbirinde YEŞİL/✓ YOKTUR:
   ayni_madde · sinyal · zayif · isaret_yok · taranamadi(taraf) · calistirilamadi
   Sonuç bloğunun HTML'inde şu kelimeler YASAK (CI iddiası): ✓ ✔ temiz güvenli uyumlu
   "etkileşim yok" "birlikte kullanılabilir" "sorun yok".

⚠ MOTOR DEĞİŞMEZ: `reference.interaction_check` ücretli /chat yolunda 5 yerden çağrılıyor.
  Dürüstlük katmanının tamamı BU sayfa katmanında kurulur.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_BOLUM_AD` | `{'interactions': 'etkileşimler', 'contraindications': 'kontrendikasyonlar', 'boxed_warning': 'kutu uyarısı', 'warnings':` | 44 |
| `_BOLUM_AD_EN` | `{'interactions': 'interactions', 'contraindications': 'contraindications', 'boxed_warning': 'boxed warning', 'warnings':` | 46 |
| `_SECTION_ALAN` | `{'drug_interactions': 'interactions'}` | 54 |
| `_ISKELET` | `re.compile('^\\s*(etiket\|label\|kÜb\|küb\|kt\|kub)\\s*:', re.I)` | 56 |
| `_URL` | `re.compile('https?://\\S+')` | 57 |
| `_UYARI_FIIL` | `re.compile('(increas\|decreas\|risk\|avoid\|contraindicat\|monitor\|caution\|reduce\|inhibit\|induc\|potentiat\|bleed\|t` | 59 |
| `_UUID` | `re.compile('^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$', re.I)` | 181 |
| `DURUMLAR` | `('ok', 'calistirilamadi', 'tek_ilac', 'kisa', 'sinif_adi')` | 291 |
| `SEVIYELER` | `('hit_var', 'isaret_yok', 'ayni_madde', 'taranamadi')` | 292 |
| `_CIFT_PASAJ_TAVAN` | `3` | 557 |
| `_SAYFA_PASAJ_TAVAN` | `24` | 558 |
| `TARAMA_SQL_MS` | `3000` | 568 |
| `TARAMA_SON_TARIH_SN` | `6.0` | 569 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_taranan_klinik` | `(card: dict \| None) -> tuple[int, list[str]]` | 65 | Bu kartta GERÇEKTEN taranabilen klinik metin: (karakter, bölüm adları). |
| `_sayac_kat` | `(n: int, bolumler: list[str], kart: dict \| None) -> tuple[int, list[str]]` | 89 | Ek kanıt kartının taranan metnini sayaca KAT — bölüm listelerini BİRLEŞTİREREK. |
| `_kirp` | `(pasaj: str, terimler) -> str \| None` | 98 | Pasajı CÜMLE SINIRINA yuvarla; eşleşen terimi kaybediyorsa BASMA. |
| `_icerir` | `(metin: str, terimler) -> bool` | 165 |  |
| `_zayif` | `(pasaj: str) -> bool` | 170 | 'Adı bir listede geçiyor' mu, yoksa düzyazı uyarı mı? |
| `_belge_link` | `(card: dict \| None) -> tuple[str, str] \| None` | 184 | Resmî etiketin DOĞRULANABİLİR linki → (url, etiket) ya da None. |
| `kok_yedegi` | `(conn, ad: str, kart: dict \| None)` | 204 | Kart klinik metinsizse AYNI molekülün dolu satırını YEREL olarak ara. |
| `esc` | `(s) -> str` | 269 |  |
| `_T` | `(lang: str, tr: str, en: str) -> str` | 295 |  |
| `sayi` | `(lang: str, n: int) -> str` | 299 | Binlik ayracı DİLE bağlı: TR '5.502' · EN '5,502'. |
| `_sayac_satiri_n` | `(lang, kalemler) -> str` | 315 | NE tarandığı — N ilaç için tek satır. `kalemler` = [(ad, karakter, bölümler), …]. |
| `_sayac_satiri` | `(lang, ad_a, na, ba, ad_b, nb, bb) -> str` | 336 | İKİ İLAÇ sarmalayıcısı — `en_sayi_bicim_verify` bunu 7 POZİSYONEL argümanla |
| `_link_html` | `(kart, lang, ad = None) -> str` | 342 |  |
| `_linkler_html` | `(ciftler, lang) -> str` | 356 | Belge linkleri satırı — ilaç ADIYLA etiketli, URL'ye göre TEKİLLEŞTİRİLMİŞ. |
| `_etiket_bilesenleri` | `(h: dict, bad: dict) -> tuple[str, str]` | 380 | Bir hit'in (kaynak adı, bölüm adı) etiketi — TEK DİLDE, TEK KEZ. |
| `_pasajlar` | `(lang, cift, by_ad)` | 402 | Bir ÇİFTİN gösterilebilir pasajları → (guclu, zayifl). |
| `_bulgular` | `(lang, guclu, zayifl, tavan = None)` | 424 | Pasaj blokları — EN'de Türkçe kanıt SONA alınır. `tavan` = basılacak en çok pasaj. |
| `_cift_adi` | `(cift) -> str` | 490 |  |
| `_cift_listesi` | `(ciftler) -> str` | 494 |  |
| `sonuc_html` | `(lang, ad_a, ad_b, kart_a, kart_b, sonuc, durum = None) -> str` | 498 | İKİ İLAÇ sarmalayıcısı — çıktısı `sonuc_html_n` ile BAYT BAYT AYNIDIR. |
| `_sinif_html` | `(lang, siniflar) -> str` | 572 | 'sinif_adi' bloğu — hekim SINIF adı yazdı, TARAMA HİÇ ÇALIŞTIRILMADI. |
| `sonuc_html_n` | `(lang, tarama, durum = None) -> str` | 646 | N İLAÇ sonuç bloğu (`<div id="etk-sonuc">` İÇİNE girer, sarmalayıcıyı BASMAZ). |
| `_kutu` | `(renk, metin, ek) -> str` | 898 |  |

## `saglik/app/etkilesim_page.py`

`762 satır` · `9 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
/etkilesim sayfa iskeleti — ilk ekran, form, N İLAÇ (2..8), TR+EN.

⚠ Dürüstlük yardımcıları `etkilesim_body.py`'de; burası yalnız SAYFA. Ayrılma sebebi:
  yardımcılar birim-test edilebilir saf fonksiyonlar, burası FastAPI/DB'ye bağımlı.

⚠⚠ N İLAÇ (founder 2026-08-18: "2'den fazla ilaç olması lazım"). TARAMA katmanı
   DEĞİŞMEDİ: hâlâ HERKESE AÇIK, ücretsiz, LLM'siz, kredi harcamaz — yalnız ilaç
   sayısı 2 → 2..`MAKS_ILAC`. Ücretli olan tek şey AI YORUMU'dur ve o AYRI bir
   bloktur (`etkilesim_yorum.py`), tarama sonucunun ALTINDA, ayrı bir istekle gelir.

⚠⚠ JS h1'e DOKUNMAZ. `calc_body.py:1725` koşulsuz `textContent = T(TX.title)` yazıyor ve
   sunucunun bastığı doğru başlığı jenerikle EZİYOR — reklamın sözü ile ilk ekranın örtüşmesi
   (mesaj uyumu) tam orada kayboluyor. Bu sayfada başlığı YALNIZ sunucu yazar; doğrulama
   scripti JS kaynağında h1'e atama OLMADIĞINI iddia eder.

⚠ Sonuç SUNUCUDA render edilir (GET ?a=&b=&d=): JS'siz çalışır, ilk boyada görünür ve
   sunucu-taraflı görünür metin SEO/kalite skoru için yeterli olur (calc dersi: JS-öncesi
   86 karakter → kalite skoru 1).

⚠⚠ GERİYE UYUM — `?a=X&b=Y` CANLI REKLAM URL'İDİR (Google AG-Etkilesim + `/interactions`
   301 takma yolu + `/panel` kartının GET formu `name="a"`/`name="b"`). KIRILMAZ: `a` ve
   `b` alanları DURUR, `d` yalnız EKLENİR ve üçü `_ilaclari_coz`da tek listeye katılır.
   Ölçüm: `scratchpad/_kamil_n_parite.py` iki-ilaç yolunun BAYT paritesini çiviler (73/73).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_ISTEK_SON_TARIH` | `TARAMA_SON_TARIH_SN` | 47 |
| `ETK_CSS` | `'\n/* Kabuk ofseti: ray akıştan çıkar (fixed) → gövde onu temizler. Sayı YAZMA, token kullan. */\n/* ⚠ ZEMİN GÖÇÜ, GEÇİC` | 66 |
| `_TXT` | `{'tr': {'h1': 'İlaç etkileşimi kontrolü', 'sub': f'2–{MAKS_ILAC} ilaç yazın: resmî ilaç etiketlerinde (ABD FDA/openFDA +` | 150 |
| `_ORNEK` | `('ibuprofen', 'aspirin')` | 238 |
| `_SINIF_UYE_TAVAN` | `12` | 298 |
| `_SINIF_YAVAS_MS` | `2000.0` | 313 |
| `_SINIF_MOLA_SN` | `120.0` | 314 |
| `_ETK_JS` | `"<script>\n(function(){\n var inp=document.getElementById('etk-d'), menu=document.getElementById('etk-menu'),\n     cips` | 688 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_dejenere` | `(s: str) -> bool` | 245 | Girdi freni: <4 harf ya da alfanümerik yok → sorgu ÇALIŞTIRILMAZ. |
| `_ilaclari_coz` | `(a: str, b: str, ilac_listesi: str)` | 252 | `?a=`, `?b=` ve `?d=` → (ilaclar, atilan, tasan). TEK NORMALİZE NOKTASI. |
| `_sinif_href` | `(ilaclar, idx: int, uye: str) -> str` | 318 | `idx`inci adı `uye` ile DEĞİŞTİREN sonuç URL'i — kalan adlar AYNEN korunur. |
| `_siniflari_coz` | `(ilaclar)` | 333 | Girilen adlardan İLAÇ SINIFI olanlar → `sinif_adi` bloğunun verisi (yoksa []). |
| `_tara` | `(request, ga: str, gb: str, lang: str = 'tr')` | 394 | (kart_a, kart_b, sonuc, durum) — durum None ise normal akış. İKİ İLAÇ. |
| `_tara_n` | `(request, ilaclar, lang: str = 'tr') -> dict` | 471 | N İLAÇ taraması → `cds.etkilesim_coklu.coklu_tara` sözlüğü. |
| `_yorum_gorunur` | `(tarama) -> bool` | 519 | AI bloğu basılsın mı? ⚠⚠ TARAMA BAŞARILI DEĞİLSE HAYIR (2026-08-18, ölçüldü). |
| `_yorum_parcalari` | `(lang: str, d, ilaclar)` | 536 | (css, blok, js) — AI yorum bloğu `etkilesim_yorum.py`nin malıdır (sözleşme). |
| `etkilesim_govde` | `(request, lang: str, a: str, b: str, d, ilac_listesi: str = '') -> str` | 557 | Tam sayfa HTML. `d` = doktor dict ya da None (yalnız nav + CTA dallanması). |

## `saglik/app/etkilesim_yorum.py`

`884 satır` · `21 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
`/etkilesim` AI YORUMU — çoklu-ilaç etiket taramasının klinik yorumu (giriş + 1 kredi).

Founder kararı 2026-08-18 (üç madde, TARTIŞILMAZ):
  1) TARAMA deterministik, LLM'siz, HERKESE AÇIK ve ÜCRETSİZ KALIR — giriş yok, kredi yok.
     Bu modül o sözleşmeye DOKUNMAZ; taramanın YANINA ikinci bir katman koyar.
  2) YORUM giriş İSTER ve 1 kredi harcar. Çıkışlı ziyaretçi butonu değil `/kayit`
     bağlantısını görür (bu sayfa ÜCRETLİ Google reklam inişi, kayıt dönüşüm hedefi).
  3) Yorum BUTONLA üretilir, OTOMATİK DEĞİL. Tarama sonucu JS'siz de görünür (SEO); yorum
     ayrı bir POST ile gelir.

────────────────────────────────────────────────────────────────────────────────
İNVARİANTLAR — BOZMA (her biri ölçülmüş bir hatadan doğdu)
────────────────────────────────────────────────────────────────────────────────
⚠⚠ İSTEMCİDEN PASAJ KABUL EDİLMEZ. İstek gövdesi YALNIZ ilaç adları taşır; tarama
   SUNUCUDA yeniden koşturulur. Aksi hâlde model uydurulmuş bir "etiket pasajına"
   `[openFDA:x]` diye atıf yapardı ve bu sayfanın SATTIĞI şey (denetlenebilirlik) çökerdi.
   Ayrıca sınırsız token yakma ve prompt enjeksiyonu yüzeyi açılırdı. `lang` de gövdeden
   ALINMAZ, `get_lang(request)`ten okunur.

⚠⚠ FAIL-OPEN YASAK. LLM istisnası, boş/kesik yanıt, zaman aşımı, semafor doluluğu ve
   çıktı kapısı ihlali → HEPSİ `{"durum":"calistirilamadi"}` + kredi İADE. Hiçbir dalda
   olumlayıcı/aklayıcı bir hüküm ÜRETİLMEZ. Bu uçta "çalıştırılamadı" demek dürüstlüktür;
   sessizce olumlu bir metin döndürmek YANLIŞ GÜVEN üretir ve bu üründe yanlış cevaptan
   tehlikelidir.

⚠⚠ SONUÇ DURUMLARI DAİMA HTTP 200 + `durum` alanı (429/500 YOK). Bir HTTP hata kodu
   istemcide sessizce yutulabilir ve hekim ekranda hiçbir şey görmeyince "bulgu yok"
   sanar. YALNIZ kimlik/girdi/kota kapıları gerçek HTTP kodu döndürür (401/400/402/403),
   çünkü onların KARŞILIĞINDA bir kullanıcı eylemi vardır.

⚠⚠ KİMLİK YOKSA 401 JSON — 303 DEĞİL. `fetch` yönlendirmeyi şeffaf izler ve istemci
   HTML gövdesini JSON sanıp sessizce boş ekran basar. Gövde `{"giris":true,
   "kayit_url":"/kayit"}` taşır; yönlendirme kararını İSTEMCİ verir.

⚠ REZERVASYON TARAMADAN ÖNCE ALINIR (`deep_analyze`ın ölçülmüş sırası): sonra alınsaydı
   kredisi bitmiş bir hesap 402'yi yemeden önce her istekte 28 çiftlik DB işini bedavaya
   koşturabilirdi.

⚠⚠ rid BAŞINA TEK commit VEYA TEK cancel (`store._settle_once` şemadan zorlar). Her hata
   dalında `cancel_reservation` ÇAĞRILIR ve o daldan SONRA `commit_reservation` YOLU YOKTUR
   (erken `return`). `llm.YanitKesildi` dalında AYRICA `record_usage(..., credits=0)`
   yazılır: kullanıcıya kredi iade edilir ama YANAN API PARASI `app.usage`'a girmelidir,
   yoksa kaçak ÖLÇÜLEMEZ (#219b).

⚠⚠ TARAMA AYRI BİR BAĞLANTIDA KOŞAR. Postgres'te başarısız tek bir ifade TÜM transaction'ı
   zehirler; tarama `QueryCanceled` alsaydı AYNI bağlantıdaki `cancel_reservation` da
   düşerdi → kredi iade EDİLEMEZDİ ve `except` bunu yutardı. Kredi bağlantısı temiz kalır.

⚠ İKİ FREN, İKİ AYRI İŞ ve İKİSİ DE `/etkilesim`inkinden AYRI NESNE:
   `_rate_ok(f"etkyorum:{doctor_id}", 20)` = KİŞİ başına tavan · `_EY_SEM` = eş zamanlı
   ağır iş tavanı. Kovayı `etk:` ile PAYLAŞMAK bir aracı kullanmayı ötekini kapatmaya
   çevirirdi (ücretsiz tarama, ücretli yorum yüzünden kilitlenemez).
   ⚠ AYRI GÜNLÜK TAVAN YOK: ölçülmüş emsalde günlük tavan yalnız 0-KREDİLİ güvenlik
   uçlarında var (`_gunluk_tavan`); burada fren KREDİnin kendisidir.

⚠ AĞIR İŞ `run_in_threadpool` İLE HAVUZA GİDER (`_etk_yorum_sync`). Rota `async def`
   kalır ve YALNIZ gövde okumasını `await`ler. İkisi birden şart, yoksa
   `scratchpad/threadpool_kapisi.py` bu ucu ölçmeyi bırakır (kör kalır).

⚠ ÇIKTI KAPISI SUNUCUDA (`cds.aklama`): aklama dili ve yönlendirme yasağı MODEL ÇIKTISINA
   uygulanır. Statik CI taraması istemcide çizilen metni GÖREMEZ — kuralın GEÇERLİ olduğu
   yüzey, UYGULANDIĞI yüzeyden geniş kalmamalı (#62b'nin aynısı).
   ⚠⚠ Kapı modülü YOKSA ya da listeleri eksikse uç SERVİS ETMEZ (`sebep="kapi_yok"`):
      doğrulanamayan çıktı "kusursuz" değildir. Bu bilinçli bir sert bağımlılıktır.

⚠ MODEL ÇIKTISI DÜZ METİNDİR. HTML'i SUNUCU üretir (`_yorum_html`, `esc` ile kaçışlı);
   istemci `innerHTML = j.yorum_html` yazar — `j.yorum` ASLA innerHTML'e verilmez.

⚠ BU MODÜL `main.py`DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur). Paylaşılan yardımcılar
   `webutil`den DOĞRUDAN okunur, main fasadından DEĞİL (`_RATE` kovası bölünmesin).

⚠ EN_PAIRS'e SIFIR SATIR: `/etkilesim` `i18n_mw`den muaf; tüm metin `_TXT`de TR+EN
   sunucuda üretilir. JS'te dile bağlı DİZE YOKTUR (`data-*` ile gelir).

⚠ MOTOR BAĞIMLILIKLARI TEMBEL IMPORT EDİLİR (`cds.etkilesim_coklu`, `cds.aklama`): eksik
   bir modül uygulamayı AÇILIŞTA çökertmesin, çalışma anında `calistirilamadi` üretsin.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 100 |
| `MOD` | `'etkilesim_yorum'` | 118 |
| `MOD_RED` | `'etk_yorum_red'` | 123 |
| `_EY_SEM` | `threading.Semaphore(2)` | 130 |
| `_SQL_TAVAN_MS` | `TARAMA_SQL_MS` | 139 |
| `_SON_TARIH_SN` | `TARAMA_SON_TARIH_SN` | 140 |
| `_RED_GUNLUK_TAVAN` | `20` | 146 |
| `_PASAJ_BUTCE_KRK` | `12000` | 150 |
| `_MIN_YANIT_KRK` | `120` | 152 |
| `_KAPANIS` | `(KAPANIS_SATIRI['tr'], KAPANIS_SATIRI['en'], ATIFLI_SATIRI['tr'], ATIFLI_SATIRI['en'])` | 159 |
| `_TXT` | `{'tr': {'h': 'Yapay zekâ yorumu', 'b': 'Yukarıdaki tarama bulgularını klinik olarak yorumlayalım: hangi hastada ağırlaşı` | 167 |
| `YORUM_CSS` | `'\n.etk-ai{background:#fff;border:1px solid var(--line);box-shadow:var(--sh-surface);\n  border-radius:var(--r-card);pad` | 223 |
| `YORUM_JS` | `"<script>\n(function(){\n var b=document.getElementById('etk-ai-go'), out=document.getElementById('etk-ai-out');\n if(!b` | 278 |
| `_BOLD` | `re.compile('\\*\\*(.+?)\\*\\*')` | 365 |
| `_BASLIK` | `re.compile('^\\*\\*(.+?)\\*\\*\\s*[—–:\\-]?\\s*(.*)$')` | 366 |
| `_MADDE` | `re.compile('^[-•–]\\s+(.*)$')` | 367 |
| `_NUMARA` | `re.compile('^(\\d{1,2})[.)]\\s+(.*)$')` | 368 |
| `_DURUM_ET` | `{'sinyal': ('SINYAL — etiket karsi ilaci aniyor', 'SIGNAL - the label mentions the other drug'), 'zayif': ('ZAYIF — ad y` | 438 |
| `_ETIKET` | `re.compile('\\[([^\\[\\]\\n]{1,60})\\]')` | 593 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 103 | `@app.post(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 111 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_T` | `(lang: str) -> dict` | 215 |  |
| `yorum_blok_html` | `(lang: str, d, ilaclar: list[str]) -> str` | 327 | `<section id="etk-ai">` — girişli hekimde BUTON, çıkışlıda `/kayit` BAĞLANTISI. |
| `_satir_ici` | `(s: str) -> str` | 371 | Kaçış ÖNCE, biçim SONRA — `**` karakterleri `esc`ten sağ çıkar, `<`/`&` çıkmaz. |
| `_yorum_html` | `(metin: str) -> str` | 376 | Modelin DÜZ METNİNİ güvenli HTML'e çevir (markdown-lite; bağlantı/tablo YOK). |
| `_du` | `(kod: str, lang: str) -> str` | 449 |  |
| `kanit_blok` | `(paket: dict, lang: str = 'tr') -> str` | 454 | `kanit_paketi` çıktısını modele giden `<tarama>` metnine çevir. |
| `_kaynaklar` | `(paket: dict) -> list[str]` | 521 | Pakette FİİLEN geçen kaynak adları — `cds.kaynaklar` kaydına göre kanonikleştirilir. |
| `_kapi_ihlali` | `(metin: str, lang: str) -> tuple[str \| None, list[str], list[str]]` | 541 | (sebep, sert_ihlaller, yumusak_ihlaller) — `sebep` None ise metin servis edilebilir. |
| `_atif_ihlali` | `(metin: str, paket: dict) -> list[str]` | 596 | PAKETTE OLMAYAN kaynağa atıf → fail-closed listesi (boşsa temiz). |
| `_kapsam` | `(tarama: dict \| None) -> dict` | 631 |  |
| `_calistirilamadi` | `(sebep: str, tarama: dict \| None = None) -> JSONResponse` | 641 | FAIL-CLOSED yanıt — DAİMA HTTP 200 + `durum`. ⚠ Hiçbir dalda olumlayıcı metin YOK; |
| `_tarama_calistir` | `(ilaclar: list[str], lang: str) -> dict \| None` | 649 | Taramayı AYRI bağlantıda koştur — kredi bağlantısını ZEHİRLEMESİN (bkz. başlık). |
| `_paket_kur` | `(tarama: dict, lang: str) -> dict \| None` | 670 |  |
| `async` `api_etkilesim_yorum` 🌐 | `(request: Request)` | 681 | `/etkilesim` taramasının AI yorumu — giriş + 1 kredi (founder kararı 2026-08-18). |
| `_ilaclar_coz` | `(body: dict) -> list[str]` | 692 | İstek gövdesinden ilaç adları — SADECE `ilaclar`; başka anahtar YOK SAYILIR. |
| `_etk_yorum_sync` | `(request, body, lang)` | 709 | Kredi zinciri — SIRA SÖZLEŞMENİN PARÇASIDIR, değiştirme (gerekçeler modül başlığında). |
| `_yanan` | `(conn, d, model, usage) -> None` | 760 | LLM DÖNDÜ ama yanıt servis EDİLMİYOR → yanan API parasını `app.usage`'a yaz. |
| `_uret` | `(conn, d, ilaclar, lang)` | 776 | Tarama → kanıt paketi → model → kapı → muhasebe. |
| `_faith` | `(metin: str, kaynaklar) -> None` | 876 | M32 atıf sadakati — SALT GÖZLEM. ⚠ `chat_routes._faith`in İKİZİ DEĞİL, ONU ÇAĞIRIR |

## `saglik/app/fiyat.py`

`265 satır` · `9 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
FİYAT GÖSTERİMİ — tutar VE para biriminin TEK KAYNAĞI (2026-08-09, #137b + #139b).

⚠⚠ NEDEN VAR — iki ölçülmüş kusur, ikisi de aynı kökten (fiyat GÖVDE METNİNDE yazılıydı):

  (1) **#137b — senkron AYRAÇ-DUYARLIYDI.** `main._apply_try_pricing` landing metninde
      fiyatı STRING olarak arayıp değiştiriyordu; anahtar TR'de nokta ("1.450"), EN'de
      virgül ("1,450") yazıldığı için **EN'de hiçbir anahtar eşleşmiyordu** →
      `IYZICO_PRICE_*` env'i fiyatı değiştirince TR düzelir, **EN eski fiyatı ilan etmeye
      devam ederdi** (sentinel ölçümü: kalan eski fiyat TR 0 / EN 8). `/kilavuz` ise
      senkronun HİÇ İÇİNDE DEĞİLDİ.

  (2) **#139b — TUTAR senkronluydu, PARA BİRİMİ değildi.** `lang=en` sayfası her koşulda
      "TL" basıyordu; 2026-08-09 "yurt dışı 35 USD" kararı hiçbir yüzeye inmedi.

ÇÖZÜM — string eşleşmesi YOK. Gövdeler (`landing.py`, `guide.py`) fiyatı **YER TUTUCU**
olarak taşır (`{{klv.fiyat.…}}`); render anında bu modül doldurur. Yazımın tamamı —
binlik ayracı, sembol, birim adı — gövdede DEĞİL burada. Gövdedeki fiyat yazımı
değişse bile senkron **no-op'a dönemez**, çünkü eşleşecek bir sayı yoktur.

⚠ Doldurma **KOŞULSUZDUR** (eski `_apply_try_pricing` yalnız provider=iyzico'da koşuyordu):
doldurulmayan yer tutucu hekime ham metin olarak görünür → kapı "0 kalıntı" ölçer.

⚠⚠ **PARA BİRİMİ 2026-08-09'DA DİLE BAĞLANDI (founder: "Türkçe hariç herkes dolar
görsün").** Yukarıdaki iki paragrafın yazıldığı gün burada TAM TERSİ yazıyordu — *"ölçüt
tahsilattan türer, dilden değil"* — ve o gerekçe YANLIŞ DEĞİLDİ: dile bakıp USD basmak
gerçekten **ilan≠tahsilat** üretir, çünkü bugün herkes iyzico'dan **TL** ödüyor.
Karar riski ortadan kaldırmıyor, **ifşayla yönetiyor**: USD gösterimi `_ifsa` satırından
AYRILAMAZ ve ifşanın koşulu bayrağa değil **gerçeğe** bağlıdır (gösterim ≠ tahsilat).
Ölçüt tek yerde: `webutil.para_birimi()` (gösterim) / `webutil.tahsilat_birimi()` (kasa).
⚠ Bu iki fonksiyonu KARIŞTIRMA: para olayları (piksel `currency`) ve yasal beyan
**tahsilat** birimini okur; ekrandaki rakam gösterim birimini.

⚠ **BU MODÜLDE ELLE YAZILI TEK BİR FİYAT YOKTUR.** TL tutarları
`iyzico.plan_price_try` (env-duyarlı), USD tutarları **gerçekten çekilen**
tutardan (`billing.PLANS` — cent) okunur.

⚠ **PARA BİRİMİNE BAĞLI DÜZ METİN BU MODÜLÜN DIŞINDA** (KDV/e-fatura/iyzico cümleleri):
onlar salt sayı değil, vergi ve satıcı beyanıdır — USD karşılıklarını uydurmak yasal metin
yazmak olur (sahibi Selim). `birim_prose_borcu()` onları SAYAR ve kapı, ölçüt USD'ye
geçtiğinde KIRMIZI basar; yani karar gerektiği an görünür olur, sessizce yanlış ilan olmaz.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ONEK` | `'{{klv.fiyat.'` | 52 |
| `OPSIYONEL` | `{'ifsa'}` | 56 |
| `_IFSA_TR` | `'Gösterilen tutar bilgi amaçlıdır; tahsilat ödeme adımında {b} olarak yapılır.'` | 181 |
| `_IFSA_EN` | `'The amount shown is indicative; payment is collected in {b} at checkout.'` | 182 |
| `_BIRIM_ADI` | `{'TRY': {'tr': 'TL', 'en': 'TRY'}, 'USD': {'tr': 'USD', 'en': 'USD'}}` | 184 |
| `PROSE_TRY_ISARET` | `('KDV dahil', 'KDV dahildir', 'VAT included', 'include VAT', 'e-fatura', 'e-arşiv', 'e-invoice', 'tax invoice')` | 258 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_sayi` | `(v: float, en: bool, ondalik: int) -> str` | 59 | Sayıyı DİLE göre yazar: TR '1.450' / '0,36' · EN '1,450' / '0.36'. |
| `bicim` | `(tutar, birim: str \| None = None, en: bool = False, tam: bool = False, request = None, doctor = None) -> str` | 68 | Tutar + para birimini TEK YERDE birleştirir. TRY → '1.450 TL' · USD → '$35'. |
| `plan_fiyat` | `(plan: str, en: bool = False, request = None, doctor = None) -> str` | 91 | Abonelik planının gösterim fiyatı ('1.450 TL' / '$35'). Bilinmeyen plan → ''. |
| `_yillik_aylik` | `(en: bool, request = None, doctor = None) -> str` | 106 | Yıllık planın AYLIK EŞDEĞERİ — landing'de '~1.208 TL/ay' diye ilan edilir. |
| `_metin_sozluk` | `(en: bool, request = None, doctor = None) -> dict[str, str]` | 122 | SAYI OLMAYAN ama para birimine bağlı metin parçaları. |
| `_ifsa` | `(en: bool, doctor = None, request = None) -> str` | 187 | İfşa satırı — **GÖSTERİM ≠ TAHSİLAT olduğunda** basılır; aksi hâlde boş dize. |
| `sozluk` | `(en: bool = False, request = None, doctor = None) -> dict[str, str]` | 209 | Yer tutucu adı → gösterim metni. Yeni bir fiyat yüzeyi eklenirse anahtar BURAYA girer. |
| `doldur` | `(html: str, en: bool = False, request = None, doctor = None) -> str` | 231 | Gövdedeki `{{klv.fiyat.<ad>}}` yer tutucularını doldurur (KOŞULSUZ). |
| `birim_prose_borcu` | `(html: str) -> list[str]` | 262 | Sayfadaki, koddan TÜREMEYEN Türk vergi/fatura beyanlarını döndürür (kapı için). |

## `saglik/app/form_listeler.py`

`83 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Kayıt/onay formunun UNVAN ve BRANŞ listeleri — SAF VERİ.

⚠⚠ BU DOSYA HİÇBİR ŞEY IMPORT ETMEZ ve etmemelidir. `webutil.py`den ayrıldı
(2026-08-08) çünkü webutil PAYLAŞILAN SÖZLÜK: şiştiğinde onu okuyan herkesi vurur ve
tavan kapısını (`dosya_boyut_verify`) tetikler. 79 branş adı ise saf referans verisidir
ve büyümeye devam edecek — burada büyümesi kimseyi etkilemez.

⚠ `_sel_opts` / `_brans_opts` BURAYA TAŞINMADI, `webutil`de kaldı: `esc` kullanırlar,
taşınsalardı bu modülün webutil'den import etmesi gerekirdi ve webutil de buradan
import ettiği için DAİRESEL olurdu (landing.py'nin aynı gerekçesi). Buraya `def`
eklemek uyarı işaretidir.

⚠ `webutil` bu adları YENİDEN DIŞA VERİR → mevcut `from .webutil import _BRANSLAR, ...`
satırlarının hiçbiri değişmedi. Yeni tüketici de webutil'den okuyabilir.

⚠ Değerler TR-KANONİK — etiketlerine `EN_PAIRS` çifti EKLEME (value attr de çevrilir →
DB'ye karışık dil yazılır).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_UNVANLAR_REG` | `['Dr.', 'Uzm. Dr.', 'Op. Dr.', 'Doç. Dr.', 'Prof. Dr.', 'Asistan Dr.', 'Psikolog', 'Tıp Öğrencisi']` | 24 |
| `_BRANS_ANA` | `['Acil Tıp', 'Adli Tıp', 'Aile Hekimliği', 'Anesteziyoloji ve Reanimasyon', 'Beyin ve Sinir Cerrahisi', 'Çocuk Cerrahisi` | 40 |
| `_BRANS_YAN` | `['Algoloji', 'Askeri Psikiyatri', 'Cerrahi Onkoloji', 'Çevre Sağlığı', 'Çocuk Acil', 'Çocuk Endokrinolojisi', 'Çocuk Enf` | 51 |
| `_BRANS_MESLEK` | `['Psikoloji']` | 69 |
| `_BRANSLAR` | `_BRANS_ANA + _BRANS_YAN + _BRANS_MESLEK + ['Diğer']` | 71 |
| `OGRENCI_BRANS` | `'Öğrenci'` | 82 |

## `saglik/app/geo.py`

`204 satır` · `12 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Ülke + şehir seçimi — kayıt/onay formlarının SUNUCU tarafı (2026-08-15, founder:
"kayıt olurken ülke şehir seçme koymamız lazım; bütün dünyadaki ülkeler ve şehirler,
seçimde zorunlu").

VERİ: `static/geo/ulkeler.json` (250 ülke, tr+en ad) + `static/geo/sehir/<CC>.json`
(ülke başına şehir listesi; TR = 81 İL, diğerleri GeoNames nüfus ≥ 15.000). Üreteç ve
lisans (CC BY 4.0, atıf) → `scripts/geo_veri_uret.py` + `static/geo/KAYNAK.md`.
⚠ Bu modül veriyi ÜRETMEZ, yalnız OKUR (ağ yok, bağımlılık yok, ilk okuma önbelleklenir).

SÖZLEŞME (form ↔ rota):
  · `ulke`  = ISO 3166-1 alpha-2 (büyük harf) — `<select name="ulke">`; sunucu `ulke_gecerli`.
  · `sehir` = listeden bir ad  YA DA  `SEHIR_DIGER` ("__diger__") → o zaman `sehir_diger`
    serbest metin (2..SEHIR_MAX krk) zorunlu. `sehir_normalize` ikisini TEK değere indirger;
    None dönerse rota 400 basar. ⚠ "Listede yok" yolu BİLEREK var: GeoNames kesimi nüfus
    ≥ 15.000; küçük yerleşimdeki hekim listede kendini bulamazsa KAYIT ENGELLENMEZ
    (telefon listesindeki "Diğer" kaçış yolunun aynısı).
  · Şehir seçeneği İSTEMCİDE ülkeye göre `/static/geo/sehir/<CC>.json`tan yüklenir
    (`_GEO_JS`, `auth_routes._ulke_sehir_alan`); SUNUCU başlangıç listesini seçili ülke için
    HTML'e gömer → JS'siz tarayıcıda da (varsayılan ülke) çalışır.
⚠ Sunucu tarafı ŞART: `required` yalnız tarayıcıyı bağlar, POST doğrudan çağrılabilir —
  doğum tarihi/telefon alanlarındaki kuralın aynısı.
⚠ Türkiye LİSTE BAŞINA sabitlenir (iki dilde de): TR-önce ürün; alfabetik yerde ("T")
  kalsaydı hekimlerin çoğu 200 satır kaydırırdı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_GEO_DIR` | `os.path.join(os.path.dirname(__file__), 'static', 'geo')` | 35 |
| `SEHIR_DIGER` | `'__diger__'` | 36 |
| `SEHIR_MAX` | `80` | 37 |
| `VARSAYILAN_ULKE` | `'TR'` | 38 |
| `_TR_ALFABE` | `'abcçdefgğhıijklmnoöprsştuüvyzqwx'` | 41 |
| `_TR_ESIT` | `{'â': 'a', 'î': 'i', 'û': 'u', 'Â': 'a', 'Î': 'i', 'Û': 'u'}` | 42 |
| `_GEO_JS` | `'<script>(function(){var u=document.getElementById("PFX-ulke"),s=document.getElementById("PFX-sehir"),d=document.getElem` | 166 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_tr_anahtar` | `(s: str) -> list[int]` | 45 |  |
| `_en_anahtar` | `(s: str) -> str` | 59 |  |
| `ulkeler` | `() -> tuple[dict, ...]` | 65 | (({"k","tr","en"}, …)) — dosya sırası (TR alfabetik). Yoksa boş demet (kapı bunu çiviler). |
| `_kod_kumesi` | `() -> frozenset[str]` | 76 |  |
| `ulke_gecerli` | `(kod: str \| None) -> bool` | 80 |  |
| `ulke_adi` | `(kod: str \| None, lang: str = 'tr') -> str` | 84 |  |
| `sehirler` | `(kod: str) -> tuple[str, ...]` | 92 | Ülkenin şehir listesi (dosya sırası). Dosya yoksa/bozuksa BOŞ — kayıt yine "Diğer" ile |
| `_ulke_sirali` | `(lang: str) -> tuple[dict, ...]` | 106 | Dile göre sıralı ülke listesi; Türkiye BAŞTA (bkz. modül başlığı). |
| `ulke_opts` | `(lang: str = 'tr', secili: str = '') -> str` | 118 | `<option>` dizisi. `secili` boşsa yalnız yer tutucu seçilidir (hekim seçmek ZORUNDA). |
| `sehir_opts` | `(kod: str, secili: str = '') -> str` | 126 |  |
| `ulke_sehir_alan` | `(pfx: str, lang: str = 'tr', secili_ulke: str = '', secili_sehir: str = '') -> str` | 132 | Ülke + şehir bloğu (etiket → select · etiket → select · gizli serbest-metin) + JS. |
| `sehir_normalize` | `(kod: str, sehir: str \| None, sehir_diger: str \| None) -> str \| None` | 181 | Form ikilisini TEK şehir değerine indirger; geçersizse None. |

## `saglik/app/gizli_kaynak.py`

`344 satır` · `8 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
GİZLİ KAYNAK — gösterimde adı GEÇMEYECEK kaynakların sunucu-taraflı süzgeci (2026-08-19).

Founder: "sitede HİÇBİR yerde statpearls geçmesin" (P0 lisans: StatPearls gerçek lisansı
CC BY-NC-ND 4.0; korpusta 'CC BY' diye YANLIŞ etiketli — `CLAUDE.md` "P0 LİSANS BULGUSU").

⚠⚠ YALNIZ GÖSTERİM. Bu modül DB'ye YAZMAZ, model girdisini/kanıt paketini/monitörü DEĞİŞTİRMEZ
   (onlar `cds/*`, Cahit/Derya). Süzgeç RENDER anında uygulanır: geçmişte `[statpearls:…]`
   etiketiyle kaydedilmiş ziyaret notları / analiz yanıtları DB'de olduğu gibi durur, ekrana
   etiketsiz gelir. Etiket SİLİNİR, yerine sahte/nötr etiket YAZILMAZ (dürüst yön: az iddia).

⚠⚠ TEK KAYNAK İKİ DİLDE: sohbet yüzeyi aynı süzgeci İSTEMCİDE uygular (`chat_body.py`
   `const _KANIT_GIZLI=[...]` + `_mGizliAtif`/`gizliSuz`). İki liste AYRIŞMASIN diye
   `rozet_atif_verify` bölüm 2 Python `GIZLI` ile JS sabitinin BİREBİR aynı olduğunu ölçer;
   `_CASE_TIMELINE_JS` ise listeyi `js_liste()` ile buradan ALIR (`__GIZLI__` yer tutucu).
   ⚠⚠ Yeni kaynak gizlenecekse ÖNCE `cds/kaynaklar.py` kaydına `"gizli": True` (bu modül ve
   `hastalik._GIZLI_KAYNAK` oradan TÜRER, #470b), SONRA `chat_body` sabiti elle güncellenir —
   `rozet_atif_verify` bölüm 2c üçünü de kayıtla kıyaslar, unutulan sürüklenme KIRMIZI olur.

⚠ Desen JS'tekiyle AYNI: `[statpearls:…]` (kimlikli) ve `[StatPearls]` (çıplak), büyük/küçük
  harf duyarsız, önündeki boşluk/sekme ile birlikte ("cümle [x]." → "cümle."). Eşleşme yoksa
  metin BAYT AYNI döner (kapı bunu ölçer — temiz not bozulmaz).

Tüketenler: `cases_routes.case_detail_page` (ziyaret notu `esc(gizli_sil(note))`, analiz yanıtı
`_md_lite(gizli_sil(ans))`, **kayıtlı zaman tüneli arşivi** `_md_lite(gizli_sil(body))`,
`__GIZLI__` enjeksiyonu) · `chat_routes.api_case_timeline` (`text = gizli_sil(text)`; Ömer).
Yeni bir yüzey hasta kaydından/model yanıtından metin basıyorsa buradan geçirilir.
⚠⚠ `app.case_timeline.body` HAM saklanır (2026-08-27 arşivi) — yani bu tablo yukarıdaki
"geçmişte etiketle kaydedilmiş" kümesine YENİ ve BÜYÜYEN bir üye ekler. Arşivi basan bir
yüzeyi süzgeçsiz yazmak, etiketi doğrudan hekimin ekranına koyar.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `GIZLI` | `tuple(sorted(_gizli_turler()))` | 47 |
| `KIMLIK_TAVAN` | `240` | 60 |
| `_RE` | `re.compile('[ \\t]*\\[(?:' + '\|'.join((re.escape(g) for g in GIZLI)) + ')(?:\\s*:[^\\]]{0,%d})?\\]' % KIMLIK_TAVAN, re.` | 62 |
| `_METIN_ALANLARI` | `('answer', 'text', 'synthesis', 'critic', 'content', 'opinion', 'pattern', 'action_hint')` | 83 |
| `_METIN_LISTE_ALANLARI` | `('notes', 'issues')` | 91 |
| `_TUT_TAVAN` | `KIMLIK_TAVAN + 64` | 194 |

### `class AkisSuzgeci(—)`  <sub>satır 272</sub>

`respond_stream` çıktısını GERİ-TUTMA TAMPONUYLA süzer (#543b).

| Metot | İmza | Satır | Açıklama |
|---|---|---|---|
| `__init__` | `(self) -> None` | 297 |  |
| `_bosalt` | `(self)` | 301 |  |
| `sar` | `(self, uretec)` | 305 |  |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `gizli_sil` | `(text: str \| None) -> str` | 66 | Metindeki gizli kaynak etiketlerini siler; eşleşme yoksa girdi BAYT AYNI döner. |
| `kaynak_suz` | `(sources)` | 94 | Rozet/kaynak listesinden gizli türleri eler (`chat_body.gizliSuz`in sunucu ikizi). |
| `_kunye_suz` | `(docs)` | 107 | Künye (source_docs) kayıtlarından gizli kaynaklı olanları eler. |
| `yuk_suz` | `(yuk)` | 115 | İstemciye gidecek JSON gövdesinin SÜZÜLMÜŞ KOPYASI (girdi MUTE EDİLMEZ). |
| `_devam_edebilir` | `(p: str) -> bool` | 197 | `p` (bir `[` ile başlayan son ek) gelecek deltalarla HALA bir gizli etikete |
| `akis_suz` | `(tampon: str) -> tuple[str, str]` | 220 | (yayımlanacak, tutulacak) — akış için tamponlu süzgeç. |
| `arama_suz` | `(yuk)` | 241 | ARAMA yükleri (`/api/query`) için: gizli kaynaklı KAYITLARI ÇIKARIR (kopya döner). |
| `js_liste` | `() -> str` | 337 | JS'e gömülecek liste (`["statpearls"]`) — `_CASE_TIMELINE_JS` `__GIZLI__` yer tutucusu. |

## `saglik/app/guide.py`

`22 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Kullanım kılavuzu (/kilavuz) — TR+EN gövde HTML + CSS.

2026-07-19 ajan paneli taslağı + uyum çürütücüsünün düzeltmeleri (durdurur→işaretler,
KVKK dosya-karartma uyarısı, süre taahhüdü kaldırıldı, Sor/Referans ilaç-kartı ayrımı).
i18n_mw /kilavuz için NO-OP (main.py) — dil seçimi rotada, EN gövdesi ayrı bakımlı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `GUIDE_CSS` | `'\n.g-wrap{max-width:820px;margin:0 auto;padding:40px 26px 90px;font-family:var(--font-body);line-height:1.65;color:var(` | 8 |
| `GUIDE_TR` | `'<div class="g-wrap">\n\n<h1>Klivance Kullanım Kılavuzu</h1>\n<p class="g-lead">Klivance, klinik sorularınıza açık tıbbi` | 10 |
| `GUIDE_EN` | `'<div class="g-wrap">\n\n<h1>Klivance User Guide</h1>\n<p class="g-lead">Klivance is a clinical decision-support tool th` | 12 |
| `GUIDE_TR` | `GUIDE_TR.replace('{DGUN}', str(_DG)).replace('{DSORU}', str(_DS))` | 20 |
| `GUIDE_EN` | `GUIDE_EN.replace('{DGUN}', str(_DG)).replace('{DSORU}', str(_DS))` | 21 |

## `saglik/app/guvenlik.py`

`235 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Güvenlik ve veri koruması sayfası (/guvenlik) — TR+EN gövde HTML.

2026-07-19 ajan paneli P1 kararı ("landing'e /guvenlik sayfası"). CSS guide.py'den
(GUIDE_CSS, .g-* sınıfları) paylaşılır. i18n_mw /guvenlik için NO-OP (main.py) —
dil seçimi rotada, EN gövdesi ayrı bakımlı.

⚠ İDDİA DİSİPLİNİ: Bu sayfadaki her teknik iddia koddan/yasal metinden teyitlidir
(Argon2id: auth.py · Fernet AES+HMAC/MultiFernet rotasyon: filecrypto.py · anında
içerik silme + ≥30g purge: store.purge_* · 30 gün hesap grace: /account/delete ·
iyzico/3DS + 10 yıl mali: _LEGAL). Yeni iddia eklerken önce koddan doğrula;
sertifika (ISO/SOC2), sızma testi, VERBIS, 2FA gibi HENÜZ OLMAYAN şeyleri YAZMA.

⚠⚠ 2026-08-05 — İKİ FOUNDER KARARI BU SAYFAYI DEĞİŞTİRDİ. Geri almadan önce oku:
 (1) **Sağlayıcı adı/ülkesi YAZILMAZ.** "Yapay zekâ işleme ve yurt dışı aktarım"
     başlıklı bölüm, içindeki "Anthropic, ABD/USA" ve "yurt dışındaki/abroad"
     ifadeleriyle birlikte KALDIRILDI (founder'a iki kez soruldu, iki kez
     "her yerden çıkar" dendi). Bölüm SİLİNMEDİ, NÖTRLEŞTİRİLDİ: sayfanın kimliği
     "ne yaptığımızı olduğu gibi yazarız" olduğu için, klinik metnin bir yapay
     zekâ altyapısında işlendiğini HİÇ söylememek sağlayıcıyı adlandırmaktan daha
     kötü olurdu. "Varsayılan olarak model eğitiminde kullanılmaz" cümlesi DOĞRU
     ve lehimize; sağlayıcı adı olmadan da söylenebildiği için KALDI.
     ⚠ `scratchpad/guvenlik_verify.py` bu ifşayı ZORUNLU KILIYORDU; iddialar
     silinmedi, TERS ÇEVRİLDİ — gerekçe orada yazılı.
 (2) **Hasta kimliği artık İŞLENEBİLİR** (avukat onaylı): hekim hasta defterine
     gerçek ad girebilir. Bu yüzden "Hiç işlenmez — sistemde kimlik alanı yoktur",
     "hasta formlarında ad alanı yoktur", "kayıt sırasında kimlik girmeme
     taahhüdünü onaylar" (3. onay kutusu KALDIRILDI, `/kayit` artık 2 kutu) ve
     "kimlik yazmama uyarısı yer alır" cümlelerinin hepsi OLGUSAL OLARAK YANLIŞ
     olmuştu ve değiştirildi. Bunlar dışarıya verilmiş TAAHHÜT metniydi —
     ürün davranışı değişince metni güncellemek zorunludur.
 ⚠ "yurt dışına aktarılmaz" ibaresi ödeme bölümünden de çıktı (founder seçenek C):
   kart güvenliği bilgisi KORUNDU ("bize hiçbir aşamada ulaşmaz"), yalnız
   "yurt dışı" ekseni düştü. Aynı karar `legal_body.py`de de uygulandı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `GUVENLIK_TR` | `'<div class="g-wrap">\n\n<h1>Güvenlik ve veri koruması</h1>\n<p class="g-lead">Bu sayfa bir pazarlama metni değildir. Kl` | 36 |
| `GUVENLIK_EN` | `'<div class="g-wrap">\n\n<h1>Security &amp; data protection</h1>\n<p class="g-lead">This page is not marketing copy. It ` | 136 |

## `saglik/app/hastalik.py`

`331 satır` · `8 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HASTALIKLAR — `/hastaliklar` (founder 2026-08-19: "ana menüye hastalıklar butonu;
hekim hastalık yazsın, bilgi gelsin, devamında soru sorma chatbox olsun, kredi harcasın").

İKİ KATMAN, İKİ FİYAT — ürünün mevcut ayrımının aynısı (`/reference` ücretsiz, `/chat` kredili):
  · KART (ücretsiz, LLM'siz): `core.disease` + `code_map` + `drug_disease` + `corpus_disease`
    — ad (TR/EN), ICD-11 kodu, tanım, eş anlamlılar, ilişkili ilaçlar, literatür.
  · SORU (1 kredi): kutuya yazılan soru MEVCUT sohbet yüzeyine taşınır (`/chat?q=`).

⚠⚠ SOHBET İSTEMCİSİ KLONLANMADI ve klonlanmamalı: `chat_body`nin akış okuyucusu daktilo
   akıtıcısı, çip ayrıştırma, atıf biçimleme ve İPTAL MUHASEBESİ taşır — hepsi ölçülmüş.
   İkinci bir kopya kaçınılmaz olarak ayrışır ve ayrışan kopya PARA YOLUNDADIR.
   Bu yüzden burada kredi kodu YOKTUR; soru `/api/chat`e mevcut yoldan gider.

⚠⚠ SORU OTOMATİK GÖNDERİLMEZ. `/chat?q=` yalnız kutuyu DOLDURUR; göndermeyi hekim yapar.
   Otomatik gönderim, paylaşılan/yer-imlenmiş bir linkin kazara KREDİ HARCAMASI demekti —
   `?welcome=1` hayalet-dönüşüm dersinin para yolundaki karşılığı.

⚠⚠ STATPEARLS KARTTA GÖSTERİLMEZ (P0 lisans: gerçek lisans CC BY-NC-ND, DB etiketi yanlış).
   `/hastaliklar` herkese açık bir yüzeydir; oradan StatPearls ilan etmek 2026-07-31'de
   kapatılan maruziyeti geri açardı (`migrate_corpus`ın varsayılan dışlamasıyla aynı gerekçe).

⚠ TANI KOYMAZ: kart bir referans kaydıdır; "şu hastalıktır" demez, kaynağı önüne koyar
   (FDA Kriter-4 dili — ürünün her yüzeyinde aynı).

⚠ Bu modül `main.py`den HİÇBİR ŞEY import ETMEZ (dairesel olur). `@_rota` + `kur(app)`.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 41 |
| `_HST_SEM` | `threading.Semaphore(3)` | 42 |
| `_GIZLI_KAYNAK` | `tuple(sorted(_gizli_turler()))` | 50 |
| `_BASLIK_STOP` | `frozenset(('unspecified', 'other', 'specified', 'without', 'with', 'disease', 'disorder', 'syndrome', 'condition', 'acut` | 54 |
| `_M` | `{'tr': {'h1': 'Hastalıklar', 'ph': 'Hastalık adı yazın…', 'ara': 'Ara', 'sub': 'Hastalığın kayıtlı tanımını, ICD-11 kodu` | 73 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(yontem: str, yol: str, **kw)` | 61 |  |
| `kur` | `(app) -> None` | 68 |  |
| `_t` | `(lang: str) -> dict` | 115 |  |
| `_ara` | `(q: str, lang: str, tavan: int = 12) -> dict` | 119 |  |
| `_kart` | `(did: int, lang: str) -> dict` | 202 |  |
| `async` `api_hastalik_ara` 🌐 | `(request: Request, q: str = '')` | 264 | Hastalık adı araması — ücretsiz, LLM yok, kayıt gerekmez. |
| `async` `api_hastalik` 🌐 | `(did: int, request: Request)` | 286 | Tek hastalık kartı — ücretsiz, LLM yok. |
| `hastaliklar_page` 🌐 | `(request: Request)` | 305 | Arama + kart + soru kutusu. Kart ücretsiz; soru mevcut sohbet yoluna taşınır. |

## `saglik/app/hastalik_body.py`

`824 satır` · `4 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HASTALIKLAR SAYFASININ GÖRSEL KATMANI — `/hastaliklar` (founder tasarımı 2026-08-19).

⚠⚠ SAF SUNUM. Bu modül DB'ye dokunmaz, rota tanımlamaz, `main.py`den ve `hastalik.py`den
   HİÇBİR ŞEY import ETMEZ. Davranış + veri `hastalik.py`de KALIR (Faz 1 `landing.py` ile
   aynı ayrım). Buraya `def` eklemek — özellikle veri okuyan bir `def` — uyarı işaretidir.
   ⚠ Ayrı dosya olmasının GEREKÇESİ mimari değil ÇARPIŞMA: `hastalik.py` eş zamanlı bir
   oturumun aktif dosyasıydı (2026-08-19'da 3 commit); founder kararı "ayrı dosyada kur".

TASARIM SÖZLEŞMESİ (founder kanvası `Hastaliklar.dc.html` — birebir uygulanan kısımlar):
  · Koyu `--ink` KAPAK: eyebrow + H1 + kurşun cümle + arama + sağda ZİNCİR şeridi
    (SORU → YANIT → DAYANAK) ve kaynak beyanı.
  · İKİ SÜTUN: solda DİZİN (sayaç + bölüm çipleri + kaydırılabilir liste),
    sağda seçili hastalık kartı (kod rozeti, ad, güncelleme, iki eylem, dört sekme).
  · Koyu "BU HASTALIKTA SOR" paneli (öneri çipleri + kredi notu) ve YASAL BANT.
  · Mikro-etiket motifi: `--font-mono`, büyük harf, harf aralıklı (DİZİN · TANIM · GÜNCELLEME).

⚠⚠ TASARIMDAN BİLEREK SAPILAN ÜÇ YER — üçü de "veri yok"tan doğdu, süsleme değil:
  (a) **KOD ROZETİ "ICD-10" DİYE SABİTLENMEZ.** Kanvas `ICD-10 I50` gösteriyor; ürünün
      hastalık tablosu **ICD-11 anahtarlı** (`disease.icd11_code`; `code_map` icd11 35.522
      vs icd10 8.450 satır). Rozet, kaydın GERÇEKTEN taşıdığı sistemi yazar — elde ICD-10
      varsa onu, yoksa ICD-11'i. Sabit "ICD-10" etiketi, satırların çoğunda YANLIŞ olurdu.
  (b) **ÇİPLER "BRANŞ" DEĞİL "BÖLÜM".** `core.disease`te uzmanlık/branş kolonu YOKTUR
      (ölçüldü). Çipler ICD-11 **bölüm** kodundan türetilir ve öyle ADLANDIRILIR. Bunlara
      "Kardiyoloji/Dahiliye" demek, veriyle desteklenmeyen bir uzmanlık iddiası olurdu.
  (c) **"İLİŞKİLİ İLAÇ SINIFLARI" DEĞİL "İLİŞKİLİ İLAÇLAR".** `drug_disease` bağı etken
      madde adı verir; sınıf verisi (`core.drug_class_cache`) BOŞ ve modülü ertelenmiş.
      "Sınıf" demek, gösterdiğimiz şeyin ne olduğu konusunda yanlış beyandır.

⚠ ATIF ROZETİ (`Europe PMC ↗`) kaynağı OLAN satırda basılır; olmayanda BASILMAZ —
  rozetsiz satır "kaynağı yok" demektir, sahte provenans üretilmez (`kaynaklar.py` ilkesi).
⚠ STATPEARLS bu yüzeyde GÖSTERİLMEZ (P0 lisans) — süzgeç `hastalik.py`de, burada değil.
⚠ TANI KOYMAZ: yasal bant sayfadan ÇIKARILAMAZ (FDA Kriter-4 dili).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `M` | `{'tr': {'eyebrow': 'HASTALIK KÜTÜPHANESİ', 'h1': 'Hastalıklar', 'lead': 'Bir hastalığın tanımını, ilişkili ilaçlarını ve` | 39 |
| `ICD11_BOLUM` | `{'1': ('Enfeksiyon', 'Infectious'), '2': ('Neoplazm', 'Neoplasms'), '3': ('Kan', 'Blood'), '4': ('Bağışıklık', 'Immune')` | 192 |
| `HASTALIK_CSS` | `'\n/* ⚠⚠ RAY OFSETİ KABUKTAN GELİR — bu sayfa `head()`+`nav()` kabuğunu\n   (`.stage > .content > .content-inner`) kulla` | 217 |
| `_SORU_SABLON` | `{'tr': {'hekim': '{x} hastasında güncel yönetim yaklaşımı nedir?', 'hasta': '{x} hastalığını hastama sade bir dille nası` | 398 |
| `_YONETIM` | `{'tr': 'Bu kaydın yönetim özeti, tam klinik anlatım içinde üretilir — iskeletli, atıflı ve bu hastalığın kaydına bağlı.'` | 412 |
| `JS` | `'<script>\n(function(){\n var E=document.getElementById(\'hst-etiket\');\n if(!E) return;\n function D(k){ return E.getA` | 526 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `T` | `(lang: str) -> dict` | 184 |  |
| `_bolum_json` | `(lang: str) -> str` | 387 | ICD-11 bölüm haritasının JS karşılığı (sunucuda dile ÇÖZÜLÜR). |
| `govde` | `(lang: str) -> str` | 420 | Sayfanın TAM gövdesi. |
| `_etiket` | `(lang: str) -> str` | 488 | JS'in okuyacağı etiket taşıyıcısı — tüm dizeler sunucuda `lang` ile üretilir. |

## `saglik/app/hekim_iz.py`

`244 satır` · `6 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HEKİM DAVRANIŞ İZİ — `app.doctor_event` kayıt katmanı (#726b, founder 2026-09-04).

Founder isteği: "hekimin kaç hastası var, hasta analizinde neler yaptı, hastalıklar sordu mu —
ne yaptığını detaylı görmemiz ve KAYIT TUTMAMIZ lazım."

⚠⚠ NE ÇÖZER (ölçüm: `scratchpad/_kamil_iz_envanteri.md`, 04.09, 124 rota AST taraması):
  1. **`app.usage.mode` ÖZELLİK DEĞİL SORU TÜRÜDÜR.** `cds/pipeline` çoğu uçta classifier'ın
     `res["mode"]`ini yazar → `/api/chat`, `/api/whatmissed`, `/api/deep-analyze` ve
     `/api/council` HEPSİ aynı `clinical` satırını üretir. Burada yüzeyi classifier değil
     **çağıran uç** yazar, dolayısıyla ayrım kaynağında doğar.
  2. **İzsiz yüzeyler.** Etkileşim taraması, böbrek dozu, SGK, hastalıklar, hesaplayıcılar,
     referans araması, hasta kartı AÇMA, dosya görüntüleme — hiçbiri satır yazmıyordu.

⚠⚠ BU BİR FREN DEĞİL, KAYIT. Hiçbir ücretsiz güvenlik ucuna kapı EKLEMEZ, hiçbir turu
  ücretlendirmez, `app.usage`a dokunmaz. `credits`/`store.FREE_MODES`/`lifetime_q` mantığı
  DEĞİŞMEDİ.

⚠⚠ FAIL-OPEN VE ÜRÜN AKIŞINI ASLA BOZMAZ. Yazma `with conn.transaction():` SAVEPOINT'i
  içindedir; hata log'a düşer, çağıran etkilenmez.
  ⚠⚠ SAVEPOINT'İN İÇİNDE `commit()` ÇAĞIRMA — psycopg3 YASAKLAR ve yazma **SESSİZCE DÜŞER**
    (bu depoda birebir yaşandı: `case_timeline` arşivi HER çağrıda düşüyordu, 27.08).
    Commit SAVEPOINT'ten SONRA atılır.
  ⚠ Ve bu yüzden kapının iddiası "fonksiyon patlamadı" DEĞİL: satır **AYRI BAĞLANTIDAN**
    geri okunur (bir bağlantı kendi commit'lenmemiş yazmasını görür → aynı bağlantı kanıt değil).

⚠ ÇIKIŞLI/ANONİM KULLANIMDA SATIR YAZILMAZ (`doctor_id` yoksa erken dönüş) — anonim iz
  tutmuyoruz. `/etkilesim` ve `/hesaplayicilar` ücretli iniş sayfalarıdır ve çıkışlı
  trafiğin tamamı bu yüzden kayıt DIŞIDIR.

⚠⚠ BİLEREK BAĞLANMAYAN DÖRT UÇ — eksiklik değil KARAR (envanterde "izsiz" görünürler).
  ⚠ Sayı 04.09'da 3'ten 4'e DÜZELTİLDİ (Fırat): aşağıda ÜÇ gerekçe maddesi var ama
    DÖRT uç sayılıyor — uçların hepsi doğru şekilde bağlanmamıştı, yanlış olan yalnız
    başlıktaki SAYIydı. Madde sayısını uç sayısı sanmak, listeyi olduğundan kısa gösterir.
  · `GET /api/suggest` — autocomplete, HER TUŞ VURUŞUNDA çağrılır. Bağlansaydı tek arama
    8-10 satır üretir, tablo şişer ve dağılım kartı okunamaz hâle gelirdi. Aranan terim
    ZATEN `referans_ara` ile bir kez ve TAM olarak kaydediliyor → kayıp bilgi YOK.
  · `GET /api/threads`, `GET /api/mycases` — istemci listeleme/yoklama uçları; hekimin bir
    KARARINI değil sayfanın kendi kendini doldurmasını ölçerler. Sayfa açılışı `panel` /
    `hasta_liste` ile zaten kayıtlı.
  · `GET /api/calendar` — aynı gerekçe; `takvim` sayfa açılışı kayıtlı.
  ⚠ Bunları eklemek isteyen önce ŞUNU ölçsün: eklenen satır hekimin bir kararını mı
    gösteriyor, yoksa istemcinin kendi trafiğini mi? İkincisi ise kayıt değil GÜRÜLTÜDÜR
    ve gerçek sinyali gizler.

⚠⚠ `detay` KİŞİSEL VERİDİR (founder K2: aranan terim saklanır) → gizlilik metniyle ATOMİK.
  Klinik serbest metin (soru gövdesi, hasta anlatısı, yanıt) BURAYA YAZILMAZ; yalnız kısa
  arama terimi ya da seçilen kaydın adı girer. Tavan `_DETAY_MAX`.

⚠⚠ SAKLAMA = HESAP ÖMRÜ, ayrı purge job'u YOK (Ömer 04.09; ayrıntı SAKLAMA bloğunda).
  Vaat TR+EN metinde: "hesabınız açık olduğu sürece saklanır; hesap kapanışında hesabınızın
  geri kalanıyla birlikte silinir." ⇒ `ON DELETE CASCADE` bu vaadi taşıyan TEK mekanizmadır,
  dekorasyon DEĞİL. Kapı onu ayrı bağlantıdan ölçer.

Kapı: `scratchpad/hekim_iz_verify.py` (CI SUITES).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_DETAY_MAX` | `500` | 83 |
| `_ANAHTAR` | `'KLIVANCE_DOCTOR_EVENT'` | 85 |
| `_UCRETSIZ_YUZEYLER` | `frozenset({'etkilesim', 'bobrek_doz', 'sgk_sayfa', 'sgk_ara', 'recete_hub', 'recete_case', 'sgk_case', 'hastaliklar', 'h` | 95 |
| `_YUZEY_BILINMEYEN` | `'siniflandirilmamis'` | 123 |
| `YUZEYLER` | `frozenset(FREN_YUZEY \| _UCRETSIZ_YUZEYLER \| {_YUZEY_BILINMEYEN})` | 125 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_temizle` | `(detay) -> str \| None` | 128 | `detay`ı kolona girmeden ÖNCE güvenli hâle getir. Geçersizse None. |
| `iz_birak` | `(conn, doctor_id, yuzey: str, detay = None) -> bool` | 151 | Hekimin bir yüzeye dokunduğunu kaydet. Döner: satır yazıldı mı. |
| `iz_istek` | `(request, yuzey: str, detay = None) -> bool` | 187 | Çağıranın AÇIK bir `conn`u yokken kullanılan sürüm — kendi bağlantısını açar. |
| `yuzey_dagilimi` | `(conn, doctor_id: int, gun: int = 90) -> list[tuple[str, int]]` | 221 | Hekimin son `gun` gündeki yüzey dağılımı — [(yuzey, adet), ...] azalan. |
| `son_eylemler` | `(conn, doctor_id: int, limit: int = 300)` | 229 | Hekimin son eylemleri (en yeni üstte) — [(created_at, yuzey, detay), ...]. |
| `toplam` | `(conn, doctor_id: int) -> int` | 239 | Hekimin TOPLAM iz sayısı (saklama penceresi içinde kalanlar). |

## `saglik/app/i18n_pairs.py`

`605 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
TR→EN çeviri çiftleri (i18n_mw tarafından uzun-önce sırayla uygulanır).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-07-29, mimari denetimi). İçerik
AYNEN taşındı — tek karakter değişmedi; render çıktısı bayt bayt doğrulandı.
main.py bu adları yeniden dışa aktarır, yani mevcut importlar (`from saglik.app.main
import ...`) ve doğrulama scriptleri KIRILMAZ.

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `EN_PAIRS` | `[('Hesabınız <b>doğrulama bekliyor</b> — tam erişim için diploma doğrulaması gerekir (deneme modundasınız).', 'Your acco` | 13 |

## `saglik/app/iyzico.py`

`423 satır` · `14 fonksiyon` · `1 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
iyzico ödeme entegrasyonu — Checkout Form (hosted, 3DS dahil). TR-önce kredi kartı tahsilatı.

MVP modeli: her dönem için TEK ÇEKİM (ön-ödemeli). Hekim 1 ay / 1 yıl öder → o dönem plan+kota
açılır (current_period_end = now + interval). Otomatik yenileme (düzenli ödeme) FAZ-2 — Türkiye'de
tekrar-çekim ayrı banka anlaşması + iyzico Abonelik ürünü ister; MVP'yi ona bağlamayız.

Akış:
  1) create_checkout(...) → iyzico CheckoutFormInitialize → paymentPageUrl (hosted forma yönlendir);
     app.iyzico_payment'e 'pending' satır (token PK) yazılır.
  2) hekim formda öder (kart + 3DS) → iyzico tarayıcıyı callbackUrl'e `token` ile POST'lar.
  3) verify(token) → CheckoutForm.retrieve → paymentStatus/fraudStatus SUNUCUDA doğrulanır;
     paidPrice beklenen tutarla karşılaştırılır (kurcalama freni). main.py callback hak-edişi
     idempotent verir (granted bayrağı atomik).

Anahtar .env'de: IYZICO_API_KEY, IYZICO_SECRET_KEY (zorunlu), IYZICO_BASE_URL (vars. SANDBOX).
Fiyatlar TL, env-güdümlü (IYZICO_PRICE_*) — canlıya geçmeden GERÇEK TL fiyatları ayarla.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_EMAIL_RE` | `re.compile('^[^@\\s]+@[^@\\s]+\\.[A-Za-z]{2,}$')` | 29 |
| `_EMAIL_FALLBACK` | `'odeme@klivance.com'` | 30 |
| `PLAN_INTERVAL` | `{'aylik': 'month', 'yillik': 'year', 'kurucu': 'month', 'kurucu_yillik': 'year'}` | 45 |
| `_PLAN_PRICE_DEFAULT` | `{'aylik': '1450.00', 'yillik': '14500.00', 'kurucu': '2900.00', 'kurucu_yillik': '29000.00'}` | 73 |
| `_TR_PLATE` | `{'adana': 1, 'adiyaman': 2, 'afyonkarahisar': 3, 'afyon': 3, 'agri': 4, 'amasya': 5, 'ankara': 6, 'antalya': 7, 'artvin'` | 124 |

### `class IyzicoMissing(RuntimeError)`  <sub>satır 40</sub>

iyzico anahtarları .env'de yok — arayüz zarifçe uyarır.

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_safe_email` | `(email: str \| None) -> str` | 33 |  |
| `_fmt` | `(v) -> str` | 82 | iyzico fiyatı 2 ondalıklı string ister ('1990.00'). |
| `plan_price_try` | `(plan: str) -> str` | 87 | Plan için TL fiyat: env IYZICO_PRICE_AYLIK / IYZICO_PRICE_YILLIK, yoksa varsayılan. |
| `_opts` | `() -> dict` | 93 |  |
| `_parse` | `(resp) -> dict` | 104 | iyzipay SDK yanıtını dict'e çevir — SDK sürümleri arası savunmacı. |
| `_zip_for_city` | `(city: str \| None) -> str` | 138 |  |
| `_sub_paygroup` | `() -> str` | 146 | Abonelik satın-alımının iyzico paymentGroup'u. Tek-çekim modelinde (iyzico recurring |
| `_buyer` | `(doctor: dict, ip: str, billing: dict \| None = None, phone: str = '') -> dict` | 157 | Alıcı objesi — iyzico zorunlu alanları. 2026-07-18: fatura profili (billing jsonb) ve |
| `_address` | `(doctor: dict, billing: dict \| None = None) -> dict` | 186 | Fatura profili (app.doctor.billing) varsa GERÇEK il/adres kullanılır (e-fatura + fraud |
| `create_checkout` | `(conn, doctor: dict, *, kind: str, plan: str, callback_url: str, ip: str = '') -> str` | 201 | Checkout Form başlat → hosted ödeme sayfası URL'si döndür. app.iyzico_payment'e |
| `verify` | `(token: str) -> dict` | 268 | Ödeme sonucunu iyzico'dan SUNUCUDA doğrula (callback token'ına GÜVENME). |
| `reconcile_pending` | `(conn, days: int = 7, limit: int = 100, apply: bool = True) -> dict` | 293 | MUTABAKAT (2026-07-29, denetim P4): hak-edişi verilmemiş son ödemeleri iyzico'dan |
| `_not_yaz` | `(conn, token: str, notu: str) -> None` | 355 | Başarısız satıra kısa not yaz — KUYRUK ROTASYONU için (best-effort). |
| `_mutabakat_makbuzu` | `(conn, info: dict, txn: str) -> bool` | 373 | Mutabakatla açılan ödemenin MAKBUZUNU gönder (best-effort). Gönderildi→True. |

## `saglik/app/iyzico_abonelik.py`

`553 satır` · `25 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
iyzico ABONELİK (recurring) — Subscription API istemcisi + hak-ediş defteri (#774c, C1).

⚠⚠ BAYRAK KAPALIYKEN ÜRÜNDE HİÇBİR ROTA BU MODÜLÜ ÇAĞIRMAZ (`credits.iyz_recurring()`
   varsayılan KAPALI). Rotalar C2'de bağlanır; yasal metin + bayrak C3'te açılır.
   Tek istisna: `iptal_best_effort` — hesap silme yolları referans DOLUYSA çağırır
   (bayrak kapalıyken referans dolu hesap yoktur → fiilen no-op).

MODEL (tasarım `scratchpad/_selim_774c_abonelik_plan.md`; belge `docs.iyzico.com`, 2026-09-12 ölçümü):
  ürün → fiyat planı (aylık/yıllık, RECURRING) → `SubscriptionCheckoutForm` (hosted
  kart formu + 3DS; yanıt `checkoutFormContent` = gömülü script, `paymentPageUrl` YOK)
  → `verify_checkout(token)` sunucu-taraflı → `abonelik_bagla` (doctor'a referanslar)
  → iyzico dönemsel çeker → webhook `subscription.order.success|failure` →
  `webhook_uygula` → `siparis_uygula` (her sipariş `app.iyzico_payment`e AYRI satır,
  `subord:` önekli token, `granted` idempotans kilidi) → `current_period_end` = iyzico
  `orders[].endPeriod` (⚠ `now+interval` DEĞİL: çekim günü ile dönem sonu kaymasın).

PARA İNVARİANTLARI (bozarsan fazla/eksik tahsilat):
  · Hak-ediş yalnız `orderStatus == 'SUCCESS'` sipariş için; webhook gövdesine GÜVENİLMEZ,
    sipariş iyzico'dan `detail()` ile yeniden okunur (callback `verify` deseni).
  · FAIL-CLOSED: imza doğrulanamıyorsa / merchantId-secret yoksa / abonelik bizde yoksa
    → hiçbir yazma. Başarısız çekim (`failure`) dönem sonunu DEĞİŞTİRMEZ, `unpaid` yazar.
  · Aynı `orderReferenceCode` (iyzico webhook'u 2xx alana dek 3 kez tekrarlar) → TEK hak-ediş.
  · İptal `current_period_end`e DOKUNMAZ (ödenen dönemin sonuna kadar erişim sürer).
  · Hesap silinirken iyzico aboneliği best-effort iptal edilir (aksi hâlde kart çekilmeye
    devam eder); iptal başarısızlığı silmeyi BLOKLAMAZ, loglanır.

WEBHOOK İMZASI (belge, ek-servisler/webhook "Abonelik Bildirimlerinin Doğrulanması"):
  X-IYZ-SIGNATURE-V3 = HMAC-SHA256(key=secretKey,
      msg = merchantId + secretKey + eventType + subscriptionReferenceCode
            + orderReferenceCode + customerReferenceCode).hex()
  ⚠ `merchantId` gövdede YOK → env `IYZICO_MERCHANT_ID`. ⚠ SDK 1.0.46'nın
  `verify_signature`ı sonucu print eder, DÖNDÜRMEZ — bu yüzden burada kendi HMAC'imiz.
  ⚠ Başlığın gönderilmesi için iyzico hesabında "webhook signature" özelliği AÇTIRILMALI
  (entegrasyon@iyzico.com) — founder aksiyonu; açık değilse uç FAIL-CLOSED kalır.

SDK: `iyzipay` 1.0.46 abonelik sınıflarını taşıyor ama (a) tekil abonelik detay ucu
(`GET /v2/subscription/subscriptions/{ref}`) SDK'da YOK, (b) `HTTPSConnection` zaman aşımı
yok → tüm çağrılar `_v2_call` ile (SDK'nın kendi imza üreticisi + timeout). Kapı takımı
`_v2_call`ı sahteyle değiştirir; gerçek ağ çağrısı YAPMAZ.

⚠ SANDBOX (2026-09-12 ölçüldü): anahtar geçerli (BinNumber success) ama abonelik uçları
HTTP 422 `100001` → sandbox üye işyerinde Abonelik modülü KAPALI; e2e için iyzico'dan
açtırılmalı. Bu modül belge şemasına göre yazıldı, canlıya istek ATILMADI.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `OLAY_BASARILI` | `'subscription.order.success'` | 58 |
| `OLAY_BASARISIZ` | `'subscription.order.failure'` | 59 |
| `DURUM_ESLEME` | `{'ACTIVE': 'active', 'PENDING': 'pending', 'UNPAID': 'unpaid', 'UPGRADED': 'active', 'CANCELED': 'canceled', 'EXPIRED': ` | 62 |
| `TOKEN_ONEK` | `'subord:'` | 64 |
| `KIND` | `'sub_recur'` | 65 |
| `_AYAR_URUN` | `'iyz_sub_product'` | 66 |
| `_AYAR_PLAN` | `{'aylik': 'iyz_sub_plan_aylik', 'yillik': 'iyz_sub_plan_yillik'}` | 67 |
| `_INTERVAL` | `{'aylik': 'MONTHLY', 'yillik': 'YEARLY'}` | 68 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_v2_call` | `(method: str, path: str, body: dict \| None = None, timeout: int = 20) -> dict` | 74 | iyzico v2 ucu çağır → dict. İmza SDK'nın kendi üreticisiyle (IYZWSv2), gövde SDK |
| `_ok` | `(r: dict) -> bool` | 95 |  |
| `_ts` | `(ms) -> datetime \| None` | 99 | iyzico epoch-milisaniye → aware datetime (UTC). Çöp → None. |
| `ensure_plans` | `(conn, planlar = ('aylik', 'yillik')) -> dict` | 110 | Ürün + fiyat planlarını iyzico'da bir kez oluştur, referansları `app.setting`e yaz. |
| `create_checkout` | `(conn, doctor: dict, *, plan: str, callback_url: str, ip: str = '', initial_status: str = 'ACTIVE') -> dict` | 153 | Abonelik checkout formunu başlat → {token, form_html, url, expire}. `app.iyzico_payment`e |
| `verify_checkout` | `(token: str) -> dict` | 190 | Form sonucunu iyzico'dan SUNUCUDA oku (callback token'ına GÜVENME). |
| `detail` | `(subscription_ref: str) -> dict \| None` | 202 | Abonelik detayı (SDK'da olmayan uç) → items[0] \| None. Siparişler `orders[]`. |
| `cancel` | `(subscription_ref: str, timeout: int = 20) -> bool` | 219 |  |
| `iyz_ref` | `(conn, doctor_id: int) -> str \| None` | 224 | Silme yolu için: referansı SİLMEDEN ÖNCE oku (satır gidince okunamaz); hata → None + log. |
| `activate` | `(subscription_ref: str) -> bool` | 234 |  |
| `retry` | `(order_ref: str) -> bool` | 239 | Başarısız siparişi yeniden çek — iyzico KENDİLİĞİNDEN tekrar denemez (belge). |
| `card_update_form` | `(subscription_ref: str, customer_ref: str, callback_url: str) -> dict` | 245 | Kart güncelleme formu → {token, form_html}. Başarısız çekim mailinin TEK linki. |
| `webhook_imza` | `(secret: str, merchant_id: str, olay: dict) -> str` | 258 |  |
| `webhook_dogrula` | `(body: bytes \| str, imza_basligi: str \| None) -> tuple[bool, dict]` | 265 | (geçerli mi, olay). ⚠ FAIL-CLOSED: başlık yok / secret yok / merchantId yok / |
| `webhook_uygula` | `(conn, olay: dict) -> dict` | 284 | Doğrulanmış olayı uygula. Döner {"uygulandi": bool, "neden": str, "doctor_id": int\|None, |
| `abonelik_bagla` | `(conn, doctor_id: int, subscription_ref: str, customer_ref: str \| None, iyz_status: str \| None = 'ACTIVE', plan: str \| None = None) -> None` | 333 | Aboneliği hekime bağla. ⚠ P1-2 (Fırat, 12.09): `plan` BURADA yazılır (ACTIVE ise) — eskiden |
| `_plan_turet` | `(conn, sub_ref: str, det: dict \| None, siparis: dict \| None, doctor_plan) -> str` | 356 | Siparişin PLANI — üç kaynak, güven sırasıyla (P1-2): (1) bu aboneliğin defter satırı |
| `durum_yaz` | `(conn, doctor_id: int, durum: str) -> None` | 383 | `subscription_status` yaz — `current_period_end`e DOKUNMAZ (erişim ödenen dönemin |
| `iptal_isaretle` | `(conn, doctor_id: int) -> None` | 390 |  |
| `_siparis_not` | `(conn, doctor_id, sub_ref, order_ref, plan, iyz_status) -> bool` | 394 | Başarısız/bekleyen siparişin izi (granted=false, hak-ediş YOK) — /admin/odeme görsün. |
| `siparis_uygula` | `(conn, doctor_id: int, subscription_ref: str, order_ref: str, *, plan: str, amount, period_end: datetime, iyz_status: str = 'ACTIVE') -> dict \| None` | 406 | BAŞARILI siparişin hak-edişini ATOMİK + İDEMPOTENT ver. Döner özet \| None (zaten verildi). |
| `iptal_best_effort` | `(conn, doctor_id: int, ref: str \| None = None) -> bool \| None` | 451 | HESAP SİLME YOLU KANCASI — referans doluysa iyzico aboneliğini iptal et. |
| `ilk_siparis_uygula` | `(conn, token: str, doctor_id: int, subscription_ref: str, customer_ref, siparis: dict, iyz_status: str = 'ACTIVE') -> dict \| None` | 473 | Abonelik BAŞLANGICINDAKİ ilk siparişi checkout TOKEN satırına yaz (callback yolu). |
| `hesap_durumu` | `(conn, doctor_id: int) -> dict` | 504 | Hesap sayfası/abonelik sayfası için recurring durumu (tek sorgu). |
| `bekleyenleri_aktive_et` | `(conn, apply: bool = True, limit: int = 25) -> dict` | 527 | PENDING (kart doğrulanmış, çekim ertelenmiş) abonelikleri ödenen dönem BİTİNCE aktive et |

## `saglik/app/kokpit_body.py`

`577 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
ANALİZ KOKPİTİ — CSS + JS sabitleri (spec: scratchpad/_omer_kokpit_spec.md §3;
#660b bilgi mimarisi kararları: scratchpad/_deniz_kokpit_660b_spec.md).

⚠⚠ NEDEN AYRI MODÜL (yerleşim kararı, 2026-09-02): spec §3.7 sabitlerin evi olarak
   `cases_body.py`yi öngörüyordu (+~430 satır) ama o dosya `dosya_boyut_verify`
   tavanında 405/409 ve girdisi "BÜYÜME BEKLENMEZ" der. Depo faz-kesim deseni tam bu
   durum için var (recete_case.py aynı gerekçeyle ayrı doğdu): YENİ SAF modül,
   `cases_routes.py` buradan import eder.

⚠⚠ SAF SABİT SÖZLEŞMESİ (`cases_body.py` ile AYNI): bu dosyaya YALNIZ modül düzeyi
   string ataması konur — def/class/import/f-string YASAK. `analiz_kokpit_verify`
   bunu AST ile çiviler (gerekçe: DAVRANIŞ burada saklanamasın).

⚠⚠ KOKPİT NE YAPAR: `#ana-ozet-list details.visit-ana` başına, model çıktısının
   11 başlıklı iskeletini (`prompts._ANALIZ_ISKELET`) İSTEMCİDE bölümleyip sinoptik
   ızgara çizer. Orijinal `.va-body` SİLİNMEZ — kokpit içindeki kapalı
   `<details class="ckpt-ham">` altına TAŞINIR. JS ölürse hiçbir şey taşınmaz →
   bugünkü görünüm birebir kalır (fail-safe). Genel Bakış çapası (`#ckpt-capa`,
   sunucu BOŞ `hidden` iskelet basar) ilk kokpitin şerit sayaçlarını AYNI kurucuyla
   kopyalar → tek sayım kaynağı; JS ölürse satır görünmez, yanlış sayı basılmaz.

⚠⚠ DEĞİŞMEZLER (spec §3.4 + Selim'in 4 şartı; ihlal = iş reddedilir):
   · KIRMIZI (#D64545) YALNIZ modelin kendi `⚠ ACİL` önekli satırından — istemci
     referans aralığından "anormal" HESAPLAMAZ, hüküm üretmez.
   · Yeşil (`--verify`) kokpit scope'unda YASAK — aklama sinyali üretilemez.
   · `_COCKPIT_JS` içinde `innerHTML` = 0 (tüm metin textContent/createElement).
   · Sunucuya seri yazılmaz · dış servis/CDN yok · localStorage'a seri yazılmaz ·
     indirme/dışa-aktarma yok (Selim, hukuk okuması — biri düşerse founder+avukat).
   · Madde metni TEK KARAKTER değiştirilmez (⚠ Dikkat paneli B9'un aynasıdır).
   · JS gövdesinde TÜRKÇE LİTERAL YOK — tüm metin `__CKPT__` (json.dumps, sunucudan).
   · Sıfır sayaç HÜKÜM değil EYLEM dilinde ("ACİL işaretlenmedi" — model önek basmadı)
     ve meta renginde (--slate), tıklanmaz (#660b-2).

⚠ #660b (2026-09-03, Deniz kararı — ölçülmüş): "Gözden geçirme listesi" karosu KALKTI
  (Dikkat+Eksik ile 7/7 birebir tekrar; 390'da 909px). Dayanak yalnız sayı/atıf
  taşımayan Dikkat maddesinin altında nötr alt satır olarak ("dayanak maddede yazılı
  değil"). Lab karosu TEK karo, İKİ blok: seri (masaüstü 2 sütun) + TEK ÖLÇÜM (tarihli
  satır — "ferritin 300 hangi gün?" belirsizliği kapandı). Şerit `max-height`/
  `overflow:hidden` KALDIRILDI: 390'da 2. satır 5px görünüyor, 3. satır tümüyle gizliydi
  (mobilde tıklanır DİKKAT hücresi fiilen yoktu).

⚠ 2026-09-04 (#B1/#B3): AMBER bayatlık hücresi (`T.bekleyen` SUNUCUDAN iner, `fetch` Selim
  şartıyla YASAK) `dortHucre`nin DIŞINDA (İ21 çapayı 4 hücrede çiviler) ve 0 iken HIÇBİR
  ŞEY basmaz ("0 bekleyen" ≠ "güncel"); `.ckpt-ara` modelin aralığı, salt SUNUM.

⚠ Yer tutucu: `__CKPT__` — cases_routes `json.dumps` çıktısıyla doldurur
  (ensure_ascii varsayılan AÇIK: TR dizgiler \uXXXX kaçışlı iner, EN_PAIRS çakışma
  yüzeyi küçülür; kapı `en(script)==script` bayt paritesini ayrıca çiviler).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_COCKPIT_CSS` | `'.ckpt{display:grid;grid-template-columns:repeat(12,1fr);gap:12px;margin:8px 13px 12px;overflow-wrap:anywhere;grid-templ` | 58 |
| `_COCKPIT_JS` | `'\n(function(){\n"use strict";\nvar T;try{T=__CKPT__;}catch(e){T=null;}\nif(!T\|\|!T.htr\|\|!T.hen\|\|!T.L)return;\nvar ` | 197 |

## `saglik/app/kritik_serit.py`

`216 satır` · `8 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
KRİTİK BULGU KADEMESİ — TAŞIYICI + ÇÖZÜCÜ (#782c, founder 2026-09-15 "aç, daha da geliştir").

SAF MODÜL: DB yok, LLM yok, HTML yok, FastAPI yok, ROTA YOK → `main.kur` çağrısı GEREKMEZ.
Tek iş: `pipeline.goruntu_on_okuma`nın `kritik` yükünü DOĞRULAMAK ve kademeyi ön-okuma
metninin İÇİNDE taşınabilir/geri okunabilir bir bloğa çevirmek. Çizim ÇAĞIRANIN
(`on_okuma.serit`/`kademe_etiketi` kart kolunda, `chat_goruntu_js` sohbette).

⚠⚠ NEDEN AYRI MODÜL: `on_okuma_kayit.py` tavanını (`scratchpad/dosya_boyut_verify.py`) tek
turda 119 satır aştı ve Ömer tavanı YÜKSELTMEYİP BÖLME dedi (15.09, #781c-3 ile aynı karar).
Blok BİREBİR taşındı — davranış değişmedi; kapı `kritik_serit_verify` taşımadan önce de
sonra da AYNI iddiaları koşturur.

⚠ `on_okuma_kayit`ten ÇEKİLMEZ, oraya BAĞIMLI DEĞİLDİR (ters yön de yok): ortak olan tek şey
`cds.isaretler.ONOKUMA_ONEK` ve o zaten TEK KAYNAKTAN okunuyor.

── HEKİME GÖRÜNEN METİN BİZİM DEĞİL ────────────────────────────────────────────────────────
⚠⚠ ÇEKİRDEK KURAL (Ömer): KIRMIZININ YANMAMASI BULGUNUN YOKLUĞU DEĞİLDİR. Nötr şerit bunu
  AÇIKÇA yazar; "temiz/normal/sorun yok" dili YASAK. Tarama HİÇ koşmadıysa nötr de
  GÖSTERİLMEZ. Bu cümlelerin hepsi `cds/kritik_goruntu.blok`tan gelir — bu modül CÜMLE
  KURMAZ, yalnız TAŞIR. Metin KISALTILMAZ, özetlenmez, cümle EKLENMEZ.

── TAŞIYICI SEÇİMİ — ÖLÇÜLDÜ, KOLON AÇILMADI (15.09) ───────────────────────────────────────
`app.case_file`de serbest/JSON kolon YOK (`on_okuma` text · `on_okuma_model` text ·
`on_okuma_durum` BEYAZ LİSTELİ · iki timestamptz). Kademeyi taşıyacak tek yer ÖN-OKUMA
METNİNİN KENDİSİ. Bunun bedava getirdiği üç şey ölçüldü:
  (a) `on_okuma_kaydet._mesaj_onokumasi` metni MESAJDAN okuyor → karta kaydet ucunda kademe
      EK KOD OLMADAN korunur;
  (b) sohbet geçmişi (`openThread`) aynı metni çiziyor → canlı ile geçmiş TEK yol;
  (c) kart kolu `on_okuma` kolonunu okuyan üç yerden geçiyor → Belgeler satırı ve `<details>`
      aynı kaynaktan.

⚠⚠ BLOĞUN YERİ ÖLÇÜLDÜ — METNİN EN BAŞINA KONMAZ: `on_okuma_kayit._icerik_dili` ve
  `onokuma_metni_mi` `lstrip().startswith(ONOKUMA_ONEK)` ile çalışıyor. Başa konan bir blok
  (1) EN ön-okumayı TR sanıp `ONOKUMA_GECMIS_ONEK`i YANLIŞ DİLDE bastırırdı, (2) `onay_reddi`nin
  önek kolunu sessizce kapatırdı (hekim makine metnini önekiyle yapıştırınca 400 yerine geçerdi).
  ⇒ blok ÖNEK SATIRINDAN SONRA yerleşir; ayrıştırma KONUMDAN BAĞIMSIZDIR (`parseChips` dersi:
  `$`-çıpalı/konum varsayan desen bir gün sessizce eşleşmeyi bırakır).

⚠⚠ `pipeline.dogrula` TUZAĞI (Cahit, 15.09): işaretli metin bir kez DAHA `dogrula`ya verilirse
  işaret satırı ilk başlıktan ÖNCE olduğu için ISKELET DIŞI sayılıp SESSİZCE düşer — kademe
  kaybolur ve hiçbir şey kırmızı basmaz. Bu yüzden `kademe_ekle` `dogrula`dan SONRA çağrılır ve
  çağrıldığı iki yerde de metin bir daha `dogrula`ya GİRMEZ. Tuzağın GERÇEK olduğu ve ürünün
  ona girmediği kapıda ölçülür (`kritik_serit_verify` A17/A18).

⚠ GÖVDE ÇOK SATIRLIDIR (`kritik_goruntu.blok` başlık + durum satırları + koşulsuz yokluk
  uyarısı + kaynaksızlık ibaresi üretir) → satır DEĞİL BLOK taşınır: açılış işareti, gövde,
  kapanış işareti.

Kapı: `scratchpad/kritik_serit_verify.py` (A sözleşme+taşıma · B node DOM-stub çizim ·
C metin disiplini · D bozma tatbikatı · E i18n · F uçtan uca DB).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KRITIK_KAPANIS` | `'[/KLV-KRITIK]'` | 63 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_kg` | `()` | 66 |  |
| `kritik_onek` | `() -> str` | 71 | Üreticinin açılış işareti öneki (`[KLV-KRITIK:`) — istemciye de BURADAN enjekte edilir |
| `kademeler` | `() -> tuple` | 77 |  |
| `kademe_coz` | `(kritik)` | 81 | `pipeline.goruntu_on_okuma` `kritik` yükünü DOĞRULA → `(kademe, durumlar, catisan)`. |
| `kademe_govdesi` | `(kritik, lang) -> str` | 115 | Hekime GÖRÜNEN kritik tarama metni — ⚠⚠ ÜRETİCİ TEK, BU KATMAN CÜMLE KURMAZ. |
| `kademe_ekle` | `(text, kritik, lang) -> str` | 138 | Kademe bloğunu ön-okuma metnine YERLEŞTİR (ÖNEK SATIRINDAN SONRA), düz metin döner. |
| `kademe_ayir` | `(text)` | 173 | `(kademe, govde, temiz_metin)` — ⚠⚠ kademe ÜÇ DURUMLU (`kritik_goruntu.kademe_oku`): |
| `kademe_ikincil` | `(govde, lang = None)` | 201 | Şeridi İKİ AĞIRLIĞA böl → `(ana_govde, ikincil_satir)` (Ömer kararı 15.09). |

## `saglik/app/landing.py`

`1343 satır` · `1 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
LANDING (herkese açık satış sayfası) — TR + EN gövdeleri ve parçaları.

⚠ `main.py`'den AYRILDI (2026-07-31, Faz 1). Sebep: main.py 8.490 satırdı, 1.469'u SAF VERİ'ydi
  (iki landing HTML'i + 4 parça sabiti); veri+rota aynı dosyada her değişikliğe bedel bindiriyordu.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ — etmemeli. Blok AST ile ölçülerek ayrıldı:
  dışarıya bağımlılığı YOK. Buraya main.py'den bir ad import edersen dairesel import doğar
  (main → landing → main) ve uygulama AÇILMAZ. Paylaşılan parçayı üçüncü bir modüle çıkar.

⚠ Landing'in DAVRANIŞI burada değil `main.py`'de: `root()` (dil seçimi, TRY fiyat uygulaması)
  ve `_landing_ctas()` (girişli/çıkışlı CTA farkı) orada kalır. Burası yalnız GÖVDE.

⚠ i18n: EN landing KENDİ sürümünü servis eder (`LANDING_HTML_EN`) → `i18n_mw` bu sayfada
  NO-OP'tur; buradaki İngilizce metin `EN_PAIRS`'e EKLENMEZ.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_HERO_LOGO` | `'<svg width="40" height="40" viewBox="0 0 128 128" xmlns="http://www.w3.org/2000/svg"><rect x="8" y="8" width="112" heig` | 18 |
| `_FOOTER_SOCIAL` | `'<div style="margin-top:14px;display:flex;gap:14px"><a href="https://www.facebook.com/profile.php?id=61591867442534" tar` | 27 |
| `_PAY_LOGOS` | `_PAY_LOGOS_HTML(False)` | 53 |
| `_PAY_LOGOS_EN` | `_PAY_LOGOS_HTML(True)` | 54 |
| `_LANDING_CSS` | `'<style>\n :root{--ink:#0C3B3E;--ink-2:#0A6B4E;--verify:#12B886;--verify-soft:#E4F3EE;--verify-ink:#053A2C;--amber:#E8A3` | 63 |
| `_LANDING_CSS_DEMO` | `'<style>\n#demo .demo-frame{max-width:760px;margin:26px auto 0;border:1px solid var(--line);border-radius:16px;backgroun` | 410 |
| `LANDING_HTML_EN` | `'<!DOCTYPE html><html lang="en"><head><meta charset="utf-8">\n<meta name="viewport" content="width=device-width, initial` | 444 |
| `LANDING_HTML` | `'<!DOCTYPE html><html lang="tr"><head><meta charset="utf-8">\n<meta name="viewport" content="width=device-width, initial` | 884 |
| `LANDING_HTML_EN` | `LANDING_HTML_EN.replace('{DGUN}', str(_DG))` | 1341 |
| `LANDING_HTML` | `LANDING_HTML.replace('{DGUN}', str(_DG))` | 1342 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_PAY_LOGOS_HTML` | `(en: bool = False) -> str` | 40 |  |

## `saglik/app/legal_body.py`

`261 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Yasal metinler — TR (_LEGAL) ve EN (LEGAL_EN).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-07-29, mimari denetimi). İçerik
AYNEN taşındı — tek karakter değişmedi; render çıktısı bayt bayt doğrulandı.
main.py bu adları yeniden dışa aktarır, yani mevcut importlar (`from saglik.app.main
import ...`) ve doğrulama scriptleri KIRILMAZ.

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_LEGAL` | `{'mesafeli': ('Mesafeli Satış Sözleşmesi', '\n<p><b>Madde 1 — Taraflar.</b> <b>SATICI:</b> ALFA 4N SAN. TİC. LTD. ŞTİ. (` | 13 |
| `LEGAL_EN` | `{'gizlilik': ('Privacy Notice', '\n<p><b>Data controller.</b> Klivance is a clinical decision-support service operated b` | 131 |

## `saglik/app/mailer.py`

`804 satır` · `14 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
E-posta gönderimi — SMTP (Resend transactional, gönderen info@klivance.com).

SMTP yapılandırılmamışsa (env yok) gönderim SESSİZCE ATLANIR ve link log'a yazılır —
uygulama kırılmaz (doğrulama non-blocking). Prod'da KLIVANCE_SMTP_* env'leri set edilince
gerçek mail gider. Sağlayıcı: Resend (smtp.resend.com:587, user='resend', pass=re_... API
anahtarı, FROM=info@klivance.com — Resend'de doğrulanmış domain). 2026-07-15'te canlıda
uçtan uca doğrulandı (Delivered → Gmail gelen kutusu). Gmail SMTP'den geçildi (Workspace
app-password kısıtı: 535 BadCredentials).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_BRAND_TEAL` | `'#0C3B3E'` | 84 |
| `_BRAND_GREEN` | `'#12B886'` | 85 |
| `_LEGAL_TR` | `'ALFA 4N SAN. TİC. LTD. ŞTİ. · VKN 0510246142 · Hamdibey Mah. İstiklal Cad. No:143, Biga/Çanakkale · KEP alfa4n@hs01.kep` | 88 |
| `_LEGAL_EN` | `'Seller: ALFA 4N SAN. TİC. LTD. ŞTİ. · Tax No 0510246142 · Biga/Çanakkale, Türkiye · KEP alfa4n@hs01.kep.tr'` | 90 |
| `_MERSIS` | `'0051024614200016'` | 105 |
| `_MERSIS_BICIM` | `_re.compile('\\d{16}')` | 108 |
| `_AY_TR` | `('Ocak', 'Şubat', 'Mart', 'Nisan', 'Mayıs', 'Haziran', 'Temmuz', 'Ağustos', 'Eylül', 'Ekim', 'Kasım', 'Aralık')` | 566 |
| `_AY_EN` | `('January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December` | 568 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `send_email` | `(to_addr: str, subject: str, html_body: str, text_body: str = '', *, unsub_url: str = '') -> bool` | 23 | HTML e-posta gönder. Döner: True (gönderildi) \| False (SMTP yok / hata → log). |
| `kunye_hazir` | `() -> bool` | 111 | Künye basılabilir mi — MERSİS numarası girilmiş **ve BİÇİMİ doğru** mu? |
| `kunye_tei` | `() -> str` | 128 | Ticari elektronik iletiye basılacak TEK SATIR kimlik; MERSİS yoksa BOŞ döner. |
| `_email_shell` | `(*, preheader: str, title: str, intro: str, cta_label: str, cta_url: str, fallback_label: str, note: str, tagline: str, callout: str = '', rows = None, legal: str = '', lang: str = 'tr', unsub_url: str = '', unsub_label: str = '') -> str` | 138 | Marka-uyumlu transactional e-posta gövdesi (TABLO tabanlı + inline stil → Gmail/Outlook/Apple |
| `verification_email_html` | `(link: str, lang: str = 'tr') -> tuple[str, str, str]` | 235 | Doğrulama maili (konu, html, text) döner. Marka-uyumlu email-güvenli şablon (_email_shell). |
| `password_reset_email` | `(link: str, hours: int = 2, lang: str = 'tr') -> tuple[str, str, str]` | 266 | Parola sıfırlama maili (konu, html, text). |
| `credit_granted_email` | `(credits: int, new_balance, note: str, doctor_name: str, link: str, lang: str = 'tr') -> tuple[str, str, str]` | 312 | Admin HEDİYE GÜN bildirimi (konu, html, text) — hekime 'süreniz uzatıldı' der. |
| `welcome_email` | `(name: str, link: str, lang: str = 'tr') -> tuple[str, str, str]` | 354 | Yeni Google kaydına 'hesabın hazır' hoş-geldin maili — doğrulama maili GEREKMEYEN Google |
| `subscription_receipt_email` | `(*, plan_label: str, credits: int, amount_disp: str, txn_id: str, interval: str, link: str, lang: str = 'tr', bonus: int = 0) -> tuple[str, str, str]` | 393 | Abonelik başlangıç + makbuz e-postası (konu, html, text). Satın alım onayı — TRANSACTIONAL |
| `first_question_email` | `(name: str, link: str, ornekler, lang: str = 'tr', unsub_url: str = '') -> tuple[str, str, str]` | 455 | Kaydolup 24 saattir HİÇ soru sormamış hekime TEK SEFERLİK hatırlatma. |
| `tarih_yaz` | `(dt, lang: str = 'tr') -> str` | 572 | `timestamptz` → insan tarihi. **SAAT BASILMAZ, zaman dilimi Europe/Istanbul.** |
| `subscription_expiry_email` | `(*, tur: str, name: str, bitis, link: str, lang: str = 'tr', unsub_url: str = '') -> tuple[str, str, str]` | 590 | Abonelik dönemi bitiyor (`tur='uyari'`) / bitti (`tur='bitti'`) bildirimi. |
| `subscription_payment_failed_email` | `(*, name: str, link: str, lang: str = 'tr') -> tuple[str, str, str]` | 728 | Tekrarlayan çekim ALINAMADI — kartı güncelleme çağrısı (konu, html, text). |
| `subscription_canceled_email` | `(*, name: str, bitis, link: str, lang: str = 'tr') -> tuple[str, str, str]` | 767 | Hekim aboneliği İPTAL ETTİ — onay + erişimin bittiği tarih (konu, html, text). |

## `saglik/app/main.py`

`1346 satır` · `38 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Klivance — FastAPI SaaS uygulaması (Faz 1a).

Çalıştır:  .venv\Scripts\uvicorn saglik.app.main:app --port 8000
Rotalar: /giris /kayit /cikis · /chat /api/chat · /reference /api/query · /cases /api/cases · /account
Oturum-korumalı; her sohbet DB'ye kaydedilir, kota + kullanım doktora bağlı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_DOCS_ON` | `_os_boot.environ.get('KLIVANCE_DOCS') == '1'` | 137 |
| `_STATIC_DIR` | `_os.path.join(_os.path.dirname(__file__), 'static')` | 183 |
| `_PROFIL_MUAF` | `('/onay', '/cikis', '/logout', '/health', '/sitemap.xml', '/robots.txt', '/static', '/auth/google', '/yasal', '/kilavuz'` | 233 |
| `_CSS_JETON` | `re.compile('"(?:\\\\.\|[^"\\\\])*"\|\'(?:\\\\.\|[^\'\\\\])*\'\|url\\([^)\\"\']*\\)\|/\\*.*?\\*/', re.S)` | 597 |
| `_CSS_BLOK` | `re.compile('(<style[^>]*>)(.*?)(</style>)', re.S \| re.I)` | 603 |
| `_XFO_EXEMPT` | `('/iyzico/callback', '/odeme/sonuc')` | 643 |
| `_NOREF_YOLLAR` | `('/dogrula/', '/parola-sifirla/', '/mail-tercihi/', '/onay')` | 652 |
| `_BOOT_TS` | `_dt.datetime.now(_dt.timezone.utc)` | 695 |
| `_SURUM` | `_surum()` | 714 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_threadpool_hizala` | `() -> None` | 145 | ⚠ THREADPOOL ↔ DB HAVUZU ORANI (mimari denetimi 2026-07-29). |
| `_favicon_ico` | `()` | 195 |  |
| `_profil_muaf` | `(p: str) -> bool` | 260 | Yol muaf listesinde mi — SEGMENT SINIRIYLA. |
| `_copy_resp_headers` | `(src, dst) -> None` | 286 | Yanıt başlıklarını KOPYALA — çoklu `Set-Cookie`'yi KORUYARAK (M21d). |
| `set_lang` | `(code: str, request: Request)` | 302 |  |
| `_tok_hekim_sync` | `(tok: str)` | 334 | Oturum jetonu → hekim sözlüğü (ya da None) — `profil_kapisi_mw`nin havuzdaki DB ayağı. |
| `async` `profil_kapisi_mw` | `(request: Request, call_next)` | 341 | PROFİL TAMAMLAMA KAPISI — girişli ama profili eksik hekimi `/onay`'a yollar. |
| `async` `i18n_mw` | `(request: Request, call_next)` | 399 |  |
| `async` `_acq_mw` | `(request: Request, call_next)` | 429 | EDİNİM KAYNAĞINI SUNUCUDA YAKALA — JS'e bağlı OLMAYAN ilk-dokunuş kaydı (2026-07-29). |
| `async` `_notrack_mw` | `(request: Request, call_next)` | 479 | İÇ TRAFİK / ÖLÇÜM TEMİZLİĞİ (2026-07-27): `klv_notrack=1` çerezi taşıyan cihazlarda |
| `_notrack_form` | `(title: str, action: str, btn: str, note: str) -> HTMLResponse` | 520 |  |
| `notrack_on_page` | `()` | 530 |  |
| `notrack_on` | `()` | 536 | Bu cihazda analitik izlemeyi kapat (founder/QA cihazları — /admin açmayan telefon vb. için). |
| `notrack_off_page` | `()` | 548 |  |
| `notrack_off` | `()` | 554 |  |
| `async` `_noindex_staging_mw` | `(request: Request, call_next)` | 564 | Staging domaini (onrender.com) Google'a İNDEKSLENMESİN — tıp ürününde 'onrender.com' arama |
| `_css_yorum_soy` | `(html: str) -> str` | 606 | HTML'deki her `<style>` bloğundan CSS yorumlarını çıkar (dize içindekiler HARİÇ). |
| `async` `_css_yorum_mw` | `(request: Request, call_next)` | 615 |  |
| `async` `_security_headers_mw` | `(request: Request, call_next)` | 657 | Temel güvenlik yanıt başlıkları (M10). |
| `_surum` | `() -> str` | 698 | Kısa commit sha. Render Docker ortamında `RENDER_GIT_COMMIT` doludur; yerelde .git'ten |
| `health` | `()` | 718 | Render sağlık kontrolü (M29). Landing render etmek yerine UCUZ bir DB dokunuşu: |
| `_short_redirect` | `(src: str, med: str)` | 748 |  |
| `sl_instagram` | `()` | 753 |  |
| `sl_facebook` | `()` | 758 |  |
| `sl_linkedin` | `()` | 763 |  |
| `sl_telegram` | `()` | 768 |  |
| `sl_interactions` | `(request: Request)` | 788 | `/interactions` → 301 → `/etkilesim` (sorgu dizesi korunur). |
| `robots_txt` | `(request: Request)` | 795 |  |
| `sitemap_xml` | `(request: Request)` | 812 |  |
| `async` `_not_found_handler` | `(request: Request, exc)` | 837 | Çıplak {"detail":"Not Found"} yerine MARKALI dil-duyarlı 404 (API yolları hariç → JSON). |
| `_header_right` | `(d, lang)` | 856 |  |
| `_apply_try_pricing` | `(html: str, en: bool = False, request = None, doctor = None) -> str` | 886 | Landing fiyat yer tutucularını doldur (`fiyat.doldur` sarmalayıcısı). |
| `_landing_ctas` | `(html: str, d, active: bool, en: bool) -> str` | 903 | Landing CTA'larını auth-aware yap (server-side; header ZATEN _header_right ile işlendi). |
| `root` | `(request: Request)` | 957 |  |
| `guide_page` | `(request: Request)` | 1202 | Kullanım kılavuzu — herkese açık (kayıt öncesi de okunabilir). Dil rotada seçilir; |
| `chatgpt_landing` | `(request: Request)` | 1273 | `/chatgpt` → 302 → `/` (sorgu dizesi korunur). Gerekçe: yukarıdaki blok. |
| `security_page` | `(request: Request)` | 1280 | Güvenlik ve veri koruması — herkese açık güven sayfası (2026-07-19 ajan paneli P1). |
| `legal_page` | `(slug: str, request: Request)` | 1312 |  |

## `saglik/app/migrate.py`

`374 satır` · `17 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Şema migrasyonu — açılışta çalışır (idempotent).

db/schema.sql (bilgi tabanı yapısı) + db/app_schema.sql (SaaS tabloları) uygulanır.

⚠⚠ DEYİM-BAZLI SÜRÜM TAKİBİ (2026-07-29, denetim V2 — her deploy'da site donma riskiydi):
Eskiden her iki dosyanın TAMAMI (≈47 DDL) TEK transaction'da, HER AÇILIŞTA çalışıyordu.
`ALTER TABLE ... ADD COLUMN IF NOT EXISTS` hiçbir şey yapmasa BİLE ACCESS EXCLUSIVE kilidi
alır ve transaction sonuna kadar BIRAKMAZ. Deploy sırasında uçuştaki bir sorgu core.corpus'ta
ACCESS SHARE tutuyorsa migrate kuyruğa girer ve kuyruktaki ACCESS EXCLUSIVE ARKASINDAKİ tüm
YENİ okumaları da bloklar → site geneli donma.

Yeni davranış:
  1. SQL dosyaları DEYİMLERE ayrılır (dolar-tırnak `$$…$$`, string, yorum farkında ayırıcı).
  2. Her deyim KENDİ transaction'ında çalışır (kilit hemen bırakılır).
  3. Her deyimin özeti `app.schema_migration`'a yazılır → sonraki açılışlar deyimi ATLAR
     (kararlı durumda migrate SIFIR DDL çalıştırır, dolayısıyla sıfır kilit).
  4. `lock_timeout` (vars. 3 sn) → kilit alınamazsa migrate PATLAR, siteyi KİLİTLEMEZ.
  5. Şemadaki tek DML (trial kotası 10→15) da böylece TEK SEFERLİK olur (denetim V8).

NOT: Bu yalnız YAPIYI kurar. Bilgi tabanı VERİSİ (275K ilaç, 31K hastalık, StatPearls…)
ayrıca yüklenmelidir — canlıda yerel DB'nin pg_dump'ı prod Postgres'e restore edilir.
Kullanım:  python -m saglik.app.migrate
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `ROOT` | `Path(__file__).resolve().parent.parent.parent` | 31 |
| `FILES` | `[ROOT / 'db' / 'schema.sql', ROOT / 'db' / 'app_schema.sql']` | 32 |
| `LOCK_TIMEOUT` | `os.getenv('KLIVANCE_MIGRATE_LOCK_TIMEOUT', '3s')` | 35 |
| `LOCK_RETRIES` | `3` | 36 |
| `_DOLLAR` | `re.compile('\\$[A-Za-z_][A-Za-z0-9_]*\\$\|\\$\\$')` | 38 |
| `_LINE_COMMENT` | `re.compile('--[^\\n]*')` | 39 |
| `_BLOCK_COMMENT` | `re.compile('/\\*.*?\\*/', re.S)` | 40 |
| `_TRACK_DDL` | `['CREATE SCHEMA IF NOT EXISTS app', 'CREATE TABLE IF NOT EXISTS app.schema_migration (\n           file       text NOT N` | 101 |
| `REINDEX_TIMEOUT` | `os.getenv('KLIVANCE_REINDEX_TIMEOUT', '600s')` | 165 |
| `_SET_STATS` | `re.compile('ALTER\\s+TABLE\\s+([\\w.\\"]+)\\s+ALTER\\s+COLUMN\\s+\\S+\\s+SET\\s+STATISTICS', re.I)` | 227 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_meaningful` | `(stmt: str) -> bool` | 43 | Yalnız yorum/boşluktan ibaret parçaları ele. |
| `_split_statements` | `(sql: str) -> list[str]` | 49 | SQL'i ';' üzerinden deyimlere ayır — ama yalnız ÜST DÜZEYDEKİ ';' üzerinden. |
| `_stmt_key` | `(stmt: str) -> str` | 95 | Deyimin kimliği: boşluk-normalize edilmiş metnin SHA-256'sı. Girinti/format değişikliği |
| `_ensure_tracking` | `(conn) -> None` | 114 |  |
| `_applied_hashes` | `(conn, fname: str) -> set[str]` | 120 |  |
| `_run_stmt` | `(conn, fname: str, idx: int, stmt: str, key: str) -> None` | 125 | Tek deyimi KENDİ transaction'ında + lock_timeout ile uygula ve kaydet. |
| `_brands_def` | `(conn) -> str \| None` | 150 | core.brands_text fonksiyonunun mevcut tanımı (yoksa None). |
| `_index_exists` | `(conn, schema: str, name: str) -> bool` | 159 |  |
| `_guard_brands_index` | `(conn, before: str \| None, after: str \| None) -> None` | 168 | ⚠ HASTA GÜVENLİĞİ FRENİ (denetim V2 eki): `core.brands_text` fonksiyonu |
| `_quote_interval` | `(v: str) -> str` | 204 | SET için güvenli literal — yalnız `<sayı><birim>` kabul edilir (SQL enjeksiyon freni). |
| `_apply_file` | `(conn, path: Path) -> list[str]` | 210 | Bir SQL dosyasını deyim deyim uygula. Döner: BU KOŞUDA uygulanan deyimlerin listesi. |
| `_analyze` | `(conn, applied: list[str]) -> None` | 231 | İSTATİSTİK BAKIMI (denetim V5): ANALYZE hiç çalışmamıştı — planlayıcı core.corpus.tsv |
| `_promote_admins` | `(conn) -> None` | 263 | KLIVANCE_ADMIN_EMAILS (virgülle ayrık) hesaplarını admin yap (idempotent). |
| `_purge_retention` | `() -> None` | 280 | KVKK backstop: grace (30 gün) dolan silme-talepli hesapları + eski soft-delete dosya |
| `_backfill_trial_used` | `() -> None` | 307 | MEVCUT hesapların e-postalarını 'deneme kullanıldı' diye işaretle (tek seferlik, |
| `_backfill_last_login` | `() -> None` | 323 | AYAKTAKİ oturumların `last_seen`'ini kalıcı `app.doctor.last_login_at`'e taşı. |
| `run` | `() -> None` | 351 |  |

## `saglik/app/olcum_js.py`

`51 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Ray tıklama ölçümü — SAF JS sabiti (2026-08-20, founder "ölçmek isterim").

⚠⚠ NEDEN AYRI MODÜL: kod `webutil._KLV_TRACK_JS`in yanına aitti ama `webutil.py`
   `dosya_boyut_verify` tavanında (987) ve o girdinin KENDİ kuralı bağlayıcı:
   "dokuzuncu artış YOK — önce `pazar` bölmesi". Aynı gün `tr_saat.py` de tam bu
   sebeple ayrılmıştı; desen budur. Buraya YALNIZ saf sabit konur (f-string yok,
   import yok, modül durumu yok).

⚠ `_ANALYTICS_TAG`in İÇİNDE toplanır (webutil.py) — bu ZORUNLU: `klv_notrack`
  middleware'i o tek sabiti `.replace(..., "", 1)` ile söker; dışarıda kalan bir
  script founder/QA gezintilerini ölçüme sızdırırdı.

NEDEN VAR (ölçülmüş körlük): takvime yatırım kararı ("hızlı yeniden-randevu çipleri")
kullanım sayısına bağlıydı. GA4'te ölçülen tek şey `/takvim` page_view'du — 28 günde
DIŞ trafikten 4 kişi, kişi başı 2,25 sn — ve bu sayı iki hipotezi AYIRAMIYOR:
  (a) hekim rayda "Takvim"i GÖRÜYOR ama tıklamıyor  → ürün/vaat sorunu
  (b) rayı görmüyor / öge ulaşılmaz yerde           → YERLEŞİM sorunu
Ayrım kararı değiştirir. Ölçülen yerleşim (b)'yi ciddi aday yapıyor: Takvim masaüstü
rayında 7. sırada, MOBİL ÇUBUKTA HİÇ YOK (yalnız "Daha fazla" menüsünde, 2 dokunuş).

TASARIM KARARLARI (her biri bir hatayı önler):
  · **Hüküm OLAY ADINDA** (`rail_takvim`), parametrede değil: GA4 olay parametresi
    custom dimension kaydı olmadan raporlanamaz; olay adı `eventName` boyutuyla
    Data API'den doğrudan okunur. Parametre yine de gider (dimension kurulursa hazır).
  · **YALNIZ `pathname`** — sorgu dizesi ASLA. `/reference?q=<ilaç>` hassas sorgudur
    (`_ANALYTICS_PATH_ONLY` kuralının aynısı, burada ikinci katman).
  · **İLK SEGMENT**: `/cases/12` → `rail_cases`. Yoksa her hasta id'si ayrı olay adı
    üretir ve GA4'ün property başına 500 farklı olay sınırı yenir.
  · **Meta'ya GİTMEZ**: `klvTrack`in 3. argümanı (fbq olay adı) VERİLMEZ. Optimizasyon
    sinyali DAR kalmalı (CLAUDE.md `klv_kayit` dersi: Meta kimi arayacağını dar
    sinyalden öğrenir; ray gezinmesi satın-alma niyeti DEĞİLDİR).
  · **`.rail-item[href]`**: çıkış `<button>`u kapsam dışı (ölçülecek bir şey yok) ve
    `closest` ile delegasyon → `nav()`e HİÇ DOKUNULMAZ, yeni ray ögesi kendiliğinden
    kapsanır (etiket/hedef tek yerde kalır).
  · `try/catch` + `if(!a)return`: soyulmuş ya da rayşız sayfada sessizce hiçbir şey yapmaz.

Kapı: `scratchpad/ray_olcum_verify.py` (CI SUITES). ⚠ GA4'e veri DÜŞTÜĞÜNÜ ölçmez —
"olay bağlandı mı"yı ölçer. ⚠ GA4 geçmişi geri getirmez: veri bu satır canlıya
çıktığı ANDAN itibaren birikir.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `RAIL_TRACK_JS` | `"<script>document.addEventListener('click',function(e){var a=e.target&&e.target.closest?e.target.closest('.rail-item[hre` | 43 |

## `saglik/app/on_okuma.py`

`688 satır` · `27 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
GÖRÜNTÜ ÖN-OKUMA (#620b YOL 2, founder 2026-09-13) — ROTA MODÜLÜ + KART HTML ÜRETİCİLERİ (WP-6, Kamil).

Sözleşme: `docs/goruntuleme-sozlesme-2026-09-13.md` §2.4 (imzalar/dallar/sıra ORADAN) · §1.4 (kokpit
`T.onokuma_iz`) · §1.7 (metinler TEK SÖZLÜK `cds.isaretler.METIN`; burada `_M` sözlüğü YOK — C.Q10).
Desen `deger_onay.py`/`rad_case.py`: `KAYIT` + `@_rota` + `kur(app)`; `include_router` YASAK (rota envanteri
okuyan ağlar körelir). Import TEK YÖNLÜ: `analyze`/`pipeline`/`on_okuma_store`/`on_okuma_kayit`/`isaretler`
buradan çekilir; `cases_routes` BENİ import eder (`kart_yuku`), `chat_routes` beni import ETMEZ.
⚠ `_sniff_mime` `cases_routes`ta (MIME literali TEK kaynak) → burada FONKSİYON İÇİ tembel import
  (modül düzeyinde dairesel olur: cases_routes → on_okuma → cases_routes).

DÖRT UÇ (hepsi POST; env `KLIVANCE_GORUNTU_ONOKUMA` YOKKEN ilk ikisi 503, düğmeler HTML'de basılmaz — C.Q12):
  1. `/api/goruntu-onoku`                       sohbet kolu (JSON) — deterministik ön-kapılar (503/400/413,
     `record_usage` YOK) → `iz_birak` → üçlü zincir → rezerv → PDF map → tespit (`zorla=0`) → `goruntu_on_okuma`
     → `add_message`×2 (`mode=goruntu`, `notice`) → `record_usage` → rezerv kaldır. `unavailable` → HTTP 200 +
     sabit amber cümle (`onokuma_calistirilamadi`), asistan satırı YİNE yazılır (yanan para kaydedilir).
  2. `/api/cases/{cid}/files/{fid}/on-oku`      kart kolu — **ÖNCE `calistirilamadi` yazılır**, başarıda `pending`
     (worker timeout'ta satır dürüst kalır). Ziyarete YAZMAZ, `extracted_text`e YAZMAZ (C.Q1).
  ⚠ #779c: 1 ve 2 ön-kapılardan SONRA `chat_routes._heartbeat_stream` NDJSON döner (10 sn `ping`, sonda `done`/`error`; Cloudflare ~100 sn kesmesi) — iş ayrı thread'de, istemci koparsa DA biter (`record_usage`/`rezerv_kaldir` işin `finally`sinde).
  3. `/api/cases/{cid}/on-okuma-onayla`         hekim kararı (Form) — LLM 0, `record_usage` 0. `onaylandi` +
     (boş ∨ makine metnine eşit ∨ `ONOKUMA_ONEK`li metin) → 400 `onokuma_400_metin`, yazma YOK
     (`on_okuma_kayit.onay_reddi`). Onayda `add_visit(ONOKUMA_ONAY_ONEK + redact_pii(metin, names=True))`.
  4. `/api/cases/{cid}/files/{fid}/belge-oku`   AÇIK istek (kağıt rapor fotoğrafı; hekim sınıflandırdı — A.10/C.Q11):
     `analyze._map_one` lab yolu (içinde `rad_normalize`); işaret dönerse `extracted_text` YAZILMAZ + notice.
  ⚠ BEŞİNCİ UÇ BU DOSYADA DEĞİL: `/api/cases/{cid}/goruntu-kaydet` → **`on_okuma_kaydet.py`** (#781c, tavan
    bölmesi 15.09). Oranın `kur(app)`u `main.py`de AYRI çağrılır; buraya yeni uç eklerken tavanı önce ölç.

DEĞİŞMEZLER: tanı koymaz · yönlendirme yasağı · FAIL-OPEN YASAK (istisna = amber/`calistirilamadi`, asla yeşil/
sessiz) · yanıt dili SORUDAN (`dil.yanit_dili`), UI metni `lang` · kabul MIME kümesi `KABUL_MIME` (C.Q8:
JPEG/PNG/GIF/WebP; TIFF/JP2/HEIC/DICOM AÇIK mesajla red — yükleme kümesi DEĞİŞMEZ, yalnız bu uç reddeder) ·
ön-okuma metni `esc` ile basılır (XSS) · JS'te TR literal YOK (metinler sunucudan) · EN düz `'` YOK.
⚠ Sözleşme imzalarına EK kwarg'lar (raporda beyan): `satir_dugmeleri(f, lang, cid=, pdfler=)` ve `serit(...,
  cid=)` — form/fetch URL'i `cid` ister, satır demeti (`list_case_files`) `case_id` taşımaz.
Kapılar: `goruntu_onokuma_verify` (KP: P4/P9/P10/P11/P12-b/P13/P20-c/P21-c/P22) · ölçüm `scratchpad/_kamil_wp6_olc.py`.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 57 |
| `KABUL_MIME` | `frozenset({'image/jpeg', 'image/png', 'image/gif', 'image/webp'})` | 61 |
| `KILITLI_DURUMLAR` | `frozenset({'pending', 'onaylandi'})` | 63 |
| `YUZEY` | `'goruntu_onoku'` | 64 |
| `_MOD` | `'goruntu'` | 65 |
| `_DURUM_KEY` | `{'pending': 'onokuma_durum_pending', 'onaylandi': 'onokuma_durum_onaylandi', 'reddedildi': 'onokuma_durum_reddedildi', '` | 80 |
| `_AMBER` | `'background:var(--amber-soft);color:var(--amber-ink)'` | 82 |
| `_GRI` | `'background:var(--mist);color:var(--slate)'` | 83 |
| `_KADEME_STIL` | `{'kirmizi': 'background:var(--alert-soft);color:var(--alert-ink)', 'amber': _AMBER, 'notr': _AMBER}` | 92 |
| `_KART_JS` | `'<script>(function(){\nwindow.onokuPost=function(url,sel,btn){var b={};if(sel){var s=document.getElementById(sel);if(s&&` | 258 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_taban_ver` | `(calls) -> list` | 68 | Pipeline istisna/kesik dalının `{"bilinmiyor": True}` usage'ını REZERV üst sınırına bağlar (#776c P4). |
| `_rota` | `(yontem: str, yol: str, **kw)` | 96 |  |
| `kur` | `(app) -> None` | 103 |  |
| `_L` | `(lang) -> str` | 108 |  |
| `_M` | `(lang) -> dict` | 112 |  |
| `_tarih` | `(t = None) -> str` | 116 | gg.aa.yyyy — TR duvar-saati (`tr_saat`); `t` verilirse o (timestamptz → yerel gün). |
| `dicom_mu` | `(head: bytes) -> bool` | 121 | DICOM Part-10: 128 bayt önsöz + `DICM`. Yalnız AÇIK red için tanınır (dönüşüm v2). |
| `_mime_red_yuk` | `(mime, lang) -> dict` | 126 |  |
| `_mime_red` | `(mime, lang) -> JSONResponse` | 130 |  |
| `_kapali` | `(lang) -> JSONResponse` | 134 |  |
| `kademe_etiketi` | `(on_okuma, lang = 'tr') -> str` | 139 | #782c — Belgeler satırındaki kritik kademe rozeti; ön-okuma METNİNDEN ayrıştırılır. |
| `_durum_etiketi` | `(durum, onay_at, lang, on_okuma = None) -> str` | 159 | `.fr-tag data-durum` — 5 durum: NULL → '' · pending/onaylandi/calistirilamadi AMBER · reddedildi GRİ. |
| `satir_dugmeleri` | `(f, lang, cid: int \| None = None, pdfler = ()) -> str` | 172 | Belgeler satırı: YALNIZ `image/*` VE analiz edilmemiş satırda `Ön-oku` + `Belge olarak oku`. |
| `serit` | `(fid: int, on_okuma, durum, onay_at, lang, cid: int \| None = None) -> str` | 209 | Satır altı `<details>`: ön-okuma metni SALT-OKUNUR (`esc`, `white-space:pre-wrap`; `max-height/overflow` |
| `kart_yuku` | `(conn, doctor_id: int, cid: int, files, lang) -> dict` | 285 | `cases_routes.case_detail_page` — `with connect()` İÇİNDE TEK çağrı → `{fid: (satir_ici, satir_alti)}`. |
| `_b64_bayt` | `(data: str) -> int` | 318 |  |
| `_b64_bas` | `(data: str, n: int = 200) -> bytes` | 323 | İlk n (4'ün katı) b64 karakteri → ham baş (DICM@128 için ≥132 bayt: 200 krk → 150 bayt). |
| `_ek_normalize` | `(raw_att) -> list[dict]` | 331 |  |
| `async` `api_goruntu_onoku` 🌐 | `(request: Request)` | 342 | Sohbet 'Görüntü ön-okuma' modu (WP-4 istemcisi). Ağır iş havuzda (M5). |
| `_goruntu_onoku_sync` | `(request, body, lang)` | 349 |  |
| `async` `_govde` | `(request: Request) -> dict` | 492 | JSON (fetch) ya da form gövdesi — ikisi de kabul (JS kapalıyken düz form). |
| `_dosya_sahiplik` | `(conn, request, cid: int, fid: int, lang)` | 503 | → (d, row) ya da (Response, None). Sahiplik `get_case_file` doctor_id WHERE + `row[1]==cid`. |
| `async` `api_on_oku` 🌐 | `(cid: int, fid: int, request: Request)` | 515 |  |
| `_on_oku_sync` | `(cid, fid, request, body, lang)` | 521 |  |
| `api_on_okuma_onayla` 🌐 | `(cid: int, request: Request, fid: str = Form(''), karar: str = Form(''), metin: str = Form(''))` | 607 | LLM 0 · `record_usage` 0. Beyaz liste dışı `karar` → 303 (yazma yok). `onaylandi` + `onay_reddi` → 400. |
| `async` `api_belge_oku` 🌐 | `(cid: int, fid: int, request: Request)` | 643 |  |
| `_belge_oku_sync` | `(cid, fid, request, lang)` | 648 | Kağıt rapor FOTOĞRAFI → hekim 'belge' dedi → `analyze._map_one` lab yolu (`rad_normalize` içinde). |

## `saglik/app/on_okuma_kaydet.py`

`193 satır` · `6 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
SOHBET GÖRSELİNİ HASTA KARTINA KAYDET (#781c, founder 2026-09-15) — TEK UÇ (Kamil).

⚠⚠ NEDEN AYRI MODÜL: `on_okuma.py` tavanını (`scratchpad/dosya_boyut_verify.py`) aştı ve Ömer
tavanı YÜKSELTMEYİP BÖLME dedi (15.09). Blok BİREBİR taşındı — davranış değişmedi; kapı
`goruntu_kaydet_verify` taşımadan önce de sonra da AYNI iddiaları koşturur.
⚠ Rota deseni `on_okuma` ile AYNI: kendi `KAYIT` listesi + `@_rota` + `kur(app)`.
  **`include_router` YASAK** (rota envanterini `app.routes`tan okuyan iki ağ sessizce körelir);
  `main.py` bu modülün `kur`unu AYRICA çağırır — eklemeyi unutmak rotayı sessizce yok eder.
⚠ `on_okuma`dan çekilenler (kopyalanmaz — ikinci doğru-kaynak açmak bu deponun en pahalı hatası):
  `KABUL_MIME` · `_M`/`_L` · `_mime_red` · `_ek_normalize` · `_MOD` · `YUZEY`.
⚠ `cases_routes` FONKSİYON İÇİ tembel import edilir (modül düzeyinde dairesel olurdu:
  cases_routes → on_okuma → …); MIME literali orada TEK kaynak.

SÖZLEŞME — `POST /api/cases/{cid}/goruntu-kaydet`
  gövde `{attachment:[{media_type,data,filename}], message_id, thread_id}`
  200  `{ok:true, fid:[...], durum:"pending"|"calistirilamadi", yeni, zaten}`
  401 oturum · 404 kart başkasının/yok · 400 ek yok / MIME / mesaj çözülemedi · 413 adet/boyut/kart
  tavanı · 429 hız · 503 `KLIVANCE_FILE_KEY` yok.
DEĞİŞMEZLER: LLM 0 · `record_usage` 0 · ziyaret notuna YAZMAZ · `extracted_text` NULL KALIR
(C.Q1) · MIME BAYTTAN (`_sniff_mime`) · Fernet · ön-okuma metni DB'den (istemciden DEĞİL).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 39 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(yontem: str, yol: str, **kw)` | 42 |  |
| `kur` | `(app) -> None` | 49 |  |
| `_m` | `(lang, key: str, yedek: str = 'onokuma_calistirilamadi') -> str` | 54 | Metin sözlüğünde anahtar YOKSA yedeğe düş — FAIL-CLOSED: #781c metinlerini `isaretler`e |
| `_mesaj_onokumasi` | `(conn, doctor_id: int, mid: int)` | 62 | `app.message` satırından ön-okuma metni — ⚠⚠ METİN İSTEMCİDEN ALINMAZ. |
| `async` `api_goruntu_kaydet` 🌐 | `(cid: int, request: Request)` | 84 | Sohbette ön-okunan görselleri hasta kartına ŞİFRELİ kaydet + ön-okumayı `pending` iliştir. |
| `_goruntu_kaydet_sync` | `(cid, request, body, lang)` | 91 | LLM 0 · `record_usage` 0 · ziyaret notuna YAZMAZ · `extracted_text`e YAZMAZ (C.Q1 aynen). |

## `saglik/app/on_okuma_kayit.py`

`321 satır` · `20 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Görüntü ön-okuması (#620b YOL 2, WP-3) — SOHBET/KAYIT tarafındaki SAF yardımcılar.

SAF MODÜL: DB yok, LLM yok, HTML yok, FastAPI yok. `chat_routes` (ve WP-6 `on_okuma`) buradan
TEK SATIRLIK çağrılar yapar; mantık burada ki `chat_routes` tavanı (dosya_boyut_verify) şişmesin.
Sözleşme: `docs/goruntuleme-sozlesme-2026-09-13.md` §1.6 (iki önek ailesi) · §2.3 "Kalıcılık /
geri-sızma" · §2.4 K14 · §4b WP-3. Önek/metin sabitleri YALNIZ `cds/isaretler`ten okunur
(kararlar C.Q10: kendi `_M` sözlüğü AÇILMAZ).

NEDEN VAR (ölçülmüş sınıf, "yanlış güven"): makine ön-okuması doğrulanmamış bulgu ADAYIDIR.
Dört geri-sızma yolu vardı — her birini buradaki bir fonksiyon kapatır:
  1. Sohbet geçmişi: `mode=goruntu` asistan turu sonraki turda olgu gibi okunurdu →
     `gecmis_cerceve` (`ONOKUMA_GECMIS_ONEK` çerçevesi; `clinical` dokunulmaz, idempotent).
  2. Hasta kaydı: hekimin ONAYLADIĞI gözlem `case_visit`e `ONOKUMA_ONAY_ONEK` ile girer ve
     "Ziyaret notları" arasında olgu gibi dururdu → `ziyaret_ayir` ayrı alt başlığa taşır
     (`onayli_bolum`). ⚠ YOL 1'in `RAD_UYARI_ONEK`li `[Dosya analizi …` çekirdeği BİLEREK
     "Ziyaret notları"nda KALIR (kararlar A.12: önek satırın içinde, model onu okur).
  3. İkincil tüketiciler (`/api/whatmissed` · `/api/simplify` · `/api/cases/{cid}/visit`):
     ön-okuma metni çerçevesiz "yanıt" gibi tüketilirdi → `onokuma_metni_mi` → 400, ücret 0.
     YALNIZ `ONOKUMA_ONEK`e bakar: `RAD_UYARI_ONEK`li YOL 1 metni GEÇER (istenen davranış).
  4. Hekim onayı: makine metnini aynen/önekli "onaylayıp" ziyarete kopyalamak → `onay_reddi`
     (boş · önekli · whitespace-normalize eşit · önek soyulmuş eşit → 400, yazma yok).
Ayrıca: `gorsel_bekleyen_notu` (görsel dosya "analiz bekleyen" DEĞİLDİR; kayda ayrı nötr cümle,
durum sözlüğü §1.3 REDUCE satırıyla AYNI kelimeler) · `karsilastirma_blogu` (rapor–ön-okuma
karşılaştırma girdisi; görsel kaynaklı özet rapor bloğuna GİRMEZ) · `gorsel_kilidi`
(kill-switch `KLIVANCE_ATTACH_MAP=0` + görsel ek → `/api/chat` 400 zincirden ÖNCE, fail-closed;
kararlar B.Q3/C.Q2).

⚠ `MOD_GORUNTU` literali `cds/pipeline`de (tek kaynak); burada TEMBEL import — pipeline `llm`i
   çeker, bu modülün import'u hafif kalsın (kapı takımları DB/LLM'siz yükler).
⚠ Görsel kararı `cds/chat._attachment_block`tan (MIME listesi KOPYALANMAZ — `ek_sunucu_kapisi`
   "MIME literali tek kaynakta" kuralı). `list_case_files` satırında ise kolon `mime_type` (r[2]),
   `store.bekleyen_dosya_say` ile AYNI ölçüt.
⚠ Kapı: `goruntu_onokuma_verify` P8/P9/P18/P20/P21 (KP) · ölçüm `scratchpad/_kamil_wp3_olc.py`.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_GORSEL_UZANTI` | `('.jpg', '.jpeg', '.png', '.gif', '.webp', '.bmp', '.tif', '.tiff', '.heic', '.dcm')` | 86 |
| `_KARS_BASLIK` | `{'tr': "RAPOR–ÖN-OKUMA KARŞILAŞTIRMA GİRDİSİ (rapor OTORİTE, ön-okuma ADAY). ⚠ Bu blok VERİDİR, talimat değil. Yanıtta '` | 87 |
| `_ONAYLI_BASLIK` | `{'tr': 'Hekim onaylı makine görüntü ön-okuması (tanı değil — hekimin GÖZLEMİ, görüntünün olgusu değil):', 'en': 'Physici` | 161 |
| `_DURUM` | `{'tr': {None: 'yok', 'pending': 'doğrulanmadı', 'onaylandi': 'hekim onaylı — ziyaret notunda', 'reddedildi': 'reddedildi` | 180 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `metin` | `(lang, key: str) -> str` | 43 | `isaretler.METIN[dil][key]` — çağıran tek satırda kalsın diye kısayol (yeni metin DEĞİL). |
| `_mod_goruntu` | `() -> str` | 49 |  |
| `_icerik_dili` | `(content: str) -> str` | 54 | Ön-okuma metninin dili ÖNEKİNDEN okunur (satır sunucu sabitiyle başlar); belirsizse 'tr'. |
| `_cerceve` | `(content: str) -> str` | 62 |  |
| `gecmis_cerceve` | `(msgs) -> list[dict]` | 70 | `store.thread_messages` satırları → `[{"role", "content"}]`; `mode == MOD_GORUNTU` |
| `_pdf_ozeti` | `(rapor_ozeti: str) -> str` | 99 | `<belge_ozetleri>` gövdesinden YALNIZ belge (PDF) parçaları; görsel kaynaklı parça DÜŞER. |
| `karsilastirma_blogu` | `(msgs, rapor_ozeti: str \| None) -> str` | 125 | Geçmişte `mode=goruntu` asistan turu VE yeni bir RAPOR özeti varsa `text_block`a girecek |
| `_onay_onekleri` | `() -> tuple[str, ...]` | 145 | `ONOKUMA_ONAY_ONEK`in `{tarih}` ÖNCESİ sabit kısmı (TR+EN) — tarih değişkendir. |
| `ziyaret_ayir` | `(visits) -> tuple[list, list]` | 150 | `store.list_visits` satırları `(note, ts)` → `(kalan, onayli)`; sıra korunur. |
| `onayli_bolum` | `(onayli, lang, kirp) -> list[str]` | 167 | `_build_case_record` `parts`ine eklenecek alt başlık bölümü (liste: boşsa `[]` → `parts +=`). |
| `gorsel_bekleyen_notu` | `(rows, lang) -> str` | 188 | `list_case_files` satırlarındaki görsel dosyalar (r[2] `image/*`) için TEK nötr cümle |
| `onokuma_izleri` | `(lang) -> list[str]` | 202 | Kokpit 6. karo amberinin ARAYACAĞI izler — ⚠⚠ ELLE YAZILMAZ, ÜRETİCİDEN TÜRETİLİR. |
| `_no_etiket` | `(mdil) -> str` | 241 |  |
| `gor_adi` | `(i: int, ad: str, mdil: str, coklu: bool) -> str` | 245 | Elenen görselin hekime gösterilen adı — çoklu turda BAŞINA MODELİN NUMARASI eklenir. |
| `numara_eslesme` | `(nolar, adlar, lang, mdil) -> str` | 257 | "Görüntü k → dosya" eşlemesi (`METIN.onokuma_numara_eslesme`); liste boşsa "". |
| `onokuma_metni_mi` | `(text) -> bool` | 282 | Metin `ONOKUMA_ONEK` (TR/EN) ile mi başlıyor? YALNIZ bu öneke bakar (`RAD_UYARI_ONEK` geçer). |
| `_norm` | `(s) -> str` | 287 |  |
| `onay_reddi` | `(metin_, on_okuma) -> bool` | 295 | Hekim onayı 400 kuralı (kararlar C.Q1): metin BOŞ · `ONOKUMA_ONEK` ile başlıyor · |
| `attach_map_kapali` | `() -> bool` | 306 | `KLIVANCE_ATTACH_MAP=0` — `cds/chat._map_attachments` ile AYNI okuma (ikiz; her çağrıda env). |
| `gorsel_kilidi` | `(attachment, lang) -> str \| None` | 311 | Kill-switch AÇIKKEN (map kapalı) görsel ek varsa 400 metni (`ek_verilmedi`, ad = MIME ailesi — |

## `saglik/app/on_okuma_store.py`

`73 satır` · `4 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Görüntü ön-okuması — `app.case_file.on_okuma*` kolonlarının TEK yazıcı/okuyucusu (#620b YOL 2, Selim WP-5).

Sözleşme: `docs/goruntuleme-sozlesme-2026-09-13.md` §2.3. Kolonlar `db/app_schema.sql` (idempotent ALTER).

⚠⚠ NEDEN AYRI MODÜL (store.py'ye DEĞİL): `store.py` tavanlı (bölme şartı) ve `on_okuma` kolonunu
  OKUYAN fonksiyonlar AST beyaz listesiyle çivilenir — {`on_okuma.py`, `on_okuma_store.py`,
  `store.soft_delete_case_file`}. Ön-okuma `extracted_text`e YAZILMAZ (görselde her koşulda NULL):
  `list_case_lab_summaries` / `_build_case_record` / analiz REDUCE ön-okumayı YAPISAL olarak göremez
  → makine ön-okuması hasta kaydına OLGU olarak sızmaz (kararlar C.Q1). Bu modülün dışına
  `on_okuma` kolonu okuyan bir SELECT yazmak o kapıyı KIRMIZI yapar.

Durum makinesi (beyaz liste `DURUMLAR`, dışı `ValueError` — sessiz düşme YOK):
  NULL ──set_on_okuma(None, "unavailable:…", "calistirilamadi")──▶ calistirilamadi   (ÖNCE yazılır: fail-closed;
        ──set_on_okuma(text, model, "pending")──────────────────▶ pending             LLM patlarsa satır bu hâlde KALIR)
  pending ──on_okuma_durum_yaz("onaylandi", now())──▶ onaylandi   (hekim metni `store.add_visit`e AYRI yazılır;
          ──on_okuma_durum_yaz("reddedildi", now())──▶ reddedildi   makine metni ziyarete KOPYALANMAZ — `on_okuma.py`)
· `pending` metinsiz OLAMAZ, `calistirilamadi` metinli OLAMAZ (invariant burada, çağıranda değil).
· Yeni üretim (`set_on_okuma`) önceki hekim kararını (`on_okuma_onay_at`) SIFIRLAR: onay eski metne aitti.
· `doctor_id` HER UPDATE/SELECT'in WHERE'inde (IDOR) + `deleted_at IS NULL` (tombstone'a yazılmaz).
· SAVEPOINT (`with conn.transaction()`) + `commit` SAVEPOINT'ten SONRA (psycopg3: SAVEPOINT içinde commit YASAK,
  yazma sessizce düşer — CLAUDE.md). ⚠ Geri okuma AYRI bağlantıdan: `scratchpad/on_okuma_kalicilik_sonda.py`.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `DURUMLAR` | `frozenset({'pending', 'onaylandi', 'reddedildi', 'calistirilamadi'})` | 25 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_durum_kontrol` | `(durum: str) -> str` | 28 |  |
| `set_on_okuma` | `(conn, doctor_id: int, fid: int, text: str \| None, model: str, durum: str) -> bool` | 35 | Ön-okuma ÜRETİMİNİ yaz (metin + model + durum + `on_okuma_at=now()`); hekim kararı sıfırlanır. |
| `on_okuma_durum_yaz` | `(conn, doctor_id: int, fid: int, durum: str, onay_at = None) -> bool` | 53 | Hekim KARARINI yaz (onaylandi/reddedildi + `on_okuma_onay_at`). Metne DOKUNMAZ. |
| `get_on_okuma` | `(conn, doctor_id: int, fid: int)` | 66 | Döner: (fid, on_okuma, on_okuma_model, on_okuma_durum, on_okuma_at, on_okuma_onay_at) ya da None |

## `saglik/app/ornek_sorular.py`

`173 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
`/chat` karşılama ekranındaki ÖRNEK SORULAR — branşa göre. SAF VERİ.

⚠⚠ BU DOSYA HİÇBİR ŞEY IMPORT ETMEZ ve etmemelidir (`form_listeler.py` ile aynı gerekçe):
saf referans verisidir ve büyümeye devam edecek; `webutil`/`chat_body` içinde büyürse onları
okuyan herkesi ve dosya-tavanı kapısını (`dosya_boyut_verify`) vurur. Buraya `def` eklemek
uyarı işaretidir — `ornekler()` dışında davranış KOYMA.

NEDEN VAR (ölçüldü 2026-08-14, prod salt-okuma):
  Kayıt olan 48 hekimin 25'i HİÇ soru sormadı. Bunların 16'sının profili tamdı, yani
  `/chat`e ULAŞTI ve karşılama ekranını GÖRDÜ — yine de sormadı. Soru soran 23 kişinin
  21'i ise ilk sorusunu kayıttan sonraki 10 DAKİKA içinde sordu. Dağılım İKİLİ: ya hemen
  soruyor ya hiç. Arada "birkaç gün sonra denedim" diyen tek kişi yok.
  ⚠ Karşılama ekranında ZATEN üç örnek kart vardı — ama ÜÇÜ DE GENELDİ (varfarin-amiodaron,
  eGFR-metformin, TSH). Bir acil hekimi ya da kadın-doğum uzmanı için o üç kart "bu araç
  benim işim için değil" izlenimi veriyordu. Bu dosya kartları branşa BAĞLAR.

⚠ KAPSAM SINIRI — DÜRÜSTLÜK: bu değişiklik 25 kişinin 16'sına hitap eder. Kalan 9'u
  `/onay` profil formunu hiç doldurmamış Google-kolu kayıtlarıdır; onlar `/chat`i HİÇ
  görmez ve bu dosya onlara SIFIR etki eder (ayrı iş → `_gorev.txt` #260b).

⚠ BRANŞ DEĞERLERİ KANONİK DEĞİL: `app.doctor.specialty` alanında `form_listeler._BRANSLAR`
  dışında ESKİ değerler de var (prod'da ölçüldü: "Dahiliye", "Yoğun Bakım", "Öğrenci",
  "Diğer" ve 13 BOŞ). Bu yüzden eşleme `_ANAHTAR`la normalize edilir ve eşleşmeyen her
  değer `GENEL`e düşer — KeyError ile karşılama ekranını KIRMAZ.

⚠ i18n: metinler burada TR+EN olarak DURUR ve sunucu `lang`e göre SEÇER (CLAUDE.md:
  "dinamik metin için sunucuda `lang` ile üret" — `EN_PAIRS`e çift EKLEME). EN metinlerde
  düz kesme (') KULLANMA: `i18n_mw` apostrof tuzağı JS string'ini kapatıp sayfanın TÜM
  JS'ini çökertiyor (kıvrık kesme U+2019 kullan ya da apostrofsuz yaz).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `GENEL` | `{'tr': [('İlaç etkileşimi', 'Varfarin ile amiodaron etkileşimi?'), ('Doz ayarı', '65y, eGFR 40 — metformin başlayayım mı` | 36 |
| `_SETLER` | `{'acil': {'tr': [('Ayırıcı tanı', '42y erkek, ani başlayan göğüs ağrısı, EKG normal — ayırıcı tanı?'), ('Acil doz', 'Sta` | 45 |
| `_ANAHTAR` | `{'acil tip': 'acil', 'cocuk acil': 'acil', 'aile hekimligi': 'aile', 'pratisyen hekim': 'aile', 'halk sagligi': 'aile', ` | 132 |
| `_TR_HARF` | `str.maketrans({'ı': 'i', 'İ': 'i', 'ş': 's', 'Ş': 's', 'ğ': 'g', 'Ğ': 'g', 'ü': 'u', 'Ü': 'u', 'ö': 'o', 'Ö': 'o', 'ç': ` | 152 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_norm` | `(s: str) -> str` | 156 | Branş adını eşleme anahtarına çevirir (Türkçe-güvenli küçültme). |
| `ornekler` | `(specialty: str, lang: str = 'tr')` | 164 | Branşa uygun 3 örnek soruyu döndürür — eşleşme yoksa GENEL. |

## `saglik/app/panel_body.py`

`684 satır` · `13 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
`/panel` — yargım deseninin uyarlaması, EK sayfa (2026-08-02, founder yönü).

Founder: "avukat işini örnek alarak bizim sistemi onun yapısına, görselliği ona
benzeteceğiz. biz yine doktorluk sistemi." Prototip: `scratchpad/_panel_yargim_v1.html`
(+ `.png`) — tasarım kararlarının gerekçesi orada.

⚠⚠ BU SAYFA MEVCUT YÜZEYE DOKUNMAZ: `nav()`/`.rail`/mevcut rotalar DEĞİŞMEDİ. `/panel`
  KENDİ sidebar'ını çizer (`nav()` ÇAĞRILMAZ) — founder bakıp karar verene kadar sıfır
  regresyon riski. Bugün ürün 64px ikon rayı + 264px beyaz kenar (iki ayrı yüzey)
  kullanıyor; yargım'da tek geniş ETİKETLİ koyu panel var (keşif daha kolay: ikon
  rayında "Araçlar" ne demek üstüne gelmeden anlaşılmıyor). Bu fark `/panel`'e taşındı.

⚠⚠ BU MODÜL `main.py`DEN (ve webutil.py DAHİL hiçbir app modülünden) HİÇBİR ŞEY IMPORT
  ETMEZ — `cases_body.py`/`chat_body.py` deseni (SAF SABİT + saf render fonksiyonu).
  Kaçış `html.escape` STDLIB'ten, `webutil.esc` DEĞİL (dairesel risk sıfıra iner).
  `panel_routes.py` veriyi DB'den çeker, ZATEN-BİÇİMLENMİŞ basit değerlerle (dict/str/int)
  buradaki `render_*` fonksiyonlarını çağırır — bu dosya satır şeklini/DB şemasını HİÇ bilmez.

⚠⚠ CSS SINIF ADLARI `pnl-` ÖNEKLİ: `ui_css.py`'de `.logo`/`.nav` ZATEN TANIMLI
  (head() her sayfaya global CSS'i enjekte eder — bu sayfa da `head()` kullanıyor).
  Önek olmadan aynı ada denk gelen bir kural CASCADE'te hangi taraf kazanır belirsiz
  hâle gelir; `pnl-` öneki çakışmayı YAPISAL olarak imkânsız kılar (ölçmeye gerek
  bırakmaz).

⚠⚠ RENK BEYANI — İLK YAZIMDAKİ HÂLİ YANLIŞTI, ÖLÇÜLDÜ (B3, 2026-08-02):
  Burada "elle hex YAZILMADI, TEK istisna amber" yazıyordu. Gerçekte **10 ham hex**
  vardı ve **6'sı hiçbir app dosyasında geçmiyordu = İCAT EDİLMİŞTİ.**
  ⚠ NEDENSEL BULGU (Hasan): üründen gelen 25 rengin **25'i** AA'yı geçiyor; AA altında
  kalan **3'ün 3'ü** icat edilmiş renklerdi. **Token disiplininden çıkılan her yerde
  kontrast düşmüş** — yani kural estetik değil, ölçülmüş bir kalite freni.
  Düzeltilenler (mevcut renk/token ile, YENİ renk icat edilmeden):
    · `#8A9B98` "YAKINDA" rozeti  2,53:1 → `var(--slate)`  4,85:1
    · `#6E9A93` "BRANŞ ARAÇLARI" 3,92:1 → `#8FB9B0`        5,69:1
    · `#7FA9A2` kişi alt satırı   4,08:1 → `#8FB9B0`        4,91:1
      (4,08 rakamı `.pnl-kisi`in `rgba(255,255,255,.05)` BİLEŞİK zeminine göredir;
       düz `--ink` üstünde 4,73 çıkar ve kusur GÖRÜNMEZ — zemin bileşikse onu hesapla.)
    · `#04231B` (verify üstü metin) → `var(--verify-ink)`: CLAUDE.md'nin AÇIK kuralı
      "`--verify` üstündeki metin daima `--verify-ink`, hex'i elle yazma".
  ⚠⚠ "KALAN İKİ ham hex" SATIRI BAYATLADI — ÖLÇÜLDÜ (Deniz, 2026-08-04): yorumlar
  sıyrılınca PANEL_CSS'te ham hex sayısı ikiden ÇOK. Güncel sayı buraya YAZILMIYOR
  (tekrar bayatlar) — ölçen araç `scratchpad/_deniz_renk_olc.py` yanındaki tek satırlık
  sayımdır: yorumları sıyır, `PANEL_CSS` içinde hex deseni say.
  ⚠ Bayatlama SEBEBİ öğretici: satır doğruydu ve sonra `#8FB9B0` (B3 kontrast düzeltmesi)
  ile `#DBEEE8` (hover kuralı) EKLENDİ; ekleyen bu satırı güncellemedi. Sabit bir sayıyı
  yoruma yazmak, onu güncelleme borcunu her sonraki düzenleyiciye devretmektir — bu depo
  bunu "kopyalanan sayı bayatlar" diye zaten kural yapmış, satırın kendisi kanıtı.
  ⚠ AA tarafı ÖLÇÜLÜYOR ve nöbetçisi var: `scratchpad/panel_renk_kapisi.py` panelde AA
  altı çift kalmadığını HESAPLANAN stilden doğrular (bileşik zeminle) → ham hex sayısı
  kaysa bile kontrast sessizce düşemez.
  ⚠ Aşağıdaki iki gerekçe GEÇERLİ: `#B9D6CF` (logo linki) ve amber çifti
  `#8A5A00`/`#FDF6EA`. ⚠ Amber için "admin_routes'tan
  AYNEN taşındı" iddiası YARIM doğruydu: `var(--amber-ink)` gerçekten oradan, **`#FDF6EA`
  hiçbir yerde YOK** — o zemin burada üretildi. Tokenlaştırılacaksa `--amber-soft`/
  `--amber-ink` ui_css.py'ye eklenmeli (ayrı iş, paylaşılan dosya).

⚠ "Kanıt geçerliliği" kartı BİLEREK YOK — yalnız `render_gecerlilik_yeri()` bir YER-TUTUCU
  döner (buton/girdi YOK, işlevmiş gibi görünmez). Derya'nın geri-çekilme veri katmanı
  01.08'de kuruldu (96 kayıt) ama founder "elensin mi işaretlensin mi" kararını henüz
  vermedi — kart inşa edilmedi, yalnız yeri açık bırakıldı.
  ⚠⚠ KART İNŞA EDİLİRSE ÜÇ DURUM ZORUNLU (geri çekilmiş / kayıt yok / **durum
  BİLİNMİYOR**): korpusun ~%89'u `unknown` ve `unknown` için yeşil "✓ GEÇERLİ" basmak
  FAIL-OPEN YASAĞININ tam ihlalidir. Founder tasarım PNG'sinde yeşil rozeti gördü ve
  onayladı — o rozet bu hâliyle YAPILMAZ.

⚠⚠ TASARIM PNG'SİNDEKİ SELAMLAMA ("Günaydın, Doktor") BİLEREK UYGULANMADI (2026-08-02).
  Gerekçe: selamlama bir UNVAN VARSAYIMI yapar; kayıtlı unvanlardan biri "Tıp Öğrencisi"
  ve `_UNVANLAR_REG`de altı farklı unvan var — herkese "Doktor" demek yanlış hitap olur,
  unvana göre dallanmak ise saat-dilimi + cinsiyet + unvan üçlüsünü panele sokar.
  Bugünkü "Panel" başlığı nötr ve doğru. ⚠ SESSİZ BIRAKILMADI, çünkü ayrım şu:
  **belgelenmiş ayrışma kusur DEĞİL, belgelenmemiş ayrışma kusurdur.** PNG ile kod
  arasındaki her fark ya burada yazılı olmalı ya da bir takım tarafından çivilenmeli.

⚠⚠ i18n — BU SAYFA `EN_PAIRS`E BİLEREK GİRMEDİ, UNUTULMADI (Ömer onayı, 2026-08-02).
  `panel_routes.py` TR/EN'i `lang == "en"` ile SUNUCUDA üretir — `cases_routes.py`nin
  AYNI deseni. Gerekçe: bu sayfa hasta kodu / kronik hastalık / soru metni gibi DİNAMİK
  klinik veri basıyor; `i18n_mw` (lang=en'de) TÜM HTML gövdesini `EN_PAIRS` ile KÖR
  arayıp-değiştiriyor. Bir hastanın `chronic`/`meds` metni ya da bir soru başlığı
  tesadüfen bir TR pazarlama ifadesine (EN_PAIRS anahtarı) denk gelirse SESSİZCE yanlış
  dile çevrilir — `en_enjeksiyon_verify`nin çivilediği sınıfın AYNISI (window.* enjeksiyonu
  yerine burada hasta verisi enjekte oluyor). `/cases` da TAM BU YÜZDEN EN_PAIRS
  KULLANMAZ. **Bu sayfaya sonradan EN_PAIRS girdisi EKLEMEK bir "eksik" DEĞİL, riski
  GERİ GETİRMEKTİR — ekleme.**
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `PANEL_CSS` | `'\n/* ⚠⚠ SIDEBAR TİPOGRAFİSİ BÜYÜTÜLDÜ (founder 2026-08-07: "masaüstünde menüdeki\n   yazıları büyütsek mi"). Ölçüm: kap` | 95 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_e` | `(v) -> str` | 90 | XSS kaçışı — `webutil.esc` ile AYNI mantık, import YOK (dosya başlığına bkz). |
| `render_kunye` | `(logo_svg: str, ad: str, tarih: str) -> str` | 500 | Sayfanın en üstündeki künye: marka · hekim · tarih (founder isteği 2026-08-08). |
| `render_header` | `(title: str, subtitle: str, btn_new_label: str, btn_new_href: str, btn_ask_label: str, btn_ask_href: str, urgent: str = '', erisim_label: str = '', erisim_href: str = '') -> str` | 520 | ⚠ `urgent` (#61, yargım deseni): gecikmiş takip VARKEN başlığın altına aciliyet |
| `card` | `(title: str, link_label: str, link_href: str, rows_html: str, empty_html: str = '') -> str` | 555 |  |
| `row_case` | `(title: str, sub: str, badges: list[tuple]) -> str` | 568 | badges: [(text, css_class)] veya [(text, css_class, title_attr)]. |
| `row_simple` | `(title: str, sub: str, right: str) -> str` | 584 |  |
| `row_appt` | `(gun: str, ay: str, title: str, sub: str, badge: tuple \| None = None) -> str` | 589 | badge: (metin, css_class) opsiyonel — aciliyet rozeti (ör. '2 gün kaldı', pnl-k-amb). |
| `render_gecerlilik_yeri` | `(badge: str, title: str, body: str) -> str` | 596 | Yer-tutucu — dosya başlığındaki kural: kutu İNŞA edilmez (buton/girdi yok), |
| `render_etk_karti` | `(*, icon_svg: str, baslik: str, aciklama: str, not_: str, ph_a: str, ph_b: str, btn: str, lbl_a: str, lbl_b: str) -> str` | 603 | KOYU VURGU KARTI — `/etkilesim` girdisi (yargım deseninin mekanik eşi, #61). |
| `render_tool_promo` | `(sim_placeholder: str, title: str, sub: str, href: str) -> str` | 637 |  |
| `render_ay_ozeti` | `(items: list[tuple]) -> str` | 642 | "Bu ay" şeridi — `items: [(sayı_metni, etiket)]`. (#126b-B) |
| `render_olcusuz` | `(rozet: str, metin: str) -> str` | 656 | ÜÇ-DURUMUN ÜÇÜNCÜSÜ: sorgu patladı → amber "ölçülemedi". (#126b-B) |
| `assemble` | `(sidebar_html: str, header_html: str, left_cards_html: str, right_cards_html: str, band_html: str = '') -> str` | 669 | ⚠ `band_html` = iki kolonun ALTINDAKİ tam genişlik bandı (founder kararı (b2)). |

## `saglik/app/panel_routes.py`

`679 satır` · `7 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
`/panel` — yargım deseninin uyarlaması, EK sayfa (2026-08-02, founder yönü).

Ayrıntı + tasarım gerekçesi: `panel_body.py` başlığı + `scratchpad/_panel_yargim_v1.html`.
`main.py`den ayrı modül (Faz 5/6/7/8 deseni): `@_rota` KAYIT listesi + `kur(app)` —
`include_router` KULLANILMAZ (bu FastAPI sürümünde tembel, `app.routes`ı düzleştirmez;
gerekçe ölçüldü — bkz. `.claude/agents/kamil.md` / diğer `*_routes.py` başlıkları).

⚠⚠ MEVCUT YÜZEYE DOKUNULMADI: `nav()` ÇAĞRILMAZ, `.rail`/`webutil.py`/main.py'nin
  mevcut rotaları DEĞİŞMEDİ. `/panel` KENDİ sidebar'ını `panel_body.py`'den render eder.
  `kur(app)` çağrısı main.py'de `_account_kur(app, 1)`'DEN SONRA (en sonda) eklendi —
  var olan hiçbir rotanın `app.routes` sırasını KAYDIRMAZ (araya değil, sona ekler).

⚠ `/panel` `_PROFIL_MUAF` listesine EKLENMEDİ — bilerek: bu bir kimlik gerektiren
  hekim panosu, `/chat`/`/cases`/`/takvim` ile AYNI kapıya (profil tamamlama +
  `/giris` yönlendirmesi) tabi olmalı. Varsayılan davranış zaten bunu yapıyor.

⚠ i18n: `cases_routes.py` deseni — TR/EN `lang == "en"` ile SUNUCUDA üretilir,
  `EN_PAIRS`e YENİ SATIR EKLENMEDİ (statik metin global tabloya bağımlı değil).
⚠⚠ AMA BU SAYFAYI KORUMAZ — İLK YAZIMDAKİ GEREKÇE MEKANİK OLARAK YANLIŞTI (P2-7,
  ölçüldü 2026-08-02): burada "dinamik hasta verisi EN_PAIRS'e denk gelirse sessizce
  çevrilir, o yüzden EN_PAIRS kullanmıyoruz" yazıyordu. **Yeni çift EKLEMEMEK, GLOBAL
  tablonun çıktıya UYGULANMASINI engellemez.** `i18n_mw` muafiyet listesi yalnız
  `/hesaplayicilar /kilavuz /guvenlik /chatgpt /etkilesim` — `/panel` ORADA DEĞİL.
  Ölçüldü: hasta `chronic` alanı "Kontrendikasyon" ise EN'de "Contraindication" olur.
  ⚠ `/cases` de AYNI sızıntıyı taşıyor → bu ÖNCEDEN VAR OLAN bir ürün kusuru, `/panel`
  onu miras aldı; `/panel`e özgü değil.
  ⚠ GERÇEK DÜZELTME (muafiyet listesine `/panel` + `/cases`) BİLEREK YAPILMADI — iki
  sayfayı birden etkiler ve karar lead'de. Burada yalnız YANLIŞ GEREKÇE düzeltildi;
  "gerekçe yanlışsa koruma da yoktur" diye bilinsin.

⚠ Tarih/saat — İKİ AYRI MODEL, KARIŞTIRILMAZ:
  · `case_note.last_visit` / `thread.created_at` = GERÇEK olay anı (timestamptz,
    UTC farkında döner) → Türkiye yerel saatine `.astimezone(_TR_TZ)` ile ÇEVRİLİR.
  · `appointment.starts_at` = DUVAR-SAATİ (memory `klivance-takvim-saat-modeli`,
    BOZMA): ham sayılar zaten Türkiye yerel saatidir, `.replace(tzinfo=None)` ile
    tzinfo SIYRILIR — `cal_routes._appt_json`'daki AYNI desen, ASLA `.astimezone()` ile
    ÇEVRİLMEZ (çevirirse ızgara/panel arasında saat kayması yeniden doğar).

⚠ "Kanıt geçerliliği" kartı İNŞA EDİLMEDİ (yalnız yer-tutucu) — Derya'nın geri-çekilme
  veri katmanı (01.08, 96 kayıt) var ama founder "elensin mi işaretlensin mi" kararını
  vermedi. `panel_body.render_gecerlilik_yeri` bkz.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_TR_TZ` | `ZoneInfo('Europe/Istanbul')` | 71 |
| `KAYIT` | `[]` | 74 |
| `_AY_TR` | `['OCA', 'ŞUB', 'MAR', 'NİS', 'MAY', 'HAZ', 'TEM', 'AĞU', 'EYL', 'EKİ', 'KAS', 'ARA']` | 107 |
| `_AY_EN` | `['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC']` | 108 |
| `_BRANS_ETK_VARSAYILAN` | `('varfarin', 'amiodaron')` | 142 |
| `_ETK_EN` | `{'varfarin': 'warfarin', 'amiodaron': 'amiodarone', 'ramipril': 'ramipril', 'spironolakton': 'spironolactone', 'sertrali` | 148 |
| `_BRANS_ICERIK` | `{'Kardiyoloji': (('varfarin', 'amiodaron'), ('cha', 'hasbled')), 'Nefroloji': (('ramipril', 'spironolakton'), ('egfr', '` | 156 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 77 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 85 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_wall` | `(dt_val) -> _dt.datetime` | 91 | Duvar-saati (appointment.starts_at) — tzinfo SIYRILIR, ÇEVRİLMEZ. Bkz. dosya başlığı. |
| `_tr_local` | `(dt_val) -> _dt.datetime` | 98 | Gerçek olay anı (thread/case timestamptz) — Türkiye yerel saatine ÇEVRİLİR. Bkz. dosya başlığı. |
| `_gun_ay` | `(dt_val, en: bool) -> tuple` | 111 |  |
| `_tarih_kisa` | `(dt_val, en: bool) -> str` | 116 |  |
| `panel_page` 🌐 | `(request: Request)` | 181 |  |

## `saglik/app/rad_case.py`

`265 satır` · `17 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
GÖRÜNTÜLEME (YOL 1) — RAPOR METNİNDEN LEZYON ŞERİDİ + TAKİP RANDEVUSU (IP2, 2026-09-13).

Sözleşme: `docs/goruntuleme-sozlesme-2026-09-13.md` §3.2 (imzalar/sabitler ORADAN). `deger_onay.py`
ikizi: hasta kartı yüklenince son 6 `extracted_text` `rad_cikar.rad_satirlari` ile ayrıştırılır,
lezyonlar bir ŞERİT olarak sunulur, hekim {3, 6, 12, 24} ay seçip `POST /api/cases/{cid}/rad-takip`
ile TAKVİME takip yazar. LLM 0 · `record_usage` 0 · `case_file`/`case_note`/`case_timeline`/`case_visit`e
yazma 0 (yalnız `app.appointment` + `hekim_iz` "rad_takip" ücretsiz yüzeyi).

⚠⚠ FAIL-OPEN YASAK: `oneriler` istisnası SESSİZ `[]` DEĞİL — `(satırlar, hata)` ikilisinde `hata` dolar,
   şerit tek amber satır basar (`rad_serit_kurulamadi`). `deger_onay.oneriler`in sessiz `[]`i BİLEREK
   klonlanmadı (kararlar: "çalıştırılamadı ≠ temiz").
⚠ RAPOR-DEDİ ↔ KLİVANCE AYRIMI: şeritteki `sinif`/`oneri` raporun beyanıdır (RAD satırı, K1). Ön-seçim
   (`checked`) YALNIZ `kaynak=evet` (alıntı kaynak metinde birebir doğrulandı) VE raporun `oneri`si
   {3, 6, 12, 24} kümesinden bir ay taşıyorsa; `hayir`/`olculemedi` → tüm düğmeler NÖTR (kararlar B.Q9).
⚠ TEK SÖZLÜK: hekime görünen metinler `cds.isaretler.METIN` (kararlar C.Q10) — bu modül KENDİ `_M`
   sözlüğünü AÇMAZ. Düğme etiketi `>Takip<` mevcut `EN_PAIRS` çiftiyle çevrilir; ay birimi
   `rad_takip_baslik` şablonunun kuyruğundan türetilir (`_ay_birimi`) — ikinci kopya YOK.
⚠ Kokpit 6. karo: `_RAD_KOKPIT_JS` (`radCoz`) `kokpit_body._COCKPIT_JS`teki `__RADCOZ__` yer tutucusuna
   `cases_routes` tarafından yerleştirilir (kokpit_body SAF sabit: import YASAK). JS'te TR literal 0,
   innerHTML 0, fetch 0; renk yok (amber KOŞULU WP-6'nın tek ifadesi, §1.4 — burada DEĞİL).
⚠ Bu modül `cases_routes`/`analiz_routes`tan HİÇBİR ŞEY import ETMEZ; onlar beni import eder (tek yön).
Kapılar: `scratchpad/rad_takip_verify.py` (T1-T6) · `scratchpad/rad_kablolama_verify.py` (KB1-KB4).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 40 |
| `AYLAR` | `(3, 6, 12, 24)` | 42 |
| `BASLIK_TAVAN` | `120` | 43 |
| `DOSYA_LIMIT` | `6` | 44 |
| `TAKIP_SAATI` | `9` | 45 |
| `_AY_RE` | `re.compile('(?<!\\d)(3\|6\|12\|24)\\s*(?:ay\|mo\|month)', re.I)` | 46 |
| `_RENK_AMBER` | `'var(--amber-ink)'` | 47 |
| `_RAD_KOKPIT_JS` | `'function(d,st){\n var K=nz(T.L.rad_oneri_iz\|\|"");\n for(var q=0;q<st.length;q++){var s=st[q],r=el("div","ckpt-satir")` | 210 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(yontem: str, yol: str, **kw)` | 50 |  |
| `kur` | `(app) -> None` | 57 |  |
| `_L` | `(lang) -> str` | 62 |  |
| `_M` | `(lang) -> dict` | 66 |  |
| `_ay_birimi` | `(lang) -> str` | 70 | `rad_takip_baslik` "… {ay} ay" / "… {ay} mo" kuyruğu → "ay" / "mo" (tek sözlük, ikinci kopya yok). |
| `rapor_ayi` | `(oneri: str) -> int \| None` | 76 | Raporun KENDİ önerisinde {3, 6, 12, 24} kümesinden bir ay geçiyor mu (ön-seçim girdisi). |
| `oneriler` | `(conn, doctor_id: int, case_id: int, lang: str) -> tuple[list[dict], str \| None]` | 82 | Son 6 dosya çıkarımındaki RAD satırları → (satırlar, hata). |
| `_boyut_goster` | `(r: dict) -> str` | 110 | `boyut=belirtilmemis` gramer TOKEN'ıdır, hekime basılmaz → '—' (dil-bağımsız; şerit + takvim başlığı). |
| `_ozet` | `(r: dict) -> str` | 116 | Lezyon satırının tek satırlık özeti — alan değerleri raporun beyanı (K1), sıfat EKLENMEZ. |
| `serit` | `(oneri: list[dict], hata: str \| None, cid: int, lang: str) -> str` | 127 | Takip şeridi. `hata` → tek amber satır; öneri yoksa BOŞ dize (şerit hiç çizilmez). |
| `serit_html` | `(conn, d: dict, cid: int, lang: str) -> str` | 169 | TEK çağrı (`cases_routes`, `conn` bloğunun İÇİNDE): okuma + HTML. İstisna → amber, asla ''. |
| `analiz_kapisi` | `(answer: str, metinler, lang: str) -> tuple[str, list[str]]` | 179 | Analiz yanıtında anılan sınıf ∖ dosya çıkarımlarının RAD kümesi → (amber_blok \| "", token listesi). |
| `ihlal_etiketi` | `(tokens: list[str], lang: str) -> str` | 193 | Ziyaret notu `tag` öneki `[⚠ N raporda geçmeyen sınıf: …] ` (boş listede ''). |
| `uyari_metni` | `(tokens: list[str], lang: str) -> str` | 201 | İstemci `alert` notu için UI dilinde tek cümle (boş listede ''). |
| `takip_tarihi` | `(ay: int, simdi: _dt.datetime \| None = None) -> _dt.datetime` | 222 | Bugün + `ay` ay, TR duvar-saati 09:00 (naive — takvim modeli `klivance-takvim-saat-modeli`). |
| `takip_basligi` | `(r: dict, ay: int, lang: str) -> str` | 230 | Başlık SUNUCUDA kurulur (form serbest metin taşımaz): ≤120 krk, `redact_pii`. |
| `rad_takip` 🌐 | `(cid: int, request: Request, bulgu_idx: str = Form(''), ay: str = Form(''))` | 237 | YALNIZ takvime yazar. Gövde `bulgu_idx` + `ay`; başka alan yok sayılır. |

## `saglik/app/recete_body.py`

`706 satır` · `14 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
REÇETE KONTROLÜ — SAF GÖVDE (CSS + metin sözlükleri + render yardımcıları).

Kapsam: `/recete-kontrolu` (hub) ve `/bobrek-doz` (böbrek doz kontrolü) sayfalarının
DB'ye/FastAPI'ye dokunmayan her parçası. Rota tarafı `recete_routes.py`.
Ayrılma sebebi `etkilesim_page`/`etkilesim_body` ile aynı: buradaki her şey birim-test
edilebilir saf fonksiyon; oradaki her şey istek/DB'ye bağlı.

⚠⚠ AKLAMA DİLİ YASAĞI BURADA DA GEÇERLİ — bu ailenin TEMEL kuralı. Hiçbir durumda
   yeşil/aklama dili YOKTUR: "temiz", "güvenli", "uyumlu", "sorun yok", "✓" ve EN
   karşılıkları hiçbir metinde geçmez. Liste TEK KAYNAK `scratchpad/_klv_aklama.py`
   (#62b) — kopyalama, oradan oku. Kapı `scratchpad/recete_kontrol_verify.py`.
   ⚠ EN listesinde `safe` var ve eşleşme ALT DİZGEdir → **`safety` de yakalanır.**
     Bu dosyanın İngilizce metinlerinde "safety" BİLEREK kullanılmadı.
   ⚠ Yasak, BİZİM cümlelerimiz içindir. Alıntılanan resmî etiket metni (blockquote)
     kapsam dışıdır ve olmak zorundadır: kaynağın kendi kelimelerini sansürlemek kanıt
     sunan bir arayüzde alıntıyı tahrif etmek olurdu (`/etkilesim` ile aynı disiplin).

⚠⚠ NE YAPAR / NE YAPMAZ (founder 2026-08-17, motor sözleşmesi `saglik/cds/renal.py`):
   HESAPLAMAZ · doz ÖNERMEZ · tanı KOYMAZ. Yalnız etiketin böbrekle ilgili ifadesini
   hekimin girdiği eGFR/CrCl değeriyle YAN YANA koyar. Sayısal eşik yakalandıysa
   "girilen değer eşiğin altında" diye İŞARETLER — kararı hekim verir (FDA Kriter-4).

⚠⚠ BEŞ DURUMUN HEPSİ ÇİZİLİR ve `ifade_yok` FAIL-CLOSED'DIR: "etikette bulunamadı"
   asla "ayar gerekmez" diye okutulmaz. Bu bir üslup tercihi değil, bu üründe YANLIŞ
   GÜVENin yanlış cevaptan tehlikeli olmasının sonucudur.

⚠ `esik_altinda` KIRMIZI METİN (`--alert-ink`), diğerleri amber/gri/mor. `/etkilesim`
  "kırmızı YOK" der çünkü orada ciddiyet hükmü BİZİM olurdu; burada kırmızı bir
  ciddiyet hükmü değil ARİTMETİK BİR OLGUdur (etiketin KENDİ söylediği eşik + hekimin
  girdiği değer). İki en önemli durumu aynı renkte basmak aracın tek ayırt edici
  çıktısını görünmez kılardı. ⚠ Tasarım DİLİ Deniz'in — değiştirmeden önce ona sor.

⚠ CSS `.rc-*` ayrı yazıldı; `etkilesim_page.ETK_CSS` PAYLAŞILMADI. Paylaşmak
  `/etkilesim`in CSS'ine dokunmak olurdu ve bu turun kuralı "`/etkilesim` davranışı
  DEĞİŞMEZ" idi. Borç: ikisi ileride ortak bir araç-CSS'ine katlanabilir.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_DURUM_RENK` | `{'esik_altinda': _ALERT, 'ifade_var': _AMBER, 'ifade_yok': _AMBER, 'taranamadi': _GRI, 'calistirilamadi': _MOR}` | 49 |
| `_KAYNAK_AD` | `{'openfda': 'openFDA', 'dailymed': 'DailyMed', 'kub': 'TİTCK KÜB'}` | 54 |
| `_ALAN_AD` | `{'tr': {'dosage': 'Doz ve uygulama', 'warnings': 'Uyarılar', 'contraindications': 'Kontrendikasyonlar', 'boxed_warning':` | 73 |
| `_ALAN_GENEL` | `{'tr': 'Etiket', 'en': 'Label'}` | 84 |
| `_BIRIM_KANONIK` | `{'ml/dk', 'ml/dak', 'ml/dakika', 'ml/min', 'ml/minute', 'mldk', 'mlmin'}` | 89 |
| `_BIRIM_AD` | `{'tr': 'mL/dk', 'en': 'mL/min'}` | 90 |
| `_KAR_AD` | `{'<=': '≤', '>=': '≥', '=<': '≤', '=>': '≥'}` | 92 |
| `MAKS_ILAC` | `8` | 94 |
| `MAKS_EGFR` | `250.0` | 95 |
| `RECETE_CSS` | `'\n/* Kabuk ofseti: ray akıştan çıkar (fixed) → gövde onu temizler (etkilesim_page deseni;\n   o dosyadaki ölçüm burada ` | 97 |
| `HUB` | `{'tr': {'h1': 'Reçete kontrolü', 'sub': 'Yazmayı düşündüğünüz reçeteyi resmî ilaç etiketlerine karşı kontrol edin. Her a` | 211 |
| `REN` | `{'tr': {'h1': 'Böbrek doz kontrolü', 'sub': 'Hastanın eGFR/CrCl değerini ve ilaçları yazın: resmî etiketlerde (openFDA/D` | 298 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_T` | `(lang: str, sozluk: dict) -> dict` | 441 |  |
| `referans_kilit_html` | `(lang: str, eksik_profil: bool = False) -> str` | 445 | Gömülü referans bölümünün KİLİTLİ hâli — görünür ama kullanılamaz. |
| `ilaclar_coz` | `(s: str) -> list[str]` | 495 | Virgül/artı ile ayrılmış listeden temiz ilaç adları. |
| `egfr_coz` | `(s: str) -> tuple[float \| None, bool]` | 521 | (değer, bozuk_mu). Boş girdi bozuk DEĞİLDİR — değersiz tarama meşru bir kip. |
| `_sayi` | `(lang: str, v: float) -> str` | 542 | Ondalık ayracı DİLE bağlı (etkilesim_body.sayi ile aynı ders, ondalık ayağı). |
| `_pasaj_metni` | `(p: dict) -> str` | 550 | Kesik cümleyi `…` ile İŞARETLE — motor kelime sınırını yuvarlar, sayfa CÜMLE |
| `birim_ad` | `(lang: str, birim: str) -> str` | 565 | Klirens birimini DİLİN yazımıyla bas; tanınmayanı OLDUĞU GİBİ bırak (ölçü uydurma). |
| `alan_ad` | `(lang: str, alan: str) -> str` | 571 | Motor alan anahtarı → hekimin okuyacağı bölüm adı. |
| `_esik_html` | `(lang: str, esikler: list[dict], T: dict) -> str` | 583 |  |
| `_pasajlar_html` | `(lang: str, r: dict, T: dict) -> str` | 605 | Pasajlar. ⚠⚠ EŞİK ASLA CÜMLESİZ BASILMAZ: motor eşik döndürüp pasaj döndürmediyse |
| `_link_html` | `(r: dict, T: dict) -> str` | 629 | Resmî belge linki — YOKSA HİÇBİR ŞEY basılır (ölü span = sahte provenans). |
| `_satir_html` | `(lang: str, r: dict, T: dict) -> str` | 638 | Tek ilacın sonuç kutusu. ⚠ BİLİNMEYEN DURUM `calistirilamadi`ya düşer — |
| `calistirilamadi_sonuc` | `(ilaclar: list[str], egfr: float \| None) -> dict` | 673 | Motor HİÇ KOŞMADIĞINDA (hız sınırı · semafor · istisna) dönen fail-CLOSED sonuç. |
| `sonuc_html` | `(lang: str, sonuc: dict \| None, egfr: float \| None, egfr_bozuk: bool, bos: bool) -> str` | 685 | `id="renal-sonuc"` kapsamının İÇİ. |

## `saglik/app/recete_case.py`

`981 satır` · `19 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HASTA KARTI · REÇETE KONTROLÜ SEKMESİ — panel gövdesi + JSON ucu (#336b, 2026-08-17).

`/cases/{cid}` 5. sekmesi: hekimin KARTA GİRDİĞİ bağlamı (ilaç listesi · eGFR · alerji
çipleri · gebelik/laktasyon) dört LLM'siz kontrolden geçirir — etkileşim · böbrek ·
alerji · duplikasyon. Founder kararı 2026-08-17 ("dört kontrol olsun, sekme yapalım").

⚠⚠ **DOKUNMADAN ÖNCE `.claude/agents/kamil.md` → "HASTA KARTI · REÇETE KONTROLÜ SEKMESİ"
   BÖLÜMÜNÜ OKU.** Bu dosyanın 87 satırlık invariant bloğu 2026-08-17'de oraya TAŞINDI
   (dosya doğduğu gün üç kez tavana dayandı; dördüncü yükseltme yerine taşıma yapıldı —
   Derya aynı gün kendi motoru için aynısını yaptı). Orada yazılı ve BURAYA KOPYALANMAZ:
   fail-closed sözleşmesi · `EN_PAIRS`e sıfır satır kuralı · makine anahtarının ekrana
   basılmaması · kırmızının yalnız BEYAN EDİLMİŞ anahtarla basılması (`_ALERT_KOD` +
   `_ALERT_ALT` yedeği ve ikisinin ayrışmasının neden sessiz olacağı) · alerjide ÜÇ HÂL ·
   `notlar[]` ayrımı · kırpma/taranamadı görünürlüğü · çok kaynaklı belge ve neden SAYI
   SABİTLENMEDİĞİ · belgesiz satırda neden hiçbir şey basılmadığı (ölçülmüş oran) ·
   kredi/fren modeli · threadpool · sahiplik · `data-lbl` ikon tuzağı · motor adının iki
   kez yazılmaması. **Her maddesi ölçülmüş bir hatadan doğdu.**

⚠⚠ MOTOR SÖZLEŞMESİ `saglik/cds/recete_kontrol.py` BAŞLIĞINDADIR — ne buraya ne ajan
   dosyasına kopyalanır (kopyalanan sözleşme ayrışır, ayrışan sözleşmenin hangi yarısının
   yürürlükte olduğu bir daha bilinmez).

⚠⚠ BU MODÜL `main.py`DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ);
   `cases_routes`tan da import ETMEZ (o BENİ import eder — ters yön DÖNGÜ olurdu).
   Paylaşılanlar `webutil`den DOĞRUDAN okunur, main fasadından DEĞİL (`_RATE` kovası
   bölünmesin). ⚠ `include_router` KULLANILMAZ (Faz 2b'de ölçülerek reddedildi):
   `@_rota` KAYIT listesi + `kur(app)`; `kur` main.py'de `_recete_kur(app)`DEN SONRA.

Kapılar/araçlar: `scratchpad/recete_sekme_verify.py` (CI) · `_kamil_recete_sekme_gercek.py`
· `_kamil_alerji_uc_hal.py` · `_kamil_recete_sekme_mobil.py` (son üçü KB/sunucu ister).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 48 |
| `_RK_SEM` | `threading.Semaphore(3)` | 54 |
| `RK_DAILY_CAP` | `200` | 68 |
| `RK_DAILY_CAP_TRIAL` | `60` | 69 |
| `_RK_GUN` | `{}` | 70 |
| `_RK_GUN_KILIT` | `threading.Lock()` | 71 |
| `T` | `{'tr': {'sek': 'Reçete kontrolü', 'h': 'Reçete kontrolü', 'note': 'Bu kartta yazılı ilaç listesini dört kontrolden geçir` | 124 |
| `_KONTROL_AD` | `('etkilesim', 'renal', 'alerji', 'duplikasyon')` | 323 |
| `_ALERT_ALT` | `{('renal', 'esik_altinda'), ('alerji', 'dogrudan')}` | 327 |
| `_ALERT_KOD` | `{'renal_esik_altinda', 'alerji_dogrudan'}` | 334 |
| `_RK_CSS` | `'<style>\n.rk-ctx{margin-top:10px}\n.rk-not{font-size:13px;color:var(--slate);line-height:1.55;margin-top:8px}\n.rk-cipl` | 456 |
| `_RK_JS` | `'<script>\n(function(){\n var b=document.getElementById(\'rk-go\'), out=document.getElementById(\'rk-sonuc\');\n if(!b\|` | 510 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_tr_bugun` | `() -> _dt.date` | 74 |  |
| `_tavan_ok` | `(d) -> bool` | 78 | Günlük tavanı tüket ve karar ver. ⚠ Tavan SEÇİMİ tek kaynaktan (`chat_routes. |
| `_rota` | `(metod: str, yol: str, **kw)` | 104 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 112 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_t` | `(lang: str) -> dict` | 337 |  |
| `sekme` | `(lang: str) -> tuple[str, str, str]` | 341 | `cases_routes._SEK` ögesi — (anahtar, ikon, etiket). Etiket SUNUCUDA üretilir. |
| `_alan` | `(etiket: str, deger: str) -> str` | 347 |  |
| `_sayi` | `(lang: str, v) -> str` | 352 | Ondalık ayracı DİLE bağlı — '28.5' bir TR okuruna 'yirmi sekiz nokta beş' diye |
| `panel` | `(case_ctx: dict, cid: int, lang: str) -> str` | 362 | Sekmenin İÇİ: bağlam özeti + düğme + boş sonuç bölgesi + JS. SAF — DB'ye/isteğe |
| `_kontrol_ad` | `(S: dict, k: str) -> str` | 618 |  |
| `_rol` | `(satir: dict) -> str` | 622 | Satırın rol rengi. ⚠⚠ KIRMIZI TAHMİN EDİLMEZ — yalnız BEYAN EDİLMİŞ anahtarlar |
| `_pasaj` | `(p) -> dict \| None` | 640 | Motor pasajı → ekran pasajı. ⚠ Kesik cümle `…` ile İŞARETLENİR: işaretsiz basmak |
| `_satir` | `(S: dict, satir: dict) -> dict` | 658 | Motor satırı → ekran satırı. Makine anahtarı EKRANA GİTMEZ: `kontrol` görünür ada |
| `_ser` | `(S: dict, k: str, ozet: str, rol: str, durum: str = '') -> dict` | 719 | Şerit kutusu. ⚠⚠ `durum` MAKİNE-OKUNUR DURUM BEYANI — ekrana BASILMAZ, yalnız |
| `_serit` | `(S: dict, lang: str, kon: dict, n_ilac: int) -> list[dict]` | 730 | Dört kontrolün özet şeridi. ⚠⚠ HER KONTROL DAİMA GÖRÜNÜR — patlayan kontrol |
| `sonuc_json` | `(ham: dict, lang: str) -> dict` | 840 | Motor çıktısı → ekranın çizeceği JSON. ⚠ Ekrana giden HER dize burada ÇEVRİLİR; |
| `calistirilamadi_json` | `(lang: str) -> dict` | 916 | Motor HİÇ KOŞMADIĞINDA (hız freni · günlük tavan · semafor · istisna) dönen |
| `async` `api_recete_kontrol` 🌐 | `(cid: int, request: Request)` | 934 | Hasta kartındaki ilaç listesini dört LLM'siz kontrolden geçir. Kredi 0. |
| `_recete_kontrol_sync` | `(cid, request, lang)` | 944 |  |

## `saglik/app/recete_routes.py`

`442 satır` · `10 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
REÇETE KONTROLÜ ROTALARI — `/recete-kontrolu` (hub) + `/bobrek-doz` (böbrek doz kontrolü).

Founder kararı 2026-08-17: ücretsiz, herkese açık kontrol araçları tek bir ÜRÜN AİLESİ
altında toplanır ("Reçete kontrolü"). Ray ögesi artık buraya iner; `/etkilesim` URL'i,
davranışı ve reklam inişi AYNEN KALIR (yalnız ray HEDEFİ değişti).

⚠⚠ `include_router` KULLANILMAZ — bu FastAPI sürümünde tembeldir: rotaları `app.routes`a
  düzleştirmez (ölçüldü Faz 2b: 95 rota → 83) ve rota envanterini `app.routes`tan okuyan
  güvenlik ağları SESSİZCE körelir. Yerine `@_rota` KAYIT listesi + `kur(app)`.
  ⚠ `kur(app)` main.py'de `_admin_hekim_kur(app)`DEN SONRA çağrılır: iki yeni yol
    tekil ve statiktir (`/recete-kontrolu`, `/bobrek-doz`), hiçbir `{param}` rotasıyla
    çakışmaz → SONA eklenir ve mevcut rota indekslerini KAYDIRMAZ (konumu ÖLÇÜLMÜŞ
    `ref_routes`/`cases_routes` bloklarının arasına girmek onları kaydırırdı).

⚠⚠ BU MODÜL `main.py`DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ).
  Paylaşılan yardımcılar `webutil`den DOĞRUDAN okunur, main fasadından DEĞİL — `_RATE`
  kovası bölünmesin (`etkilesim_page`in aynı düzeltmesi, P2 2026-07-31).

────────────────────────────────────────────────────────────────────────────────
İNVARİANTLAR — BOZMA
────────────────────────────────────────────────────────────────────────────────
⚠⚠ FAIL-CLOSED: hız sınırı · semafor · HER istisna `calistirilamadi`ya düşer ve satırlar
  YİNE BASILIR (`recete_body.calistirilamadi_sonuc`). Boş sonuç döndürmek 'çalıştırılamadı'yı
  'bulgu yok' diye göstermek olurdu. `/etkilesim`in `QueryCanceled` kuralıyla aynı.

⚠ İKİ FREN, İKİ AYRI İŞ: `_rate_ok(f"renal:{ip}", 20)` KİŞİ başına tavan (kötüye kullanım),
  `_REN_SEM` EŞ ZAMANLI DB işi tavanı (anonim sayfa worker havuzunu yemesin). Biri
  diğerinin yerine geçmez — `/etkilesim`de ikisi de var ve gerekçeleri orada ölçüldü.
  ⚠ Kovanın ADI `renal:` — `etk:` ile PAYLAŞILMAZ; paylaşsaydı bir aracı kullanmak
    diğerini kapatırdı ve hekim sebebini göremezdi.

⚠⚠ HİÇBİR DURUMDA YEŞİL/AKLAMA DİLİ YOK ve `ifade_yok` AMBER'dır — "etikette bulunamadı"
  asla "ayar gerekmez" değildir. Metinlerin tamamı `recete_body`de, yasak liste
  `scratchpad/_klv_aklama.py`de (TEK KAYNAK), kapı `scratchpad/recete_kontrol_verify.py`.

⚠ MOTOR ÇAĞRISI TEK YERDE (`renal_check`) ve sözleşmesi `saglik/cds/renal.py` başlığında.
  Motor gövdesi 2026-08-17'de yazıldı (Derya, `renal_motor_verify` 93/93) ve sayfada
  HİÇBİR değişiklik gerektirmedi — beş durum zaten çiziliyordu. Gerçek motorla uçtan uca
  ölçüm: `scratchpad/_kamil_recete_gercek_motor.py` (beş durumun beşi de üretildi).

⚠ ÖRNEK SONUÇ OTOMATİK KOŞTURULMAZ (`/etkilesim`ten BİLİNÇLİ FARK): motor iskeletken boş
  sorguda örnek koşturmak, herkese açık bir sayfada "araç çalışmıyor" ekranı yayınlamak
  olurdu. Yerine tıklanabilir örnek BAĞLANTI var. Motor gelince örnek dalı açılabilir.

⚠ `/bobrek-doz` HASSAS SORGULU: ilaç adları `?d=` ile URL'dedir → `webutil._ANALYTICS_PATH_ONLY`
  desenine EKLENDİ (GA4'e sorgusuz `page_location`, Meta pikseli HİÇ başlamaz).
  `/recete-kontrolu` hub'ında sorgu YOK → BİLEREK desen dışında: huni ölçümü yaşasın.
  İkisinin ayrımı `scratchpad/admin_panel_verify.py` `_PRIV_OLMALI`/`_PRIV_OLMAMALI`da çivili.

⚠ İKİSİ DE `i18n_mw`DEN MUAF (main.py yol listesi) — TR/EN gövdeyi sunucuda `lang` ile
  üretiyorlar. Muafiyet düşerse `en()` kendi ürettiğimiz İngilizce gövdenin üstünden
  geçer. ⚠ `_PROFIL_MUAF`ta da olmalılar, yoksa girişli-ama-profili-eksik hekim ücretsiz
  araca giderken `/onay`a sapar (`/etkilesim`in ölçülmüş gerekçesi).

⚠ AUTOCOMPLETE SESSİZCE DEĞERSİZ OLABİLİR — ÖLÇÜLDÜ, GİZLENMEDİ: `/api/suggest` kimlik
  ister (`ref_routes.api_suggest` → oturumsuzda `[]`). Bu sayfa herkese açık olduğu için
  ÇIKIŞLI ziyaretçide öneri kutusu hiç açılmaz; alan düz metin/virgüllü olarak tam çalışır.
  Ucu anonime açmak AYRI bir karardır (anonim + DB'ye vuran uç = kendi IP kovası +
  kötüye kullanım freni ister) ve tek başıma verilmedi.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 78 |
| `_REN_SEM` | `threading.Semaphore(3)` | 82 |
| `_REF_SINIR` | `'</div>\n<script>'` | 139 |
| `_REF_CAPA` | `'<div id="out"></div>'` | 140 |
| `_RENAL_JS` | `"<script>\n(function(){\n var inp=document.getElementById('rc-d'), menu=document.getElementById('rc-menu'),\n     cips=d` | 368 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 85 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 93 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `_kabuk` | `(lang: str, T: dict, canonical: str, robots: str, d, govde: str) -> str` | 99 | head + CSS + ray + gövde. ⚠ EN'de `<html lang>` ELLE çevrilir: `head()` 'tr' |
| `_band` | `(T: dict, d, lang: str = 'tr') -> str` | 109 | Dönüşüm bandı — CTA auth-farkında (çıkışlı → `/kayit`, girişli → `/chat`). |
| `_ref_parcala` | `() -> tuple[str, str] \| None` | 143 | `_REF_BODY` → (HTML gövde, `<script>` bloğu). Beklenmedik biçimde `None`. |
| `_referans_bolumu` | `(lang: str, T: dict, d) -> str` | 169 | Hub'ın referans bölümü — çıkışlıda kilitli, girişlide tam gövde. |
| `recete_hub_page` 🌐 | `(request: Request)` | 218 | İki ücretsiz kontrol aracının giriş kapısı. Kayıt yok, kredi yok, LLM yok. |
| `sgk_odeme_page` 🌐 | `(request: Request)` | 255 | Ek-4/A liste araması — hub'ın üçüncü aracı. Kayıt yok, kredi yok, LLM yok. |
| `_tara` | `(request, ilaclar: list[str], egfr: float \| None, lang: str) -> dict` | 278 | Motoru fail-CLOSED çağırır. Her hata yolu `calistirilamadi` satırları döndürür. |
| `bobrek_doz_page` 🌐 | `(request: Request, egfr: str = '', d: str = '')` | 302 | Etiket-temelli böbrek doz kontrolü — kayıt gerekmez, kredi harcamaz, LLM yok. |

## `saglik/app/ref_body.py`

`427 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
/reference sayfa gövdesi (HTML + JS).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-07-29, mimari denetimi). İçerik
AYNEN taşındı — tek karakter değişmedi; render çıktısı bayt bayt doğrulandı.
main.py bu adları yeniden dışa aktarır, yani mevcut importlar (`from saglik.app.main
import ...`) ve doğrulama scriptleri KIRILMAZ.

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_REF_BODY` | `'\n<button class="sbtoggle" type="button" onclick="document.getElementById(\'refside\').classList.toggle(\'open\')" aria` | 13 |

## `saglik/app/ref_routes.py`

`344 satır` · `7 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
REFERANS + ÜCRETSİZ ARAÇ YÜZEYİ (Faz 8, 2026-08-01).

`main.py`den ayrıldı. Kapsam: 5 rota —
`/reference` · `/etkilesim` · `/hesaplayicilar` · `/api/query` · `/api/suggest`.

⚠⚠ BU MODÜL `main.py`DEN HİÇBİR ŞEY IMPORT ETMEZ (dairesel olur → uygulama AÇILMAZ).
  Ölçüldü (`scratchpad/faz8_kapsam_olc.py`): taşınan kodun main.py'de kalan tek
  bağımlılığı `app` nesnesiydi; o da `kur(app)` ile parametre olarak geliyor.
  Paylaşık yardımcı 0, arada yabancı rota 0.

⚠⚠ `include_router` KULLANILMAZ — bu FastAPI sürümünde TEMBEL: rotaları `app.routes`a
  düzleştirmiyor (ölçüldü Faz 2b: 95 rota → 83). Ürün çalışmaya devam eder ama rota
  envanterini `app.routes`tan okuyan güvenlik ağları SESSİZCE körelir. Bunun yerine
  `@_rota` KAYIT listesi + `kur(app)` → `app.get/post` public API'si.

⚠ `kur(app)` ÇAĞRISININ YERİ SÖZLEŞMENİN PARÇASI: main.py'de blok tam nerede
  duruyorsa oraya konur. Ölçüldü — bu rotalar `app.routes`ta 51-55; komşuları
  50 (`DELETE /api/calendar/{aid}`) ve 56 (`/cases`).

────────────────────────────────────────────────────────────────────────────────
BU DOSYADAKİ İNVARİANTLAR — hepsi ölçülmüş hatalardan doğdu, BOZMA
────────────────────────────────────────────────────────────────────────────────
⚠⚠ `/api/query` ÜÇ SATIRLIK ÇÖKME FRENİ (`len(q.strip()) < 3 or not any(isalnum)`):
  `"%"` ya da 1-2 karakterlik sorgu `core.drug` (433.804 satır) üzerinde GIN indeksini
  devre dışı bırakıp **Postgres'i SEGFAULT ettiriyordu — TÜM veritabanı düşer.**
  Tetikleyici dışarıdan gelen egzotik girdi DEĞİL: ürünün KENDİ autocomplete'i
  üretebiliyor (36 TİTCK markası `%` ile başlıyor). Kapısı:
  `scratchpad/api_giris_freni_verify.py` (CI SUITES'te; frenin ÖNÜNDE/ARKASINDA
  `route()` çağrılıp çağrılmadığını saplamayla ölçer).

⚠ `/hesaplayicilar` ve `/etkilesim` `i18n_mw`'den MUAFTIR. Muafiyet BU DOSYADA DEĞİL,
  `main.py`de yol listesindedir (`i18n_mw` + `_PROFIL_MUAF`). Dil `CALC_LANG`/`REF_LANG`
  ile JS'te üretilir — EN_PAIRS apostrof tuzağından korunmak için. Muafiyet düşerse
  o sayfaların TÜM JS'i sessizce çöker.

⚠⚠ `/hesaplayicilar` ÜCRETLİ GOOGLE REKLAM İNİŞİDİR. `head(...)` çağrısı `?c=`
  çözümünden SONRA gelir — önce çağrılsaydı başlık/açıklama/canonical geç kalır ve
  sayfa JENERİK başlıkla çıkardı (Google 15 kelimeyi tek sayfa sayardı; o fatura bir
  kez ödendi). Sıra SESSİZCE bozulabilir: bugün bunu tutan bir test YOK
  (mutasyonla ölçüldü — `if c:` → `if False:` 7 süiti de yeşil bıraktı).

⚠ `/api/suggest` AYRI HIZ KOVASI (`sg:` öneki + `SUGGEST_RATE_MAX`): aynı kovayı
  kullansaydı 8 harflik bir arama hekimin SOHBET hakkını tüketirdi.

⚠ Kimliksiz davranış BİLEREK farklı: `/api/query` → 401 + İKİ DİLLİ metin (API JSON'u
  `i18n_mw`den GEÇMEZ) · `/api/suggest` → sessiz `[]` (konsol temiz kalsın).

⚠ `/etkilesim` HERKESE AÇIK + FAIL-CLOSED: hiçbir durumda yeşil/aklama dili YOKTUR
  (yasak kelimeler CI'da taranır). Dürüstlük katmanının tamamı `etkilesim_body.py`de.

⚠ `translate_result` dalında FAIL-OPEN DOĞRUDUR: çeviri hatasında `tr_map={}` ile
  ORİJİNAL gösterilir. Bu bir klinik hüküm değil SUNUM katmanıdır — güvenlik
  katmanlarındaki "fail-open YASAK" kuralı buraya uygulanmaz.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 82 |
| `_ETK_SEM` | `threading.Semaphore(3)` | 130 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(metod: str, yol: str, **kw)` | 85 | `@app.get(...)`in yerini tutar — `app` bu modülde YOK (dairesel olurdu). |
| `kur` | `(app) -> None` | 96 | Bu modülün rotalarını uygulamaya bağlar. main.py TEK KEZ çağırır. |
| `reference_page` 🌐 | `(request: Request)` | 118 |  |
| `etkilesim_page` 🌐 | `(request: Request, a: str = '', b: str = '', d: str = '')` | 134 | Etiket-temelli ilaç etkileşimi taraması — kayıt gerekmez, kredi harcamaz. |
| `calculators_page` 🌐 | `(request: Request, c: str = '')` | 158 | Klinik hesaplayıcılar — istemci-tarafı, ücretsiz (kredi harcamaz). HERKESE AÇIK (2026-07-19): |
| `api_query` 🌐 | `(request: Request, q: str = '')` | 270 |  |
| `api_suggest` 🌐 | `(request: Request, q: str = '')` | 322 | Referans arama autocomplete — ilaç adı önerileri (aktif segment prefiksi). SALT DB, LLM/kota YOK. |

## `saglik/app/sgk_ara.py`

`393 satır` · `7 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
SGK BEDELİ ÖDENECEK İLAÇLAR LİSTESİNDE ARAMA (founder 2026-08-19).

İki yüzey, TEK motor ve TEK uç:
  · `/sgk-odeme`            — Reçete kontrolü hub'ının üçüncü aracı (herkese açık, kayıtsız)
  · `/reference` içine gömülü arama kutusu (`arama_bloku`) — founder: "referansın içine
    direkt SGK Bedeli Ödenecek İlaçlar araması koyalım"
  · `GET /api/sgk-ara?q=`   — ikisinin de kullandığı JSON ucu

⚠ LLM YOK · kredi 0 · ağ YOK · kayıt gerekmez (`/etkilesim` · `/bobrek-doz` sınıfı).

⚠⚠ "SGK ÖDER" DEMEZ — sözleşme `saglik/cds/sgk.py` başlığında, buraya KOPYALANMAZ.
   Ek-4/A rapor/endikasyon/branş koşulu İÇERMEZ; çıktı "liste şunu gösteriyor + yürürlük
   tarihi". Garanti dili kapısı: `scratchpad/sgk_odeme_verify.py` (bu dosyayı da tarar).

⚠⚠ ÇIPLAK SAYFADA REÇETELİ MOLEKÜL ADI YASAK (`inis_kaynak_beyani_verify` §7 — 2026-08-17'de
   canlıda yakalandı): bu sayfanın STATİK metninde örnek ilaç adı YAZILMAZ. Sonuçlar
   kullanıcının yazdığı sorgudan doğar, statik metin değildir.

⚠ ÇÖKME FRENİ: `/api/query`nin üç satırlık freninin aynısı (<3 karakter / alfanümerik yok
   → 400). Gerekçe orada ölçülmüş: `%` ya da 1-2 karakterlik sorgu GIN indeksini devre dışı
   bırakıp Postgres'i SEGFAULT ettiriyordu — TÜM veritabanı düşer.

⚠ Bu modül `main.py`den ve `recete_routes`tan HİÇBİR ŞEY import ETMEZ (dairesel olur);
   `recete_routes` BENİ import eder. `sgk_case`ten YALNIZ ölçülmüş iki yardımcı okunur
   (o da beni import etmez) — mantık İKİ KEZ YAZILMAZ, çünkü o iki fonksiyonun her satırı
   ölçülmüş bir hatadan doğdu (etken madde↔ürün köprüsü · Türkçe İ · kombinasyon önceliği).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 40 |
| `_SGKA_SEM` | `threading.Semaphore(3)` | 41 |
| `_M` | `{'tr': {'h': 'SGK Bedeli Ödenecek İlaçlar Listesi (Ek-4/A)', 'ph': 'İlaç ya da etken madde adı yazın…', 'ara': 'Ara', 'b` | 56 |
| `_STIL` | `'<style>\n.sgka{background:#fff;border:1px solid var(--line);box-shadow:var(--sh-surface);\n  border-radius:var(--r-card` | 236 |
| `_JS` | `'<script>\n(function(){\n var q=document.getElementById(\'sgka-q\'), b=document.getElementById(\'sgka-go\'),\n     out=d` | 311 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(yontem: str, yol: str, **kw)` | 44 |  |
| `kur` | `(app) -> None` | 51 |  |
| `_t` | `(lang: str) -> dict` | 114 |  |
| `_ara` | `(q: str, tavan: int = 25) -> dict` | 118 | Ürün adı VE etken madde üzerinden Ek-4/A araması. |
| `_bos_durum` | `(surum, markalar: list[str], olcum: dict) -> str` | 181 | Boş sonucun ÜÇ nedeninden hangisi (`cds/sgk.py` ile AYNI sözcükler). |
| `async` `api_sgk_ara` 🌐 | `(request: Request, q: str = '')` | 203 | Ek-4/A araması. Herkese açık, kredi 0, LLM yok. |
| `arama_bloku` | `(lang: str, baslik: bool = True) -> str` | 280 | Gömülebilir arama kutusu (referans sayfası + `/sgk-odeme` aynı parçayı kullanır). |

## `saglik/app/sgk_case.py`

`641 satır` · `16 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
HASTA KARTI · SGK GERİ ÖDEME KONTROLÜ — AYRI DÜĞME (founder 2026-08-19).

Reçete kontrolü sekmesindeki dört kontrolün (etkileşim · böbrek · alerji · duplikasyon)
YANINDA, KENDİ düğmesiyle çalışan beşinci kontrol: hastanın ilaç listesindeki her ürün
SGK Ek-4/A "Bedeli Ödenecek İlaçlar Listesi"nde görünüyor mu?

⚠⚠ NEDEN AYRI DÜĞME (founder kararı, "sgk kontrol butonunu ayrı koyalım"): dört kontrol
   KLİNİK GÜVENLİK sorusudur (hasta zarar görür mü); SGK sorusu MALİ/İDARİdir (hasta
   ödeyecek mi). Tek düğmeye katmak ikisini aynı ağırlıkta gösterirdi — hekim güvenlik
   bulgusunu idari satırın arasında kaybedebilir. Ayrıca bu kontrol farklı bir kaynağa
   (tebliğ eki) ve farklı bir tazelik rejimine (haftalık cron) bağlıdır.

⚠⚠ "SGK ÖDER" DEMEZ — sözleşmenin tamamı `saglik/cds/sgk.py` başlığında; buraya
   KOPYALANMAZ. Ek-4/A rapor/endikasyon/branş koşulu İÇERMEZ; çıktı daima "liste şunu
   gösteriyor + yürürlük tarihi". Kapı `scratchpad/sgk_odeme_verify.py` garanti dilini
   iki yönlü tarar (bu dosya da taranır).

⚠ ÜÇ DURUM AYRI ÇİZİLİR (`listede_var` · `listede_yok` · `eslestirilemedi`): üçüncüsünü
  ikinciye katlamak hekime YANLIŞ OLUMSUZ hüküm göstermektir.
⚠ LLM YOK · kredi 0 · ağ YOK (yalnız `core.sgk_odeme` okunur).
⚠ Bu modül `main.py`den ve `cases_routes`tan HİÇBİR ŞEY import ETMEZ (dairesel olur);
  `recete_case` BENİ import eder — ters yön YASAK.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KAYIT` | `[]` | 37 |
| `_SGK_SEM` | `threading.Semaphore(3)` | 41 |
| `_METIN` | `{'tr': {'go': 'SGK geri ödeme kontrolü', 'calisiyor': 'Kontrol ediliyor…', 'baslik': 'SGK Bedeli Ödenecek İlaçlar Listes` | 56 |
| `_ADAY_TAVAN` | `20000` | 150 |
| `_JOKER` | `'iıİIoöOÖuüUÜsşSŞgğGĞcçCÇaâAÂ'` | 155 |
| `_MARKA_TAVAN` | `400` | 164 |
| `_EK4A_PARCA` | `80` | 165 |
| `_JS` | `"<script>\n(function(){\n var b=document.getElementById('sgkc-go'), out=document.getElementById('sgkc-sonuc');\n if(!b\|` | 567 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_rota` | `(yontem: str, yol: str, **kw)` | 44 |  |
| `kur` | `(app) -> None` | 51 |  |
| `_t` | `(lang: str) -> dict` | 84 |  |
| `buton` | `(cid: int, lang: str, kapali: bool = False) -> str` | 88 | Reçete kontrolü panelinin ALTINA eklenen ayrı düğme + kendi sonuç bölgesi. |
| `async` `api_sgk_kontrol` 🌐 | `(cid: int, request: Request)` | 112 | Hastanın ilaç listesindeki ürünlerin Ek-4/A liste durumu. Kredi 0, LLM yok. |
| `_sgk_kontrol_sync` | `(request: Request, cid: int)` | 124 |  |
| `_kelime_basinda` | `(hedef: str, metin: str) -> bool` | 182 | `hedef`, `metin` içinde bir kelimenin BAŞINDA geçiyor mu (ek serbest). |
| `_ayracsiz` | `(s: str) -> str` | 192 | `_cf` + harf/rakam disi her sey atilir. Ayrac farki eslesmeyi BOZMASIN diye. |
| `_kok_jenerikleri` | `(conn, kok: str, tavan: int = 200) -> set[str]` | 198 | Bir MARKA KOKUNU TİTCK jeneriklerine çözer. |
| `_kok_esles` | `(a: str, b: str) -> bool` | 235 | İki marka kökü aynı ürün ailesini mi gösteriyor (ayraçsız, ÖNEK, iki yönlü). |
| `_base_brand_yerel` | `(b: str)` | 250 |  |
| `_jenerik_ortusuyor` | `(a: set[str], b: set[str]) -> bool` | 255 | İki jenerik kümesi aynı molekülü mü gösteriyor (ek/tuz serbest). |
| `urun_yolu_suz` | `(conn, sorgu: str, adlar: list[str]) -> list[str]` | 264 | Ek-4/A ÜRÜN ADI yolundan gelen satırları süzer: kelime-ORTASI eşleşmeler ancak |
| `_titck_markalari` | `(conn, etken: str, tavan: int = _MARKA_TAVAN, *, olcum: dict \| None = None) -> list[str]` | 317 | Etken maddenin TÜRKİYE'DEKİ ÜRÜN adları (`core.drug`, TİTCK satırları). |
| `_ek4a_ara` | `(conn, markalar: list[str], tavan: int = 6, *, olcum: dict \| None = None) -> list[dict]` | 418 | Marka listesinin TAMAMINI Ek-4/A'da TEK sorguda arar. |
| `_hesapla` | `(ctx: dict, lang: str) -> dict` | 482 | Ağır iş THREADPOOL'da (olay döngüsü bloklanmaz — deponun kuralı). |

## `saglik/app/sohbet_baslik.py`

`656 satır` · `20 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
#736c · OTOMATİK SOHBET BAŞLIĞI — hekimin kendi el yazısındaki gibi klinik kısaltma.

Founder (2026-09-06): *"sohbetin başlığına da hekimin anlayacağı bir kısaltma üretelim,
hastayı bulması kolay olan; biz üretelim, hekim yeni isim verene kadar."*
Örnek: `64K · HT 180/92 · takipne` · `6E · OSB · irritabilite` · `Warfarin + amiodaron · INR`.

⚠⚠ **LLM YOK, 0 MALİYET.** Başlık tamamen deterministik: yaş/cinsiyet deseni · ilaç adı
(KB'den, `engine._drugs_in_query` SALT OKUMA) · kavram (kapalı kısaltma sözlüğü) · sayısal
bulgu (TA/eGFR/HbA1c/INR/ateş…). Bir tur için ek LLM çağrısı YAPILMAZ.

⚠⚠ **ÜZERİNE YAZMA KURALI — `app.thread.title_auto`:**
  · `create_thread` yeni thread'i `title_auto=true` açar → otomatik başlık YAZILABİLİR.
  · `rename_thread` `title_auto=false` yapar → o thread'e bir daha ASLA dokunulmaz.
  · **Eski thread'lerde kolon NULL** ve NULL "yazılabilir" SAYILMAZ (`IS TRUE` şartı):
    geçmişi toplu yeniden adlandırmak hekimin bildiği listeyi bir gecede değiştirirdi.
  ⚠ `IS TRUE` yazımı ŞART; `= true` NULL'da NULL döner ama `WHERE`de aynı davranır —
    yine de niyeti okunur kılmak için `IS TRUE` kullanılır ve kapı bunu ölçer.

⚠⚠ **BOŞ BAŞLIK > YANLIŞ BAŞLIK.** Klinik parça çıkmazsa `uret()` **None** döner ve mevcut
başlık (ilk soru / "Yeni sohbet") AYNEN kalır. Fragman, çip yankısı ("🚫 Gebelik: DIŞLANDI"),
selamlama gibi girdilerde kısaltma ÜRETİLMEZ — uydurma bir kısaltma hekime yanlış hastayı
açtırır ve bu, boş başlıktan pahalıdır.

⚠ **KAVRAM SÖZLÜĞÜ KAPALIDIR VE ÖYLE KALMALI.** "Sorudaki en uzun kelimeyi al" gibi bir
sezgisel, hekimin kendi yazdığı hasta adını ya da alakasız bir kelimeyi başlığa taşırdı.
Sözlükte olmayan konu = kavramsız başlık (ya da başlık yok) — kapsam ÖLÇÜLÜR, uydurulmaz.

⚠ **KVKK — `redact_pii` UYGULANMIYOR, GEREKÇE:** başlık soru metninin türevidir ve
`thread.title` bugün zaten ham sorunun ilk 80 karakterini tutuyor; ürettiğimiz başlık ondan
**daha az** şey açığa çıkarır çünkü içeriği BEYAZ LİSTEDEN gelir (yaş · cinsiyet · KB'den
çözülmüş ilaç adı · kapalı sözlükten kavram · sayısal bulgu) — serbest metin span'i
kopyalanmaz, dolayısıyla hasta adı yapısal olarak GİREMEZ. Tek istisna hekimin KENDİ kart
etiketidir (08.05'ten beri gerçek ad girilebiliyor): o zaten hekimin kendi verisi ve sohbet
seçicisinde ekranda duruyor → yeni bir veri sınıfı doğmuyor. `redact_pii` bu kısa, beyaz
listeli dizgede yalnız zarar verirdi (ör. "64K"yı bozabilecek maskeleme) ve yokluğu bir
boşluk DEĞİL, ölçülmüş bir karardır.

⚠ Yazma **best-effort + SAVEPOINT** (`soru_kayit` deseni): başlık patlarsa yanıt/akış/para
yolu ETKİLENMEZ. ⚠ SAVEPOINT'in İÇİNDE `commit()` çağrılmaz (psycopg3 yasaklar) — commit
SAVEPOINT'ten sonra atılır.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `MAX_KRK` | `40` | 48 |
| `ILAC_TIMEOUT_MS` | `1500` | 49 |
| `AYRAC` | `' · '` | 50 |
| `_MAX_ILAC` | `2` | 51 |
| `_ESLEME` | `{'ı': 'i', 'İ': 'i', 'I': 'i', 'ş': 's', 'Ş': 's', 'ğ': 'g', 'Ğ': 'g', 'ü': 'u', 'Ü': 'u', 'ö': 'o', 'Ö': 'o', 'ç': 'c',` | 58 |
| `_YAS` | `re.compile('\\b(\\d{1,3})\\s*(?:yasinda\|yasindaki\|yas\|y)\\b\|\\b(\\d{1,3})[- ]?(?:year[- ]?old\|yo)\\b')` | 80 |
| `_AY` | `re.compile('\\b(\\d{1,2})\\s*(?:aylik\|ayl[ıi]k\|ay)\\b')` | 82 |
| `_KADIN` | `re.compile('\\b(kadin\|bayan\|kiz\|hanim\|female\|woman\|girl)\\b')` | 83 |
| `_ERKEK` | `re.compile('\\b(erkek\|bay\|oglan\|male\|man\|boy)\\b')` | 84 |
| `_KAVRAM` | `[('akut bobrek hasari', 'ABH'), ('kronik bobrek hastaligi', 'KBH'), ('bobrek hasari', 'ABH'), ('akut koroner sendrom', '` | 89 |
| `_TAM_ESLESME` | `frozenset({'odem'})` | 154 |
| `_KAVRAM_EN` | `[('acute kidney injury', 'AKI'), ('kidney injury', 'AKI'), ('acute coronary syndrome', 'ACS'), ('atrial fibrillation', '` | 157 |
| `_YAS_SONRA` | `'(?!\\s*(?:yasinda\|yasindaki\|yas\\b\|y\\b\|aylik\|year\|yo\\b))'` | 177 |
| `_BULGU` | `[(re.compile('\\b(\\d{2,3})\\s*/\\s*(\\d{2,3})\\b'), lambda m: f'TA {m.group(1)}/{m.group(2)}'), (re.compile('\\begfr\\D` | 178 |
| `_TICARI` | `re.compile('\\bfiyat\\w*\|\\bucret\\w*\|\\bsirket\\w*\|\\bfirma\\w*\|\\bhisse\\w*\|\\bpatent\\w*\|\\breklam\\w*\|\\bkamp` | 281 |
| `_NOTR_EK` | `frozenset({'HCI', 'HCL', 'HBR', 'HIDROKLORUR', 'HİDROKLORÜR', 'HYDROCHLORIDE', 'HYDROBROMIDE', 'MONOHIDRAT', 'MONOHİDRAT` | 418 |
| `_KIMLIK_TUZ` | `frozenset({'sulfat', 'sulfate', 'sitrat', 'citrate', 'glukonat', 'gluconate', 'klavulanat', 'clavulanate', 'karbonat', '` | 425 |
| `_ROMEN` | `re.compile('^[ivx]+$')` | 431 |
| `_TUZ_SON` | `re.compile('(?:at\|ur\|ate\|ide\|id\|it)$')` | 434 |
| `_NOTR_KATLI` | `frozenset((katla(x) for x in _NOTR_EK))` | 435 |
| `ILAC_BUTCE` | `26` | 459 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `katla` | `(s: str) -> str` | 62 | Küçük harf + aksansız, **karakter sayısı DEĞİŞMEDEN** (indeksler kaysın istemiyoruz). |
| `_hasta` | `(nq: str, lang: str) -> str \| None` | 192 | `64K` / `6E` / `3 aylık` — cinsiyet yoksa yalnız yaş, yaş yoksa None. |
| `_ad_parcasi_mi` | `(orijinal: str, konum: int) -> bool` | 220 | Eşleşen token bir ÖZEL AD parçası mı ("Deniz **Ateş**")? — SEZGİSEL, sınırı yazılı. |
| `_kavram` | `(nq: str, lang: str, orijinal: str = '') -> list[str]` | 238 | Soruda geçen kavramlar — **KONUM SIRASINDA** (tablo sırası DEĞİL), tekilleştirilmiş. |
| `_klinik_baglam` | `(nq: str) -> bool` | 286 | İlaç TEK PARÇAYKEN başlık kurulabilir mi? (ticari metinse HAYIR) |
| `_bulgu` | `(nq: str) -> str \| None` | 291 |  |
| `_sigdir` | `(parcalar: list[str]) -> str` | 299 | Parçaları sırayla ekle; TAVANI AŞANI ATLA (kırpma YOK — yarım kısaltma yanıltır). |
| `uret` | `(soru: str, lang: str = 'tr', kart_etiketi: str \| None = None, ilaclar: list[str] \| None = None) -> str \| None` | 311 | Klinik kısaltma başlığı — üretilemiyorsa **None** (çağıran mevcut başlığı KORUR). |
| `_kart_etiketi` | `(conn, doctor_id: int, case_id) -> str \| None` | 362 |  |
| `_ilaclar` | `(conn, soru: str) -> list[str]` | 370 | KB'den çözülmüş ilaç adları (SALT OKUMA). ⚠ Kendi eşleştirmemi YAZMIYORUM: |
| `_ad_sadelestir` | `(ad: str) -> str` | 438 | `AMIODARONE HCI` → `Amiodarone`; `MAGNEZYUM SÜLFAT` → `Magnezyum sülfat` (tuz KALIR). |
| `_ad_kisalt` | `(ad: str) -> str` | 462 | Uzun jenerik adı başlığa sığacak KISA biçime indir: **taban + kimlik tuzu**. |
| `_ilac_metni` | `(ilac: list[str]) -> str \| None` | 480 | İlaç parçası: TAM ad sığıyorsa tam, sığmıyorsa KISA biçim (tekilleştirilmiş). |
| `_soru_adaylari` | `(soru_n: str) -> set[str]` | 500 | Sorunun sözcükleri + TR→EN ilaç köprüsünden gelen adayları (SALT OKUMA). |
| `_gecti_mi` | `(aday: set[str], tok: str) -> bool` | 517 | Token soruda GEÇİYOR mu — birebir ya da ≥6 harflik ortak önek (çekim/yazım payı). |
| `hekim_adlandirdi_mi` | `(soru_n: str, ad: str, kart: dict \| None = None, aday: set[str] \| None = None) -> bool` | 527 | ⚠⚠ TEK İNVARYANT (Fırat P1b, 2026-09-06): **başlığa giren ilacı hekim ADLANDIRMIŞ |
| `_adlandirilan_etken` | `(aday: set[str], ad: str) -> str` | 560 | Çok etkenli kombinasyonda **hekimin YAZDIĞI** etkeni başlığa koy. |
| `_tuz_uyusuyor_mu` | `(soru_n: str, adlar: list[str]) -> list[str]` | 581 | Çözülen jenerikleri SORUNUN TUZUYLA karşılaştır — uyuşmayan ad BAŞLIĞA GİRMEZ. |
| `arka_planda_yaz` | `(thread_id, doctor_id, soru: str, case_id = None, lang: str = 'tr') -> None` | 605 | Başlığı YANIT YOLUNUN DIŞINDA yaz — çağıran HİÇ beklemez (P1, 2026-09-06). |
| `yaz_sessiz` | `(conn, thread_id, doctor_id, soru: str, case_id = None, lang: str = 'tr') -> str \| None` | 630 | Otomatik başlığı yaz — HER HATA YUTULUR (yanıt yolu bundan etkilenmez). |

## `saglik/app/soru_analiz.py`

`531 satır` · `14 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
#715b K2 · TÜRETME — `app.question_insight` (SPEC: docs/soru-analitigi-spec-2026-09-04.md).

Founder 2026-09-04: *"hangi branşlar neler soruyor. Bizim alameti farikamız bu."*
Bu modül K1'in (kayıt) ve K3'ün (panel) ARASINDAKİ katmandır: `app.message`teki ham soruyu
**türetilmiş, sorgulanabilir** bir satıra çevirir. Yazma ucu Cahit'te (`soru_kayit.py`),
panel Kamil'de; burada YALNIZ türetme vardır.

⚠⚠ **TÜRETİLMİŞ VERİ ASLA TEK KAYNAK DEĞİLDİR.** `app.question_insight` silinip
`app.message`tan yeniden üretilebilir ve taksonomi geliştikçe yeniden ÜRETİLİR — bu yüzden
her satır `uretim_surumu` taşır ve batch **idempotent**tir (aynı sürümde iki koşum aynı
sonucu verir; kapı şartı). Bir sayı burada "kaynak" gibi okunursa taksonomi borcunu
dondurmuş oluruz.

⚠⚠ **`app.query_stat`a ve `store._stat_topic`e DOKUNULMAZ.** O tablo bilinçli olarak
ANONİM (doctor_id ve ham metin YOK) ve bir regresyon frenidir; bu iş `app.message` üzerinden
gider. `_stat_topic`in kusuru — İLK EŞLEŞENİ alıp durması, "gebede warfarin dozu"nu tek
kutuya düşürmesi — burada TEKRARLANMAZ: `konu` **ÇOK ETİKETLİDİR** ve eşleşen HER etiket
yazılır. Aynı sebeple o listeyi import etmiyorum: paylaşılan liste, donmuş olması gereken
freni benim taksonomi işim yüzünden hareket ettirirdi.

⚠ **KLİNİK METİN YAZILMAZ.** Bu tabloya soru/yanıt metni, hasta verisi, alıntı GİRMEZ —
yalnız etiketler, sayılar, bayraklar ve KENDİ KB'mizin adları (ilaç jeneriği, ICD bölümü).
`uzunluk` karakter SAYISIDIR, metnin kendisi değil.

ÜÇ DURUM DİSİPLİNİ: ölçülemeyen alan **NULL** kalır, `False`/`0` YAZILMAZ. `kanit_n = 0`
"paket boştu" demektir; `NULL` "künye yok, ölçemedik" demektir (K1 öncesi mesajların tamamı
böyledir). İkisini birleştirmek paneli fail-open yapardı.

SINIFLANDIRMA ÜÇ KADEME (ucuzdan pahalıya, SPEC bölüm 3):
  1. **Deterministik, 0 maliyet** — kendi KB'miz sınıflandırıcıdır: ilaç adları
     `engine._drugs_in_query`, ICD bölümü `retrieval` TR terim köprüsü, soru tipi regex ailesi.
  2. **ARTIK ÖLÇÜLÜR** — kuralla etiketlenemeyen satır oranı RAPOR EDİLİR (`rapor()`),
     `"diger"` kutusuna SÜPÜRÜLMEZ. Kapanmayan artık taksonomi borcudur.
  3. **LLM yalnız artık için** — `llm_istem_paketi()` istemi HAZIRLAR, çağrıyı YAPMAZ;
     koşturma founder onaylı ayrı adımdır (`yontem` kolonu kural/llm ayrımını taşır).
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `URETIM_SURUMU` | `1` | 44 |
| `KONU_TAKSONOMISI` | `[('etkilesim', ('etkiles', 'interaksiyon', 'birlikte kullan', 'kombinasyon', 'beraber kullan')), ('gebelik-laktasyon', (` | 57 |
| `SORU_TIPLERI` | `[('doz', ('doz', 'mg/kg', 'titrasyon', 'azaltmali', 'kac mg', 'kac tablet', 'doz ayar', 'maksimum doz')), ('etkilesim', ` | 111 |
| `ICD_BOLUM` | `{'1': 'enfeksiyon-parazit', '2': 'neoplazi', '3': 'kan-hematopoetik', '4': 'bagisiklik', '5': 'endokrin-metabolik', '6':` | 136 |
| `_KATLA` | `str.maketrans('çğıöşüâîû', 'cgiosuaiu')` | 147 |
| `_ISKELET_TR` | `('kısa yanıt', 'gerekçe', 'dikkat', 'pratik öneri')` | 148 |
| `_ISKELET_EN` | `('bottom line', 'rationale', 'caution', 'practical guidance')` | 149 |
| `_SINIR_ONCE` | `'(?<![a-z0-9])'` | 163 |
| `_KONU_DESEN` | `None` | 189 |
| `_TIP_DESEN` | `None` | 190 |
| `_SATIR_SQL` | `"\nSELECT m.id, m.content, m.created_at, m.mode, t.case_id,\n       d.specialty, d.unvan, d.city,\n       (SELECT a.cont` | 295 |
| `_UPSERT` | `'\nINSERT INTO app.question_insight\n (message_id, soruldu_at, brans, unvan, sehir, konu, icd_bolum, soru_tipi, ilaclar,` | 353 |
| `_LLM_ISTEM_BASI` | `'Aşağıdaki klinik soruları SADECE verilen etiket kümesinden etiketle. Her soru için 0-3 etiket ver, JSON dizi döndür, aç` | 478 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_desen` | `(kw: str)` | 166 | Anahtardan kelime-sınırlı desen üretir. |
| `_normalize` | `(s: str) -> str` | 182 | Karşılaştırma için katla: küçült + U+0307 at + Türkçe harfleri ASCII'ye indir. |
| `konular` | `(soru: str) -> list[str]` | 193 | ÇOK ETİKETLİ konu listesi — eşleşen HER etiket döner (ilk eşleşen DEĞİL). |
| `soru_tipleri` | `(soru: str) -> list[str]` | 206 | Soru tipi — ÇOK ETİKETLİ (gerekçe: `SORU_TIPLERI` başlığı). |
| `icd_bolumleri` | `(conn, soru: str) -> list[str]` | 215 | Sorguyu ICD-11 BÖLÜMLERİNE indirger (LLM'siz, mevcut TR terim köprüsüyle). |
| `ilaclar` | `(conn, soru: str, tavan: int = 5) -> list[str]` | 249 | Sorudaki ilaçların JENERİK adları (KB'den çözülür; hasta metni DEĞİL). |
| `_iskelet_tam` | `(yanit: str, dil: str \| None) -> bool \| None` | 270 | Yanıt `DOCTOR_SYSTEM` iskeletini taşıyor mu (Kısa yanıt/Gerekçe/Dikkat/Pratik öneri). |
| `_dil` | `(soru: str, yanit: str \| None) -> str \| None` | 283 | Sorunun dili — ürünün kendi LLM'siz tespitini kullanır (`cds.dil.yanit_dili`). |
| `satir_uret` | `(conn, ham: tuple, ilac: bool = True) -> dict` | 320 | Tek bir kullanıcı mesajından türetilmiş satır sözlüğü (SAF: DB'ye yazmaz). |
| `_yaz` | `(conn, satir: dict) -> bool` | 375 | Tek satırı yazar; YAZILAMAZSA batch'i DÜŞÜRMEZ (döner `False`). |
| `uret` | `(conn, limit: int \| None = None, yeniden: bool = False, ilac: bool = True) -> dict` | 400 | Batch türetme. Döner: {'bakilan','yazilan','atlanan','artik','artik_orani'}. |
| `rapor` | `(conn) -> dict` | 445 | ARTIK ORANI VE KAPSAM RAPORU — gizlenmez, `"diger"` kutusuna süpürülmez (SPEC 3.2). |
| `llm_istem_paketi` | `(sorular: list[str], parti: int = 40) -> list[str]` | 485 | Artık sorular için toplu istem metinleri (ÇAĞRI YAPILMAZ, yalnız metin üretilir). |
| `main` | `() -> int` | 514 |  |

## `saglik/app/soru_kayit.py`

`357 satır` · `14 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
#715b K1 · KAYIT — katman sonucu + kanıt paketi künyesi (SPEC: docs/soru-analitigi-spec-2026-09-04.md).

Founder 2026-09-04: *"soru ve cevapları detaylı kayıt yapabileceğimiz bir sistem kur… hangi
branşlar neler soruyor. Bizim alameti farikamız bu."* Bu modül o sistemin YAZMA ucudur;
türetme (K2, `app.question_insight`) ve panel (K3) AYRI sahiplerdedir.

NE YAZAR — İKİ ŞEY, İKİSİ DE KLİNİK METİNSİZ:
  · `app.answer_layer`  = güvenlik/yardımcı katmanın SONUÇ SINIFI (katman ne YAPTI).
    ⚠⚠ Katmanın ÜRETTİĞİ METİN YAZILMAZ (founder: "kimliksiz özet"). Bugün 501 redflag /
    287 dose_check / 27 critic … çağrısının çıktısı HİÇBİR YERDE yok (#707b) → bu üründeki
    en tehlikeli kusur sınıfı ("yanlış güven") geriye dönük ölçülemiyordu.
  · `app.evidence_doc`  = MODELE GİDEN belgelerin künyesi (KB kimliği + başlık + sıra).
    ⚠⚠ ALINTI ve LİSANS BİLEREK YOK — `chat._source_docs_from`un 2026-08-02 kararı
    (pasajı süresiz saklama) AYAKTA; founder'ın onayladığı KİMLİK+BAŞLIK+SIRA'dır.

ÜÇ DURUM, İKİ DEĞİL: `tetiklendi` NULL = ÖLÇÜLEMEDİ. "Çalıştırılamadı ≠ temiz" bu deponun
invariantı; `unavailable`ı `False`a ezmek paneli fail-open yapardı. Şema CHECK'i de yasaklar.

⚠⚠ YAZMA ASLA ÜRÜNÜ BOZMAZ:
  · her yazma `with conn.transaction():` SAVEPOINT'i içinde (Postgres'te başarısız TEK ifade
    transaction'ı zehirler → sarmalanmazsa AYNI bağlantıdaki `record_usage` da düşer ve
    kaçak ÖLÇÜLEMEZ hâle gelir — ölçülmüş sınıf, `cases_api_verify` bölüm 14);
  · `commit()` SAVEPOINT'in DIŞINDA (psycopg3 Transaction context içinde commit'i YASAKLAR
    ve yazma SESSİZCE düşer);
  · hata YUTULUR ama SESSİZ DEĞİL (log satırı) ve dönüş değeri yazılan satır sayısıdır —
    çağıran "kaydedildi" diye bir şey İDDİA ETMEZ, yalnız ölçüm için okur;
  · kill-switch `KLIVANCE_SORU_KAYIT=0` → hiçbir şey yazılmaz, ürün aynen çalışır.

⚠ `store.py`ye DOKUNULMADI (eş zamanlı sahip) ve `app.query_stat` ANONİM KALIR — bu modül
  ona hiç dokunmaz; yeni iş `app.message` üzerinden gider.

────────────────────────────────────────────────────────────────────────────────────────
ÇAĞRI YERLERİ — GEREKÇE BURADA, `chat_routes`ta YALNIZ İŞARET DURUR
⚠ Bu prosa 2026-09-04'te `chat_routes.py`den BURAYA TAŞINDI (Ömer kararı: o dosyanın tavan
  kaydında "çırcır borcu" var, bir sonraki büyümede BÖLME konuşulacak). SİLİNMEDİ, taşındı —
  gerekçesi kaybolan satır bir sonraki oturumda "bu ne işe yarıyor" denip DÜŞÜRÜLÜR.

1. `/api/chat` akışı, `add_message`ten SONRA (`_ndjson`) — `map` katmanı + kanıt künyesi.
   `_mid` = `son_mesaj_id`. ⚠ REDDEDİLEN turda mesaj YOKTUR → `_mid` None: künye yazılmaz
   (bağsız künye hiçbir soruyu cevaplamaz), katman satırı bağsız YAZILIR — sayı ölçülür,
   "hangi soru" cevapsız kalır ve bu GİZLENMEZ.
2. AKIŞ İPTALİ (`_settle_abort`) — `final` YOKTUR, elde yalnız yanan çağrılar (`calls_partial`)
   vardır. Yazılmazsa "ek çıkarımı koştu ama tur bitmedi" turu ölçümde HİÇ GÖRÜNMEZ; gerekçe
   `record_usage`ın iptalde de yazılmasıyla aynı: **ölçülemeyen kaçak yok sayılır.**
   `tetiklendi` NULL kalır (katman koştu, bulgusu bilinmiyor — `False` "bulgu yok" demek olurdu).
3. İKİNCİ-GEÇİŞ GÜVENLİK UÇLARI (`/api/dose-check`, `/api/redflag`, `/api/simplify`) — ÜÇ DURUM:
   `warn`/`urgent` = bulgu var · `clean`/`applied` = bulgu yok · belirsiz/`unavailable` =
   ÖLÇÜLEMEDİ. ⚠⚠ `unavailable`ı `false`a EZME: "çalıştırılamadı ≠ temiz" invariantı tam
   burada VERİYE geçer; ezilirse panel "sorun yok" diye okur (fail-open'ın veri hâli).
   ⚠ Bu uçlar mesaj ÜRETMEZ → `message_id` İSTEMCİDEN gelir (`istemci_mesaj_id`, sahiplik
   doğrulamalı). `chat_body` bugün göndermiyor (Kamil ayağı) → satırlar bağsız yazılıyor.
   ⚠ Kapıda durdurulan çağrı (oturum yok · hız sınırı · günlük tavan) KAYDA GİRMEZ: yalnız
   GERÇEKTEN denenen katman yazılır. Bilinçli sınır — aksi hâlde kötüye kullanım altında
   sınırsız satır yazılırdı; o sayaçlar `app.usage`/`count_today_mode` tarafında.
4. `/api/deep-analyze` — `critic` + KOŞULLU `revise`, mesaja BAĞLI (bu uç mesajı kendi yazar).
   ⚠ critic `unavailable` ise `tetiklendi` NULL: "sorun bulunmadı" DEĞİL, "denetim koşmadı".
   Revize satırı YALNIZ revize koştuysa yazılır (yoksa satır da yok — sahte 0 üretilmez).
5. `/api/council` — `specialist`: uzman görüşü ÜRETİLDİ mi. Branş SAYISI kolonu YOK; yeni alan
   açmadan ölçülebilen şey "katman bir şey üretti mi"dir, sayı K2'nin (`question_insight`) işi.
6. `/api/cases/{cid}/timeline` — ⚠ `message_id` YAPISAL OLARAK YOK: zaman tüneli bir
   `app.message` üretmez (arşivi `app.case_timeline`). Bağsızlık burada EKSİK DEĞİL, katmanın
   DOĞASI; panelde "mesaja bağlı değil" diye okunur.
7. `/api/drug-card` — ilaç kartı da `app.message` YAZAR → künye mesaja BAĞLANIR (katman
   satırı YOK: bu uçta ayrı bir güvenlik/yardımcı katman koşmuyor, tek Opus çağrısı var).
   ⚠ ÖLÇÜLDÜ (#715b-K1-g, 04.09, LLM'siz 20 girdi): gerçek ilaç adı `drug` moduna düşer ve
   künye 0'dır (kartın belgesi yoktur — boş satır yazmak SAHTE PROVENANS olurdu); çok
   kelimeli/klinik girdi `interaction`/`clinical`a düşer ve künye 4–8 gelir. Yani bu yol
   "künye anlamsız" DEĞİL, ölçüm deliğiydi.
   ⚠⚠ Bu uç `out`u OLDUĞU GİBİ döndürür → yükü POP ETMEK ZORUNLU (aşağıdaki bölüm).

YÜKLER NEDEN POP EDİLİR (`chat_routes` `_ndjson` + `deep_analyze`/`council`/`drug-card`)
⚠⚠ `cds/chat` ölçüm yüklerini `_katmanlar` / `_kanit_kunye` adlarıyla, `_calls`/`_timing` gibi
  `_` ÖNEKLİ taşır ve tüketici onları `done`dan POP EDER. Kozmetik DEĞİL: künye `hastalik`
  modunda GİZLİ KAYNAĞIN ADINI taşır (#470b, founder "sitede hiçbir yerde görünmesin") —
  tarayıcıya gitseydi gösterim süzgeci hiç devreye girmeden ağ sekmesinde okunurdu.
⚠ Yeni bir uç bu yükü taşıyan bir sözlüğü OLDUĞU GİBİ döndürüyorsa önce POP ET.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `KATMANLAR` | `frozenset({'redflag', 'dose_check', 'critic', 'revise', 'specialist', 'map', 'simplify', 'timeline', 'retrieval'})` | 90 |
| `DURUMLAR` | `frozenset({'ok', 'unavailable', 'error'})` | 92 |
| `KUNYE_MAX_BELGE` | `40` | 95 |
| `KUNYE_BASLIK_MAX` | `200` | 96 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `acik` | `() -> bool` | 101 | Kill-switch. ⚠ Varsayılan AÇIK: ölçüm için var olan bir hattın varsayılanı kapalı |
| `satir` | `(katman: str, durum: str, tetiklendi = None, ms = None, calls = None) -> dict` | 107 | Tek katman satırı kurar; maliyeti `calls`tan HESAPLAR (tahmin etmez). |
| `satirlar` | `(ham) -> list[dict]` | 128 | `cds/chat`in `_katmanlar` yükünü (`{katman,durum,tetiklendi,ms,_calls}`) satıra çevirir. |
| `satirlar_calls` | `(calls) -> list[dict]` | 146 | `record_usage` biçimindeki `[(model, usage, mod), …]` listesinden katman satırları. |
| `yaz` | `(conn, message_id, katman, durum, tetiklendi = None, ms = None, calls = None) -> int` | 164 | Tek satırlık kestirme — çağrı yerinde TEK satır kalsın (para/akış kodunu şişirmeyelim). |
| `katman_yaz` | `(conn, message_id, kayitlar) -> int` | 169 | `app.answer_layer`a satırları yaz (best-effort). Yazılan satır sayısını döner. |
| `kunye_satirlari` | `(belgeler) -> list[dict]` | 196 | `cds/chat`ten gelen ham künye listesini yazılabilir satırlara indirger (SAF). |
| `kunye_yaz` | `(conn, message_id, belgeler) -> int` | 223 | `app.evidence_doc`a kanıt paketi künyesini yaz (best-effort). Satır sayısını döner. |
| `son_mesaj_id` | `(conn, thread_id, role: str = 'assistant')` | 247 | Az önce yazılan mesajın kimliği. None dönebilir — çağıran bunu TOLERE ETMELİ. |
| `mesaj_sahibi_mi` | `(conn, message_id, doctor_id) -> bool` | 266 | İstemcinin gönderdiği `message_id` GERÇEKTEN bu hekimin mi (IDOR freni). |
| `ms` | `(t0) -> int` | 283 | Katmanın duvar-saati süresi (ms) — ölçüm için; para/akış yolunu ETKİLEMEZ. |
| `marker_rag_sorgu` | `(conn, body, doctor_id, question: str) -> str \| None` | 289 | #726b — ÇİP/PANEL YANITINDA RETRIEVAL GİRDİSİ (D4). Sinyal yoksa None (fail-safe). |
| `marker_oncesi_soru` | `(conn, message_id, doctor_id) -> str \| None` | 316 | #726b — `marker_mid` ile gösterilen YANITI ÜRETEN hekim sorusu (yoksa None). |
| `istemci_mesaj_id` | `(conn, body, doctor_id)` | 344 | İkinci-geçiş uçlarının gövdesinden `message_id` — doğrulanmış ya da None. |

## `saglik/app/store.py`

`2553 satır` · `101 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Kalıcılık + erişim — sohbet thread/mesaj, pseudonim vaka, kullanım kaydı.

Metering'in DB sürümü: her sohbet turu app.usage'a yazılır (maliyet ölçümü + günlük fren).

⚠⚠ 2026-08-21: **KREDİ MUHASEBESİ KALDIRILDI** (founder kararı — sınırsız abonelik).
Bu dosyadan çıkan 14 fonksiyon: `try_reserve` · `commit_reservation` · `cancel_reservation`
· `_settle_once` · `reserved_portions` · `add_topup`/`add_gift` · `get_topup_balance`/
`get_gift_balance` · `wallet_status` · `grant_topup_once`/`grant_gift_once`/
`grant_yearly_bonus_once` · `month_credits_used`. Yerlerine **`erisim_durumu`** (kim
girebilir) + **`count_today_agir`** (sessiz günlük fren) geldi.
⚠ **`record_usage` KALDI ve iptalde de yazılır** — kredi kalksa da "kaçak ÖLÇÜLEMEZ
olmasın" invariantı geçerli; `app.usage` artık hem maliyet hem fren kaynağı.
⚠ Şemadan kolon DÜŞÜRÜLMEDİ (`topup_balance`, `gift_balance`, `monthly_quota`,
`app.quota_reservation`, `app.topup_grant`, `app.usage.credits`) — okunmuyor, yazılmıyor.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `FREE_MODES` | `['refused', 'classify', 'map', 'translate', 'dose_check', 'redflag', 'critic', 'revise', 'simplify', 'triage', 'speciali` | 32 |
| `RESERVED` | `'reserved'` | 60 |
| `RESERVED_ORPHAN` | `'reserved_orphan'` | 61 |
| `_RES_MODES` | `[RESERVED, RESERVED_ORPHAN]` | 62 |
| `_BILLABLE_EXCL` | `FREE_MODES + _RES_MODES` | 65 |
| `_TR_TZ` | `'Europe/Istanbul'` | 74 |
| `_TR_BUGUN` | `f"((now() AT TIME ZONE '{_TR_TZ}')::date)"` | 75 |
| `_SUB_ACTIVE_SQL` | `f"(plan <> 'trial' AND (current_period_end IS NULL OR current_period_end > now() OR (iyzico_subscription_ref IS NOT NULL` | 82 |
| `_COMP_STATUS` | `'comp'` | 103 |
| `_COMP_SQL` | `f"(d.subscription_status = '{_COMP_STATUS}')"` | 104 |
| `_PAYING_SQL` | `"(d.subscription_status = 'active')"` | 107 |
| `_GIRIS_DAMGA_BASI` | `datetime(2026, 7, 30, 2, 0, tzinfo=timezone.utc)` | 116 |
| `_IC_ADMIN_SQL` | `"COALESCE(d.role, '') = 'admin'"` | 132 |
| `_IC_TEST_SQL` | `'COALESCE(d.is_test, false)'` | 133 |
| `_IC_SQL` | `f'({_IC_ADMIN_SQL} OR {_IC_TEST_SQL})'` | 134 |
| `IC_ETIKET` | `'iç hesaplar (admin/test) hariç'` | 136 |
| `_PERIOD_START_SQL` | `"CASE\n          WHEN d.current_period_end IS NOT NULL AND d.current_period_end > now() THEN\n            d.current_peri` | 201 |
| `_STAT_TOPICS` | `[('etkilesim', ('etkileş', 'birlikte', 'kombin', 'interaksiyon', ' ile ')), ('gebelik-laktasyon', ('gebe', 'hamile', 'em` | 536 |
| `_DUVAR_OLAYLAR` | `('duvar_trial', 'duvar_abone', 'fiyat', 'fren_429', 'butce_dolu', 'butce_kismi', 'butce_olculemedi', 'butce_ucusta', 'bu` | 629 |
| `_FIYAT_KAYNAK` | `frozenset({'duvar', 'bant', 'panel', 'tekrar', 'dogrudan', 'butce', 'diger'})` | 661 |
| `_FREN_YUZEY` | `FREN_YUZEY` | 675 |
| `_OLAY_YUZEY` | `{'fiyat': _FIYAT_KAYNAK, 'fren_429': _FREN_YUZEY, 'butce_dolu': _FREN_YUZEY, 'butce_kismi': _FREN_YUZEY, 'butce_olculeme` | 678 |
| `_ILAC_ADLARI_CACHE` | `None` | 728 |
| `UPCOMING_LIMIT` | `60` | 1396 |
| `OVERDUE_LIMIT` | `30` | 1397 |
| `_UNSET` | `object()` | 1471 |
| `SETTING_MAX` | `120` | 1852 |
| `_UNSUB_SALT_KEY` | `'mail_unsub_salt'` | 2000 |
| `_ADOPTION` | `[('chat', 'Sohbet (Sor)', 'app.thread', ''), ('case', 'Hasta kartı', 'app.case_note', ''), ('visit', 'Ziyaret notu', 'ap` | 2442 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `_tr_gun` | `(kolon: str) -> str` | 139 | timestamptz kolonunu TÜRKİYE yerel gününe indir (panel sayımları için). |
| `_like_escape` | `(s: str) -> str` | 144 | LIKE/ILIKE metakarakterlerini (\ % _) kaçır → kullanıcı girdisi LİTERAL aransın. |
| `record_usage` | `(conn, doctor_id: int, calls: list[tuple], credits: int = 1) -> float` | 151 | calls = [(model, usage, call_mode), ...]. Her çağrıyı KENDİ moduyla app.usage'a yazar. |
| `_period_start` | `(conn, doctor_id: int)` | 215 | Aylık pencerenin başlangıcı — `_PERIOD_START_SQL`ün tek satırlık okuması (gerekçe orada). |
| `purge_reservation_marks` | `(conn, days: int = 7) -> int` | 223 | Eski rezervasyon kesinleştirme izlerini sil (tablo şişmesin). rid'ler bigserial olduğu |
| `grant_iyzico_subscription` | `(conn, doctor_id: int, plan: str, interval: str, bonus_key: str \| None = None) -> int` | 233 | iyzico ön-ödemeli dönem tahsilatını app.doctor'a uygula (Stripe _apply_subscription'ın |
| `pending_payments` | `(conn, days: int = 7, limit: int = 200)` | 266 | Hak-edişi VERİLMEMİŞ son ödeme denemeleri (mutabakat kuyruğu). |
| `settle_payment` | `(conn, token: str, payment_id: str \| None = None) -> dict \| None` | 290 | DOĞRULANMIŞ bir iyzico ödemesinin hak-edişini ATOMİK + İDEMPOTENT ver. |
| `usage_summary` | `(conn, doctor_id: int) -> dict` | 344 |  |
| `erisim_durumu` | `(conn, doctor_id: int) -> dict` | 367 | Hekimin erişim hakkı — **TEK DOĞRU KAYNAK**. Tüm ücretli uçlar bunu okur. |
| `hediye_gun_ver` | `(conn, anahtar: str, doctor_id: int, gun: int, plan: str = 'aylik') -> bool` | 428 | Admin hediyesi — **GÜN** hediye eder (2026-08-21: eski `grant_gift_once` kredi |
| `erisim_bitisi` | `(conn, doctor_id: int)` | 473 | Hekimin erişiminin bittiği an (admin gösterimi + hediye sonrası geri okuma). |
| `count_today_agir` | `(conn, doctor_id: int, modlar = None) -> int` | 478 | Bugün (TÜRKİYE günü) tüketilen AĞIR çağrı sayısı — sınırsız aboneliğin sessiz freni. |
| `create_thread` | `(conn, doctor_id: int, title: str = '', case_id: int \| None = None) -> int` | 504 |  |
| `add_message` | `(conn, thread_id: int, role: str, content: str, sources = None, mode: str = None, cost_usd: float = None, notice: str = None) -> None` | 513 | ⚠ `notice` (#620b-C): hekime gosterilecek, MODELDEN BAGIMSIZ uyari (ornegin |
| `_stat_topic` | `(question: str) -> str` | 552 |  |
| `_stat_kinds` | `(sources) -> list[str]` | 560 |  |
| `log_query_stat` | `(question: str, mode: str, sources, lang: str) -> None` | 571 | ANONİM türetilmiş sinyal yaz — KENDİ bağlantısı (ana işlem/yanıtı ASLA bozmaz, best-effort). |
| `_duvar_yuzey` | `(yuzey: str) -> str` | 686 | Yüzey etiketini kanonikleştir: küçük harf, [a-z0-9_], en çok 24 karakter. |
| `duvar_say` | `(conn, olay: str, yuzey: str) -> None` | 692 | ANONİM sayaç artır (best-effort). Kimlik/saat/sorgu YAZILMAZ — bkz. şema notu. |
| `_bilinen_ilac_adlari` | `(conn) -> set` | 731 | Redaksiyonda KORUNACAK ilaç adları (kök marka + jenerik), küçük harfli. |
| `keep_questions_for_doctor` | `(conn, doctor_id: int, reason: str) -> int` | 758 | Hekim SİLİNMEDEN ÖNCE sorularını `app.question_kept`e kopyala (founder kararı 2026-07-29). |
| `kept_questions` | `(conn, limit: int = 200, only_gaps: bool = False) -> list[dict]` | 812 | Silinmiş üyelerden KORUNAN sorular (admin görünürlüğü). Hekim kimliği YOKTUR. |
| `list_threads` | `(conn, doctor_id: int, limit: int = 50)` | 825 | Sohbet kenar çubuğu. Döner: `(id, baslik, created_at, mesaj_n, son_etkinlik)`. |
| `delete_thread` | `(conn, doctor_id: int, thread_id: int) -> bool` | 851 | Konuşmayı sil (sahiplik zorunlu). Mesajlar FK CASCADE ile birlikte silinir. |
| `rename_thread` | `(conn, doctor_id: int, thread_id: int, title: str) -> bool` | 860 | Konuşmayı yeniden adlandır (sahiplik zorunlu; başlık kırpılır). |
| `thread_owned` | `(conn, doctor_id: int, thread_id: int) -> bool` | 870 | thread_id gerçekten bu doktora mı ait? Yazma yolunda IDOR guard'ı için. |
| `thread_messages` | `(conn, doctor_id: int, thread_id: int)` | 877 | Doktorun kendi thread'inin mesajları (sahiplik kontrolü zorunlu). |
| `thread_case_id` | `(conn, doctor_id: int, thread_id: int) -> int \| None` | 892 | Sohbete bağlı hasta kartının id'si — YALNIZ kart hâlâ VARSA ve BU hekiminse; yoksa None. |
| `create_case` | `(conn, doctor_id: int, code: str, age_band: str, sex: str, narrative: str, chronic: str = '', meds: str = '', egfr = None, kreatinin = None, kilo_kg = None, gebelik_haftasi = None, boy_cm = None, allergy_status: str = '', allergies: str = '', gebe_olabilir = None, emziriyor = None, yas = None, birlestir: bool = True) -> tuple[int \| None, bool]` | 910 | Pseudonim hasta kartı aç. Aynı takma kod varsa GÜNCELLER — ancak BOŞ gelen |
| `update_case` | `(conn, doctor_id: int, case_id: int, age_band: str, sex: str, narrative: str, chronic: str, meds: str, egfr = None, kreatinin = None, kilo_kg = None, gebelik_haftasi = None, boy_cm = None, allergy_status: str = '', allergies: str = '', gebe_olabilir = None, emziriyor = None, yas = None) -> bool` | 996 | Hasta kartı başlık bilgilerini güncelle (sahiplik zorunlu). Sayısal alanlar |
| `case_silme_sayilari` | `(conn, doctor_id: int, case_id: int) -> dict \| None` | 1022 | Bir hasta kartına bağlı satırların TAM sayımı (sahiplik zorunlu) → dict \| None. |
| `delete_case` | `(conn, doctor_id: int, case_id: int) -> dict \| None` | 1052 | Hasta kartını KALICI sil (sahiplik zorunlu) → silinenlerin sayımı; kart yoksa None. |
| `get_case` | `(conn, doctor_id: int, case_id: int)` | 1079 | Tek hasta kartı (sahiplik zorunlu) → satır veya None. |
| `list_cases` | `(conn, doctor_id: int, limit: int = 300)` | 1097 | Hasta kartları — son ziyarete göre; ilaç + dosya sayısı + klinik-param bayrağı ile. |
| `add_visit` | `(conn, doctor_id: int, case_id: int, note: str) -> int \| None` | 1123 | Ziyaret notu ekle (sahiplik zorunlu) + son ziyaret zamanını güncelle. |
| `link_visit_thread` | `(conn, doctor_id: int, visit_id: int, thread_id: int) -> None` | 1144 | Analiz danışması thread'ini ziyarete bağla (sahiplik zorunlu). İlk analiz kazanır — |
| `list_visits` | `(conn, case_id: int, limit: int = 200)` | 1153 |  |
| `list_visits_full` | `(conn, case_id: int, limit: int = 200)` | 1160 | Hasta detay 'Ziyaret geçmişi' için: her ziyaret + bağlı analiz thread'i ve o thread'in |
| `add_case_timeline` | `(conn, doctor_id: int, case_id: int, body: str, model: str = '', lang: str = 'tr') -> int \| None` | 1182 | Üretilen zaman tünelini arşive yaz (sahiplik zorunlu). Döner: id \| None (sahiplik yok). |
| `list_case_timelines` | `(conn, doctor_id: int, case_id: int, limit: int = 20)` | 1203 | Kayıtlı zaman tünelleri, EN YENİ ÜSTTE. Döner: (id, body, created_tr, lang). |
| `count_recent_signups` | `(conn, ip: str, window_minutes: int = 60) -> int` | 1218 | Aynı IP'den son N dakikadaki kayıt sayısı (kayıt spam'i freni). |
| `count_today_mode` | `(conn, doctor_id: int, mode: str) -> int` | 1228 | Bugün (takvim günü) doktorun verilen moddaki çağrı sayısı — ücretsiz uç tavanı için. |
| `admin_doctor_funnel` | `(conn, doctor_id: int) -> dict` | 1239 | Hekim detayı: maliyet + aktivasyon funnel tek sorguda. lifetime_usd = yaşam-boyu API maliyeti |
| `admin_get_doctor` | `(conn, doctor_id: int)` | 1262 | Tek hekim tam detay (admin). Döner: dict \| None. |
| `admin_set_fields` | `(conn, doctor_id: int, plan = None, monthly_quota = None, verification_status = None, topup_balance = None) -> bool` | 1285 | Admin: plan / DENEME GÜNÜ / doğrulama durumu güncelle (verilen alanlar). |
| `admin_stats` | `(conn) -> dict` | 1309 | Panel üst özet: toplam hekim, aktif abone, bu ay toplam kredi + API maliyeti (USD). |
| `add_appointment` | `(conn, doctor_id: int, starts_at, kind: str = 'randevu', title: str = '', case_id: int \| None = None) -> int` | 1400 | Takvim etkinliği ekle (randevu/takip). |
| `list_appointments` | `(conn, doctor_id: int, start, end)` | 1429 | [start, end) aralığındaki etkinlikler (takvim ızgarası için). Hasta kodu JOIN'li. |
| `list_upcoming_appointments` | `(conn, doctor_id: int, since, limit: int = UPCOMING_LIMIT, include_done: bool = True)` | 1441 | Yaklaşan (since'ten sonra) etkinlikler — 'Yaklaşanlar' panosu için. |
| `list_overdue_followups` | `(conn, doctor_id: int, now, limit: int = OVERDUE_LIMIT)` | 1456 | Tarihi GEÇMİŞ ve tamamlanmamış klinik TAKİPLER — 'Gecikmiş takip' bloğu için. |
| `update_appointment` | `(conn, doctor_id: int, appt_id: int, starts_at = None, kind: str \| None = None, title: str \| None = None, case_id = _UNSET) -> bool` | 1474 | Etkinliği düzenle — YALNIZ verilen alanlar değişir (kısmi güncelleme). |
| `list_case_appointments` | `(conn, doctor_id: int, case_id: int, limit: int = 20)` | 1518 | Bir hastanın yaklaşan/geçmiş etkinlikleri (hasta-detay sayfası için). |
| `count_case_upcoming` | `(conn, doctor_id: int, case_id: int, since) -> int` | 1527 | Hastanın tamamlanmamış YAKLAŞAN etkinlik SAYISI (hasta kartı 'N yaklaşan'). |
| `set_appointment_done` | `(conn, doctor_id: int, appt_id: int, done: bool = True) -> bool` | 1537 |  |
| `delete_appointment` | `(conn, doctor_id: int, appt_id: int) -> bool` | 1544 |  |
| `list_cases_min` | `(conn, doctor_id: int, limit: int = 500)` | 1551 | Composer açılır menüsü + masaüstü aktarıcı için hafif liste. |
| `link_thread_case` | `(conn, doctor_id: int, thread_id: int, case_id: int) -> bool` | 1563 | Bir sohbeti pseudonim hasta kartına bağla (her ikisi de bu hekime ait olmalı). |
| `list_threads_for_case` | `(conn, doctor_id: int, case_id: int, limit: int = 50)` | 1577 | Bu hasta kartına bağlı sohbetler (en yeni önce). |
| `add_case_file` | `(conn, doctor_id: int, case_id: int, filename: str, mime: str, size: int, sha256: str, enc_bytes: bytes) -> int \| None` | 1590 | Şifreli dosyayı vakaya ekle (sahiplik zorunlu). Döner: file_id veya None (sahiplik yok). |
| `add_case_file_tekil` | `(conn, doctor_id: int, case_id: int, filename: str, mime: str, size: int, sha256: str, enc_bytes: bytes) -> tuple[int, bool] \| None` | 1600 | `add_case_file`in raporlayan hâli. Döner: (file_id, yeni_mi) ya da None (sahiplik yok). |
| `list_case_files` | `(conn, doctor_id: int, case_id: int)` | 1632 | Aktif (silinmemiş) dosyalar — content_enc HARİÇ (blob taşınmasın). |
| `get_case_file` | `(conn, doctor_id: int, file_id: int)` | 1644 | Tek dosya + içerik (indirme/analiz). IDOR guard doctor_id WHERE'de. |
| `bekleyen_dosya_say` | `(rows) -> int` | 1653 | Analiz edilmemiş dosya sayısı (`list_case_files` satırı, r[5]=analyzed). ⚠ #620b: görsel (`image/*`, r[2]) |
| `soft_delete_case_file` | `(conn, doctor_id: int, file_id: int) -> bool` | 1659 | Dosyayı sil (sahiplik zorunlu): ŞİFRELİ İÇERİĞİ + çıkarım özetini GERÇEKTEN temizler |
| `set_extracted_text` | `(conn, file_id: int, text: str, model: str) -> None` | 1671 | MAP cache yazımı. file_id çağıran katmanda get_case_file ile doctor_id'ye karşı |
| `request_account_deletion` | `(conn, doctor_id: int) -> None` | 1681 | Hesabı silme için damgala (grace başlar). İdempotent: zaten damgalıysa saati SIFIRLAMAZ |
| `cancel_account_deletion` | `(conn, doctor_id: int) -> None` | 1690 | Silme talebini geri al (grace içinde). deletion_requested_at=NULL. |
| `account_deletion_at` | `(conn, doctor_id: int)` | 1697 | Silme talebi damgası (timestamptz) veya None — /account banner/iptal için. |
| `purge_due_accounts` | `(conn, grace_days: int = 30) -> list[int]` | 1704 | Grace süresi (varsayılan 30 gün) dolan silme-talepli hesapları KALICI sil. |
| `purge_soft_deleted_files` | `(conn, days: int = 30) -> int` | 1756 | Soft-delete edilmiş case_file tombstone satırlarını (içerik zaten sıfırlanmış) fiziksel sil. |
| `list_case_lab_summaries` | `(conn, doctor_id: int, case_id: int, limit: int = 25)` | 1768 | Analiz edilmiş dosyaların çıkarım özetleri (tahlil değerleri) — chat'in hasta |
| `get_cached_translations` | `(conn, hashes: list[str], lang: str) -> dict` | 1780 | Verilen kaynak-hash'ler için önbellekteki çevirileri {hash: translated} döndür. |
| `put_cached_translations` | `(conn, items: list[tuple], lang: str, model: str) -> None` | 1790 | items = [(src_hash, translated), ...] → önbelleğe yaz (çakışmada dokunma). |
| `save_consents` | `(conn, doctor_id: int, ip: str \| None, unvan: str = '', phone: str = '', birth_date = None, country: str \| None = None, city: str \| None = None) -> None` | 1813 | 2 hukuki onayı zaman+IP damgasıyla yazar; unvan/telefon/doğum tarihi/ülke-şehir |
| `has_consents` | `(conn, doctor_id: int) -> bool` | 1840 |  |
| `get_setting` | `(conn, key: str) -> str \| None` | 1847 |  |
| `set_setting` | `(conn, key: str, value: str) -> None` | 1855 | Ayarı yaz. ⚠⚠ **SINIRI AŞAN DEĞER SESSİZCE KIRPILMAZ — REDDEDİLİR.** |
| `_trial_salt` | `(conn) -> str` | 1897 | Özet tuzu — bir kez üretilip `app.setting`'te saklanır (koda GÖMÜLMEZ). |
| `trial_salt_var` | `(conn) -> bool` | 1921 | Tuz DB'de KAYITLI mı? (panelde 'ölçülemedi' basmak için — üretmez, yalnız bakar). |
| `trial_email_hash` | `(conn, email: str) -> str` | 1929 | E-postanın tuzlu HMAC özeti. |
| `trial_already_used` | `(conn, email: str) -> bool` | 1944 | Bu e-posta daha önce ücretsiz deneme aldı mı? |
| `mark_trial_used` | `(conn, email: str) -> None` | 1955 | Denemeyi 'kullanıldı' işaretle (idempotent). ⚠ commit ÇAĞIRANA ait değil — burada |
| `backfill_trial_used` | `(conn) -> int` | 1969 | MEVCUT hesapların e-postalarını işaretle (tek seferlik, idempotent). |
| `_unsub_salt` | `(conn) -> str` | 2003 | Abonelikten-çıkma imza tuzu — bir kez üretilip `app.setting`te saklanır. |
| `mail_unsub_token` | `(conn, doctor_id: int) -> str` | 2023 | `<id>.<imza>` — maildeki çıkış bağlantısına gömülen jeton. |
| `mail_unsub_coz` | `(conn, token: str) -> int \| None` | 2042 | Jetonu doğrula → doctor_id, geçersizse None. |
| `mail_opt_out_durum` | `(conn, doctor_id: int) -> bool \| None` | 2062 | Bu hekim listeden çıkmış mı? True/False; hekim yoksa (ya da kolon yoksa) None. |
| `mail_opt_out_yaz` | `(conn, doctor_id: int, *, cik: bool = True) -> bool` | 2077 | Çıkış talebini uygula (idempotent). Döner: yazma başarılı mı. |
| `admin_update_profile` | `(conn, doctor_id: int, *, full_name: str, email: str, unvan: str, specialty: str, kurum: str, phone: str, is_test: bool \| None = None) -> str \| None` | 2101 | Admin üye bilgisi düzenler. Döner: None=başarı, aksi hâlde hata kodu. |
| `admin_archive_and_delete` | `(conn, doctor_id: int, admin_id: int, reason: str = '') -> bool` | 2145 | Üyeyi ARŞİVLEYEREK sil (kullanıcı kararı: 'silinenler komple kaybolmasın'). |
| `list_archive` | `(conn, limit: int = 100) -> list[dict]` | 2204 | Admin'in ARŞİVLEDİĞİ üyeler + her satır için o adresin ŞU AN aktif hesabı var mı. |
| `admin_dashboard` | `(conn) -> dict` | 2234 | Founder panosu verileri — tek round-trip'lik hafif sorgular (admin sayfası başına). |
| `clarify_stats` | `(conn) -> dict` | 2390 | 'Önce netleştirin' önce-sor turu ÖLÇÜMÜ (bu ay). Bu tur hekime ÜCRETSİZ (kredi iade) |
| `set_feedback_status` | `(conn, fid: int, status: str) -> None` | 2452 |  |
| `get_billing` | `(conn, doctor_id: int) -> dict \| None` | 2460 |  |
| `set_billing` | `(conn, doctor_id: int, data: dict) -> None` | 2465 | Fatura profili: {'type':'bireysel'\|'mukellef','name','tckn_vkn','tax_office','address','city'}. |
| `count_kurucu` | `(conn) -> int` | 2477 | Kurucu plana (kurucu/kurucu_yillik) geçmiş hekim sayısı — GERÇEK kontenjan sayacı. |
| `save_feedback` | `(conn, doctor_id: int, category: str, message: str, page: str = '') -> int` | 2487 | Hekim önerisini/geri bildirimini kaydeder → id döner. Alanlar kırpılır (kötüye kullanım freni). |
| `feedback_sayaclari` | `(conn) -> dict` | 2497 | Geri bildirim kuyruğunun GERÇEK boyu — `list_feedback` LIMIT'inden BAĞIMSIZ. |
| `list_feedback` | `(conn, limit: int = 200) -> list[dict]` | 2517 | Admin paneli için geri bildirimler (yeni → eski), gönderen hekim bilgisiyle. |

## `saglik/app/tr_saat.py`

`38 satır` · `2 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
TÜRKİYE GÜN SINIRI — takvim · panel · hasta kartı için TEK "şimdi" (2026-08-20, takvim denetimi kalem 2).

⚠ NEDEN VAR: "bugün" sınırı ÜÇ yüzeyde ayrı hesaplanıyordu, ikisi çıplak sunucu-saati
  (`datetime.now()`) idi → Render UTC'de TR 00:00–03:00 arası `/api/calendar` DÜNÜ bugün
  sayarken `/panel` doğru sayıyordu: aynı takibe üç ayrı hüküm. Tek kaynak burada.
⚠ NEDEN AYRI MODÜL, `webutil` DEĞİL: `webutil` tavan girdisinin kendi kuralı
  (`dosya_boyut_verify`: "dokuzuncu artış YOK — önce `pazar.py` bölmesi") bağlayıcı; bu
  yardımcı da paylaşılan sözlükle (esc/nav/i18n) HİÇBİR ortak durum taşımıyor. SAF: DB yok,
  HTML yok, `main.py`/`webutil` import'u yok → dairesel import riski sıfır.
⚠⚠ NAİVE döner (tzinfo SIYRILIR): karşılaştırılan `appointment.starts_at` DUVAR-SAATİ'dir
  (memory `klivance-takvim-saat-modeli`); tz-aware değerle kıyas saat kaymasını geri getirir.
  Sabit `+03` YAZMA — zaman dilimi ADI (yaz saati/ileride değişiklik).
⚠ TEK MONKEYPATCH NOKTASI `tr_saat.tr_simdi` (`tr_gun_basi` ondan türetir). Tüketiciler
  `from . import tr_saat as _ts` + `_ts.tr_simdi()` biçiminde MODÜL ÜZERİNDEN çağırır —
  `from .tr_saat import tr_simdi` + ad çağrısı patch'i GÖRMEZ (`takvim_verify` §7 AST ile çiviler).
⚠ `store._TR_BUGUN`/`_tr_gun()` bunun SQL tarafıdır (sorgu içi `AT TIME ZONE`); bu modül
  Python tarafı (naive duvar-saatiyle kıyas). İkisi aynı dilim ADINI kullanır, biri diğerinin
  yerine GEÇMEZ.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_TR_TZ` | `ZoneInfo('Europe/Istanbul')` | 26 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `tr_simdi` | `() -> _dt.datetime` | 29 | Türkiye duvar-saati ŞİMDİ — naive (ofsetsiz). |
| `tr_gun_basi` | `(now: _dt.datetime \| None = None) -> _dt.datetime` | 34 | Türkiye'de BUGÜNÜN BAŞI (00:00, naive). `now` verilmezse `tr_simdi()`. |

## `saglik/app/ui_css.py`

`1267 satır` · `0 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
Uygulama CSS'i — TEK KAYNAK (TR ve EN sayfalar aynı sabiti kullanır).

⚠ Bu dosya `saglik/app/main.py`'den AYRILDI (2026-07-29, mimari denetimi). İçerik
AYNEN taşındı — tek karakter değişmedi; render çıktısı bayt bayt doğrulandı.
main.py bu adları yeniden dışa aktarır, yani mevcut importlar (`from saglik.app.main
import ...`) ve doğrulama scriptleri KIRILMAZ.

⚠ Buraya YALNIZ saf sabit konur: f-string kullanma, modül durumuna dokunma, import etme.
Dinamik bir şey gerekiyorsa main.py'de kalmalı.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `CSS` | `'\n/* ==========================================================================\n   KLIVANCE — PAYLAŞILAN APP KABUĞU CS` | 13 |

## `saglik/app/webutil.py`

`1256 satır` · `34 fonksiyon` · `0 sınıf`

<details><summary><b>Modül başlığı (sözleşme)</b></summary>

```
WEBUTIL — uygulamanın PAYLAŞILAN SÖZLÜĞÜ (Faz 2a, 2026-07-31).

`main.py`'den ayrıldı. Sebep: admin'i ayırmak (Faz 2) doğrudan mümkün DEĞİLDİ —
admin kodu main.py'den 18 ada bağlıydı ve taşımak dairesel import üretirdi
(main → admin_routes → main). Bu modül o bağı kırar: HEM main.py HEM ileride
gelecek router modülleri buradan okur.

⚠⚠ BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ — etmemeli. Ederse dairesel
  import doğar ve uygulama AÇILMAZ.

⚠⚠ SIRA ÖNEMLİ: `_ANALYTICS_TAG`, `_ANALYTICS_PATH_ONLY`'yi DEĞER olarak içerir
  (AST ile doğrulandı). Tanımların sırası main.py'deki orijinal sırasıdır; yeniden
  sıralarsan NameError alırsın. Yeni ad eklerken bağımlılığından SONRA yaz.

⚠ HASSAS ADLAR — değiştirirken ilgili invariantı oku:
  · `esc`  — XSS kalkanı; her HTML üretimi buna bağlı.
  · `en`   — i18n çevirisi (EN_PAIRS). Apostrof tuzağı buradan doğar: İngilizce
             değerlerde kıvrık kesme ’ (U+2019) kullan, düz ' JS string'ini kapatır.
  · `head` / `nav` — her sayfanın ortak girdisi; `nav` dil-duyarlı ve `_rh()` ile
             çıkışlıda kimlik-isteyen rayı /kayit'a çevirir.
  · `_doctor` — oturum çözümü, kimlik kararlarının tek kaynağı.
  · `_ANALYTICS_TAG` / `_ANALYTICS_PATH_ONLY` — `klv_notrack` tag'in TAMAMINI soyar;
             path-only onun İÇİNDE olduğu için o da gider. Ayırma.

⚠ main.py bu adları `# noqa: F401` ile YENİDEN DIŞA VERİR (fasad): main.py'de 108
  fonksiyon ve 6 dış doğrulama script'i onları `main.<ad>` olarak okuyor.
```

</details>

**Modül sabitleri**

| Ad | Değer (kırpılmış) | Satır |
|---|---|---|
| `_PW_MESAJ` | `{'short': f'Parola en az {auth.PW_MIN} karakter olmalı.'}` | 57 |
| `_PW_IPUCU` | `f'En az {auth.PW_MIN} karakter.'` | 64 |
| `LOGO` | `'<svg width="32" height="32" viewBox="0 0 128 128" xmlns="http://www.w3.org/2000/svg"><rect x="8" y="8" width="112" heig` | 66 |
| `_FONTS` | `'<link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" cros` | 71 |
| `_FAVICON` | `'<link rel="icon" href="/static/favicon.svg" type="image/svg+xml"><link rel="icon" href="/static/favicon-32.png" sizes="` | 76 |
| `GA4_MEASUREMENT_ID` | `'G-FKK3Z0TWZ9'` | 84 |
| `ADS_CONVERSION_ID` | `'AW-18330764987'` | 91 |
| `ADS_CALC_CTA_SEND_TO` | `f'{ADS_CONVERSION_ID}/Jv79CK2ig9ccELuN5aRE'` | 92 |
| `_PROD_ANALYTICS` | `_os.getenv('KLIVANCE_COOKIE_SECURE') == '1'` | 93 |
| `_ANALYTICS_PATH_ONLY` | `'<script>window.KLV_PRIV=/^\\/(reference\|cases\|admin\|etkilesim\|bobrek-doz\|onay)(\\/\|$\|\\?)/.test(location.pathnam` | 108 |
| `_GA4_TAG` | `f"""<script async src="https://www.googletagmanager.com/gtag/js?id={GA4_MEASUREMENT_ID}"></script><script>window.dataLay` | 110 |
| `META_PIXEL_ID` | `_os.getenv('META_PIXEL_ID', '').strip()` | 121 |
| `_META_PIXEL_TAG` | `f"""<script>!function(f,b,e,v,n,t,s){{if(f.fbq)return;n=f.fbq=function(){{n.callMethod?n.callMethod.apply(n,arguments):n` | 122 |
| `_KLV_TRACK_JS` | `f"<script>window.klvTrack=function(g,gp,f,fp){{try{{if(window.gtag)gtag('event',g,gp\|\|{{}});}}catch(e){{}}try{{if(wind` | 146 |
| `_SRC_CAPTURE_JS` | `'<script>(function(){try{var q=new URLSearchParams(location.search);var g=q.get("gclid"),wb=q.get("wbraid"),gb=q.get("gb` | 172 |
| `_ANALYTICS_TAG` | `_ANALYTICS_PATH_ONLY + _GA4_TAG + _META_PIXEL_TAG + _KLV_TRACK_JS + RAIL_TRACK_JS + _SRC_CAPTURE_JS` | 194 |
| `_ICSVG` | `'viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"` | 230 |
| `IC_CHAT` | `f'<svg {_ICSVG}><path d="M21 11.8a8.2 8.2 0 0 1-8.2 8.2 8.4 8.4 0 0 1-3.7-.8L3.2 20.8l1.4-5A8.2 8.2 0 1 1 21 11.8z"/></s` | 232 |
| `IC_REF` | `f'<svg {_ICSVG}><circle cx="10.8" cy="10.8" r="6.8"/><path d="M20.6 20.6l-5-5"/></svg>'` | 233 |
| `IC_CASE` | `f'<svg {_ICSVG}><circle cx="9.6" cy="8" r="3.4"/><path d="M3 21v-1.2A4.8 4.8 0 0 1 7.8 15h3.6a4.8 4.8 0 0 1 4.8 4.8V21"/` | 234 |
| `IC_HASTALIK` | `f'<svg {_ICSVG}><path d="M12 6.4v13.4"/><path d="M12 6.4C10.4 4.9 8.4 4.2 6 4.2H3.6v13.4H6c2.4 0 4.4.7 6 2.2"/><path d="` | 238 |
| `IC_CAL` | `f'<svg {_ICSVG}><rect x="3" y="5.2" width="18" height="15.8" rx="2.6"/><path d="M16.2 3v4.4M7.8 3v4.4M3 10.6h18"/></svg>` | 239 |
| `IC_CALCU` | `f'<svg {_ICSVG}><rect x="4.2" y="3" width="15.6" height="18" rx="2.6"/><rect x="7.4" y="6.2" width="9.2" height="3.6" rx` | 240 |
| `IC_ADMIN` | `f'<svg {_ICSVG}><path d="M12 3l7.6 3.4v5.4c0 4.6-3.2 7.4-7.6 9.2-4.4-1.8-7.6-4.6-7.6-9.2V6.4z"/><path d="M9.2 12.2l2 2 3` | 241 |
| `IC_ACC` | `f'<svg {_ICSVG}><circle cx="12" cy="8.2" r="3.6"/><path d="M4.6 21v-1a5.6 5.6 0 0 1 5.6-5.6h3.6A5.6 5.6 0 0 1 19.4 20v1"` | 242 |
| `IC_OUT` | `f'<svg {_ICSVG}><path d="M9.6 21H5.6A2.6 2.6 0 0 1 3 18.4V5.6A2.6 2.6 0 0 1 5.6 3h4"/><path d="M16 16.8l4.8-4.8L16 7.2"/` | 243 |
| `IC_HOME` | `f'<svg {_ICSVG}><path d="M3 11.4 12 4.2l9 7.2"/><path d="M5.6 10V21h12.8V10"/></svg>'` | 244 |
| `IC_PANEL` | `f'<svg {_ICSVG}><rect x="3" y="3" width="7.6" height="7.6" rx="1.8"/><rect x="13.4" y="3" width="7.6" height="7.6" rx="1` | 247 |
| `IC_GERI` | `f'<svg {_ICSVG} stroke-width="2"><path d="M15 18l-6-6 6-6"/></svg>'` | 248 |
| `IC_DAHA` | `f'<svg {_ICSVG}><circle cx="5" cy="12" r="1.7" fill="currentColor" stroke="none"/><circle cx="12" cy="12" r="1.7" fill="` | 252 |
| `UST_SERIT_ISARET` | `'<!--KLV_UST_SERIT-->'` | 260 |
| `_RAIL_JS` | `'<script>(function(){var n=document.querySelector(".rail-nav");if(!n)return;var a=n.querySelector(".rail-item.active");i` | 328 |
| `PAZAR_TR` | `'tr'` | 606 |
| `PAZAR_INTL` | `'intl'` | 607 |
| `_PLAN_ADLARI` | `{'trial': ('Deneme', 'Trial'), 'aylik': ('Standart', 'Standard'), 'yillik': ('Standart Yıllık', 'Standard Annual'), 'kur` | 730 |
| `_ADMIN_PLANS` | `{k: v[0] for k, v in _PLAN_ADLARI.items()}` | 748 |
| `_ADMIN_VER` | `{'pending': 'Değerlendirmede', 'verified': 'Onaylı', 'rejected': 'Reddedildi'}` | 749 |
| `_ADM_AMBER` | `'background:#FFF4E0;border:1px solid #F0DCA8;color:var(--amber-ink)'` | 757 |
| `_GIRIS_DAMGA_BASI` | `store._GIRIS_DAMGA_BASI` | 767 |
| `_IYZ_KOD` | `{'10005': ('bankanız işlemi onaylamadı — bankanızı arayın', 'your bank declined the transaction — call your bank'), '100` | 811 |

| Fonksiyon | İmza | Satır | Açıklama |
|---|---|---|---|
| `esc` | `(v) -> str` | 41 | Kullanıcı verisini HTML'e basmadan önce kaçış (stored XSS koruması). |
| `head` | `(title, desc: str = '', canonical: str = '', robots: str = '', *, izsiz: bool = False)` | 196 | Ortak `<head>` — başlık + (opsiyonel) açıklama/canonical/robots. |
| `ust_serit` | `(baslik, *, geri = None, rozet = None, meta = None, cta = None, ikincil = None, lang = 'tr', baslik_id = None, ustetiket = None)` | 263 | D4 · sayfa üst şeridi: ← geri · başlık · durum rozeti · meta · TEK birincil CTA. |
| `nav` | `(doctor, active = '', lang = 'tr')` | 346 |  |
| `_doctor` | `(request: Request, conn)` | 528 |  |
| `get_lang` | `(request: Request) -> str` | 531 | Dil: ?lang= query > cookie > Accept-Language (birincil etiket tr ise tr, değilse en) > tr. |
| `en` | `(html: str) -> str` | 545 | App HTML'indeki Türkçe arayüz ifadelerini İngilizceye çevir (yalnız lang=en). |
| `_sel_opts` | `(items, cur = '')` | 558 |  |
| `_brans_opts` | `(cur = '')` | 561 | Branş seçimi — optgroup'lu (Ana dallar / Yan dallar) + Diğer; uzun listeyi taranabilir tutar. |
| `_num_or_none` | `(s)` | 574 | Form sayısal alanı → float ya da None (boş/geçersiz → None; virgül→nokta). |
| `_int_or_none` | `(s)` | 581 | Form tamsayı alanı → int ya da None (gebelik haftası gibi). |
| `_usd_try` | `(db_value: str \| None = None) -> float` | 585 | USD/TRY kuru — admin birim-ekonomi kartı için. Öncelik: admin panel ayarı |
| `pazar` | `(d = None) -> str` | 610 | Hekimin PAZARI: 'tr' \| 'intl'. `d` yoksa/telefon yoksa **'tr'** (bugünkü tek pazar). |
| `_pay_provider` | `(d = None) -> str` | 622 | Bu hekim için ödeme sağlayıcısı ('stripe'\|'iyzico') — UI dallanması ve rota için. |
| `tahsilat_birimi` | `(doctor = None) -> str` | 634 | TAHSİL EDİLECEK para birimi ('TRY'\|'USD') — **kasanın gerçekte ne yapacağı.** |
| `para_birimi` | `(request = None, doctor = None, en = None) -> str` | 643 | GÖSTERİLECEK para birimi ('TRY'\|'USD'). **Fiyat basan HER yüzey bunu çağırır.** |
| `klv_cur_js` | `(request = None, doctor = None) -> str` | 669 | `window.KLV_CUR` için İSTEK-BAZLI düzeltme script'i (yoksa boş dize). |
| `_price_try_fmt` | `(amount_str) -> str` | 688 | '1900.00' → '1.900 TL' (tam sayı ondalıksız; Türk binlik ayracı). |
| `plan_adi` | `(plan: str, en: bool = False) -> str` | 739 | Planın ÜRÜN adı (fiyatsız) — tek doğru kaynak. Bilinmeyen slug kendisi döner. |
| `_aktif_yok` | `(kayit) -> str` | 768 | `last_active` NULL iken basılacak hücre. `kayit` = hesabın created_at'i (None ise |
| `_tr_tutar_coz` | `(metin: str) -> float \| None` | 784 | TR yazımlı para girdisini float'a çevir; çözemezse None (0 UYDURMAZ). |
| `_rate_ok` | `(doctor_id, limit: int = RATE_MAX) -> bool` | 844 | Süreç-içi hız sınırı. `limit` ile uç-başına daha SIKI tavan verilebilir (M12). |
| `_profil_eksik` | `(d) -> bool` | 869 | Zorunlu profil alanları tamam mı? (consents + telefon + unvan + branş) |
| `_client_ip` | `(request: Request) -> str` | 893 | Gerçek istemci IP — sunucu yapılandırma zincirinden (gunicorn env → UvicornWorker → |
| `async` `_json_body` | `(request) -> dict` | 913 | İstek gövdesini güvenle sözlük olarak oku (await gereken TEK kısım). |
| `_case_ctx_base` | `(c)` | 927 | case_note satırından ortak hasta bağlamı sözlüğü (record HARİÇ — çağıran ekler). |
| `_email_gate` | `(d, lang = 'tr')` | 943 | Ücretsiz deneme kötüye kullanımı freni (2026-07-19): DENEME kullanıcısı e-postasını |
| `adil_kullanim_html` | `(lang = 'tr', href = '/yasal/iade')` | 961 | Adil kullanım ibaresi + koşul LİNKİ (4 yüzeyde ortak). |
| `fiyat_nudge` | `(er) -> str` | 989 | Hekime fiyat yolu gösterilmeli mi? Döner: "duvar" \| "bitiyor" \| "". |
| `comp_metni` | `(lang: str = 'tr', durum: str \| None = None, kalan: int \| None = None)` | 1023 | Bedelsiz (`subscription_status='comp'`) erişimi olan hekime ÖZEL cümle (Selim). |
| `_duvar_yuzeyi` | `(yuzey: str) -> str` | 1070 | Duvarın hangi uçta gösterildiğini bul — ÇAĞIRANI DEĞİŞTİRMEDEN (#490b). |
| `_erisim_402` | `(d, conn, lang, yuzey: str = '')` | 1098 | Erişim kapısı — **ücretli her ucun İLK kontrolü**. Geçerse None, geçmezse 402 JSON. |
| `_gunluk_fren_429` | `(d, conn, lang, yuzey: str = '')` | 1187 | Sessiz günlük tavan — aşıldıysa 429, aşılmadıysa None. |
| `butce_nudge` | `(h, lang: str = 'tr') -> str` | 1236 | `/chat` üst bandı için BÜTÇE metni — `fiyat_nudge` kalıbı (§6 "hekim soruyu yazıp |

