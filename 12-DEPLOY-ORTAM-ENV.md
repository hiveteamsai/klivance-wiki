# 12 — DEPLOY, ORTAM VE ENV DEĞİŞKENLERİ

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
>
> ⚠⚠ **BU DOSYADA HİÇBİR SIR DEĞERİ YOKTUR** — yalnız env **adları** ve ne işe yaradıkları.
> Sır değerleri `.env` (yerel, commit edilmez) ve Render panosundadır (`sync:false`).

---

## 1. Ortam topolojisi

```
┌────────────────────────┐   git push origin main    ┌─────────────────────────────┐
│  YEREL                 │ ────────────────────────▶ │  GitHub                     │
│  D:\sağlık             │                           │  fatihalkanalfa/klivance    │
│  .venv (Python 3.12)   │                           │  (PRIVATE), dal: main       │
│  Docker `saglik-pg`    │                           └──────────┬──────────────────┘
│  (postgres:16)         │                                      │ autoDeploy: true
│  uvicorn :8000         │                                      ▼
└────────────────────────┘                           ┌─────────────────────────────┐
                                                     │  RENDER (Oregon)            │
                                                     │  web  srv-d9agd4mcjfls739r3ja0│
                                                     │  Docker · gunicorn          │
                                                     │  PostgreSQL 18              │
                                                     └─────────────────────────────┘
```

| Ortam | URL |
|---|---|
| Yerel | `http://127.0.0.1:8000` |
| Staging (canlı) | `https://klivance.onrender.com` |
| Hedef | `https://klivance.com` |

---

## 2. YEREL GELİŞTİRME

### 2.1 Veritabanı

```bash
docker compose up -d          # kaldır  (container: saglik-pg, postgres:16, :5432)
docker compose down           # durdur  (VERİYİ KORUR)
docker compose down -v        # SIFIRLA (volume'u siler — DİKKAT)
```

### 2.2 Sunucu

```bash
# repo KÖKÜNDEN (D:\sağlık) — MUTLAKA modül yolu ile
./.venv/Scripts/python.exe -m uvicorn saglik.app.main:app --host 127.0.0.1 --port 8000
```

⚠⚠ **`saglik/` dizininin İÇİNDEN çalıştırma** — `saglik/http.py` stdlib `http`'yi gölgeler
→ uvicorn çöker.

### 2.3 Şema

```bash
./.venv/Scripts/python.exe -m saglik.app.migrate      # idempotent
./.venv/Scripts/python.exe scripts/init_db.py
./.venv/Scripts/python.exe scripts/verify_db.py
```

### 2.4 ⚠⚠ Yerel ortamın bilinen kararsızlıkları

| Sorun | Durum |
|---|---|
| **Yerel PostgreSQL segfault ediyor** | **AÇIK, kök neden bulunamadı** (2026-08-02, 3 ajan). Çökmeler 2026-07-09'a kadar geriye gidiyor = herhangi bir günün kodundan bağımsız. DB WAL redo ile toparlanır. **Uzun ölçümü yarıda kesen şey budur** — script'ini suçlamadan önce `docker logs saglik-pg`'ye bak |
| **Makine donanım kararsızlığı** | 2026-08-06'da tırmandı (BSOD, boştayken de) → **yerel KIRMIZI da kanıt değil; nihai kanıt CI** |
| ⚠ Prod maruziyeti | **BİLİNMİYOR** (farklı PG major + imaj) — yerelden çıkarımla kurma |
| ⚠ `ts_headline` | Ürün kodundakine **dokunulmadı ve dokunulmamalı** — ortam sorunu ürün regresyonuyla kapatılmaz |
| **Yerel PG zaman dilimi** | `SHOW timezone` = **`Etc/UTC`** (ölçüldü 2026-09-01) → yerel artık prod'la aynı eksende, yani bu sınıf hatayı yerelde **arayabilirsin**. ⚠ Ama TZ'i **ölçmeden VARSAYMA** (iki yönü de bir kez yanlış bilindi) |

### 2.5 ⚠ Windows'a özgü tuzaklar

| Tuzak | Doğrusu |
|---|---|
| ⚠⚠ **Kaynak `.py` dosyasını PowerShell ile YAZMA** | `Set-Content -Encoding UTF8` başa **BOM** ekler → `SyntaxError: invalid non-printable character U+FEFF`; bir kez **tüm uygulama düştü**. Okumak sorun değil |
| **Bash heredoc backslash bozar** | `<< 'PY'` içinde ters eğik çizgi öngörülemez şekilde gerçek newline'a dönüşebilir. Backslash içeren scriptleri **Write ile dosyaya yaz**, öyle çalıştır |
| **`curl -d` Türkçe karakter bozar** | JSON gövdeyi Python ile UTF-8 dosyaya yaz, `--data-binary @file` kullan |
| **Toplu düzenleme** | Python script + **tmp dosyaya yaz → `os.replace`** (truncate riski yok). ⚠ `open('w')` surrogate-emoji **yazamaz** → dosya boşalır; literal emoji ya da `\U0001F4D6` kullan. **Her yazımdan önce `ast.parse`** |
| Konsol encoding | `sys.stdout.reconfigure(encoding='utf-8')` |

---

## 3. PROD DEPLOY

### 3.1 Akış

```
git push origin main  →  Render autoDeploy  →  docker build
                      →  CMD: python -m saglik.app.migrate && gunicorn …
```

**Dockerfile CMD:**
```sh
python -m saglik.app.migrate && exec gunicorn saglik.app.main:app \
  -k uvicorn.workers.UvicornWorker \
  -w ${WEB_WORKERS:-2} -b 0.0.0.0:${PORT:-8000} \
  --timeout ${WEB_TIMEOUT:-300} --forwarded-allow-ips '*'
```

### 3.2 Dockerfile kararları

| Karar | Gerekçe |
|---|---|
| `python:3.12-slim` | |
| **Root olmayan kullanıcı** (uid 10001 `klivance`) | RCE/path-traversal doğrudan root vermesin. Uygulama hiçbir yere yazmıyor (dosyalar DB'de bytea, `.pyc` kapalı) → salt-okuma kaynakla sorunsuz. Port 8000 (>1024) → ayrıcalık gerekmez |
| `FORWARDED_ALLOW_IPS="*"` | Render proxy'sinin `X-Forwarded-For`'una güven → `request.client.host` **gerçek istemci IP'si** olur (iyzico `buyer.ip` datacenter IP'si değil = fraud sinyali düzelir; IP-bazlı kayıt limiti kullanıcı başına çalışır) |
| `PYTHONDONTWRITEBYTECODE=1` | |
| İmaja giren | `saglik/` · `db/` · `sources.yaml` |

⚠ **BİLİNEN RİSK (belgelendi, bilinçli değiştirilmedi):** `"*"` = "her kaynaktan gelen
`X-Forwarded-For`'a güven". Container'a doğrudan erişebilen biri istemci IP'sini
**istediği gibi uydurabilir**. Render ağ yalıtımı pratikte engeller ama **savunma
TEK KATMANDIR**.
⚠⚠ **DARALTMAYA KALKMA (ölçmeden):** Render/Cloudflare çıkış IP'leri sabit **değil** ve
haber vermeden değişir. Yanlış daraltma **sessizce** eski davranışa döner (herkes
datacenter IP'si görünür) → `buyer.ip` + `gclid`/kayıt atfı **bozulur**, üstelik hiçbir
hata vermez. Doğru çözüm ölçüyle gelir: gerçek proxy IP aralığı doğrulanıp CIDR listesi.

### 3.3 `render.yaml` — sıfırdan kurulum reçetesi

⚠⚠ **BU DOSYA "SIFIRDAN YENİDEN KURULUM" REÇETESİDİR.** Eksik env = **sessiz özellik
kaybı**. Önceki hâlinde `KLIVANCE_FILE_KEY`, iyzico anahtarları, SMTP, Google OAuth, admin
listesi ve Meta pixel **hiç yoktu** → felaket kurtarma senaryosunda site "ayakta" görünüp
dosya kütüphanesi, ödeme, e-posta ve ölçüm **çalışmıyor** olurdu.

Kurtarma sırası ve doğrulama adımları: **`docs/kurtarma-runbook.md`**

### 3.4 ⚠⚠ Postgres plan uyumsuzluğu (ölçüldü 2026-07-29)

```
yerelde prod'a giden şemalar:  core ≈ 2.152 MB  +  app ≈ 2 MB  ≈ 2,15 GB
                              (raw ≈ 4,2 GB PROD'A ALINMAZ)
render.yaml plan: basic-1gb   ⚠ YETERSİZ
```

**`basic-1gb` bu veriyi ALMAZ** — restore disk dolunca yarıda kalır. Yeniden provision
ederken **en az `basic-4gb`**. ⚠ Dump sıkıştırılmış ~1,3 GB'dır; **dump boyutuna bakıp
karar VERME** — restore sonrası boyut yukarıdaki rakamdır.

### 3.5 `/health` ve Render sağlık probu

```yaml
healthCheckPath: /health
```

⚠⚠ **Bu satır `main.py`'deki `/health` rotasıyla BİRLİKTE deploy edilmelidir.** Rota
olmadan blueprint uygulanırsa sağlık probu 404 alır ve Render servisi **sürekli yeniden
başlatır**.

⚠ `/health` **landing render etmez** — ucuz bir DB dokunuşu. Neden: `/` tüm landing'i
render edip DB'ye bağlanıyordu; DB yavaşladığında prob timeout → restart → açılışta
migrate → yük artar → yine timeout (**kaskad**).

### 3.6 Bilgi tabanı verisini taşıma

```bash
# Yerelde dump
docker exec saglik-pg pg_dump -U postgres -d saglik -Fc -f /tmp/saglik.dump
docker cp saglik-pg:/tmp/saglik.dump ./saglik.dump

# Prod'a restore (Render dış bağlantı URL'si ile)
pg_restore --no-owner --no-privileges -d "<RENDER_EXTERNAL_DATABASE_URL>" saglik.dump
```

⚠ **Şema yapısını `migrate.py` kurar; VERİYİ bu restore taşır.** İkisi ayrıdır.
⚠ Delta taşıma: `scripts/migrate_kb_delta.py` · `scripts/migrate_corpus.py` (dry-run varsayılan).

---

## 4. BAĞLANTI HAVUZU — `saglik/db.py`

**Neden var (2026-07-29, denetim V1):** her `connect()` YENİ bağlantı açıyordu
(TCP + TLS + auth el sıkışması; Render'da 25–60 ms). 70+ `with connect()` var, tek sohbet
turu birkaç bağlantı açıyor ve biri **LLM streaming'i boyunca açık kalıyordu** →
`WEB_WORKERS` × eşzamanlı istek arttığında Postgres **`FATAL: too many connections`** veriyordu.

| | |
|---|---|
| ⚠ **Çağrı API'si DEĞİŞMEDİ** | `with connect() as conn:` aynen çalışır (commit'te commit, istisnada rollback) |
| Kill-switch | `KLIVANCE_DB_POOL=0` → eski davranış |
| Boyut | `KLIVANCE_DB_POOL_MIN` (vars. 1) / `KLIVANCE_DB_POOL_MAX` (vars. 8) — **WORKER BAŞINA** |
| Havuz kurulamazsa | **Sessizce doğrudan bağlantıya düşer** |
| Kaçış yolu | `connect_direct()` — havuzu **bypass** eder (migrate, bakım scriptleri) |

⚠⚠ **`_pool_reset`** — havuza dönen bağlantıyı **temiz bırakır**. Kritik: oturum-düzeyi
`SET` (ör. retrieval'ın `SET max_parallel_workers_per_gather=0`) bir sonraki isteğe
**SIZMAMALI**. Açık işlem geri alınır, sonra `RESET ALL`.

⚠ **Threadpool ↔ DB havuzu oranı** açılışta hizalanır (`main._threadpool_hizala`).

---

## 5. ENV DEĞİŞKENLERİ — tam envanter

> Kaynak: `saglik/config.py` + kod genelinde `os.getenv`/`os.environ` AST/grep taraması +
> `render.yaml`. ⚠ **Değerler yoktur.**

### 5.1 Temel — ZORUNLU

| Env | Ne | Yoksa |
|---|---|---|
| `DATABASE_URL` | PostgreSQL bağlantı dizesi | `config.database_url()` **RuntimeError** |
| `ANTHROPIC_API_KEY` | Claude API | **CDS yanıt üretilemez** (ürünün çekirdeği). Açılışta uyarı |
| `KLIVANCE_FILE_KEY` | Hasta dosyası Fernet **birincil** anahtarı | Yükleme/analiz `err=svc`. ⚠⚠ **KAYBI = KALICI VERİ KAYBI** |
| `KLIVANCE_BASE_URL` | Checkout dönüş URL'leri (sonda `/` yok) | `http://127.0.0.1:8000` |
| **`KLIVANCE_COOKIE_SECURE`** | ⚠⚠ **ÇİFT GÖREVLİ**: oturum çerezi `Secure` bayrağı **+ TÜM GA4/Meta ölçümü** | `'1'` değilse ölçüm **sessizce kapanır**; `startup_warnings` bağırır |

### 5.2 Ödeme

| Env | Ne |
|---|---|
| `KLIVANCE_PAY_PROVIDER` | `iyzico` (varsayılan) \| `stripe` |
| `KLIVANCE_PAY_ROUTING` | Pazara göre yönlendirme — **varsayılan `0` = KAPALI, zorunluluk** (bkz. `07` §6) |
| `IYZICO_API_KEY` · `IYZICO_SECRET_KEY` | **ŞART** — yoksa `IyzicoMissing` |
| `IYZICO_BASE_URL` | ⚠⚠ **ŞEMASIZ HOST**. Vars. **sandbox** (kazara canlı çekim olmasın) |
| `IYZICO_MERCHANT_ID` | |
| `KLIVANCE_IYZ_PAYGROUP` | iyzico `paymentGroup` |
| `KLIVANCE_IYZ_RECURRING` | Otomatik yenileme bayrağı (`credits.iyz_recurring()`) — **KAPALI** |
| `STRIPE_SECRET_KEY` · `STRIPE_WEBHOOK_SECRET` | ABD kolu (pasif) |
| `KLIVANCE_STRIPE_TAX` | Stripe vergi |
| `KLIVANCE_USD_TRY` | Kur (admin ayarına düşer) |

⚠⚠ **Render'da kalmış bayat `IYZICO_PRICE_*` env'i yeni fiyatı SESSİZCE EZER.**
(`IYZICO_PRICE_S50/S100/S150` ve `_TOPUP_PRICE_DEFAULT` 2026-08-21'de **öldü.**)

### 5.3 LLM / model kademesi

| Env | Varsayılan |
|---|---|
| `KLIVANCE_CLASSIFIER_MODEL` | `claude-haiku-4-5` |
| `KLIVANCE_ANSWER_MODEL` | `claude-opus-4-8` |
| `KLIVANCE_MAP_MODEL` | `claude-sonnet-5` |
| `KLIVANCE_ATTACH_MAP_MODEL` · `KLIVANCE_ATTACH_MAP` | ek eşleme |
| `KLIVANCE_CRITIC_MODEL` | `claude-opus-4-8` |
| `KLIVANCE_MID_MODEL` | `claude-sonnet-5` |
| `KLIVANCE_COUNCIL_MODEL` | `claude-opus-4-8` |
| `KLIVANCE_GORUNTU_MODEL` | `claude-sonnet-5` |
| `KLIVANCE_EFFORT` | `high` |
| `KLIVANCE_MAP_WORKERS` | MAP paralelliği |

⚠ `llm.fiyatsiz_modeller()` — `PRICES` tablosunda karşılığı olmayan **etkin** modelleri
döndürür; boş değilse **maliyet ölçümü sessizce yanlıştır** (kapı `maliyet_korlugu_verify`).

### 5.4 Özellik bayrakları

| Env | Ne | Varsayılan |
|---|---|---|
| `KLIVANCE_GORUNTU_ONOKUMA` | Görüntü ön-okuma (YOL 2) | **YOKKEN KAPALI** |
| `KLIVANCE_BUTCE_MOD` | `gozlem` \| `uygula` | `gozlem`. ⚠ `uygula` **ifşa maddesi yokken KODDA REDDEDİLİR** |
| `KLIVANCE_SORU_KAYIT` · `KLIVANCE_QUERY_STAT` · `KLIVANCE_WALL_STAT` | Kayıt katmanları | |
| `KLIVANCE_CIHAZ_UI` · `KLIVANCE_CIHAZ_UI_HEKIM` | Cihaz eşleştirme UI | |
| `KLIVANCE_DOCS` | `/docs` `/redoc` `/openapi.json` | **`1` ile yalnız yerelde** |
| `KLIVANCE_CONFIG_CHECK` | Açılış uyarıları | `1` (`0` susturur) |

### 5.5 E-posta (Resend SMTP)

`KLIVANCE_SMTP_HOST` · `_PORT` (vars. 587) · `_USER` · `_PASS` · `_FROM` (vars. `_USER`).
**Eksikse `mailer` sessizce atlar** ve link log'a yazılır.

### 5.6 Kimlik / yönetim

| Env | Ne |
|---|---|
| `GOOGLE_OAUTH_CLIENT_ID` · `_SECRET` | Yoksa "Google ile devam et" **gizlenir** |
| `KLIVANCE_ADMIN_EMAILS` | Virgülle ayrık admin e-postaları (`/admin` erişimi) |
| `KLIVANCE_ADMIN_EMAIL` · `KLIVANCE_ADMIN_LOG` | |

### 5.7 Ölçüm / reklam (app runtime **import etmez** — scriptler kullanır)

`META_PIXEL_ID` (app kullanır) · `META_ACCESS_TOKEN` · `META_ADS_TOKEN` ·
`META_SYSTEM_USER_TOKEN` · `META_TOKEN` · `META_AD_ACCOUNT_ID` · `META_PAGE_ID` ·
`META_PAGE_TOKEN` · `GOOGLE_ADS_DEVELOPER_TOKEN` · `GOOGLE_ADS_REFRESH_TOKEN` ·
`GOOGLE_ADS_CLIENT_ID` · `GOOGLE_ADS_CLIENT_SECRET` · `GOOGLE_ADS_LOGIN_CUSTOMER_ID` ·
`GOOGLE_ADS_CUSTOMER_ID`

### 5.8 Veri hattı

`OPENFDA_API_KEY` (opsiyonel, kota yükseltir) · `ICD11_CLIENT_ID` · `ICD11_CLIENT_SECRET`
(OAuth2) · `KLIVANCE_KUB_GORSEL_SAYFA` · `KLV_V8_GORSEL_DIZIN`

### 5.9 Altyapı ayarı

`KLIVANCE_DB_POOL` · `_MIN` · `_MAX` · `_TIMEOUT` · `KLIVANCE_MIGRATE_LOCK_TIMEOUT`
(vars. `3s`) · `KLIVANCE_REINDEX_TIMEOUT` · `WEB_WORKERS` (vars. 2) · `WEB_TIMEOUT`
(vars. 300) · `PORT` (vars. 8000) · `FORWARDED_ALLOW_IPS` · `RENDER` · `RENDER_GIT_COMMIT`
(sürüm damgası)

### 5.10 ⚠⚠ Politika env'i

| Env | Kural |
|---|---|
| **`PROD_DATABASE_URL`** | `.env`'de **VARDIR** → kural teknik zorunluluk değil **POLİTİKA**: prod DB'ye **founder onayı olmadan BAĞLANILMAZ**; onaylıysa **salt-okunur** (`SET default_transaction_read_only=on`) |

---

## 6. AÇILIŞTA YANLIŞ-YAPILANDIRMA AVCISI — `config.startup_warnings()`

Import anında (yani her worker açılışında) stderr'e **gürültülü** basar. Render log'unda
görünür. Susturmak: `KLIVANCE_CONFIG_CHECK=0`.

Kontrol ettikleri:
1. `KLIVANCE_BASE_URL` https ama `KLIVANCE_COOKIE_SECURE ≠ 1` → **çerez Secure değil + ölçüm KAPALI**
2. `KLIVANCE_FILE_KEY` yok → dosya kütüphanesi çalışmaz
3. `PAY_PROVIDER=iyzico` ama anahtar yok → **satış YAPILAMAZ**
4. `PAY_PROVIDER=stripe` ama `STRIPE_SECRET_KEY` yok → satış yapılamaz
5. `IYZICO_BASE_URL` **sandbox** ama site canlı adreste → **gerçek tahsilat OLMAZ**
6. `ANTHROPIC_API_KEY` yok → CDS üretilemez

> **Neden var (2026-07-29 denetimi):** kritik env'lerin kaybı **hiçbir iz bırakmadan**
> davranış değiştiriyordu. Reklam bütçesi akarken ölçüm ölmüş olabiliyordu.

---

## 7. GIT DİSİPLİNİ (paylaşılan ağaç — eş zamanlı çalışma var)

| Kural | Neden |
|---|---|
| **Dosyayı düzenlemeden önce `git status` ile KİRLİ Mİ bak** | `git log` **yetmez** — commit'e bakmak çalışma ağacındaki işi göstermez |
| ⚠⚠ **Başkasının commit'ini DÜŞÜRME — commit bölmek için `reset` KULLANMA** | `reset --mixed HEAD~1` sonraki **her** commit'i geri alır (üç commit sessizce düştü, belirti yoktu). Kurtarma: `git reflog` → birebirliği **ÖLÇ** |
| ⚠⚠ **`git commit -- <yol>` İZOLASYON SAĞLAMAZ** | O anki ağacı alır → eşzamanlı yazanın satırı senin commit'ine girer; içerik doğru kalır ama **ATIF bozulur**. Yalnız **kendi hunk'ın** (`add -p`) + `git show --stat` ile **geri oku** |
| ⚠⚠ **PUSH DALIN TAMAMINI YAYINLAR** | Onay bekleyen tutulanı da. Push'tan hemen önce `git log origin/main..HEAD` ile **LİSTELE**; başkasınınki varsa `git push origin <sha>:main` ile **sabitle** (HEAD'e değil), sonra `fetch` + geri oku |
| ⚠⚠ Sabitleme de yetmez | Altında oluşan commit yine yayınlanır → **stage bir REZERVASYON DEĞİL**; bekletme, hemen commit et |
| ⚠⚠ **`git fsck` push öncesi** | Bu makinede **git nesneleri yazarken bozuluyor**; *"commit atıldı"* kanıt değildir (memory `makine-git-nesne-bozulmasi`) |
| Commit mesajı sonu | `Co-Authored-By: Claude <model> <noreply@anthropic.com>` — **oturumun GERÇEK modeli**; sabit ad yazma (bir kez çivilendi ve bayatladı) |

---

## 8. YAYIN ÖNCESİ KONTROL LİSTESİ

- [ ] `KLIVANCE_COOKIE_SECURE=1` (HTTPS)
- [ ] `IYZICO_BASE_URL` = `api.iyzipay.com` (sandbox **değil**), **şemasız**
- [ ] `KLIVANCE_FILE_KEY` **Render dışında da yedeklendi**
- [ ] `app.setting 'trial_salt'` yedeklendi (**tek nokta arızası**)
- [ ] `ANTHROPIC_API_KEY` rotasyonlandı
- [ ] `curl https://klivance.com/health` → 200 **+ sürüm damgası** doğru sha
- [ ] Render'da bayat `IYZICO_PRICE_*` env'i **yok**
- [ ] `/docs` `/redoc` `/openapi.json` → 404
- [ ] CI yeşil (`ci_kapi_verify` dahil)
- [ ] `git fsck` temiz, `git log origin/main..HEAD` yalnız kendi commit'lerin

---

## 9. KURTARMA

Tam runbook: **`docs/kurtarma-runbook.md`**

| Senaryo | Kritik nokta |
|---|---|
| DB restore | ⚠⚠ **`trial_salt` tuzunu yedeklemeden restore etme** — deneme freni düşer |
| `KLIVANCE_FILE_KEY` kaybı | **Kurtarılamaz** — tüm hasta dosyaları kalıcı okunamaz |
| Anahtar rotasyonu | `KLIVANCE_FILE_KEY_OLD` + `scripts/rotate_file_key.py` |
| `main.py` bozuldu | `git checkout HEAD -- saglik/app/main.py` |
| Bozuk git nesnesi | `git fsck` → `git reflog` → prune reçetesi (memory `makine-git-nesne-bozulmasi`) |
| Yedek | `scripts/yedek_al.py` (kapı `yedek_ezme_verify`) |

---

**Sonraki:** [`13-SCRIPT-ENVANTERI.md`](13-SCRIPT-ENVANTERI.md)
