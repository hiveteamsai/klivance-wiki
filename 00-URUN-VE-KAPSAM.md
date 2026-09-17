# 00 — ÜRÜN VE KAPSAM

> Devralan ekibin **ilk okuyacağı** dosya. Buradaki hiçbir madde "geliştirme tercihi" değil;
> her biri ya bir regülasyon sınırı, ya bir lisans kısıtı, ya ölçülmüş bir founder kararıdır.
> Değiştirmeden önce `16-ISLETME-KURALLARI-VE-KARARLAR.md`'yi okuyun.

---

## 1. Ürün tek cümlede

**Klivance**, Türk hekimler için **klinik karar-destek (CDS — Clinical Decision Support)**
SaaS'ıdır. Açık tıbbi literatür ve kurumsal veriyi (ilaç etiketleri, ilaç etkileşimleri,
klinik korpus, FAERS yan etki bildirimleri, ICD-11, TİTCK KÜB) tarar ve hekime **atıflı,
denetlenebilir** yanıt verir.

**Klivance TANI KOYMAZ.** Karar-desteği sunar; son karar hekimindir. Bu bir pazarlama
cümlesi değil, ürünün mimari sınırıdır (bkz. §4).

| | |
|---|---|
| **Canlı (staging)** | https://klivance.onrender.com |
| **Hedef alan adı** | https://klivance.com |
| **Pazar** | Türkiye (TR-önce). ABD kolu Faz-2, kod hazır ama **kapalı**. |
| **Arayüz dili** | TR varsayılan · EN otomatik + toggle |
| **Kullanıcı** | Hekim, tıp öğrencisi, diş hekimi, eczacı (kayıt formundaki unvan listesi) |
| **Tüzel kişi** | ALFA 4N SAN. TİC. LTD. ŞTİ. (VKN 0510246142, Biga/Çanakkale) — **veri sorumlusu** |
| **ABD tüzel kişi** | Klivance, LLC (Delaware, Stripe Atlas) — Faz-2 |

---

## 2. Ne yapar — hekime görünen yüzeyler

Ürün tek bir "chatbot" değil; **birbirini besleyen sekiz yüzeyden** oluşur. Rota karşılıkları
`03-ROTA-ENVANTERI.md`'de.

### 2.1 Klivance'a Sor — `/chat`
Ana CDS yüzeyi. Hekim klinik soru sorar; sistem kapsam kilidi → RAG → akıl yürütme
zincirinden geçirip **kaynak atıflı** yanıt üretir (`05-CDS-YANIT-ZINCIRI.md`).
- Yanıt iskeleti **zorunlu ve sabittir**: `Kısa yanıt` / `Gerekçe` / `⚠ Dikkat` /
  `Pratik öneri` + kapanış satırı (EN: `Bottom line` / `Rationale` / `⚠ Caution` /
  `Practical guidance`). Arayüz bu iskelete güvenebilir.
- Yanıt **akar** (streaming). İptalde bile `record_usage` yazılır.
- Yanıt altında **tıklanabilir kaynak pill'leri**: `europepmc:PMCxxx`, `openFDA:ilaç`,
  `pubmed:pmid`. Diğerleri (StatPearls/LiverTox/TİTCK/FAERS/ICD-11) **bilerek ölü span**
  — deep-link'lenemedikleri için başlıkla arama-linki **eklenmez** (yanlış makaleye
  götürmek, denetlenebilirlik vaadini bozar).
- Hasta bağlamı bağlanabilir (`case_ctx`) → alerji, gebelik/emzirme, böbrek fonksiyonu,
  boy/kilo prompta girer.
- Görsel/PDF eklenebilir (görüntü ön-okuma — §2.7).

### 2.2 Hastalarım — `/cases`
Pseudonim **veya gerçek adlı** hasta defteri. Ziyaret notları, tahliller, alerji çipleri,
doğurganlık durumu, boy. Şifreli dosya kütüphanesi (`app.case_file.content_enc` = Fernet).
Zaman tüneli arşivi (`app.case_timeline`) sentezi üretildiği tarihle kalıcı saklar.

### 2.3 Reçete & Referans — `/recete-kontrolu`, `/reference`
- **Reçete kontrolü ailesi**: böbrek doz kontrolü (`/bobrek-doz`), etkileşim, alerji
  çapraz-reaktivite, duplikasyon. LLM'siz, deterministik motorlar.
- **Referans arama**: ilaç kartı + klinik sorgu. `/reference` artık **303 yönlendirme**;
  gövdesi `/recete-kontrolu` içine taşındı (#356b).
- ⚠ Derin bağlantı `?q=` **değil** `localStorage klv_refq` — hassas sorgu URL'e/GA4'e sızmasın.

### 2.4 İlaç etkileşimi — `/etkilesim`
**Herkese açık, ücretsiz, LLM'siz, kimlik istemez.** Ücretli reklam inişi (Google + Meta).
2–8 ilaç taranır (`?d=a,b,c`). ⚠ Eski `?a=&b=` formatı **korunmuştur** (canlı reklam inişi).
Altı durum; **hiçbirinde yeşil/aklama dili yok** (CI'da taranır). AI yorumu ayrıdır ve
hesap ister.

### 2.5 Hesaplayıcılar — `/hesaplayicilar`
İstemci-tarafı klinik hesaplayıcılar (JS), 0 maliyet, ücretli reklam inişi.
⚠ Formül ekler/değiştirirsen `node scratchpad/calc_verify.js` koş — **tıbbi güvenlik değişmezi**.

### 2.6 Hastalıklar — `/hastaliklar`
Hastalık monografi modu → `/chat?hst=`. Kanıt paketi `cds/hastalik_kanit.py`,
prompt `prompts.HASTALIK_SYSTEM`.

### 2.7 Görüntüleme — iki ayrı yol
| Yol | Girdi | Çıktı | Durum |
|---|---|---|---|
| **YOL 1** | Radyoloji **rapor metni** | RAD satırı (`cds/rad_cikar.py`) → lezyon şeridi + takip randevusu | Açık |
| **YOL 2** | **Görüntünün kendisi** (ön-okuma) | `pipeline.goruntu_on_okuma` tek giriş; ayrı kolon | Env `KLIVANCE_GORUNTU_ONOKUMA` **yokken KAPALI** |

⚠ YOL 2, #620b'deki "hiç okuma" duruşunun **bilinçli değişimidir**. `extracted_text`e
YAZILMAZ; `DOCTOR_SYSTEM` DOKUNULMAZ. Sözleşme: `docs/goruntuleme-sozlesme-2026-09-13.md`.

### 2.8 Takvim + Panel + Admin
- `/takvim` — randevu ve klinik takip. **Duvar saati** modeli (bkz. §6).
- `/panel` — hekim kokpiti (yargım deseni uyarlaması).
- `/admin*` — kurucu paneli. Analytics'e **sızmaz** (`_notrack_mw`).

---

## 3. Ne YAPMAZ — kapsam dışı ve reddedilmiş

| Reddedilen | Gerekçe |
|---|---|
| **Tanı koymak** | Regülasyon. Menüde "Tanı" adlandırması REDDEDİLDİ; "Klivance'a Sor" kullanılır. |
| **Hekime API anahtarı vermek** | Klivance tek Anthropic hesabı tutar. |
| **Hekime/branşa yönlendirme** | "Uzmana danışın", "… konsültasyonu düşünülebilir", "karar hekimindir, ona bırakın" **YASAK** (founder 2026-08-15). Fiilî girişim, yönetim adımı olarak yazılır. |
| **Rol eskalasyonu** | Tıp öğrencisi de hekimle **aynı derinlikte** yanıtlanır. Öz-beyan ("psikologum") perspektifi değiştirmez. `rol_eki` her unvanda BOŞ. |
| **Kredi sistemi** | 2026-08-21'de kaldırıldı → sınırsız abonelik + deneme (§5). |
| **Top-up satın alma** | 2026-08-21'de kaldırıldı. |
| **`include_router`** | Ölçülerek reddedildi — rota envanteri okuyan iki ağ sessizce körelir. |
| **Yeni kişi/olay tablosu** | `app.person`/`app.account_event` reddedildi; sıfır yeni kalıcı veri ilkesi. |
| **UpToDate'i adıyla hedef alan reklam** | Asimetrik hukuki risk. |

---

## 4. Regülasyon duruşu — FDA CDS ve "tanı koymaz"

Yürürlükteki FDA CDS rehberi: **29 Ocak 2026**. Riskli olan **Kriter 4**: hekim, önerinin
dayanağını **bağımsız gözden geçirebilmeli** ve öneriye **birincil olarak dayanmamalı**.

⚠⚠ **"Tanı koymaz" TEK BAŞINA muafiyet SAĞLAMAZ** — FDA fonksiyona bakar. 2026 değişikliği
muafiyet değil, **geri çekilebilir enforcement discretion**'dır.

Bunun mimariye yansıması:
1. **Her iddia atıflıdır** — `cds/kaynaklar.py` atıf zincirinin tek doğru kaynağıdır.
2. **Atıf sadakati ölçülür** — `cds/faithfulness.py` (LLM'siz, deterministik, salt-gözlem).
3. **Rozet, modele giden kanıtla AYNI kümeden üretilir** — olmayan kart rozet almaz
   (sahte provenans), olan almalı (yoksa meşru atıf elenir, hekim uydurma sanır).
4. **`dose_check` / `redflag` tek-çıktı yönlendirmesinden kaçınır.**
5. Landing'deki **Kriter-3/4 kalkan cümleleri YERİNDE** (commit `c98b72e`) — **SİLME.**

Kaynak: `docs/rakip-bosluk-arastirma-2026-07-16.md`.

---

## 5. İş modeli — erişim ve para

⚠⚠ **FİYAT SAYISI VE SÜRE BU DOSYAYA YAZILMAZ** (bir kez bayatladı). Tek doğru kaynaklar:

| Bilgi | Kaynak (koddan oku) |
|---|---|
| Deneme süresi | `credits.DENEME_GUN` |
| Günlük soru tavanı | `credits.AGIR_GUNLUK_TAVAN` / `AGIR_GUNLUK_TAVAN_TRIAL` |
| Tavana sayılan modlar | `credits.AGIR_MODLAR` |
| Satılabilir planlar | `credits.SATILABILIR_PLANLAR` |
| Fiyat | `iyzico._PLAN_PRICE_DEFAULT` |
| Dosya/ek limitleri | `credits.MAX_ANALYZE_FILES`, `MAX_ATTACH_COUNT`, `MAX_ONOKUMA_GORSEL` |

**Model (2026-08-21 founder kararı):**
- Aktif abonelik → **sınırsız**
- Yeni kayıt → `DENEME_GUN` gün deneme, günde `AGIR_GUNLUK_TAVAN_TRIAL` soru
  (⚠ "sınırsız deneme" dili **YASAK**)
- İkisi de yoksa → duvar

⚠⚠ **"SINIRSIZ" İLANI + GİZLİ TAVAN = ADİL KULLANIM İBARESİ ZORUNLU** (yanıltıcı ticari
beyan riski): fiyat/landing'de görünür ibare + yasal metinde madde. **İbareyi kaldıran,
tavanı da kaldırmalıdır.** Üretici: `credits.adil_kullanim_ibaresi()`.

- Fiyat **KDV dahil**. Yıllık = aylık × 10 (2 ay bedava — **tek avantajı budur**).
- Sağlayıcı: **iyzico** (TR/TRY, Checkout Form, 3DS). Stripe ABD kolu için hazır, **pasif**.
- ⚠ Otomatik yenileme (#774c) **kodda var, bayrak KAPALI** (`credits.iyz_recurring()`).
- ⚠⚠ **FİKTİF İŞLEM YASAĞI**: çerçeve sözleşme founder'ın kendi kartıyla kendine satışını
  yasaklar (tespitinde tek taraflı durdurma).

---

## 6. Ürünün beş "değişmez"i

Bunlar tek tek ölçülmüş hatalardan doğdu. Her biri CI kapısıyla çivilidir.

### D1 — FAIL-OPEN YASAK: "çalıştırılamadı" ≠ "temiz"
Hata / JSON-parse hatası / hız sınırında `status="unavailable"` + `critic_status` döner;
UI amber "elle doğrulayın" basar, **asla yeşil tik**. Bu sınıf hata "yanlış cevap" değil
**YANLIŞ GÜVEN** üretir — bu üründe daha tehlikeli.

### D2 — Son adım bağlanmamış (en sık hata sınıfı)
İş yapılmış, üretilen şey **tüketiciye bağlanmamış**, yüzeysel bakınca "tamam" görünüyor.
**Kural: "X eklendi" demeden önce X'i TÜKETEN katmanda ölç** (HTML/JSON/canlı URL).
Birim testi fonksiyonu test eder, **bağlantıyı değil** — 26/26 yeşilken özellik hekime
görünmüyordu.

### D3 — Hassas sorgu piksele GİTMEZ
`_ANALYTICS_PATH_ONLY`: `/reference?q=`, `/cases/*`. Meta pikselinin JS kolu yüklenmez.
⚠⚠ `<noscript>` kolu **muaftır** → "hiç yüklenmez" hükmü kurma (#105b).
Yeni hassas-sorgulu yol eklersen deseni oraya ekle.

### D4 — Gün sınırı = TÜRKİYE saati
`store._TR_BUGUN` / `_tr_gun()` tek kaynak. `now()::date` Render'da UTC → TR 00:00–03:00
arası DÜNÜ "bugün" sanıyordu. ⚠ Sabit `+03` ofset **yazma**; zaman dilimi **adı** kullan.
Takvim randevusu ise **duvar saati**: `_appt_json` ofsetsiz ISO döner, sunucu `_wall()`
naive yerel — bozarsan ızgara 09:00 / yan panel 12:00 çelişir.

### D5 — Ölçüm sinyali GENİŞ, optimizasyon sinyali DAR
Aynı olayı iki amaca koşma. Üç kayıt olayı, üç ayrı amaç:
| Olay | Amaç | Kapsam |
|---|---|---|
| GA4 `sign_up` | Google Ads **birincil dönüşümü** | `/chat?welcome=1` |
| Meta `CompleteRegistration` | **Optimizasyon** | **DAR**: `email_verified` VE unvan ≠ `Tıp Öğrencisi` |
| Meta `klv_kayit` | **Hariç tutma** | **GENİŞ**: filtresiz |

---

## 7. Rekabet konumu

- **Asıl rakip UpToDate değil, ücretsiz OpenEvidence** — ama ABD NPI kilidi + AB geo-block
  nedeniyle TR'de rakip değildir.
- Konumlandırma: **"UpToDate'in YANINA, yerine değil."**
- ⚠⚠ **SEKİZ İDDİA DOĞRULAMADA ÇÜRÜDÜ** — landing'e/reklama **koyma**: "daha güncel" ·
  "UpToDate LLM'ini gizliyor" · "UpToDate'ten pahalıyız" (aksine ~%15 ucuzuz) · "içeriğin
  2/3'ü uzman görüşü" · "araması kötü" · "FDA tek-çıktı kısıtını kaldırdı" · "Nature
  Medicine kanıtladı" · COI için "rüşvet/gizleme" (gerçeği **beyan uyumsuzluğu**).
  Her birinin çürütme kanıtı `.claude/agents/hasan.md`'de.

---

## 8. Bu export'un kapsamı ve sınırı

**İÇERİR:** kaynak kod mimarisi, 142 rota, 55 tablo, 1.514 fonksiyon + 55 metot, 320 doğrulama kapısı,
env değişkeni **adları**, deploy topolojisi, iş kuralları, bilinen tuzaklar, teknik borç.

**İÇERMEZ (bilinçli):**
- **Hiçbir sır/anahtar değeri** — yalnız env **adları** ve ne işe yaradıkları.
- **Satır sayıları / KB hacimleri** — bayatlar; ölçüm reçetesi verilir.
- **Kişisel veri** — hiçbir hekim/hasta kaydı örneği yok.
- **Hukuki yorum** — founder kuralı: hukuki boyut istenmedikçe üretilmez. Kodda duran
  kapılar (NC/ND süzgeci, lisans alanına birebir kopya, StatPearls görünmezliği)
  **yorum değil veri ve koddur**; koşar, tartışılmaz.

---

**Sonraki:** [`01-MIMARI.md`](01-MIMARI.md) — katmanlar ve istek yaşam döngüsü.
