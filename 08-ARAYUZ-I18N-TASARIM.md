# 08 — ARAYÜZ, i18n VE TASARIM SİSTEMİ

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> Burada "tasarım tercihi" görünen çoğu şey **ölçülmüş bir erişilebilirlik ya da mobil
> dokunma-hedefi kararıdır.** Değiştirmeden önce ilgili kapıyı oku.

---

## 1. Render modeli — şablon motoru YOK

Klivance'ta Jinja, React, SPA ve **build adımı yoktur**. Sayfa gövdeleri **Python string
sabitleridir** ve çalışma anında birleştirilir:

```python
html = head(baslik, desc=…) + nav(doctor, active="chat", lang=lang) + _CHAT_BODY
if lang == "en":
    html = en(html)          # EN_PAIRS ile TR→EN
return HTMLResponse(html)
```

**Neden:** böylece her yüzey **CI'da metin olarak ölçülebilir**. 320 doğrulama kapısının
büyük kısmı üretilen HTML'i sondalar. Şablon motoruna geçmek bu kapıları kırar.

### 1.1 Paylaşılan sözlük — `saglik/app/webutil.py`

⚠⚠ **BU MODÜL `main.py`'DEN HİÇBİR ŞEY IMPORT ETMEZ.** Ederse dairesel import doğar ve
**uygulama açılmaz**.

⚠⚠ **SIRA ÖNEMLİ:** `_ANALYTICS_TAG`, `_ANALYTICS_PATH_ONLY`'yi **değer olarak** içerir
(AST ile doğrulandı). Tanım sırası `main.py`'deki orijinal sıradır; yeniden sıralarsan
`NameError` alırsın. **Yeni adı bağımlılığından SONRA yaz.**

| Ad | Rol | Dokunma uyarısı |
|---|---|---|
| `esc(v)` | **XSS kalkanı** | Her HTML üretimi buna bağlı |
| `en(html)` | i18n çevirisi | **Apostrof tuzağı buradan doğar** (§4.2) |
| `head(title, desc, canonical, robots, izsiz=)` | Ortak `<head>` | |
| `nav(doctor, active, lang)` | Sol ray | Dil-duyarlı; `_rh()` ile çıkışlıda kimlik-isteyen rayı `/kayit`'a çevirir |
| `ust_serit(...)` | Sayfa üst şeridi (D4) | ← geri · başlık · rozet · meta · **TEK birincil CTA** |
| `_doctor(request, conn)` | Oturum çözümü | **Kimlik kararlarının tek kaynağı** |
| `get_lang(request)` | Dil kararı | |
| `_ANALYTICS_TAG` / `_ANALYTICS_PATH_ONLY` | Ölçüm etiketi | `klv_notrack` tag'in **TAMAMINI** soyar; path-only onun **içinde** → **AYIRMA** |
| `_erisim_402` / `_gunluk_fren_429` | Erişim kapıları | Bkz. `07-KIMLIK-ERISIM-ODEME.md` §2.2 |

⚠ **Yeni kimlik-isteyen ray ögesi eklersen `_rh()` ile SAR.**

### 1.2 Gövde modülleri (`*_body.py`) — SAF

**Yalnız saf sabit konur:** f-string **yok**, modül durumu **yok**, import **yok**.
Dinamik bir şey gerekiyorsa rota modülünde kalır.

| Dosya | Yüzey |
|---|---|
| `landing.py` | Herkese açık satış sayfası (TR + EN **ayrı gövde**) |
| `chat_body.py` | `/chat` — HTML + JS (kaynak pill'leri, `addSources`, `srcLabel` burada) |
| `cases_body.py` | `/cases` şablonları |
| `calc_body.py` | `/hesaplayicilar` — istemci-tarafı JS |
| `ref_body.py` | `/reference` |
| `recete_body.py` | `/recete-kontrolu` + `/bobrek-doz` |
| `etkilesim_body.py` + `etkilesim_page.py` | `/etkilesim` |
| `hastalik_body.py` | `/hastaliklar` |
| `cal_body.py` | `/takvim` |
| `panel_body.py` · `kokpit_body.py` | `/panel` · analiz kokpiti |
| `legal_body.py` | `_LEGAL` (TR) + `LEGAL_EN` |
| `guide.py` · `guvenlik.py` | `/kilavuz` · `/guvenlik` |
| `abonelik_body.py` | Abonelik sayfaları |
| `ui_css.py` | **Uygulama CSS'i — TEK KAYNAK** (TR ve EN aynı sabiti kullanır) |
| `i18n_pairs.py` | `EN_PAIRS` |
| `chat_goruntu_js.py` · `olcum_js.py` | Saf JS sabitleri |
| `form_listeler.py` · `ornek_sorular.py` | **Saf veri** (import 0) |

---

## 2. TASARIM SİSTEMİ

### 2.1 Tokenlar — `ui_css.CSS` `:root`

| Token | Değer | Rol |
|---|---|---|
| `--ink` | `#0C3B3E` | Ana koyu (petrol) |
| `--ink-2` | `#0A6B4E` | ⚠ **Yeşil METİN için bunu kullan** (WCAG) |
| `--verify` | `#12B886` | Yeşil — **kanıt/CTA dozu** |
| `--verify-soft` / `--verify-ink` | `#E4F3EE` / `#053A2C` | |
| `--slate` | `#5C6B6C` | İkincil metin |
| `--line` | `#DCE5E3` | Kenarlık |
| `--mist` / `--paper` | `#E9F1EF` / `#FBFDFC` | |
| `--alert` / `--alert-soft` | `#D64545` / `#FDF3F3` | |
| `--amber` / `--amber-ink` / `--amber-soft` | `#E8A33D` / … / `#FDF6EA` | ⚠ **`--amber` DOLGU, `--amber-ink` METİN** |
| `--surface-card` / `--ink-body` | `#fff` / `#0E1B1C` | |
| **`--rail-w`** | **`272px`** | Sol ray genişliği — **TEK genişlik, admin DAHİL** |
| **`--altbar-h`** | **`64px`** | Mobil alt çubuk yüksekliği |

⚠ `--gold` **ÇIKARILDI** (v2: dijital çekirdek 2-renk Ink + Verify; gold yalnız basılı-prestij).

### ⚠⚠ İKİ AYRI KONTRAST EŞİĞİ — karıştırma

| Rol | Eşik |
|---|---|
| **Kenar** (border) | **3:1** |
| **Metin zemini** | **4,5:1** |

`--line` `#E6ECEC` → `#DCE5E3` değişimi **AA düzeltmesi DEĞİL, görünürlük ayarıdır**
(kenar beyaz kartta 1,19 → 1,28; **ikisi de 3:1 ALTINDA** — "AA tamam" sanma).
`--slate` bu değer üzerinde **4,334 = eşik altı**; 145 kullanımın 4'ü zemin ve
**dördü de metinsiz** (ölçüldü). **Zemine METİN koyan yeni kural yazan ÖNCE bu oranı ölçsün.**

### 2.2 ⚠⚠ İKİ KABUK ÖLÇÜSÜ, TEK KAYNAK — **SAYI YAZMA, TOKEN KULLAN**

`--rail-w` ve `--altbar-h`. `--altbar-h` beş dosyada **elle 56px yazılıydı**
(chat çekmecesi/scrim, hesaplayıcılar, panel); biri unutulunca çubuk içeriğin **üstüne biner**.

**Kural: kabuk ofseti yazacaksan `var(--rail-w)` kullan, sayı yazma.**
Kapılar: `kabuk_ofset_verify` · `uzay_suruklenme_kapisi` · `token_tek_kaynak_kapisi` ·
`amber_token_kapisi`.

### 2.3 Tipografi

- **Sora** (başlık) + **Inter** (gövde) — Google Fonts
- `--font-head` token'ı üzerinden

### 2.4 Düzen

Apilex-tarzı: sol ray (`.rail`) → mobilde alt-bar (`@820px`). Sonuç kartları,
`.field-grid`, `.apx-table`.

⚠ **Paylaşımlı mobil `@media` kurallarını CSS sabitinin SONUNA yaz** — base'i source-order
ile ezsin. Başa koyarsan sonraki base **ezer**.
✅ Uygulama CSS'i tek-kaynak → `@media` TR+EN **otomatik senkron**.

### 2.5 Sol ray — ölçülmüş kararlar

| Karar | Tarih | Durum |
|---|---|---|
| `.rail-dar` / `--rail-dar-w` (64px admin dalı) | 2026-08-08 | **KALDIRILDI** (founder). Kapı `admin_mobil_bar_verify` |
| Kısa ray etiketi "Sor"/"Ask" | 2026-08-03 (#45b) | **KALDIRILDI** → bileşik "Klivance'a Sor" / "Ask Klivance" |
| "Hastalar" | 2026-08-03 | → **"Hastalarım" / "My Patients"** |
| Ray öge SIRASI | 2026-08-03 | `/panel` sidebar'ına uyduruldu: Klivance'a Sor → Hastalarım → Referans → Araçlar → Takvim (Panel/Ana sayfa/Hesap yerinde) |
| Ters yön (paneli raya çevirmek) | | **REDDEDİLDİ** — bir hedefe **TEK ad** |
| Ray puntosu | 2026-08-11 | 9,5 → **11px** (deponun okunabilirlik tabanının altındaydı) |

**Bedeli ölçüldü ve KAPANDI** (bağımsız ölçüm 2026-08-08 + 2026-08-11):
- "Daha fazla" taşma menüsü geldikten sonra 360×740 TR'de en küçük dokunma hedefi **44×55px**
  (menü açık/kapalı ikisi de; açık menü satırları 344×48).
- ✅ TR+EN × 360×740 + 320×700 **dördünde de** en küçük hedef **45×55px**, satır 1,
  dikey/yatay kırpılma 0, bar+sayfa taşması 0 → **bedel sıfır, hedef 1px BÜYÜDÜ.**
- Para yolu (Giriş/Kayıt/Hesap) `.rail-foot` kuralıyla **44px korunur**.

Kapılar: `ray_olcum_verify` · `ray_sira_verify` · `admin_mobil_bar_verify` ·
`ray_panel_verify` (SKIP) · `alt_bar_hedef_verify` (SKIP).

### 2.6 Marka adlandırma — **DEĞİŞMEZ**

| Yüzey | Ad |
|---|---|
| Chat | **"Klivance'a Sor" / "Ask Klivance"** (kıvrık kesme **U+2019**) |
| Hasta defteri | **"Hastalarım" / "My Patients"** |
| Reddedilen | **"Tanı"** (regülasyon) |

---

## 3. XSS VE ÇIKTI GÜVENLİĞİ

| Kural | Detay |
|---|---|
| `webutil.esc(v)` | Kullanıcı verisi HTML'e basılmadan önce **zorunlu** |
| Kaynak pill metni | **`escH` zorunlu** — `E()` `innerHTML` kullanır → XSS |
| `_css_yorum_mw` | `<style>` yorumlarını yanıttan soyar (#93b) |
| `_security_headers_mw` | Güvenlik yanıt başlıkları (M10) |
| `reference._like_escape` | ILIKE'a kullanıcı girdisi → `%`/`_` kaçır (yoksa `a_pirin`→ASPIRIN yanlış kart / tek `%`→seq-scan) |

Kapılar: `xss_reference_verify` · `css_yorum_verify` · `olu_css_kapisi` · `like_escape_verify` ·
`en_enjeksiyon_verify`.

---

## 4. i18n

### 4.1 Dil kararı

```
get_lang(request):  ?lang= query  >  cookie klv_lang  >  Accept-Language
                    (birincil etiket 'tr' → tr, değilse en)  >  tr
```

| Yüzey | Yöntem |
|---|---|
| Uygulama HTML'i | `i18n_mw` → `webutil.en()` → `EN_PAIRS` (TR→EN, **uzun-önce sıralı**) + `<html lang>` swap |
| **Landing + legal** | **Kendi İngilizce sürümünü servis eder** (`LANDING_HTML_EN`, `LEGAL_EN`) → middleware **no-op** |
| `/hesaplayicilar` | `CALC_LANG` ile **JS tarafında** → `i18n_mw`'den **MUAF** |
| `/reference` | `REF_LANG` sunucudan enjekte — bu sayfada **`KLV_LANG` YOK** |
| Runtime JS | `KLV_LANG` (cookie'den) ile dallan |

**Yeni app metni eklerken:** statik ise `EN_PAIRS`'e çift ekle **veya** sunucuda `lang` ile
üret (dinamik/çoğul için **tercih et**).

### 4.2 ⚠⚠ APOSTROF TUZAĞI — sessiz P0

`en()` EN modunda **TÜM HTML'i (script blokları DAHİL)** `EN_PAIRS` ile değiştirir.

```
EN çevirisi düz kesme ( ' ) içeriyor
  + TR anahtarı tek-tırnaklı bir JS string içinde   →  w.innerHTML='...<p>TR metin</p>...'
  ⇒ string kapanır ⇒ SyntaxError ⇒ O SAYFANIN TÜM JS'İ ÇÖKER (sessiz; sayfa boş yüklenir)
```

**KURAL: `EN_PAIRS` İngilizce değerlerinde kıvrık kesme `’` (U+2019) kullan.**

**Doğrulama:**
```
head() + nav(doc,'chat','en') + _CHAT_BODY  →  en()  →  <script> çıkar  →  node --check
```

Bu, *"EN'de sohbet geçmişi kayboluyor"* bug'ının köküydü (2026-07-15).

### 4.3 ⚠ Çakışma tuzağı

`i18n_mw` EN modunda **JS string literallerini de çevirir** → TR literal'ini `EN_PAIRS`
anahtarlarıyla (**"Kaynaklar"** gibi) **çakışmayacak** şekilde yaz
(ör. `"Tıbbi kaynaklar taranıyor"`) ya da JS'te `KLV_LANG` ile dallan.

### 4.4 Çeviri MUAFİYETLERİ (bilinçli)

| Yüzey | Neden |
|---|---|
| **Admin paneli** | Bilerek **Türkçe tek-dilli** → **admin metnini `EN_PAIRS`'e EKLEME** |
| **Unvan/branş etiketleri** | **TR-KANONİK** → `EN_PAIRS` çifti **EKLEME** |
| `/hesaplayicilar` | JS tarafında iki dilli |

Kapılar: `i18n_muafiyet_verify` · `parite_maske_verify` · `en_sayi_bicim_verify` ·
`dil_ctop_verify` · `kart_dili_verify`.

### 4.5 ⚠⚠ YANIT DİLİ ≠ ARAYÜZ DİLİ

Modele giden dil **SORUDAN** tespit edilir (`cds/dil.py::yanit_dili`, LLM'siz; belirsizse
önceki turlar → `get_lang`). `chat_routes`ta **5 üretici çağrı** ondan geçer; **hata metni ve
paneller UI dilinde kalır.** Kapı: `yanit_dili_verify` (CI).

---

## 5. SAYFA-BAZLI ÖZEL KURALLAR

### 5.1 `/chat` — `chat_body.py`

- **Kaynak pill'leri burada** (`main.py` DEĞİL — Faz 6'da taşındı): `addSources` / `srcLabel`.
  String'ler `chat._sources_from`; regex ve etiket haritası `cds/kaynaklar.py`'den
  **`chat_routes` enjekte eder**.
- Tıklanabilir: `europepmc:PMCxxx` · `openFDA:ilaç` (→`/reference?q=`) · `pubmed:pmid`
- ⚠ `statpearls` / `livertox` / `TİTCK` / `FAERS` / `ICD-11` = **ölü span BİLE BİLE**
  (deep-linklenemez; **başlıkla arama-linki EKLEME** = yanlış-makale, denetlenebilirlik
  vaadini bozar)
- ⚠ Pill metni **`escH` zorunlu**
- Üst bant: bütçe/fiyat nudge (`butce_bant.py`, `webutil.fiyat_nudge`, `butce_nudge`)

### 5.2 `/hesaplayicilar` — `calc_body.py`

⚠⚠ **TIBBİ GÜVENLİK DEĞİŞMEZ:** formül ekler/değiştirirsen `node scratchpad/calc_verify.js`.
Ücretli reklam inişi. Arama normalizasyonu · üç kapı · ölçülmüş vakalar →
`docs/kamil-ozellik-defteri.md`.

### 5.3 `/etkilesim` — herkese açık, LLM'siz, ücretli iniş

- Altı durum, **hiçbirinde yeşil/aklama dili yok** (CI'da taranır)
- `id="etk-sonuc"` **silinmez**
- ⚠ Eski `?a=&b=` formatı **korunmuştur** (canlı reklam inişi, kıran P0)

### 5.4 `/reference` → 303

Gövdesi `/recete-kontrolu` içine taşındı (#356b, founder 2026-08-27).
Çıkışlıda bölüm **görünür ama KİLİTLİ** (`/api/suggest` + `/api/query` kimlik ister;
çalışmayan arama kutusu yayınlanmaz); ücretsiz üç kart **açık** kalır.
⚠ `ref_body.py` **değişmedi** — gövde çalışma anında kesilir (`_ref_parcala`, fail-closed).
⚠ Derin bağlantı **`?q=` DEĞİL** `localStorage klv_refq` (hassas sorgu URL'e/GA4'e sızmasın).

### 5.5 Landing

- Mobil hero sıralaması · CTA auth-aware (`.cta-buy`) · fold ölçümü →
  `docs/kamil-ozellik-defteri.md`; sayı `scratchpad/inis_fold_verify.py` çıktısından
- **FDA Kriter-3/4 kalkan cümleleri YERİNDE** (commit `c98b72e`) — **SİLME**
- Fiyat **yer tutucudur** (`{{klv.fiyat.…}}`) → `fiyat.doldur()` (bkz. `07` §4)
- Masaüstü kayıt sahnesi `.auth-reg`

Kapılar: `inis_fold_verify` · `inis_kaynak_beyani_verify` · `guvence_fold_verify` ·
`inis_kontrast_verify` (SKIP) · `ana_sayfa_donus_verify`.

### 5.6 404 ve yönlendirmeler

`main._not_found_handler` — çıplak `{"detail":"Not Found"}` yerine **markalı, dil-duyarlı**
404 (API yolları hariç → JSON).

| Eski | Yeni | Kod |
|---|---|---|
| `/interactions` | `/etkilesim` | 301 |
| `/chatgpt` | `/` | 302 (sorgu dizesi korunur) |
| `/reference` | `/recete-kontrolu` | 303 |

Kısa linkler: `/ig` `/fb` `/li` `/tg`.

---

## 6. MOBİL

| Kural | Değer |
|---|---|
| Sol ray → alt bar | `@media (max-width: 820px)` |
| İkinci kırılım | `640px` |
| En küçük dokunma hedefi (ölçülmüş) | **45×55px** (TR+EN × 360×740 + 320×700) |
| Para yolu hedefi | **44px** (`.rail-foot`) |
| Alt çubuk yüksekliği | `var(--altbar-h)` — **sayı yazma** |

Test boyutları: **360×740** ve **320×700**, TR **ve** EN.

---

## 7. ARAYÜZ DEĞİŞTİRİRKEN KOŞULACAK KAPILAR

| Dokunduğun yer | Kapı |
|---|---|
| CSS / token | `token_tek_kaynak_kapisi` · `amber_token_kapisi` · `olu_css_kapisi` · `css_yorum_verify` · `uzay_suruklenme_kapisi` · `panel_tipo_kapisi` |
| Sol ray | `ray_olcum_verify` · `ray_sira_verify` · `admin_mobil_bar_verify` |
| `EN_PAIRS` / i18n | `i18n_muafiyet_verify` · `parite_maske_verify` · `en_enjeksiyon_verify` · `en_sayi_bicim_verify` |
| `/chat` gövdesi | `chat_sayfa_js_check` · `chat_liste_verify` (JS) · `_kamil_yanit_js_check` · `kanit_ciz_verify` |
| `/hesaplayicilar` | `calc_verify` (JS) · `calc_arama_verify` (JS) · `calc_sayfa_js_check` · `calc_cta_verify` · `calc_seo_verify` · `calc_v9_duzen_verify` |
| `/etkilesim` | `etkilesim_govde_verify` · `etkilesim_dil_sira_verify` |
| Landing | `inis_fold_verify` · `guvence_fold_verify` · `inis_kaynak_beyani_verify` |
| `/cases` | `cases_govde_verify` · `kart_form_sozlesme_verify` |
| Atıf arayüzü | `atif_arayuz_kapsam_verify` · `rozet_kapsam_verify` · `pill_rol_kapisi` |
| XSS | `xss_reference_verify` |

---

**Sonraki:** [`09-ADMIN-VE-ANALITIK.md`](09-ADMIN-VE-ANALITIK.md)
