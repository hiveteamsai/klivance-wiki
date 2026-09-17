# 18 — DEVİR NOTLARI (hiveteams.ai için)

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> **Bu dosya "nasıl devralınır" sorusunun cevabıdır.** İlk saat, ilk gün, ilk hafta.

---

## 1. İlk 30 dakika — okuma sırası

| Sıra | Dosya | Neden |
|---|---|---|
| 1 | **`CLAUDE.md`** (depo kökü, ~71 KB) | Her oturumun yüklediği değişmez kurallar + dosya haritası. **Bu export onun genişletilmiş hâlidir.** |
| 2 | [`00-URUN-VE-KAPSAM.md`](00-URUN-VE-KAPSAM.md) | Ne yapar, ne yapmaz, hangi sınırlar |
| 3 | [`01-MIMARI.md`](01-MIMARI.md) | Katmanlar, istek yaşam döngüsü |
| 4 | [`16-ISLETME-KURALLARI-VE-KARARLAR.md`](16-ISLETME-KURALLARI-VE-KARARLAR.md) | "Neden böyle" — tekrar tartışmamak için |
| 5 | [`17-BILINEN-TUZAKLAR-VE-BORCLAR.md`](17-BILINEN-TUZAKLAR-VE-BORCLAR.md) | Aynı çukura düşmemek için |

⚠ **Bu export'un tavanı vardır.** Derin alan bilgisi `.claude/agents/*.md`'de, ölçüm
anlatıları `docs/tuzaklar-ve-denetim-anlatilari.md`'de kalır. Export onlara **işaret eder**,
içeriklerini kopyalamaz (kopyalanan sayı bayatlar, kopyalanan ad hayalet olur).

---

## 2. Ortamı kurma

### 2.1 Gereksinimler
- Python **3.12**
- Docker (PostgreSQL 16)
- Node.js (JS kapıları için)
- Git

### 2.2 Adımlar

```bash
# 1. Depo
git clone <repo> && cd <repo>

# 2. Sanal ortam
python -m venv .venv
./.venv/Scripts/python.exe -m pip install -r requirements.txt
# Reklam scriptleri gerekiyorsa AYRICA:
./.venv/Scripts/python.exe -m pip install -r requirements-ads.txt

# 3. Veritabanı
docker compose up -d            # container: saglik-pg, :5432

# 4. .env dosyası (kökte, COMMIT EDİLMEZ)
#    Zorunlu: DATABASE_URL, ANTHROPIC_API_KEY, KLIVANCE_FILE_KEY
#    Tam liste: 12-DEPLOY-ORTAM-ENV.md §5
#    KLIVANCE_FILE_KEY üretmek için:
#    python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"

# 5. Şema
./.venv/Scripts/python.exe -m saglik.app.migrate

# 6. Sunucu — REPO KÖKÜNDEN, modül yolu ile
./.venv/Scripts/python.exe -m uvicorn saglik.app.main:app --host 127.0.0.1 --port 8000
```

⚠⚠ **`saglik/` dizininin İÇİNDEN çalıştırma** — `saglik/http.py` stdlib `http`'yi gölgeler.

### 2.3 ⚠ Bilgi tabanı VERİSİ ayrıca gerekir

Şema boş gelir; ürünün çalışması için `core.*` verisi lazım (ilaç, hastalık, korpus).
Kaynak: prod ya da yerel `pg_dump` (bkz. `12-DEPLOY-ORTAM-ENV.md` §3.6).

**Verisiz de çalışan yüzeyler:** kayıt/giriş, arayüz, hesaplayıcılar (istemci-tarafı).
**Verisiz çalışmayan:** CDS yanıtları, ilaç kartı, etkileşim, referans.

### 2.4 Doğrulama

```bash
./.venv/Scripts/python.exe scripts/verify_db.py     # tablo bazında satır sayısı
./.venv/Scripts/python.exe scripts/smoke_test.py    # duman testi
curl http://127.0.0.1:8000/health                   # 200 + sürüm damgası
```

---

## 3. İlk değişikliği yapmadan önce — 6 adımlık çek listesi

1. **`git status`** — dosya kirli mi? (`git log` **yetmez**)
2. **Dosyanın kendi başlık docstring'ini oku.** Bu depoda invariantlar dosya
   başlıklarındadır (~278.000 karakter). `⚠⚠` ile başlayan her satır bir **sözleşmedir**.
3. **Hangi kapı bu dosyayı koruyor?**
   `grep -l "<dosya adı>" scratchpad/*_verify.py`
4. **Dosya tavanı:** `python scratchpad/dosya_boyut_verify.py`
5. **Değişikliği yap** → ilgili kapıları **koş**
6. **Syntax:** `python -c "import ast; ast.parse(open('<dosya>',encoding='utf-8').read())"`

---

## 4. Kod tabanının "okuma anahtarları"

| Gördüğün | Anlamı |
|---|---|
| `⚠⚠` | **Sözleşme.** Bozmadan önce oku; çoğu ölçülmüş bir hatadan doğdu |
| `⚠` | Uyarı / tuzak |
| `#NNNb` / `#NNNc` | `_gorev.txt` kalem numarası — o kalemi oku |
| `@_rota("get", "/yol")` | Rota tanımı (`include_router` yok) |
| `def kur(app)` | `main.py`'nin çağırdığı kayıt fonksiyonu |
| `# noqa: F401 (fasad)` | `main.py`'nin **yeniden dışa verdiği** ad — silmeden önce AST ile dış tüketici tara |
| `*_body.py` | **SAF** sabit (f-string yok, import yok) |
| `SAF MODÜL: … import 0` | Hiçbir şey import etmez ve **etmemeli** |
| `ÜRETİLMİŞ DOSYA — ELLE DÜZENLEME` | Üreteci var, elle dokunma |
| `store._admin_safe` → `ok`/`error` | Üç-durum disiplini |
| `status="unavailable"` | **Fail-open yasağı** — asla "clean" |

---

## 5. Bu deponun beş "kişiliği"

Kod, ölçülmüş hatalardan doğan beş refleks taşır. Bunları **anlamadan** değiştirmek
regresyon üretir:

1. **"Çalıştırılamadı ≠ temiz."** Belirsizlik **amber**dir, asla yeşil.
2. **"Son adımı ölç."** Fonksiyon değil, **tüketici katman** ölçülür.
3. **"Sayıyı kopyalama."** Sayı tek yerde durur; belge ona **işaret eder**.
4. **"Aracı da sına."** Ölçüm aracının kendi öz-testi vardır, **iki yönlü**.
5. **"Reddedileni yeniden önerme."** Reddedilmiş her şeyin **gerekçesi yazılı**dır.

---

## 6. ÜRETİLMİŞ DOSYALARI YENİDEN ÜRETME

Bu export'un dört dosyası **mekanik olarak üretildi** — elle güncelleme:

| Dosya | Kaynak |
|---|---|
| `03-ROTA-ENVANTERI.md` | Canlı `app.routes` |
| `04-VERITABANI.md` | `information_schema` + `pg_indexes` + `pg_constraint` |
| `15a/b/c-FONKSIYON-REFERANSI-*.md` | Python `ast` ile kaynak ağacın tamamı |
| `veri/*.json` | Yukarıdakilerin ham çıktısı |

### Yeniden üretim reçetesi

```python
# 1) Modül + fonksiyon envanteri (AST)
#    saglik/ altındaki her .py için: docstring, import, fonksiyon imzaları,
#    sınıflar, BÜYÜK_HARF sabitler → veri/moduller.json

# 2) Rota envanteri — UYGULAMANIN KENDİSİNDEN
import sys; sys.path.insert(0, "<repo kökü>")
from saglik.app.main import app
for r in app.routes:
    r.path, sorted(r.methods or []), r.name, r.endpoint.__module__
# ⚠ Desenden ÇIKARMA — `include_router` bilerek yok, `app.routes` düz.

# 3) Şema — yerel DB'den, SALT-OKUNUR
conn.execute("SET default_transaction_read_only=on")
# information_schema.columns + pg_indexes + pg_constraint
# ⚠ SATIR SAYISI ALMA — bayatlar.

# 4) Kapı envanteri
# .github/workflows/test.yml içinden SUITES / SKIP_SUITES / JS_SUITES dizelerini
# regex ile çıkar (⚠ `(?<![A-Z_])SUITES=` — yoksa SKIP_SUITES'le de eşleşir),
# her kapının docstring ilk satırını ekle.
```

⚠ `veri/*.json` dosyaları makine-okunurdur; bir agentic araç bunları doğrudan tüketebilir.

---

## 7. AGENTIC ARAÇ İÇİN NOTLAR

Klivance'ı bir AI SDLC ürününe devrederken bilinmesi gerekenler:

### 7.1 Bu depo **kendi kendini belgeler**
~278.000 karakter docstring. Bir modülü değiştirmeden önce **başlığını okumak** neredeyse
her zaman yeterlidir. Ajanlara verilecek ilk talimat bu olmalı.

### 7.2 Kapılar **davranış sözleşmesidir**, test değil
320 kapı, "bu fonksiyon doğru mu"yu değil **"bu davranış hâlâ bağlı mı"**yı ölçer.
Bir ajan kod ürettiğinde **ilgili kapıyı koşturmalı**, ve kapı kırmızıysa **kapıyı değil
kodu** düzeltmeli (istisna: kapı gerçekten bayatladıysa — o zaman kabul testi
*"eskisinin kaçırdığını"* göstermeli).

### 7.3 Türkçe kod tabanı
Değişken/fonksiyon adları, docstring'ler, yorumlar **Türkçe**dir.
⚠ **Türkçe İ tuzağı gerçektir**: `'TİTCK'.lower()` = `ti̇tck` ≠ `cf('TİTCK')` = `titck`.
Python `.lower()` ile Postgres `lower()` **farklı** sonuç verir. Normalizasyon için
`kaynaklar.cf` / `reference._cf` kullanılır.

### 7.4 Gizli alanlar — otomatik ajanın **dokunmaması gerekenler**

| Alan | Neden |
|---|---|
| **Para yolu** (`iyzico*.py`, `billing.py`, `account_routes` ödeme dalları) | Founder onayı + kontrolcü denetimi ister |
| **Canlı reklam** (`scripts/meta_*`, `gads_*`) | Para harcar; canlı hesaba yazar |
| **Mail gönderimi** (`scripts/*mail*`, `mailer`) | Gerçek hekimlere gider |
| **Prod DB** (`PROD_DATABASE_URL`) | Founder onayı; onaylıysa **salt-okunur** |
| **Yasal metin** (`legal_body.py`) | Founder kararları + avukat teyidi |
| **Prompt tabanı** (`prompts.DOCTOR_SYSTEM`) | Cache tabanı + 4 kapı + founder kuralları |
| **`credits.py` sabitleri** | Para; ölçülmeden değişmez |

### 7.5 ⚠⚠ Ücretli API kuralı
Founder kuralı (2026-09-06): **abonelikle yapılabilen hiçbir şey ücretli API ile yapılmaz.**
Deney/ölçüm/sınıflandırma **alt ajanla**; ürün API yolu yalnız kod yolundan geçmesi
**zorunlu** olan ön-kayıtlı tek doğrulama koşumu içindir.
Ayrıca test çağrıları **günde ≤10** ve **adet + maliyetle beyan edilir**.

### 7.6 Değişiklik riski haritası

| Risk | Dosyalar | Gerekçe |
|---|---|---|
| 🔴 **Yüksek** | `cds/chat.py` · `cds/engine.py` · `cds/reference.py` · `cds/prompts.py` · `app/store.py` · `app/credits.py` · `app/iyzico.py` · `app/auth.py` | Hasta güvenliği / para / atıf zinciri |
| 🟡 **Orta** | `*_routes.py` · `app/webutil.py` · `app/main.py` | Rota sırası, fasadlar, middleware |
| 🟢 **Düşük** | `*_body.py` · `ui_css.py` · `i18n_pairs.py` | Saf sunum (⚠ ama apostrof tuzağı + token kuralları var) |

---

## 8. "BİR ÖZELLİK EKLE" AKIŞI — referans

```
1.  Hangi katman?        → 01-MIMARI.md §2 · 05-CDS §5.1 (Derya/Cahit ayrımı)
2.  Yeni rota mı?        → *_routes.py içinde @_rota + kur(app)
                           ⚠ main.py'deki kur() SIRASINI kontrol et (eşleşme sırası)
                           ⚠ /api/cases/{cid}/... ise SON SEGMENT STATİK olsun
3.  Ücretli mi?          → ÜÇLÜ ZİNCİR: _erisim_402 → _butce_fren → _gunluk_fren_429
                           ⚠ Güvenlik ucuysa zincirin DIŞINDA + _gunluk_tavan
4.  Ağır iş var mı?      → run_in_threadpool (event loop'ta çalıştırma)
5.  DB yazıyor mu?       → CASCADE'i açıkça yaz · SAVEPOINT disiplini
                           ⚠ Yeni app.doctor kolonu → arşive gitmeli mi KARAR VER
6.  Metin var mı?        → EN_PAIRS'e çift ekle VEYA sunucuda lang ile üret
                           ⚠ Kıvrık kesme U+2019 · EN_PAIRS anahtarlarıyla çakışma
7.  Yeni kaynak mı?      → 06-BILGI-TABANI §8 (9 adım)
8.  Ölçüm/piksel mi?     → _ANALYTICS_TAG'in İÇİNE · hassas sorgu → _ANALYTICS_PATH_ONLY
9.  Kapı yaz             → scratchpad/<ad>_verify.py + test.yml SUITES'e EKLE
10. Tüketici katmanda ölç → "X eklendi" demeden önce X'i TÜKETEN katmanda gör
11. ci_kapi_verify koş   → yeni kapı bağlı mı
12. Commit + push        → git fsck · git log origin/main..HEAD
```

---

## 9. KRİTİK SIRLAR VE YEDEKLEME

Devir sırasında **mutlaka** aktarılması gerekenler (değerleri bu export'ta **yoktur**):

| Sır | Kaybı ne olur |
|---|---|
| **`KLIVANCE_FILE_KEY`** | ⚠⚠ **TÜM HASTA DOSYALARI KALICI OKUNAMAZ** — kurtarılamaz |
| **`app.setting 'trial_salt'`** | ⚠⚠ Deneme freni düşer + panel "hepsi ilk kez geldi" der. **Yedeklemeden DB restore etme** |
| `ANTHROPIC_API_KEY` | CDS çalışmaz |
| `IYZICO_API_KEY` / `_SECRET_KEY` | Tahsilat durur |
| `DATABASE_URL` (prod) | — |
| GitHub secret `DATABASE_URL` | 4 cron durur (besleme, KVKK purge, retraction, SGK) |
| Google OAuth client secret | ⚠ Google **orijinal secret'ı GÖSTERMEZ** — yenisi üretilmeli |
| Resend SMTP anahtarı | Mail susar (sessizce) |
| `META_*` / `GOOGLE_ADS_*` | Reklam yönetimi durur |

⚠ **`.env` commit edilmez** (`.gitignore`). Sırlar Render panosunda `sync:false`.

---

## 10. DIŞ HESAPLAR / SERVİSLER

| Servis | Kimlik | Not |
|---|---|---|
| GitHub | `fatihalkanalfa/klivance` (**private**), dal `main` | `autoDeploy` açık |
| Render | web `srv-d9agd4mcjfls739r3ja0` + PostgreSQL 18 (Oregon) | |
| iyzico | Alfa 4N üye işyeri | |
| Stripe | Klivance, LLC (Delaware, Atlas) — **aktif, canlı ödemeye hazır**; tek eksik payout banka | Kodda **pasif** |
| Anthropic | Tek hesap (hekimlere anahtar verilmez) | |
| Google Cloud | Proje **"Klivance"** (`alfa4n`'den **ayrı**) | OAuth marka adı gerekçesi: `07` §1.6 |
| Google Ads | MCC `9209924231` → hesap `5989180376` | |
| GA4 | `G-FKK3Z0TWZ9` · property `properties/545762878` | |
| Meta | App ID `1334867008814787` · Pixel env'de | |
| Resend | `info@klivance.com` | |

---

## 11. BU EXPORT'UN SINIRI — dürüst beyan

**İçermez:**
- Sır/anahtar **değerleri** (bilinçli)
- KB satır sayıları ve hacimler (bayatlar — ölçüm reçetesi verildi)
- Kişisel veri örneği
- Hukuki yorum (founder kuralı)
- Reklam kreatif külliyatı ve performans defteri (`reklam/ornek-vakalar.md`,
  `.claude/agents/hasan.md`) — ürün kodu değil
- `.claude/agents/*.md` derin alan kurallarının **tamamı** — işaret edildi, kopyalanmadı

**Doğrulanmamış olanlar (bu export sırasında ölçülmedi):**
- Prod DB'nin güncel durumu (founder onayı gerekir)
- Canlı reklam hesaplarının durumu
- GitHub Actions'ın güncel yeşil/kırmızı durumu (`gh` kurulu değil)
- `#639b`'nin (CI yedi gündür kırmızı) güncel geçerliliği

**Bu export sırasında ÖLÇÜLENLER:**
- 142 rota, 7 middleware — canlı `app.routes`'tan
- 55 tablo — yerel PostgreSQL'den, salt-okunur
- 140 modül, 1.514 modül-düzeyi fonksiyon + 23 sınıf / 55 metot — AST ile
- 320 CI kapısı — `test.yml` + her kapının docstring'i
- 177 script — AST + docstring
- `ci_kapi_verify` → **37 geçti · 4 kaldı** (KAPISIZ: `_firat_782c_kayit_kapisi`)
- 3 script modül düzeyinde çıplak `sys.exit` + `__main__` kapısı yok

---

## 12. KİMİNLE KONUŞULUR

Klivance'ın iç çalışma modeli **tek giriş noktalıdır**: founder tek yerle konuşur.
Devralan ekip için karşılığı: **ürün kararları founder'ındır**; teknik kararlar kod
tabanının kendi disiplinine (kapılar + dosya başlıkları) tabidir.

**Onay gerektiren işler:**
- Para harcayan / dışarı çıkan her işlem (reklam, mail, ödeme)
- Canlı reklam ya da canlı site değişikliği
- Prod DB'ye bağlanma
- Yasal metin değişikliği
- Founder'a bir **sayı** ya da **hüküm** iletilmesi (önce doğrulanır)

---

**← Başa dön:** [`INDEX.md`](INDEX.md)
