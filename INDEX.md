# KLIVANCE — SİSTEM ENVANTERİ VE MİMARİ DOKÜMANTASYONU

**Devir paketi — hiveteams.ai (Agentic AI SDLC) için hazırlandı.**

| | |
|---|---|
| **Ürün** | Klivance — Türk hekimler için klinik karar-destek (CDS) SaaS'ı |
| **Canlı** | https://klivance.onrender.com · hedef: https://klivance.com |
| **Depo** | `fatihalkanalfa/klivance` (private), dal `main` |
| **Ölçüm anı** | **2026-09-17** · commit **`cb16758d`** |
| **Dil** | Türkçe (kaynak kodun, docstring'lerin ve kararların dili) |

---

## 📐 Ölçülen büyüklükler

Aşağıdaki sayılar **bu export sırasında ölçüldü** — tahmin değil.

| Ne | Sayı | Nasıl ölçüldü |
|---|---:|---|
| Python modülü | **140** | `ast` ile kaynak ağacın tamamı |
| Python satırı (`saglik/`) | **~69.000** | `wc -l` |
| Fonksiyon (modül düzeyi) | **1.514** | `ast` |
| Sınıf / sınıf metodu | **23 / 55** | `ast` |
| HTTP rotası | **142** | canlı `app.routes` |
| Middleware | **7** | `app.user_middleware` |
| Veritabanı tablosu | **55** | `information_schema` (app 32 · core 22 · raw 1) |
| CI doğrulama kapısı | **320** | `test.yml` (SUITES 234 · SKIP 70 · JS 16) |
| Operasyon scripti | **177** | `scripts/` + `ast` |
| Docstring hacmi | **~278.000 karakter** | `ast.get_docstring` |
| `_gorev.txt` kalemi | **578** | başlık regex'i |

---

## 📚 DOKÜMAN HARİTASI

### Başlangıç (bu sırayla oku)

| # | Dosya | Boyut | İçerik |
|---|---|---:|---|
| 00 | [**ÜRÜN VE KAPSAM**](00-URUN-VE-KAPSAM.md) | 12 KB | Ne yapar, ne yapmaz, regülasyon duruşu, iş modeli, beş değişmez |
| 01 | [**SİSTEM MİMARİSİ**](01-MIMARI.md) | 21 KB | Yığın, katman haritası, bağımlılık yönü, istek yaşam döngüsü, middleware, i18n mimarisi, reddedilmiş kararlar |
| 02 | [**DİZİN VE DOSYA ENVANTERİ**](02-DIZIN-VE-DOSYA-ENVANTERI.md) | 21 KB | Her dizin ve **140 modülün** ne yaptığı, adlandırma konvansiyonları |

### Referans (üretilmiş — makine doğru)

| # | Dosya | Boyut | İçerik |
|---|---|---:|---|
| 03 | [**ROTA ENVANTERİ**](03-ROTA-ENVANTERI.md) | 23 KB | **142 rota**: metot, yol, endpoint, kaynak satırı, açıklama · erişim kapıları |
| 04 | [**VERİTABANI ŞEMASI**](04-VERITABANI.md) | 63 KB | **55 tablo** kolon kolon + indeks + kısıt · bağlantı disiplini · hesap yaşam döngüsü |
| 15a | [**FONKSİYON REF — APP**](15a-FONKSIYON-REFERANSI-APP.md) | 416 KB | `saglik/app/` — 88 modül, imzalar, modül başlıkları, sabitler |
| 15b | [**FONKSİYON REF — CDS**](15b-FONKSIYON-REFERANSI-CDS.md) | 183 KB | `saglik/cds/` — 37 modül |
| 15c | [**FONKSİYON REF — ALTYAPI**](15c-FONKSIYON-REFERANSI-ALTYAPI.md) | 39 KB | Kök modüller + `connectors/` + `ads/` |

### Alan alan derinlik

| # | Dosya | Boyut | İçerik |
|---|---|---:|---|
| 05 | [**CDS YANIT ZİNCİRİ**](05-CDS-YANIT-ZINCIRI.md) | 27 KB | Ürünün çekirdeği: classify → RAG → answer → ikinci geçiş · promptlar · **atıf zinciri** · rozet sözleşmesi · fail-open yasağı |
| 06 | [**BİLGİ TABANI VE KAYNAKLAR**](06-BILGI-TABANI-VE-KAYNAKLAR.md) | 15 KB | Veri topolojisi, connector sözleşmesi, **lisans disiplini**, reddedilen kaynaklar, KÜB/SGK/geri-çekilme hatları |
| 07 | [**KİMLİK, ERİŞİM, ÖDEME**](07-KIMLIK-ERISIM-ODEME.md) | 20 KB | Argon2id + oturum · erişim modeli · **üçlü kapı zinciri** · bütçe kapısı · iyzico/Stripe · fatura · mail |
| 08 | [**ARAYÜZ, i18n, TASARIM**](08-ARAYUZ-I18N-TASARIM.md) | 14 KB | Render modeli, tasarım tokenları, kontrast eşikleri, sol ray, **apostrof tuzağı**, sayfa kuralları |
| 09 | [**ADMİN VE ANALİTİK**](09-ADMIN-VE-ANALITIK.md) | 15 KB | Admin paneli, denetim logu, **üç-durum disiplini**, soru analitiği, GA4/Meta, üç kayıt olayı |
| 10 | [**GÜVENLİK, KVKK, UYUM**](10-GUVENLIK-KVKK-UYUM.md) | 17 KB | Yanıt başlıkları, şifreleme, PII redaksiyonu, hasta kimliği kararı, hesap yaşam döngüsü, FDA CDS, avukat teyidi |

### Operasyon

| # | Dosya | Boyut | İçerik |
|---|---|---:|---|
| 11 | [**TEST VE CI KAPILARI**](11-TEST-VE-CI-KAPILARI.md) | 48 KB | **320 kapının tam envanteri** · sürüklenme kapısı · ölçüm aracı körlükleri |
| 12 | [**DEPLOY, ORTAM, ENV**](12-DEPLOY-ORTAM-ENV.md) | 18 KB | Yerel kurulum, Docker, Render, **tam env envanteri**, bağlantı havuzu, git disiplini, kurtarma |
| 13 | [**SCRIPT ENVANTERİ**](13-SCRIPT-ENVANTERI.md) | 27 KB | **177 script** + `--apply` işareti + reçeteler |
| 14 | [**MASAÜSTÜ VE EKLENTİ**](14-MASAUSTU-VE-EKLENTI.md) | 7 KB | e-Nabız Chrome eklentisi (DENEME kilidi), Electron programı |

### Karar ve devir

| # | Dosya | Boyut | İçerik |
|---|---|---:|---|
| 16 | [**İŞLETME KURALLARI VE KARARLAR**](16-ISLETME-KURALLARI-VE-KARARLAR.md) | 16 KB | Değişmez kurallar, **18 denetim invariantı**, tarihli founder kararları, **reddedilmiş şeyler** |
| 17 | [**BİLİNEN TUZAKLAR VE BORÇLAR**](17-BILINEN-TUZAKLAR-VE-BORCLAR.md) | 14 KB | Yaşanmış tuzaklar, **9 açık P0**, açık borçlar, öncelik önerisi |
| 18 | [**DEVİR NOTLARI**](18-DEVIR-NOTLARI.md) | 13 KB | İlk 30 dakika, ortam kurulumu, okuma anahtarları, **agentic araç notları**, risk haritası, sırlar |

### Makine-okunur ekler — `veri/`

| Dosya | Boyut | İçerik |
|---|---:|---|
| [`veri/rotalar.json`](veri/rotalar.json) | 43 KB | 142 rota — canlı `app.routes` dökümü |
| [`veri/moduller.json`](veri/moduller.json) | 940 KB | 140 modül — docstring, import, fonksiyon imzaları, sınıflar, sabitler |
| [`veri/sema.json`](veri/sema.json) | 100 KB | 55 tablo — kolon, tip, null, varsayılan, indeks, kısıt |
| [`veri/middleware.json`](veri/middleware.json) | 1 KB | Middleware yığını (yürütme sırası) |

> Bu JSON'lar bir agentic aracın doğrudan tüketebileceği biçimdedir.
> Yeniden üretim reçetesi: [`18-DEVIR-NOTLARI.md` §6](18-DEVIR-NOTLARI.md).

---

## 🎯 GÖREVE GÖRE NEREYE BAKMALI

| İstediğin | Dosya |
|---|---|
| "Bu rota ne yapıyor?" | `03` → `15a/b/c` |
| "Bu tablo neden böyle?" | `04` |
| "Yanıt neden kötü geldi?" | `05` §5.1 — **Derya/Cahit ayrımını ÖNCE yap** |
| "Yeni kaynak ekleyeceğim" | `06` §8 (9 adım) |
| "Ödeme akışı nasıl?" | `07` §5 |
| "Sayfayı değiştireceğim" | `08` + `02` (dosya haritası) |
| "Neden bu kadar çok test var?" | `11` §1 |
| "Nasıl deploy ediyoruz?" | `12` |
| "Bu env ne işe yarıyor?" | `12` §5 |
| "Bu kararı değiştirebilir miyim?" | `16` — muhtemelen daha önce tartışıldı |
| "Nereden başlamalıyım?" | `18` |
| "Ne bozuk?" | `17` |

---

## ⚠️ BU DOKÜMANTASYONU OKURKEN — dört kural

### 1. `⚠⚠` bir SÖZLEŞMEDİR
Bu depoda `⚠⚠` ile başlayan her satır, **yaşanmış bir hatadan** doğmuştur ve genellikle
bir CI kapısıyla çivilidir. Değiştirmeden önce oku.

### 2. Sayılar ÖLÇÜLDÜ, kopyalanmadı — ama BAYATLAR
Buradaki sayılar **2026-09-17'de ölçülmüştür**. Bir sayıya dayanarak karar verecekseniz
**yeniden ölçün**; her dosya ölçüm reçetesini verir.
⚠ **Bilinçli olarak yazılmayan sayılar:** fiyat · deneme süresi · günlük tavan · KB satır
sayıları. Bunların **tek doğru kaynağı koddur** (`credits.py`, `iyzico.py`); dokümana
yazmak bir kez bayatladı.

### 3. Bu export'un TAVANI var
Derin alan bilgisi **`.claude/agents/*.md`**'de, ölçüm anlatıları
**`docs/tuzaklar-ve-denetim-anlatilari.md`**'de, tarihli kayıt
**`docs/durum-gunlugu.md`**'de, açık kalemler **`_gorev.txt`**'de kalır.
Export onlara **işaret eder**, içeriklerini kopyalamaz — çünkü *kopyalanan sayı bayatlar,
kopyalanan ad hayalet olur.*

### 4. Üretilmiş dosyaları ELLE DÜZENLEME
`03` · `04` · `15a/b/c` · `veri/*` mekanik olarak üretildi. Kod değişince
**yeniden üretilir** (reçete `18` §6).

---

## 🔴 DEVİR ANINDA BİLİNEN DURUM

| Konu | Durum |
|---|---|
| **CI** | 🔴 `ci_kapi_verify` **kırmızı** — 37 geçti · 4 kaldı. Sebep: bugün eklenen `_firat_782c_kayit_kapisi` hiçbir listede değil. **Düzeltme tek satır** (`11` §3) |
| **Açık P0** | **9 kalem** — dördü hasta güvenliği (`17` §4.1) |
| **Ödeme** | iyzico çalışıyor ama **uçtan uca doğrulanmadı** (#768c) |
| **Dönüşüm** | ⚠ **Ürün parayı hiç istemiyor**: 6 haftada fiyat sayfasını 16 kişi açtı (#489b) |
| **Stripe / ABD kolu** | Kod hazır, **bayrak kapalı** — 4 kod-dışı engel (`07` §6) |
| **iyzico recurring** | Kodda var, **bayrak kapalı** (#774c) |
| **Görüntü ön-okuma** | Env yokken **kapalı** |
| **e-Nabız eklentisi** | **DENEME kilidi** — yalnız `127.0.0.1:8000` |

---

## 📄 DEPODAKİ DİĞER KAYNAKLAR (export dışı, ama kritik)

| Dosya | Ne |
|---|---|
| **`CLAUDE.md`** | Proje kılavuzu — her oturumun yüklediği değişmez kurallar (~71 KB) |
| **`_gorev.txt`** | 578 kalem: açık işler, ölçümler, kararlar (~720 KB) |
| **`docs/tuzaklar-ve-denetim-anlatilari.md`** | Ölçüm anlatıları — tuzakların tam hikâyesi |
| `docs/durum-gunlugu.md` | Tarihli kayıt |
| `docs/kamil-ozellik-defteri.md` | Sayfa/özellik kararları |
| `docs/derya-veri-defteri.md` | KB/veri ölçümleri (tarihli) |
| `docs/kurtarma-runbook.md` | Felaket kurtarma |
| `docs/goruntuleme-sozlesme-2026-09-13.md` | Görüntüleme iki-yol sözleşmesi |
| `docs/maliyet-tavani-mimari-2026-09-02.md` | Bütçe kapısı tasarımı |
| `docs/iyzico-abonelik-acilis-2026-09-12.md` | Recurring açılış şartları |
| `docs/rakip-bosluk-arastirma-2026-07-16.md` | FDA CDS + rekabet kaynağı |
| `.claude/agents/*.md` | Alan uzmanı derin kuralları |

---

*Bu export `D:\sağlık\hive team export` altında üretildi. Sır değeri, kişisel veri ve
hukuki yorum içermez.*
