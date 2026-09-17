# 16 — İŞLETME KURALLARI VE VERİLMİŞ KARARLAR

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
>
> Bu dosya **"neden böyle"** sorularının cevabıdır. Buradaki maddeler tercih değil
> **karar**dır: her biri ya founder tarafından verilmiş, ya ölçülerek kurulmuş, ya da bir
> regülasyon/lisans sınırıdır. **Değiştirmeden önce oku; çoğu tekrar tartışıldı ve aynı
> sonuca varıldı.**

---

## 1. DEĞİŞMEZ ÜRÜN KURALLARI

| # | Kural | Kaynak |
|---|---|---|
| Ü1 | **"Tanı koymaz"** konumlandırması korunur | Regülasyon (FDA CDS) |
| Ü2 | Menüde **"Klivance'a Sor"**; **"Tanı"** adlandırması **REDDEDİLDİ** | Regülasyon |
| Ü3 | **Yönlendirme yasağı** — hiçbir hekime/branşa/uzmana yönlendirme | Founder 2026-08-15 |
| Ü4 | **Profesör tonu** — tıp öğrencisi de hekimle aynı derinlikte | Founder 2026-08-15 |
| Ü5 | **Rol eskalasyonu YOK** — `rol_eki` her unvanda BOŞ; öz-beyan perspektifi değiştirmez | Founder, 2026-08-08 + ikinci teyit |
| Ü6 | Chat adı = **marka bileşiği** "Klivance'a Sor" / "Ask Klivance" (kıvrık kesme U+2019) | 2026-07-14 ajan paneli |
| Ü7 | **Bir hedefe TEK ad** — sol menü de bileşiği kullanır; ters yön (paneli raya çevirmek) reddedildi | Founder 2026-08-03, #45b |
| Ü8 | **Hekime API anahtarı verilmez** | — |
| Ü9 | Yanıt iskeleti **zorunlu ve sabit** (Kısa yanıt/Gerekçe/⚠ Dikkat/Pratik öneri + kapanış) | `prompts.DOCTOR_SYSTEM` |
| Ü10 | **Kapanış satırı iki hâlli**; `Atıfsızdır.` kuyruğu KALKTI | Founder 2026-09-11, #769c |
| Ü11 | Türkçe yanıtta "agent" → **"ilaç"**, "ajan" değil | Founder 2026-09-01 |

---

## 2. DENETİM İNVARİANTLARI (2026-07-29 — **BOZMA**)

Bunlar tek tek ölçülmüş hatalardan doğdu; her biri kapıyla çivilidir.

### İ1 — ⚠⚠ "SON ADIM BAĞLANMAMIŞ" = EN SIK HATA SINIFI
Bir günde **dört kez** çıktı. İş yapılmış, üretilen şey tüketiciye bağlanmamış, yüzeysel
bakınca "tamam" görünüyor.
**KURAL: "X eklendi" demeden önce X'i TÜKETEN katmanda ölç** (HTML/JSON/canlı URL).
Birim testi fonksiyonu test eder, **bağlantıyı değil** — 26/26 yeşilken özellik hekime
görünmüyordu.

### İ2 — ⚠⚠ FAIL-OPEN YASAK: "çalıştırılamadı" ≠ "temiz"
Hata / JSON-parse hatası / hız sınırında `status="unavailable"` + `critic_status`;
UI amber "elle doğrulayın" basar, **asla yeşil tik**.
⚠ Yeni güvenlik katmanı eklerken **anahtarı gerçekten DÖNDÜR** (`redflag_scan`da `status`
hiç yoktu). Bu sınıf hata "yanlış cevap" değil **YANLIŞ GÜVEN** üretir.

### İ3 — ROZET, MODELE GİDEN KANITLA AYNI KÜMEDEN ÜRETİLİR
Olmayan kart rozet **almaz** (sahte provenans); olan **almalı** (yoksa `citedSources`
süzgeci meşru atfı eler, hekim uydurma sanır). Eşleme `cds/kaynaklar.py` kaydından doğar →
**elle senkron YOK**.

### İ4 — Kaynak rozeti kartın **GERÇEK `source`**'undan üretilir
TİTCK kaydı "openFDA" rozeti **almaz** (yanlış deep-link = sahte provenans).

### İ5 — Hasta bağlamı: `parts` boş olabilir, `record` yine de GİTMELİ
Yoksa tahliller/ziyaret notları/alerji-teratojen-renal talimatları prompta **hiç girmez**.

### İ6 — ÖLÇÜM SİNYALİ GENİŞ, OPTİMİZASYON SİNYALİ DAR
Aynı olayı iki amaca **koşma**.

### İ7 — HASSAS SORGU PİKSELE GİTMEZ
⚠⚠ `<noscript>` kolu **MUAF** → "hiç yüklenmez" hükmü **kurma** (#105b).

### İ8 — AKIŞ İPTALİNDE KOŞULSUZ İADE YASAK
Teslim edilen karaktere göre ücretlendir; iptalde de `record_usage` yaz.

### İ9 — `purchase` olayı **DB'den** üretilir, URL'den değil

### İ10 — Self-heal reserved satırını **SİLMEZ**, `reserved_orphan` işaretler
Silince **çift tahsil**.

### İ11 — AĞIR İŞ EVENT LOOP'TA ÇALIŞMAZ (`run_in_threadpool`)
İki eşzamanlı dosya analizi siteyi **herkes için** donduruyordu.

### İ12 — `/docs`, `/redoc`, `/openapi.json` KAPALI

### İ13 — ÜCRETSİZ GÜVENLİK UÇLARINDA TAVAN = KÖTÜYE KULLANIM FRENİ
`dose-check`/`redflag`/`simplify` 0 maliyet ve `_email_gate`'in **dışında** — bu **doğru**;
güvenlik katmanı kota/duvar arkasına konmaz.
⚠⚠ Fren **kapatmıyor, AZALTIYOR** — "kapandı" **deme**.

### İ14 — PANEL DOĞRULUĞU
Hata → pozitif hüküm **YASAK** (sıfırın sebebini iddia eden cümle de dahil) ·
**YOKLUK en sessiz hâldir** · sayaçlar **AYRIK** · ROI sütunu **PARAYI** ölçer ·
`last_active` geçici kaynağa dayanamaz · `/health` sürüm damgası **503 dalında DA** taşır.

### İ15 — PANEL GÜN SINIRI = TÜRKİYE SAATİ
`store._TR_BUGUN`/`_tr_gun()` tek kaynak. ⚠ Sabit `+03` ofset **yazma**; zaman dilimi **adı** kullan.

### İ16 — ⚠⚠ COMMIT EDİLMEK YETMEZ, KAPIYA BAĞLANMASI GEREKİR
`SUITES`/`SKIP_SUITES`'te olmayan takım **sessizce ÖLÜ ağdır**.
⚠ **Gizli KB bağımlılığı grep'le bulunmaz → AST.**
⚠⚠ **KAPIYI YENİDEN YAZMA — KAPININ KENDİSİNİ ÇALIŞTIR** (yeniden yazılan kapı 13/13 yeşil
basarken CI kırmızıydı).

### İ17 — DOĞRULAMA SCRIPTLERİ COMMIT EDİLİR
⚠ *"(24/24 geçti)"* iddiası görünce **önce dosyanın VAR olduğunu doğrula** — koşulamayan
test, **olmayan güvencedir**.
⚠ **Test iddiası uygulamadan ayrışırsa TESTİ düzelt** (bir kez bir hatayı "beklenen davranış"
olarak çivilemişti).

### İ18 — ⚠⚠ YEREL YEŞİL ≠ CI YEŞİL
İşletim sistemi farkı; **50 koşu kaybedildi**.

---

## 3. FOUNDER KARARLARI — tarihli

### 2026-07-14 · Dosya saklama kuralı TERSİNE döndü
Eski *"yüklenen dosya SAKLANMAZ"* kaldırıldı → hasta dosyaları **kalıcı + şifreli**
(Fernet, DB bytea). `KLIVANCE_FILE_KEY` **master anahtar**, kaybı = kalıcı veri kaybı.

### 2026-07-23 · *"Bu metin taslak niteliğindedir"* ibaresi KALDIRILDI

### 2026-07-30 · Admin panosu — görünürlük katmanı
5 kart + atıf oranı + **ödeme DENEMELERİ** (redler görünür).

### 2026-08-03 · #45b — Sol menü bileşiği kullanır
"Sor"/"Ask" kaldırıldı · "Hastalar"→"Hastalarım" · ray sırası `/panel` sidebar'ına uyduruldu.
**Bedeli ödendi ve kapandı** (bağımsız ölçüm 2026-08-08 + 2026-08-11).

### 2026-08-03 · ⚠⚠ API PARA YAKMA YASAĞI
Test soruları hariç ücretli API ile para yakmak **YASAK**; test soruları **ekip toplamı
günde ≤10**. Her test çağrısı raporda **adet + maliyetle beyan edilir**. 10'u aşacak ya da
test-dışı her çağrı **founder onayı** ister. (Müşterinin ürün kullanımı bu kuralın dışında.)

### 2026-08-03 · Mail kuralları
(1) Her **yeni mail tasarımı/metni** önce founder'a test gönderimiyle gider.
(2) Tam künye **eklenmez**, **tek satır kimlik** zorunlu (Ticari İleti Yön. m.8/2).

### 2026-08-05 · #118b — HASTA KİMLİĞİ KURALI TERSİNE DÖNDÜ
Hekim **gerçek ad girebilir**; takma kod isteğe bağlı. Kayıtta **2 onay**.
⚠ Eski `consents` `patient: true` **silinmez**.
⚠ *"Kimlik işlenmez / yalnız takma kod"* diyen dili **ÜRETME**.

### 2026-08-05 · #119b — YASAL METİN İFŞASI (iki kez teyit)
`anthropic` / `ABD` / `m.9` / `açık rıza` = **0**. **Eksik sanıp EKLEME.**
⚠ Kararın çürütmediği olgu: aktarım gerçekten oluyor — karar **ifşaya** dair, **olguya** değil.

### 2026-08-07 · PROMPT'TAN META-İTİRAF KALDIRILDI
Ürünün kendi altyapısından söz eden cümleler çıktı; **klinik güvenlik uyarısı KALDI**,
dili klinik gerekçeye çevrildi. ⚠ Kapsam **yalnız model çıktısı** — `/etkilesim`,
`pipeline` status, `chat_body` fail-open UI dalları **kapsam dışıdır**.

### 2026-08-08 · `.rail-dar` (64px admin dalı) KALDIRILDI
**TEK genişlik, admin dahil.**

### 2026-08-09 · #139b — Para birimi DİLE bağlandı
*"Türkçe hariç herkes dolar görsün."* ⚠ Risk (**ilan ≠ tahsilat**) ortadan kaldırılmadı,
**ifşayla yönetiliyor**: USD gösterimi `_ifsa` satırından **ayrılamaz**.

### 2026-08-12 · Push için onay sorma
Doğrulanınca gönder. ⚠ Kontrolcü denetimi ve para/dışarı-çıkan işlem onayı **DURUYOR**.

### 2026-08-14 · `/admin/sorular` HAM gösterir
Soruyu **ve** yanıtı, **hekim kimliğiyle**.

### 2026-08-15 · #275b — YANIT DİLİ ≠ ARAYÜZ DİLİ
Modele giden dil **sorudan** tespit edilir.

### 2026-08-15 · Yönlendirme yasağı + profesör tonu

### 2026-08-15 · CI `paths-ignore` ekonomisi
Yalnız not dosyası değişen push'ta koşma (~300 dk/ay tasarruf).

### 2026-08-17 · Reçete kontrolü ailesi
*"Reçete kontrolü olsun ismi, önce yapalım"* → hub `/recete-kontrolu`, ilk araç `/bobrek-doz`.
⚠ `/etkilesim` URL'i ve ücretli reklam inişi **aynen duruyor**.

### 2026-08-17 · e-Nabız DENEME KİLİDİ
*"Hasta kartı butonunu canlıya alma."* → manifest kapsamı **yalnız `127.0.0.1:8000`**.

### 2026-08-18 · #345b — `/etkilesim` çoklu ilaç
*"İlaç etkileşim kontrolünde 2'den fazla ilaç olması lazım."* (2–8)
⚠ Eski `?a=&b=` **korundu** (canlı reklam inişi).

### 2026-08-19 · Hastalıklar sayfası
*"Ana menüye hastalıklar butonu; hekim hastalık yazsın, bilgi gelsin, devamında soru sorma
chatbox olsun."*

### 2026-08-19 · #362b(a) — Analizden çıkan değeri karta işleme
Zincir bugüne dek **kırıktı**: tahlil → analiz metni → **⛔** → hasta kartı → tetik → uyarı.

### 2026-08-19 · SGK Ek-4/A + aylık reklam harcaması girişi

### 2026-08-20 · Takvim (Faz 9)

### 2026-08-21 · ⚠⚠ KREDİ SİSTEMİ KALDIRILDI (#396b)
**Sınırsız abonelik + deneme + sessiz günlük tavan.** Top-up yolu silindi.
Kredi kolonları şemada **duruyor ama okunmuyor**.

### 2026-08-21 · Fenotip motoru — karar "B"
Ölçülmüş **ölü veri** üstüne kuruldu.

### 2026-08-25 · Yaş aşımı kararı "b"

### 2026-08-27 · #356b — `/reference` `/recete-kontrolu` içine taşındı
Eski yol **303**. Çıkışlıda bölüm görünür ama **kilitli**.

### 2026-08-29 · *"Her yazdığım ajan kadroda olsun"* → `ekip_kadro_verify`

### 2026-08-31 · #470b — StatPearls GÖRÜNMEZ
*"Kullanalım, bilgileri alalım, ama hekim karşısına StatPearls diye çıkmasın."*
⇒ **BY (atıf) da düşer.** Bilinerek alındı — eksik sanıp atıf **EKLEME**.

### 2026-08-31 · #522b — Admin erişim/denetim logu
`_admin()` = rol **VE** denetim kapısı.

### 2026-09-01 · #596b — Cihaz eşleştirme

### 2026-09-02 · ⚠⚠ DÖNEM BÜTÇE KAPISI — *"zarar etmeyelim"*
İki mod (`gozlem` varsayılan / `uygula`); `uygula` **ifşa maddesi yokken KODDA REDDEDİLİR.**

### 2026-09-02 · *"Sınırsız deneme" dili YASAK*

### 2026-09-04 · #715b — Soru analitiği
*"Hangi branşlar neler soruyor. Bizim alameti farikamız bu."*
*"Soru ve cevapları detaylı kayıt yapabileceğimiz bir sistem kur."*

### 2026-09-04 · #726b — Hekim davranış izi
*"Hekimin kaç hastası var, hasta analizinde neler yaptı… KAYIT TUTMAMIZ lazım."*

### 2026-09-03 · #668b — ⚠⚠ e-Nabız: PDF KAYNAK DEĞİL, AKORDİYON KAYNAK
Founder kendi hesabında **gösterdi**: PDF **eksik kopya**, sayfa HTML'i **tam veri**.

### 2026-09-06 · ⚠⚠ ABONELİKLE YAPILABİLEN HİÇBİR ŞEY ÜCRETLİ API İLE YAPILMAZ
Deney/ölçü/metin/sınıflandırma → alt ajan. Ürün API yolu yalnız kod yolundan geçmesi
**zorunlu** olan ön-kayıtlı tek doğrulama koşumu.

### 2026-09-06 · #748b — Kopan istemcinin toparlanması
Hekim telefonda soruyu sorup yanıtı beklerken sekmeyi değiştirince üretim düşüyordu.

### 2026-09-11 · ⚠⚠ AVUKAT TEYİDİ (taşa kazınmış — iki kural)
1. 2026-09-11'e kadar açılmış **her** hukuk/lisans kalemi avukat onaylıdır; **teyit izin
   değil, DURUŞ ONAYIDIR.** Bu kalemleri bir daha soru olarak **çıkarma**.
2. **"Ben hukuki boyutu düşün demedikçe sen DÜŞÜNME."** Hiçbir işte hukuki yorum/lisans
   hükmü/KVKK-FSEK-telif analizi **üretilmez**.

### 2026-09-11 · #769c — Kapanış satırı iki hâlli

### 2026-09-12 · #774c — iyzico abonelik (recurring)
**Kodda var, bayrak KAPALI.** Açma founder'ın, üç şart:
`docs/iyzico-abonelik-acilis-2026-09-12.md`.

### 2026-09-13 · #776c — GÖRÜNTÜLEME İKİ YOL
YOL 1 rapor metni → RAD satırı · YOL 2 görüntü ön-okuma.
⚠⚠ #620b *"hiç okuma"* duruşu **BİLİNÇLİ DEĞİŞTİ**.

### 2026-09-14 · #778c — Görüntülemede görüntü olmadan metin
*"Görüntülemede görüntü olmadan da yazı yazma olsun."*

### 2026-09-15 · #781c / #782c — Sohbet görselini karta kaydet · kritik bulgu kademesi
*"Aç, daha da geliştir."*

---

## 4. REDDEDİLMİŞ ŞEYLER — **tekrar önerme**

### 4.1 Mimari
| Reddedilen | Gerekçe |
|---|---|
| `include_router` | Rota envanteri okuyan iki ağ sessizce körelir |
| `app/db.py` | `connect()` `saglik/db.py`'de |
| Şablon motoru / SPA | Gövde ölçen 320 kapıyı kırar |
| `app.person` / `app.account_event` | Sıfır yeni kalıcı veri ilkesi |
| `.rail-dar` (64px) | Founder 2026-08-08 |
| Kısa ray etiketi | Bir hedefe TEK ad |
| `ts_headline`'a dokunmak | Ortam sorunu ürün regresyonuyla kapatılmaz |

### 4.2 Veri kaynakları
**DDInter** (CC-BY-NC) · **DrugBank tam** (ücretli) · **WHO ATC dosyası** ·
**MedlinePlus** (karışık lisans) · **PMC doğrudan** · **USPSTF** ·
**NCBI Bookshelf yolu** ("OA subset" ≠ ticari) · **TEMD/TKD/ESC/SIGN/KDIGO** · **TGA**

### 4.3 Prod'a taşınmayanlar (ölçüldü, kazanç 0)
**DailyMed** · **`name_alias`** · **`code_map`-rxnorm**

### 4.4 ⚠⚠ REKLAM/LANDING'DE SÖYLENMEYECEK SEKİZ İDDİA
**Hepsi doğrulamada ÇÜRÜDÜ:**
1. "Daha güncel"
2. "UpToDate LLM'ini gizliyor"
3. "UpToDate'ten pahalıyız" (aksine **~%15 UCUZUZ**)
4. "İçeriğin 2/3'ü uzman görüşü"
5. "Araması kötü"
6. "FDA tek-çıktı kısıtını kaldırdı"
7. "Nature Medicine kanıtladı"
8. COI için "rüşvet/gizleme" (gerçeği **beyan uyumsuzluğu**, kanıtlanmış suistimal DEĞİL)

⚠ *"Ama bu doğru"* demeden önce `.claude/agents/hasan.md`'ye bak — her birinin çürütme
kanıtı orada.

⚠ **UpToDate'i adıyla hedef alan karşılaştırmalı reklam YASAK** (asimetrik hukuki risk).

### 4.5 Çürütülmüş hipotezler (iniş sayfası darboğazı — P0)
⛔ **Tekrar kovalama:** sayfa yavaşlığı · mobil düzen kırılması · Meta piksel olaylarının
GA4'te görünmemesi.
`/chatgpt` A/B'si **geçersizdi**. Açık uç: IG uygulama-içi tarayıcı.

---

## 5. REKABET KONUMU

| | |
|---|---|
| **Asıl rakip** | UpToDate **değil**, ücretsiz **OpenEvidence** |
| Ama | ABD NPI kilidi + AB geo-block ⇒ **TR'de rakip değil** |
| Konumlandırma | **"UpToDate'in YANINA, yerine değil"** |
| FDA CDS | Yürürlükteki rehber **29 Ocak 2026**; riskli olan **Kriter 4** |

Landing'deki **Kriter-3/4 kalkan cümleleri YERİNDE** (commit `c98b72e`) — **SİLME.**

---

## 6. FİYAT / ERİŞİM KURALLARI

| Kural | Detay |
|---|---|
| ⚠⚠ Fiyat sayısı **hiçbir dokümana yazılmaz** | Kaynak `iyzico._PLAN_PRICE_DEFAULT` |
| ⚠⚠ Süre/tavan sayısı **yazılmaz** | Kaynak `credits.*` |
| Fiyat | **KDV dahil** |
| Kurucu plan | Ömür boyu **ORAN** taahhüdü (legacy; **sabit-TL DEĞİL**) |
| Yıllık | Aylık × 10 → **2 ay bedava, TEK avantajı budur** |
| Kredi bonusu / top-up | **YOK** |
| ⚠⚠ "Sınırsız" + gizli tavan | **Adil kullanım ibaresi ZORUNLU** (iki yerde: görünür ibare + yasal madde) |
| ⚠ "Her abone kârlı" | **DENMEZ** — tek fren günlük tavan, o da seçildi/ölçülmedi |
| ⚠⚠ Fiktif işlem | Founder'ın kendi kartıyla kendine satışı **YASAK** |

---

## 7. ÇALIŞMA DİSİPLİNİ (devralan ekip için)

Bu kurallar Klivance'ın iç çalışma tarzıdır; devralan ekip kendi süreçlerini kurabilir ama
**neden bu kadar çok kapı olduğunu** anlamak için faydalıdır.

| Kural | Özü |
|---|---|
| **Kanıtla, iddia etme** | Bir sayı ya da hüküm iletilecekse **ölç**; desen **ADAY** üretir, **HÜKÜM** üretmez |
| **Ölçüt: para ya da hasta güvenliği** | İkisi de değilse bugün yapılmaz |
| **Derinlik 1** | Ölç → düzelt → tek kapı → commit. Bulgunun bulgusunu kazma; yolda bulunan başka kusur **kalem** olur, kovalanmaz |
| **Rapor ≤10 satır + tek karar sorusu** | |
| **Yokluk iddiası, varlık iddiasından çok ölçüm ister** | *"X yok"* demeden önce **X'in ailesini listele ve her üyesini dene** |
| **Bir ölçütü değiştiriyorsan** | Kabul testi **eskisinin kaçırdığını** göstermeli (eski YEŞİL / yeni KIRMIZI) |
| **Yeni kural ilgili dosyanın başlığına yazılır** | ⚠ Ama **sayı/dosya adı KOPYALAMA**: kopyalanan sayı bayatlar, kopyalanan ad **hayalet** olur |
| **Eş zamanlı çalışma** | `git status` ile kirli mi bak · başkasının commit'ini düşürme · push dalın tamamını yayınlar |

---

**Sonraki:** [`17-BILINEN-TUZAKLAR-VE-BORCLAR.md`](17-BILINEN-TUZAKLAR-VE-BORCLAR.md)
