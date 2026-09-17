# 17 — BİLİNEN TUZAKLAR VE AÇIK BORÇLAR

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
>
> **Bu dosyanın amacı devralan ekibin aynı çukura ikinci kez düşmemesidir.**
> Buradaki her madde **yaşanmış**tır; hiçbiri teorik risk değildir.

---

## 1. TEKNİK TUZAKLAR — dil / platform

### T1 — ⚠⚠ PostgreSQL: aynı transaction'da "olursa olur" yazma → **SAVEPOINT**
Başarısız bir ifade **tüm transaction'ı zehirler**; `try/except` hatayı yutsa bile sonraki
`commit()` `InFailedSqlTransaction` alır ve **o transaction'daki ASIL yazma da düşer.**
Kritik olmayan ek yazmayı `with conn.transaction():` içine al.
**Doğrulama deseni:** hatayı **mock ile patlat** ve asıl yazmanın **sağ kaldığını** ölç.

### T2 — ⚠⚠ SAVEPOINT'İN İÇİNDE `commit()` ÇAĞIRMA
psycopg3 **yasaklar** (*"Explicit commit()/rollback() forbidden within a Transaction
context"*) ve yazma **SESSİZCE DÜŞER**. `store.*` fonksiyonlarının çoğu sonunda
`conn.commit()` yapar; biri `with conn.transaction():` içine alındığı an **ölür**.
Doğrusu commit'i **SAVEPOINT'ten SONRA** atmak.

> **Ölçüldü 2026-08-27:** `case_timeline` arşivi **HER çağrıda** düşüyordu.
> `cases_api_verify` bölüm 14 yakaladı — yani bu sınıfı **yazmayı DB'den geri okuyan** bir
> iddia olmadan **göremezsin**.

### T3 — ⚠⚠ GERİ OKUMA AYNI BAĞLANTIDAN YAPILIRSA KANIT DEĞİLDİR
psycopg3'te `autocommit` **kapalıyken** `conn.execute("SET …")` zaten transaction açar →
sonraki `conn.transaction()` bir **SAVEPOINT'e** döner ve `close()` dış transaction'ı
**ROLLBACK** eder; üstelik **bir bağlantı kendi commit'lenmemiş yazmalarını görür**.

**KURAL:** yazma iddiasını **AYRI BAĞLANTIDAN** doğrula · `autocommit=True` ya da açık
`conn.commit()` · **`rowcount` yazma kanıtı DEĞİLDİR.** Prod ve para yollarında da geçerli.

### T4 — ⚠⚠ TİPSİZ SQL PARAMETRESİ + `except: pass` = SESSİZ VERİ KAYBI (P0)
- Parametreye bağlam yoksa **tipi VER** (`%s::text`)
- `except: pass` ile sarılan DB yazmasını **DB'den okuyarak** doğrula ("HTTP 303 döndü"
  kanıt **değil**)
- Aynı `with connect()` bloğunda tek bir başarısız ifade **sonraki HER yazmayı** sessizce düşürür

> Doğrulama e-postası bu yüzden **hiç gitmiyordu.** Kapı `kayit_kaynak_verify`.

### T5 — ILIKE'a kullanıcı girdisi → `reference._like_escape`
`%`/`_` kaçır; yoksa `a_pirin` → ASPIRIN **yanlış kart**, tek `%` → **seq-scan**.

### T6 — Prod gecikme: unnest-ILIKE seq-scan
2026-07-20: etkileşim prod'da **75 s** → `find_drugs` unnest-ILIKE seq-scan;
`brands_text() ILIKE` + GIN ile **154×** hızlandı.
**KURAL: unnest-ILIKE kullanma, prod'da ölç.** Memory `klivance-prod-latency-seqscan`.

### T7 — ⚠⚠ i18n APOSTROF TUZAĞI (sessiz P0)
EN çevirisinde **düz kesme (`'`)** + TR anahtarı tek-tırnaklı JS string içinde ⇒
`SyntaxError` ⇒ **o sayfanın TÜM JS'i çöker** (sessiz; sayfa boş yüklenir).
**KURAL: `EN_PAIRS` İngilizce değerlerinde kıvrık kesme `’` (U+2019).**

### T8 — `store.get_case` **tuple** döner (dict değil), **18-geniş**
Yeni alan **SONA** eklenir; okuyucular `_case_ctx_base` **index-tabanlı**.

### T9 — Kota penceresi = `store._period_start` (takvim ayı **DEĞİL**)
Abonede `current_period_end` aylık yıldönümü; trial/expired takvim ayı.

### T10 — Sonnet-5'te `thinking` verilmezse **adaptive VARSAYILAN AÇIK**
`max_tokens`'i yer → `llm._no_thinking(model)` ile kapat.
⚠ **Haiku'ya GEÇİRME → HTTP 400.**

### T11 — ⚠⚠ Prompt-cache tabanını **EZBERE YAZMA**
Minimum önbelleklenebilir önek **modele bağlı** ve nesiller arası **monotonik DEĞİL**.
`token_butce_verify._CACHE_MIN` haritasından oku. ⚠ Yanlış sabitin yönü "güvenli" değil:
fazla temkinli eşik **ters yönde yanlış alarm** üretir.

### T12 — Takvim randevusu = **duvar saati**
`_appt_json` ofsetsiz ISO (JS yerel kabul), sunucu `_wall()` naive yerel.
Bozarsan ızgara 09:00 / yan panel 12:00 çelişir.

---

## 2. ORTAM TUZAKLARI — Windows / araçlar

### O1 — ⚠⚠ Kaynak `.py` dosyasını PowerShell ile **YAZMA**
`Set-Content -Encoding UTF8` başa **BOM** ekler → `SyntaxError: invalid non-printable
character U+FEFF`. **Bir kez tüm uygulama düştü.** Okumak sorun değil.

### O2 — Bash heredoc backslash bozar
`<< 'PY'` içinde ters eğik çizgi öngörülemez şekilde gerçek newline'a dönüşebilir
(JS regex'ini/string'ini kırar). Backslash içeren scriptleri **dosyaya yaz**, öyle çalıştır.

### O3 — `curl -d` Türkçe karakter bozar (Windows)
JSON gövdeyi Python ile UTF-8 dosyaya yaz, `--data-binary @file`.

### O4 — Toplu düzenleme
Python script + **tmp dosyaya yaz → `os.replace`**. ⚠ `open('w')` surrogate-emoji
**yazamaz** → dosya **boşalır**. Her yazımdan önce **`ast.parse`**.

### O5 — ⚠⚠ BORU (`| tail`) ÇIKIŞ KODUNU MASKELER (#407b, **açık kalem**)
`python x.py 2>&1 | tail -12` sonrası `$?` **borunun** son komutunun kodudur, script'in
**değil** → **kırmızı bir kapı YEŞİL görünür.**
**Doğrusu:** `${PIPESTATUS[0]}` ya da dosyaya yönlendirip ayrı okuma.

### O6 — ⚠⚠ YEREL POSTGRES SEGFAULT EDİYOR — **AÇIK, kök neden bulunamadı**
2026-08-02, 3 ajan. Çökmeler 2026-07-09'a kadar geriye gidiyor = **herhangi bir günün
kodundan bağımsız**. DB WAL redo ile toparlanır.
**Uzun ölçümü yarıda kesen şey budur** — script'ini suçlamadan önce
`docker logs saglik-pg`'ye bak.
⚠⚠ **2026-08-06'da TIRMANDI** (BSOD, boştayken de) → **yerel KIRMIZI da kanıt değil;
nihai kanıt CI.**
⚠ **Prod maruziyeti BİLİNMİYOR** (farklı PG major + imaj) — yerelden çıkarımla kurma.
Baş şüpheli **donanım** → memory `makine-donanim-kararsizligi`.

### O7 — ⚠⚠ GIT NESNELERİ YAZARKEN BOZULUYOR
*"Commit atıldı"* **kanıt değil** → **push öncesi `git fsck`**.
Vaka `5cb5c540` 2026-08-15'te temizlendi (reflog girişi + prune reçetesi memory'de).
**Tekrar görülürse YENİ vaka.**

### O8 — Oturum ADLARI ÇAKIŞIR
Çıplak adla mesaj yanlış oturuma düşer → sahiplik/onay **yanlış kişiye** sorulur.
Memory `makine-oturum-adi-cakismasi`.

### O9 — `gh` CLI **KURULU DEĞİL**
CI log'unu okuma reçetesi: `docs/tuzaklar-ve-denetim-anlatilari.md`. **Aramaya vakit harcama.**

---

## 3. ÖLÇÜM ARACI KÖRLÜKLERİ

⚠⚠ **ÖLÇÜM/TARAMA ARACI YAZDIYSAN ARACIN KENDİSİNİ SINA.**

| # | Körlük |
|---|---|
| M1 | Çok-satırlı yapıyı **regex**'le arama → **AST** |
| M2 | **Öz-referans** — araç kendi dosyasını taramaz |
| M3 | **Ada göre desen** — `SUITES=` deseni `SKIP_SUITES=` ile de eşleşir |
| M4 | **Girdiyi TAHMİN etmek** — alan adlarını **rotanın imzasından OKU**. En tehlikelisi paritenin yine korunması = kusur **SESSİZ** |
| M5 | Bozma tatbikatında `git checkout --` **tek başına geri almaz** — `__pycache__` de silinmeli |
| M6 | **Platform altından değişir** — yeşil bir kapı, kodu hiç değişmeden **ölçmeyi bırakabilir** |
| M7 | **Kapıyı düzeltince ölçüm aracı da bayatlar** → her çıkarıcıya *"tarayıcı ölü değil"* koruması |
| M8 | Öz-test **İKİ YÖNLÜ** olsun |
| M9 | **Desen ADAY üretir, HÜKÜM üretmez** (#108b) |
| M10 | Bir aracın kusurunu, o aracın **DENETLEDİĞİ** kişiye düzelttirme |
| M11 | ⚠⚠ **DEĞERİ ÖLÇ, ADI DEĞİL** — 2026-08-01'de **beş kez** çıktı, biri P0'dı |
| M12 | ⚠⚠ **KAPIYI YENİDEN YAZMA — KAPININ KENDİSİNİ ÇALIŞTIR** |
| M13 | `scripts/*.py`: **`__main__` kapısı ŞART** + hata gövdesine **üyelikle** bak |

---

## 4. AÇIK BORÇLAR — `_gorev.txt` ölçümü (2026-09-17)

> **Kaynak:** `_gorev.txt` başlık etiketleri, AST/regex ile sayıldı.
> ⚠ Bu bir **NOT dosyasıdır, GERÇEK DEĞİL** — bir kalemin gerçekten açık olduğunu
> iddia etmeden önce **koddan/canlıdan ölç**.

| Etiket | Adet |
|---|---|
| Toplam numaralı kalem | **578** |
| `AÇIK` | **184** |
| `KALEM (kovalanmadı)` | **54** |
| `KAPANDI/BİTTİ` | **96** |
| `KAYIT (bilgi)` | 12 |
| `FOUNDER KARARI` | 8 |
| `REDDEDİLDİ` | 1 |
| Etiketsiz/diğer | 223 |

**Açık + kalem içinde öncelik:** P0 **9** · P1 **66** · P2 **79** · önceliksiz 87

### 4.1 ⚠⚠ AÇIK P0'LAR (etikete göre — dokuz kalem)

| # | Başlık | Sahip / alan |
|---|---|---|
| **#484b** | *P0 adayı:* Mobilde **"Klivance'a Sor" alt çubukta GÖRÜNMÜYOR** — çıkışlı 6 sayfanın 6'sında %0. `.rail-nav` taşıyor ve **açılışta kaydırılmış** geliyor | Arayüz |
| **#489b** | **Ürün parayı hiç istemiyor**: 6 haftada fiyat sayfasını **16 kişi** açtı. GA4 hunisi: `/kayit` 397 → kayıt ~86 → `/chat` 155 → `/account` 45 → **`/abonelik` 16** | Para / dönüşüm |
| **#511b** | **Canlı Google reklamında süre vaadi** — founder'ın 2026-08-28 kuralı **çiğneniyor** (`KLV-TR-DISPLAY-REMARKETING`) | Reklam |
| **#565b** | **Yanlış hastanın kartına sessiz birleşme** — `store.create_case` aynı `code` gelirse GÜNCELLER (`ON CONFLICT DO UPDATE`); founder'ın "sağ üstteki addan bakarak kayıt açacak" tarifiyle birleşince **aynı adlı iki hasta birleşir** | **Hasta güvenliği** |
| **#566b** | **Aktarım doz güvenliği alanlarını NULL'lar** — `store.update_case` COALESCE'siz tek UPDATE, 15 kolon. Gönderilmeyen alan NULL olur; NULL'lananlar arasında `kilo_kg`, `boy_cm`, `gebelik_haftasi`, `yas`, `egfr`, `kreatinin` = **DOZ GÜVENLİĞİ** | **Hasta güvenliği** |
| **#623b** | **Eşleştir hasta kartına yanlış kreatinin/eGFR yazabilir** — bugünkü değer temiz ama ⚠⚠ **"ŞANS, GÜVENCE DEĞİL"** | **Hasta güvenliği** |
| **#639b** | **Ana dalın CI'ı yedi gündür kırmızı** (2026-08-25 son yeşil; 95 koşum, sıfır yeşil). ⚠ Bu kayıt 2026-09-03 tarihlidir — **güncel CI durumu ayrıca ölçülmeli** (`gh` kurulu değil) | Teslim engeli |
| **#703b** | *P0 adayı:* "Aspirin ve **Celaxan**" → kart üretilemedi, model **ezberden** "Celaxan (sitalopram, SSRI)" dedi; hekim "**Clexane** = enoksaparin" diye düzeltti. **MI bağlamında ANTİKOAGÜLAN "SSRI" diye elendi** | **Klinik doğruluk** |
| **#704b** | **Hasta kartı bağlamı aralıklı düşüyor** — aynı soruya 2 dk arayla önce "veri yok", sonra tam lab dökümü. Aynı thread'de metformin+tirzepatid/eGFR58 hastaya "hasta bağlamı verilmedi" deyip **ilaç-kontrendikasyon kontrolünü HİÇ yapmadı**. **Değişmez-5'in canlı ihlali** | **Hasta güvenliği** |

### 4.2 Kredi kaldırma açık kalemleri (#396b-A..J) — **ikisi P1 ve PARA**

| # | Kalem |
|---|---|
| **A** | Günlük tavan **seçildi, ÖLÇÜLMEDİ** — maliyeti sınırlayan **TEK şey odur**; `app.usage`tan p95/p99 ile kalibre edilmeli |
| **B** | **Abone başına maliyet ölçümü YOK** → admin birim ekonomi kartı marj yerine **"ölçülmedi"** basıyor |
| diğer | Canlı reklamlarda kredi vaadi taraması |

### 4.3 P0 — DARBOĞAZ İNİŞ SAYFASI (hüküm açık)

**Reklam iş görüyor, sayfa tutmuyor.** Google'da 12 kelimenin 12'sinde *"iniş sayfası
deneyimi ORT-ALTI"* iken **alaka iyi** ⇒ sorun **reklam metni değil SAYFA**.

⛔ **ELENDİ, tekrar kovalama:** sayfa yavaşlığı · mobil düzen kırılması · Meta piksel
olaylarının GA4'te görünmemesi.
`/chatgpt` A/B'si **geçersizdi**. Açık uç: **IG uygulama-içi tarayıcı**.

### 4.4 Diğer açık iş başlıkları

| Alan | Kalem |
|---|---|
| **Ödeme** | iyzico **uçtan uca DOĞRULANMADI** (#768c, dört maddelik çek listesi). ⚠ Tek yetkili kaynak `/admin/odeme` (DB); **GA4 kanıt DEĞİL** |
| **Ödeme** | `reconcile_pending` ile açılan ödemede **makbuz gönderimi** doğrulanmalı |
| **Ödeme** | `iyzico_payment`'ta **iade alanı yok** (#523b) |
| **KVKK** | e-Nabız **kopyası geri alınamıyor** (#442b) |
| **Veri** | **TR sorguda kanıt paketi boş** (#453b) · retrieval sorguyu **tek kelimeye indirgiyor** (#702b) |
| **Veri** | Geri çekilme **rozet/süzgeci hâlâ YOK** |
| **Veri** | **Türkçe kılavuz içeriği sıfır** (#455b) |
| **Atıf** | **Sahte provenans**: rozet basılan 148 yanıtın 26'sında (%17,6) (#698b) |
| **Ölçüm** | **Conversions API yok** → ölçüm %100 tarayıcıya bağlı (#496b) |
| **Ölçüm** | `klv_kayit` hekim başına **iki kez** ateşleniyor (#492b) |
| **Ürün** | `/api/query` **tümüyle süzgeçsiz** → StatPearls adı JSON'da (#551b) |
| **Ürün** | `/chat?q=<vaka>` URL'inin kendisi GA4'e düşüyor (#560b-B) |
| **ABD** | Delaware LLC → Stripe Atlas → `sk_live_`, canlı ürünler, `STRIPE_WEBHOOK_SECRET`, DNS |
| **SEO** | `/icd` genel arama sayfası (~2.780 arama/ay) — ⚠ **sayfa olmadan o kelimelere teklif VERİLMEZ** |
| **Varlık** | `static/img/` altında **~2 MB ölü görsel**, founder kararı bekliyor. ⚠ `og-klivance-en.png` dışarıdan `og:image` olarak önbelleklenmiş **olabilir** |

---

## 5. 🔴 BU EXPORT SIRASINDA ÖLÇÜLEN

```
scratchpad/ci_kapi_verify.py   →   37 geçti · 4 kaldı
KAPISIZ: _firat_782c_kayit_kapisi
```

Bugün eklenen bir kapı (`#782c-a`, commit `1344e26a`) `SUITES`/`SKIP_SUITES`'in hiçbirinde
değil → **sürüklenme kapısı kırmızı**, dolayısıyla **CI de kırmızı olur**.
**Düzeltme tek satır:** adı `SUITES`e ekle (DB/anahtar gerektirmiyorsa) ya da gerekçesiyle
`SKIP_SUITES`e. Bu export **belgeler, düzeltmez.**

---

## 6. DEVRALAN EKİP İÇİN ÖNCELİK ÖNERİSİ

⚠ Bu bir **öneri**dir, karar founder'ındır.

| Sıra | İş | Neden |
|---|---|---|
| 1 | **#565b + #566b + #623b + #704b** | Dördü de **hasta güvenliği**; ikisi sessiz veri bozulması, biri bağlam düşmesi |
| 2 | **`ci_kapi_verify` kırmızısı** | Tek satır; kırmızı CI her şeyi bloklar |
| 3 | **#489b** (ürün parayı istemiyor) + iniş sayfası darboğazı | **Para** — ürün çalışıyor, dönüşüm yok |
| 4 | **#396b-A/B** (tavan kalibrasyonu + abone başı maliyet) | Maliyeti sınırlayan tek şey ölçülmemiş |
| 5 | **#511b** (canlı reklamda süre vaadi) | Uyum + canlı yüzey |
| 6 | **#703b** (yazım hatası toleransı) | Klinik doğruluk |
| 7 | **iyzico uçtan uca doğrulama** (#768c) | Para yolu kanıtlanmamış |

---

## 7. AYRINTI DOSYALARI — bu tuzakların tam anlatısı

| Dosya | İçerik |
|---|---|
| **`docs/tuzaklar-ve-denetim-anlatilari.md`** | **Ölçüm anlatıları + denetim gerekçeleri** — tuzakların tam hikâyesi |
| `docs/durum-gunlugu.md` | Tarihli kayıt (ne zaman ne yapıldı) |
| `docs/kamil-ozellik-defteri.md` | Sayfa/özellik kararları, elenen hipotezler |
| `docs/derya-veri-defteri.md` | KB/veri ölçümleri (tarihli) |
| `docs/kurtarma-runbook.md` | Felaket kurtarma |
| `_gorev.txt` | **578 kalem** — açık işler, ölçümler, founder kararları |
| `.claude/agents/*.md` | Alan uzmanı derin kuralları (derya · cahit · kamil · selim · hasan · deniz · asaf · firat) |

---

**Sonraki:** [`18-DEVIR-NOTLARI.md`](18-DEVIR-NOTLARI.md)
