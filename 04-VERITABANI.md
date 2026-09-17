# 04 — VERİTABANI ŞEMASI

> **Üretilmiş dosya — elle düzenleme.** Kaynak: yerel PostgreSQL `information_schema` + `pg_indexes` + `pg_constraint` (salt-okunur).
> Ölçüm anı: 2026-09-17 · commit `cb16758d`. Yeniden üretmek için export kökündeki
> `veri/` JSON'larını besleyen çıkarıcıyı koştur (bkz. `18-DEVIR-NOTLARI.md` §6).

**PostgreSQL 16 (yerel Docker `saglik-pg`) / 18 (prod Render, Oregon).** 3 şema · 55 tablo.

⚠⚠ **SATIR SAYISI BU DOSYADA YOKTUR — bilinçli.** Depoda satır sayıları bir kez
bayatladı; iddia kuracak olan yeniden ölçer. Ayrıca **prod KB ≠ yerel KB**: cron
(`sync-feed.yml`) `openfda` + `clinicaltrials`i **doğrudan prod'a** yazar, yani prod
bu ikisinde yerelden İLERİDEDİR.

## Şema ayrımı

- **`app`** (32 tablo) — SaaS uygulama verisi — hekim hesabı, sohbet, hasta defteri, ödeme, ölçüm. **Kişisel veri buradadır.**
- **`core`** (22 tablo) — Bilgi tabanı (KB) — ilaç, hastalık, korpus, yan etki, kodlar. **Kişisel veri YOKTUR.**
- **`raw`** (1 tablo) — Ara-staging (ham connector çıktısı, ~4 GB). **Prod'a ALINMAZ.**

## Bağlantı disiplini (ÜRÜN KURALI — bozma)

- `connect()` **`saglik/db.py`**'de ve **`@contextmanager`** → `with connect() as conn:`
  (`saglik/app/db.py` YOKTUR; çıplak ad bir kez bir aracı yanılttı).
- Havuzlu (`psycopg_pool`) — env `KLIVANCE_DB_POOL*`.
- ⚠⚠ **Aynı transaction'da "olursa olur" yazma → SAVEPOINT** (`with conn.transaction():`).
  Postgres'te başarısız bir ifade **tüm transaction'ı zehirler**; `try/except` hatayı
  yutsa bile sonraki `commit()` `InFailedSqlTransaction` alır ve **asıl yazma da düşer**.
- ⚠⚠ **SAVEPOINT'in İÇİNDE `commit()` ÇAĞIRMA** — psycopg3 yasaklar, yazma **sessizce düşer**.
  `store.*` fonksiyonlarının çoğu sonunda `conn.commit()` yapar; biri `with conn.transaction():`
  içine alındığı an ölür. Doğrusu commit'i SAVEPOINT'ten SONRA atmaktır.
- ⚠⚠ **Geri okuma aynı bağlantıdan yapılırsa kanıt değildir.** Bir bağlantı kendi
  commit'lenmemiş yazmalarını görür; `autocommit` kapalıyken `conn.execute("SET …")`
  zaten transaction açar → `close()` dışarıyı **ROLLBACK** eder. **Yazma iddiasını AYRI
  BAĞLANTIDAN doğrula.** `rowcount` yazma kanıtı DEĞİLDİR.
- ILIKE'a kullanıcı girdisi → `reference._like_escape` (`%`/`_` kaçır).
- Tipsiz SQL parametresi + `except: pass` = **sessiz veri kaybı (P0)** → `%s::text` ile tip ver.

## Hesap yaşam döngüsü — 2 doğum, 2 ölüm yolu (BAĞLAYICI)

| Yön | Fonksiyon |
|---|---|
| doğum | `auth.create_doctor` · `auth.upsert_google_doctor` |
| ölüm | `store.admin_archive_and_delete` · `store.purge_due_accounts` |

⚠ **Yeni `doctor_id`'li tablo eklersen CASCADE'i AÇIKÇA yaz** — yoksa KVKK silmesi FK
hatasıyla bloklanır. ⚠ Yeni kişi/olay tablosu **kurulmaz** (`app.person`/`app.account_event`
reddedildi): "geri döndü" arşivde okuma anında `lower(email)` JOIN'iyle, "tekrar kayıt"
mevcut HMAC ile üretilir → sıfır yeni kalıcı veri, sıfır yeni ifşa yükümlülüğü.

⚠⚠ **Kanonik e-posta = Python `.strip().lower()`; HMAC'i ASLA SQL'de üretme** — Türkçe İ
yüzünden Python ile Postgres `lower()` FARKLI hash üretir.

---

# Şema `app`

SaaS uygulama verisi — hekim hesabı, sohbet, hasta defteri, ödeme, ölçüm. **Kişisel veri buradadır.**

## `app.admin_erisim`

> Admin denetim logu (`_admin()` kapısı yazar). ⚠ Loga **İÇERİK YAZILMAZ** (metin · `?q=` terimi).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.admin_erisim_id_seq'::regclass)` |
| 2 | `admin_id` | `bigint` | **hayır** | |
| 3 | `hedef_id` | `bigint` | evet | |
| 4 | `hedef_kume` | `ARRAY` | evet | |
| 5 | `kayit_sayisi` | `integer` | evet | |
| 6 | `yol` | `text` | **hayır** | |
| 7 | `eylem` | `text` | **hayır** | |
| 8 | `at` | `timestamp with time zone` | **hayır** | `now()` |
| 9 | `ip` | `inet` | evet | |
| 10 | `hedef_kayit` | `bigint` | evet | |

**Kısıtlar:**

- `PK` `admin_erisim_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `admin_erisim_admin_idx`
  ```sql
  CREATE INDEX admin_erisim_admin_idx ON app.admin_erisim USING btree (admin_id, at DESC)
  ```
- `admin_erisim_at_idx`
  ```sql
  CREATE INDEX admin_erisim_at_idx ON app.admin_erisim USING btree (at DESC)
  ```
- `admin_erisim_hedef_idx`
  ```sql
  CREATE INDEX admin_erisim_hedef_idx ON app.admin_erisim USING btree (hedef_id, at DESC) WHERE (hedef_id IS NOT NULL)
  ```
- `admin_erisim_kayit_idx`
  ```sql
  CREATE INDEX admin_erisim_kayit_idx ON app.admin_erisim USING btree (hedef_kayit, at DESC) WHERE (hedef_kayit IS NOT NULL)
  ```
- `admin_erisim_kume_idx`
  ```sql
  CREATE INDEX admin_erisim_kume_idx ON app.admin_erisim USING gin (hedef_kume)
  ```
- `admin_erisim_pkey`
  ```sql
  CREATE UNIQUE INDEX admin_erisim_pkey ON app.admin_erisim USING btree (id)
  ```

## `app.answer_layer`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.answer_layer_id_seq'::regclass)` |
| 2 | `message_id` | `bigint` | evet | |
| 3 | `katman` | `text` | **hayır** | |
| 4 | `durum` | `text` | **hayır** | |
| 5 | `tetiklendi` | `boolean` | evet | |
| 6 | `ms` | `integer` | evet | |
| 7 | `cost_usd` | `numeric` | evet | |
| 8 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `CHECK` `answer_layer_durum_ck` → `CHECK ((durum = ANY (ARRAY['ok'::text, 'unavailable'::text, 'error'::text])))`
- `FK` `answer_layer_message_id_fkey` → `FOREIGN KEY (message_id) REFERENCES app.message(id) ON DELETE CASCADE`
- `PK` `answer_layer_pkey` → `PRIMARY KEY (id)`
- `CHECK` `answer_layer_tetik_ck` → `CHECK (((durum = 'ok'::text) OR (tetiklendi IS NULL)))`

**İndeksler:**

- `answer_layer_katman_idx`
  ```sql
  CREATE INDEX answer_layer_katman_idx ON app.answer_layer USING btree (katman, created_at)
  ```
- `answer_layer_msg_idx`
  ```sql
  CREATE INDEX answer_layer_msg_idx ON app.answer_layer USING btree (message_id)
  ```
- `answer_layer_pkey`
  ```sql
  CREATE UNIQUE INDEX answer_layer_pkey ON app.answer_layer USING btree (id)
  ```

## `app.appointment`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.appointment_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `case_id` | `bigint` | evet | |
| 4 | `kind` | `text` | **hayır** | `'randevu'::text` |
| 5 | `starts_at` | `timestamp with time zone` | **hayır** | |
| 6 | `title` | `text` | **hayır** | `''::text` |
| 7 | `done` | `boolean` | **hayır** | `false` |
| 8 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `appointment_case_id_fkey` → `FOREIGN KEY (case_id) REFERENCES app.case_note(id) ON DELETE SET NULL`
- `FK` `appointment_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `appointment_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `appointment_case_idx`
  ```sql
  CREATE INDEX appointment_case_idx ON app.appointment USING btree (case_id) WHERE (case_id IS NOT NULL)
  ```
- `appointment_doctor_time_idx`
  ```sql
  CREATE INDEX appointment_doctor_time_idx ON app.appointment USING btree (doctor_id, starts_at)
  ```
- `appointment_pkey`
  ```sql
  CREATE UNIQUE INDEX appointment_pkey ON app.appointment USING btree (id)
  ```

## `app.case_file`

> Hasta dosya kütüphanesi. `content_enc` = Fernet(`KLIVANCE_FILE_KEY`). Render diski ephemeral → içerik **DB bytea**'da.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.case_file_id_seq'::regclass)` |
| 2 | `case_id` | `bigint` | **hayır** | |
| 3 | `doctor_id` | `bigint` | **hayır** | |
| 4 | `filename` | `text` | **hayır** | |
| 5 | `mime_type` | `text` | **hayır** | |
| 6 | `size_bytes` | `bigint` | **hayır** | |
| 7 | `sha256` | `text` | **hayır** | |
| 8 | `content_enc` | `bytea` | **hayır** | |
| 9 | `extracted_text` | `text` | evet | |
| 10 | `extracted_model` | `text` | evet | |
| 11 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 12 | `deleted_at` | `timestamp with time zone` | evet | |
| 13 | `on_okuma` | `text` | evet | |
| 14 | `on_okuma_model` | `text` | evet | |
| 15 | `on_okuma_durum` | `text` | evet | |
| 16 | `on_okuma_at` | `timestamp with time zone` | evet | |
| 17 | `on_okuma_onay_at` | `timestamp with time zone` | evet | |

**Kısıtlar:**

- `FK` `case_file_case_id_fkey` → `FOREIGN KEY (case_id) REFERENCES app.case_note(id) ON DELETE CASCADE`
- `FK` `case_file_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `case_file_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `case_file_case_idx`
  ```sql
  CREATE INDEX case_file_case_idx ON app.case_file USING btree (case_id) WHERE (deleted_at IS NULL)
  ```
- `case_file_deleted_idx`
  ```sql
  CREATE INDEX case_file_deleted_idx ON app.case_file USING btree (deleted_at) WHERE (deleted_at IS NOT NULL)
  ```
- `case_file_doctor_idx`
  ```sql
  CREATE INDEX case_file_doctor_idx ON app.case_file USING btree (doctor_id)
  ```
- `case_file_pkey`
  ```sql
  CREATE UNIQUE INDEX case_file_pkey ON app.case_file USING btree (id)
  ```

## `app.case_note`

> Hasta kartı. ⚠ 2026-08-05'ten beri **gerçek ad girilebilir** (takma kod isteğe bağlı) → KVKK m.6 özel nitelikli veri.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.case_note_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `code` | `text` | **hayır** | |
| 4 | `age_band` | `text` | evet | |
| 5 | `sex` | `text` | evet | |
| 6 | `narrative` | `text` | evet | |
| 7 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 8 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |
| 9 | `chronic` | `text` | evet | |
| 10 | `meds` | `text` | evet | |
| 11 | `last_visit` | `timestamp with time zone` | evet | |
| 12 | `egfr` | `numeric` | evet | |
| 13 | `kreatinin` | `numeric` | evet | |
| 14 | `kilo_kg` | `numeric` | evet | |
| 15 | `gebelik_haftasi` | `smallint` | evet | |
| 16 | `boy_cm` | `numeric` | evet | |
| 17 | `allergy_status` | `text` | evet | |
| 18 | `allergies` | `text` | evet | |
| 19 | `gebe_olabilir` | `boolean` | evet | |
| 20 | `emziriyor` | `boolean` | evet | |
| 21 | `yas` | `smallint` | evet | |

**Kısıtlar:**

- `UNIQUE` `case_doctor_code_uniq` → `UNIQUE (doctor_id, code)`
- `FK` `case_note_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `CHECK` `case_note_egfr_chk` → `CHECK (((egfr IS NULL) OR ((egfr >= (0)::numeric) AND (egfr <= (250)::numeric))))`
- `CHECK` `case_note_gebelik_chk` → `CHECK (((gebelik_haftasi IS NULL) OR ((gebelik_haftasi >= 0) AND (gebelik_haftasi <= 42))))`
- `PK` `case_note_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `case_doctor_code_uniq`
  ```sql
  CREATE UNIQUE INDEX case_doctor_code_uniq ON app.case_note USING btree (doctor_id, code)
  ```
- `case_doctor_idx`
  ```sql
  CREATE INDEX case_doctor_idx ON app.case_note USING btree (doctor_id)
  ```
- `case_note_doctor_visit_idx`
  ```sql
  CREATE INDEX case_note_doctor_visit_idx ON app.case_note USING btree (doctor_id, last_visit DESC NULLS LAST)
  ```
- `case_note_pkey`
  ```sql
  CREATE UNIQUE INDEX case_note_pkey ON app.case_note USING btree (id)
  ```

## `app.case_timeline`

> Zaman tüneli arşivi — üretilen sentez tarihiyle KALICI, gövde HAM saklanır.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.case_timeline_id_seq'::regclass)` |
| 2 | `case_id` | `bigint` | **hayır** | |
| 3 | `doctor_id` | `bigint` | **hayır** | |
| 4 | `body` | `text` | **hayır** | |
| 5 | `model` | `text` | evet | |
| 6 | `lang` | `text` | **hayır** | `'tr'::text` |
| 7 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `case_timeline_case_id_fkey` → `FOREIGN KEY (case_id) REFERENCES app.case_note(id) ON DELETE CASCADE`
- `FK` `case_timeline_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `case_timeline_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `case_timeline_case_idx`
  ```sql
  CREATE INDEX case_timeline_case_idx ON app.case_timeline USING btree (case_id, created_at DESC)
  ```
- `case_timeline_doctor_idx`
  ```sql
  CREATE INDEX case_timeline_doctor_idx ON app.case_timeline USING btree (doctor_id)
  ```
- `case_timeline_pkey`
  ```sql
  CREATE UNIQUE INDEX case_timeline_pkey ON app.case_timeline USING btree (id)
  ```

## `app.case_visit`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.case_visit_id_seq'::regclass)` |
| 2 | `case_id` | `bigint` | **hayır** | |
| 3 | `doctor_id` | `bigint` | **hayır** | |
| 4 | `note` | `text` | **hayır** | |
| 5 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 6 | `thread_id` | `bigint` | evet | |

**Kısıtlar:**

- `FK` `case_visit_case_id_fkey` → `FOREIGN KEY (case_id) REFERENCES app.case_note(id) ON DELETE CASCADE`
- `FK` `case_visit_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `case_visit_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `case_visit_pkey`
  ```sql
  CREATE UNIQUE INDEX case_visit_pkey ON app.case_visit USING btree (id)
  ```
- `visit_case_idx`
  ```sql
  CREATE INDEX visit_case_idx ON app.case_visit USING btree (case_id, created_at)
  ```

## `app.chat_turn`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `turn` | `text` | **hayır** | |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `durum` | `text` | **hayır** | `'uretiliyor'::text` |
| 4 | `thread_id` | `bigint` | evet | |
| 5 | `message_id` | `bigint` | evet | |
| 6 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 7 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `chat_turn_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `chat_turn_pkey` → `PRIMARY KEY (turn)`

**İndeksler:**

- `chat_turn_created_idx`
  ```sql
  CREATE INDEX chat_turn_created_idx ON app.chat_turn USING btree (created_at)
  ```
- `chat_turn_pkey`
  ```sql
  CREATE UNIQUE INDEX chat_turn_pkey ON app.chat_turn USING btree (turn)
  ```

## `app.cihaz_eslesme`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.cihaz_eslesme_id_seq'::regclass)` |
| 2 | `istek_ozet` | `text` | **hayır** | |
| 3 | `kod` | `text` | evet | |
| 4 | `cihaz_sir_ozet` | `text` | evet | |
| 5 | `cihaz_ad` | `text` | evet | |
| 6 | `doctor_id` | `bigint` | evet | |
| 7 | `durum` | `text` | **hayır** | `'bekliyor'::text` |
| 8 | `istek_ip` | `inet` | evet | |
| 9 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 10 | `onay_at` | `timestamp with time zone` | evet | |
| 11 | `son_kullanim` | `timestamp with time zone` | evet | |
| 12 | `expires_at` | `timestamp with time zone` | **hayır** | |
| 13 | `cihaz_expires_at` | `timestamp with time zone` | evet | |

**Kısıtlar:**

- `FK` `cihaz_eslesme_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `cihaz_eslesme_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `cihaz_doctor_idx`
  ```sql
  CREATE INDEX cihaz_doctor_idx ON app.cihaz_eslesme USING btree (doctor_id)
  ```
- `cihaz_eslesme_pkey`
  ```sql
  CREATE UNIQUE INDEX cihaz_eslesme_pkey ON app.cihaz_eslesme USING btree (id)
  ```
- `cihaz_kod_uq`
  ```sql
  CREATE UNIQUE INDEX cihaz_kod_uq ON app.cihaz_eslesme USING btree (kod) WHERE (kod IS NOT NULL)
  ```
- `cihaz_sir_uq`
  ```sql
  CREATE UNIQUE INDEX cihaz_sir_uq ON app.cihaz_eslesme USING btree (cihaz_sir_ozet) WHERE (cihaz_sir_ozet IS NOT NULL)
  ```

## `app.doctor`

> Hekim hesabı. ⚠ `to_jsonb(d)` ile arşive kopyalanır → **yeni kolon eklersen arşive gitmeli mi diye KARAR VER** (`store.admin_archive_and_delete`).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.doctor_id_seq'::regclass)` |
| 2 | `email` | `text` | **hayır** | |
| 3 | `password_hash` | `text` | **hayır** | |
| 4 | `full_name` | `text` | evet | |
| 5 | `specialty` | `text` | evet | |
| 6 | `diploma_no` | `text` | evet | |
| 7 | `verification_status` | `text` | **hayır** | `'pending'::text` |
| 8 | `plan` | `text` | **hayır** | `'trial'::text` |
| 9 | `monthly_quota` | `integer` | **hayır** | `15` |
| 10 | `role` | `text` | **hayır** | `'doctor'::text` |
| 11 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 12 | `stripe_customer_id` | `text` | evet | |
| 13 | `stripe_subscription_id` | `text` | evet | |
| 14 | `subscription_status` | `text` | evet | |
| 15 | `current_period_end` | `timestamp with time zone` | evet | |
| 16 | `signup_ip` | `text` | evet | |
| 17 | `unvan` | `text` | evet | |
| 18 | `kurum` | `text` | evet | |
| 19 | `phone` | `text` | evet | |
| 20 | `email_verified` | `boolean` | **hayır** | `false` |
| 21 | `verify_token` | `text` | evet | |
| 22 | `verify_sent_at` | `timestamp with time zone` | evet | |
| 23 | `topup_balance` | `integer` | **hayır** | `0` |
| 24 | `deletion_requested_at` | `timestamp with time zone` | evet | |
| 25 | `consents` | `jsonb` | evet | |
| 26 | `billing` | `jsonb` | evet | |
| 27 | `referral_code` | `text` | evet | |
| 28 | `signup_source` | `text` | evet | |
| 29 | `gift_balance` | `integer` | **hayır** | `0` |
| 30 | `gclid` | `text` | evet | |
| 31 | `gclid_at` | `timestamp with time zone` | evet | |
| 32 | `reset_token` | `text` | evet | |
| 33 | `reset_sent_at` | `timestamp with time zone` | evet | |
| 34 | `last_login_at` | `timestamp with time zone` | evet | |
| 35 | `is_test` | `boolean` | **hayır** | `false` |
| 36 | `birth_date` | `date` | evet | |
| 37 | `first_q_mail_at` | `timestamp with time zone` | evet | |
| 38 | `country` | `text` | evet | |
| 39 | `city` | `text` | evet | |
| 40 | `mail_opt_out` | `timestamp with time zone` | evet | |
| 41 | `deneme_gun` | `integer` | **hayır** | `15` |
| 42 | `yenileme_uyari_donem` | `timestamp with time zone` | evet | |
| 43 | `yenileme_bitis_donem` | `timestamp with time zone` | evet | |
| 44 | `iyzico_customer_ref` | `text` | evet | |
| 45 | `iyzico_subscription_ref` | `text` | evet | |

**Kısıtlar:**

- `UNIQUE` `doctor_email_key` → `UNIQUE (email)`
- `PK` `doctor_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `doctor_created_idx`
  ```sql
  CREATE INDEX doctor_created_idx ON app.doctor USING btree (created_at DESC)
  ```
- `doctor_deletion_idx`
  ```sql
  CREATE INDEX doctor_deletion_idx ON app.doctor USING btree (deletion_requested_at) WHERE (deletion_requested_at IS NOT NULL)
  ```
- `doctor_email_key`
  ```sql
  CREATE UNIQUE INDEX doctor_email_key ON app.doctor USING btree (email)
  ```
- `doctor_iyz_sub_idx`
  ```sql
  CREATE INDEX doctor_iyz_sub_idx ON app.doctor USING btree (iyzico_subscription_ref) WHERE (iyzico_subscription_ref IS NOT NULL)
  ```
- `doctor_last_login_idx`
  ```sql
  CREATE INDEX doctor_last_login_idx ON app.doctor USING btree (last_login_at DESC NULLS LAST)
  ```
- `doctor_mail_opt_out_idx`
  ```sql
  CREATE INDEX doctor_mail_opt_out_idx ON app.doctor USING btree (mail_opt_out) WHERE (mail_opt_out IS NOT NULL)
  ```
- `doctor_period_end_idx`
  ```sql
  CREATE INDEX doctor_period_end_idx ON app.doctor USING btree (current_period_end) WHERE (current_period_end IS NOT NULL)
  ```
- `doctor_pkey`
  ```sql
  CREATE UNIQUE INDEX doctor_pkey ON app.doctor USING btree (id)
  ```
- `doctor_referral_code_uq`
  ```sql
  CREATE UNIQUE INDEX doctor_referral_code_uq ON app.doctor USING btree (referral_code) WHERE (referral_code IS NOT NULL)
  ```
- `doctor_reset_token_idx`
  ```sql
  CREATE INDEX doctor_reset_token_idx ON app.doctor USING btree (reset_token) WHERE (reset_token IS NOT NULL)
  ```
- `doctor_signup_ip_idx`
  ```sql
  CREATE INDEX doctor_signup_ip_idx ON app.doctor USING btree (signup_ip, created_at)
  ```
- `doctor_verify_token_idx`
  ```sql
  CREATE INDEX doctor_verify_token_idx ON app.doctor USING btree (verify_token) WHERE (verify_token IS NOT NULL)
  ```

## `app.doctor_archive`

> Admin silmelerinin kalıcı snapshot'ı. ⚠ Hekimin KENDİ KVKK talebi (`purge_due_accounts`) buraya kopya ALMAZ.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.doctor_archive_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `email` | `text` | **hayır** | |
| 4 | `full_name` | `text` | evet | |
| 5 | `reason` | `text` | evet | |
| 6 | `snapshot` | `jsonb` | **hayır** | |
| 7 | `deleted_by` | `bigint` | evet | |
| 8 | `deleted_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `doctor_archive_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `doctor_archive_email_idx`
  ```sql
  CREATE INDEX doctor_archive_email_idx ON app.doctor_archive USING btree (email)
  ```
- `doctor_archive_pkey`
  ```sql
  CREATE UNIQUE INDEX doctor_archive_pkey ON app.doctor_archive USING btree (id)
  ```

## `app.doctor_event`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.doctor_event_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `yuzey` | `text` | **hayır** | |
| 4 | `detay` | `text` | evet | |
| 5 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `doctor_event_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `doctor_event_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `doctor_event_created_idx`
  ```sql
  CREATE INDEX doctor_event_created_idx ON app.doctor_event USING btree (created_at)
  ```
- `doctor_event_doctor_idx`
  ```sql
  CREATE INDEX doctor_event_doctor_idx ON app.doctor_event USING btree (doctor_id, created_at DESC)
  ```
- `doctor_event_pkey`
  ```sql
  CREATE UNIQUE INDEX doctor_event_pkey ON app.doctor_event USING btree (id)
  ```

## `app.evidence_doc`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.evidence_doc_id_seq'::regclass)` |
| 2 | `message_id` | `bigint` | evet | |
| 3 | `sira` | `smallint` | **hayır** | |
| 4 | `source` | `text` | **hayır** | |
| 5 | `doc_id` | `text` | **hayır** | |
| 6 | `baslik` | `text` | evet | |
| 7 | `skor` | `real` | evet | |
| 8 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `evidence_doc_message_id_fkey` → `FOREIGN KEY (message_id) REFERENCES app.message(id) ON DELETE CASCADE`
- `PK` `evidence_doc_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `evidence_doc_doc_idx`
  ```sql
  CREATE INDEX evidence_doc_doc_idx ON app.evidence_doc USING btree (source, doc_id)
  ```
- `evidence_doc_msg_idx`
  ```sql
  CREATE INDEX evidence_doc_msg_idx ON app.evidence_doc USING btree (message_id)
  ```
- `evidence_doc_pkey`
  ```sql
  CREATE UNIQUE INDEX evidence_doc_pkey ON app.evidence_doc USING btree (id)
  ```

## `app.feedback`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.feedback_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `category` | `text` | evet | |
| 4 | `message` | `text` | **hayır** | |
| 5 | `page` | `text` | evet | |
| 6 | `status` | `text` | **hayır** | `'new'::text` |
| 7 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `feedback_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `feedback_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `feedback_created_idx`
  ```sql
  CREATE INDEX feedback_created_idx ON app.feedback USING btree (created_at DESC)
  ```
- `feedback_doctor_idx`
  ```sql
  CREATE INDEX feedback_doctor_idx ON app.feedback USING btree (doctor_id)
  ```
- `feedback_pkey`
  ```sql
  CREATE UNIQUE INDEX feedback_pkey ON app.feedback USING btree (id)
  ```

## `app.iyzico_payment`

> Ödeme defteri — `/admin/odeme` panelinin TEK yetkili kaynağı. `granted` alanı `purchase` olayını üretir.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `token` | `text` | **hayır** | |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `kind` | `text` | **hayır** | |
| 4 | `plan` | `text` | evet | |
| 5 | `interval` | `text` | evet | |
| 6 | `credits` | `integer` | evet | |
| 7 | `amount` | `numeric` | evet | |
| 8 | `payment_id` | `text` | evet | |
| 9 | `granted` | `boolean` | **hayır** | `false` |
| 10 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 11 | `fail_info` | `text` | evet | |
| 12 | `subscription_ref` | `text` | evet | |
| 13 | `order_ref` | `text` | evet | |
| 14 | `period_end` | `timestamp with time zone` | evet | |
| 15 | `iyz_status` | `text` | evet | |

**Kısıtlar:**

- `FK` `iyzico_payment_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `iyzico_payment_pkey` → `PRIMARY KEY (token)`

**İndeksler:**

- `iyzico_payment_created_idx`
  ```sql
  CREATE INDEX iyzico_payment_created_idx ON app.iyzico_payment USING btree (created_at DESC)
  ```
- `iyzico_payment_doctor_idx`
  ```sql
  CREATE INDEX iyzico_payment_doctor_idx ON app.iyzico_payment USING btree (doctor_id)
  ```
- `iyzico_payment_order_idx`
  ```sql
  CREATE INDEX iyzico_payment_order_idx ON app.iyzico_payment USING btree (order_ref) WHERE (order_ref IS NOT NULL)
  ```
- `iyzico_payment_pending_idx`
  ```sql
  CREATE INDEX iyzico_payment_pending_idx ON app.iyzico_payment USING btree (created_at DESC) WHERE (NOT granted)
  ```
- `iyzico_payment_pkey`
  ```sql
  CREATE UNIQUE INDEX iyzico_payment_pkey ON app.iyzico_payment USING btree (token)
  ```
- `iyzico_payment_sub_idx`
  ```sql
  CREATE INDEX iyzico_payment_sub_idx ON app.iyzico_payment USING btree (subscription_ref) WHERE (subscription_ref IS NOT NULL)
  ```

## `app.login_event`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.login_event_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `login_at` | `timestamp with time zone` | **hayır** | `now()` |
| 4 | `last_seen` | `timestamp with time zone` | **hayır** | `now()` |
| 5 | `logout_at` | `timestamp with time zone` | evet | |

**Kısıtlar:**

- `FK` `login_event_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `login_event_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `login_event_doctor_idx`
  ```sql
  CREATE INDEX login_event_doctor_idx ON app.login_event USING btree (doctor_id, login_at DESC)
  ```
- `login_event_pkey`
  ```sql
  CREATE UNIQUE INDEX login_event_pkey ON app.login_event USING btree (id)
  ```

## `app.message`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.message_id_seq'::regclass)` |
| 2 | `thread_id` | `bigint` | **hayır** | |
| 3 | `role` | `text` | **hayır** | |
| 4 | `content` | `text` | **hayır** | |
| 5 | `sources` | `jsonb` | evet | |
| 6 | `mode` | `text` | evet | |
| 7 | `cost_usd` | `numeric` | evet | |
| 8 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 9 | `notice` | `text` | evet | |

**Kısıtlar:**

- `PK` `message_pkey` → `PRIMARY KEY (id)`
- `FK` `message_thread_id_fkey` → `FOREIGN KEY (thread_id) REFERENCES app.thread(id) ON DELETE CASCADE`

**İndeksler:**

- `message_pkey`
  ```sql
  CREATE UNIQUE INDEX message_pkey ON app.message USING btree (id)
  ```
- `message_role_created_idx`
  ```sql
  CREATE INDEX message_role_created_idx ON app.message USING btree (role, created_at)
  ```
- `message_thread_idx`
  ```sql
  CREATE INDEX message_thread_idx ON app.message USING btree (thread_id)
  ```
- `message_thread_role_idx`
  ```sql
  CREATE INDEX message_thread_role_idx ON app.message USING btree (thread_id, role, id)
  ```

## `app.query_stat`

> **ANONİM** soru istatistiği: `doctor_id` YOK, ham soru metni YOK. ⚠ Biri eklenirse KVKK ifşası ZORUNLU olur (aynı commit'te).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.query_stat_id_seq'::regclass)` |
| 2 | `created` | `date` | **hayır** | `CURRENT_DATE` |
| 3 | `lang` | `text` | evet | |
| 4 | `mode` | `text` | evet | |
| 5 | `sourced` | `boolean` | evet | |
| 6 | `refused` | `boolean` | evet | |
| 7 | `source_kinds` | `ARRAY` | evet | |
| 8 | `topic` | `text` | evet | |

**Kısıtlar:**

- `PK` `query_stat_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `query_stat_created_idx`
  ```sql
  CREATE INDEX query_stat_created_idx ON app.query_stat USING btree (created)
  ```
- `query_stat_pkey`
  ```sql
  CREATE UNIQUE INDEX query_stat_pkey ON app.query_stat USING btree (id)
  ```
- `query_stat_topic_idx`
  ```sql
  CREATE INDEX query_stat_topic_idx ON app.query_stat USING btree (topic)
  ```

## `app.question_insight`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.question_insight_id_seq'::regclass)` |
| 2 | `message_id` | `bigint` | **hayır** | |
| 3 | `soruldu_at` | `timestamp with time zone` | evet | |
| 4 | `brans` | `text` | evet | |
| 5 | `unvan` | `text` | evet | |
| 6 | `sehir` | `text` | evet | |
| 7 | `konu` | `ARRAY` | **hayır** | `'{}'::text[]` |
| 8 | `icd_bolum` | `ARRAY` | **hayır** | `'{}'::text[]` |
| 9 | `soru_tipi` | `ARRAY` | **hayır** | `'{}'::text[]` |
| 10 | `ilaclar` | `ARRAY` | **hayır** | `'{}'::text[]` |
| 11 | `kanit_bos` | `boolean` | evet | |
| 12 | `kanit_n` | `smallint` | evet | |
| 13 | `kanit_turleri` | `ARRAY` | evet | |
| 14 | `atif_var` | `boolean` | evet | |
| 15 | `red` | `boolean` | evet | |
| 16 | `iskelet_tam` | `boolean` | evet | |
| 17 | `hasta_baglami` | `boolean` | evet | |
| 18 | `ek_dosya` | `boolean` | evet | |
| 19 | `dil` | `text` | evet | |
| 20 | `uzunluk` | `integer` | evet | |
| 21 | `tur_sirasi` | `text` | evet | |
| 22 | `mod` | `text` | evet | |
| 23 | `yontem` | `text` | **hayır** | `'kural'::text` |
| 24 | `uretim_surumu` | `smallint` | **hayır** | `1` |
| 25 | `uretildi_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `CHECK` `question_insight_kanit_ck` → `CHECK (((kanit_bos IS NULL) OR (kanit_n IS NOT NULL)))`
- `FK` `question_insight_message_id_fkey` → `FOREIGN KEY (message_id) REFERENCES app.message(id) ON DELETE CASCADE`
- `UNIQUE` `question_insight_message_id_key` → `UNIQUE (message_id)`
- `PK` `question_insight_pkey` → `PRIMARY KEY (id)`
- `CHECK` `question_insight_tur_ck` → `CHECK (((tur_sirasi IS NULL) OR (tur_sirasi = ANY (ARRAY['ilk'::text, 'takip'::text]))))`
- `CHECK` `question_insight_yontem_ck` → `CHECK ((yontem = ANY (ARRAY['kural'::text, 'llm'::text, 'kural+llm'::text])))`

**İndeksler:**

- `question_insight_brans_idx`
  ```sql
  CREATE INDEX question_insight_brans_idx ON app.question_insight USING btree (brans)
  ```
- `question_insight_konu_idx`
  ```sql
  CREATE INDEX question_insight_konu_idx ON app.question_insight USING gin (konu)
  ```
- `question_insight_message_id_key`
  ```sql
  CREATE UNIQUE INDEX question_insight_message_id_key ON app.question_insight USING btree (message_id)
  ```
- `question_insight_pkey`
  ```sql
  CREATE UNIQUE INDEX question_insight_pkey ON app.question_insight USING btree (id)
  ```
- `question_insight_zaman_idx`
  ```sql
  CREATE INDEX question_insight_zaman_idx ON app.question_insight USING btree (soruldu_at)
  ```

## `app.question_kept`

> `asked_at` = `date` (timestamp DEĞİL — saniye hassasiyeti küçük tabanda yeniden-kimliklendirme vektörü). Kimlik bağı YOK.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.question_kept_id_seq'::regclass)` |
| 2 | `asked_at` | `date` | **hayır** | |
| 3 | `lang` | `text` | evet | |
| 4 | `mode` | `text` | evet | |
| 5 | `question` | `text` | **hayır** | |
| 6 | `answer` | `text` | evet | |
| 7 | `sourced` | `boolean` | **hayır** | `false` |
| 8 | `source_kinds` | `ARRAY` | evet | |
| 9 | `kept_at` | `timestamp with time zone` | **hayır** | `now()` |
| 10 | `kept_reason` | `text` | evet | |

**Kısıtlar:**

- `PK` `question_kept_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `question_kept_asked_idx`
  ```sql
  CREATE INDEX question_kept_asked_idx ON app.question_kept USING btree (asked_at DESC)
  ```
- `question_kept_pkey`
  ```sql
  CREATE UNIQUE INDEX question_kept_pkey ON app.question_kept USING btree (id)
  ```

## `app.referral`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.referral_id_seq'::regclass)` |
| 2 | `referrer_id` | `bigint` | **hayır** | |
| 3 | `referred_id` | `bigint` | **hayır** | |
| 4 | `status` | `text` | **hayır** | `'pending'::text` |
| 5 | `referrer_reward` | `integer` | **hayır** | `0` |
| 6 | `referred_reward` | `integer` | **hayır** | `0` |
| 7 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 8 | `rewarded_at` | `timestamp with time zone` | evet | |

**Kısıtlar:**

- `PK` `referral_pkey` → `PRIMARY KEY (id)`
- `FK` `referral_referred_id_fkey` → `FOREIGN KEY (referred_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `UNIQUE` `referral_referred_id_key` → `UNIQUE (referred_id)`
- `FK` `referral_referrer_id_fkey` → `FOREIGN KEY (referrer_id) REFERENCES app.doctor(id) ON DELETE CASCADE`

**İndeksler:**

- `referral_pkey`
  ```sql
  CREATE UNIQUE INDEX referral_pkey ON app.referral USING btree (id)
  ```
- `referral_referred_id_key`
  ```sql
  CREATE UNIQUE INDEX referral_referred_id_key ON app.referral USING btree (referred_id)
  ```
- `referral_referrer_idx`
  ```sql
  CREATE INDEX referral_referrer_idx ON app.referral USING btree (referrer_id)
  ```

## `app.referral_invite`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.referral_invite_id_seq'::regclass)` |
| 2 | `referrer_id` | `bigint` | **hayır** | |
| 3 | `email` | `text` | **hayır** | |
| 4 | `joined` | `boolean` | **hayır** | `false` |
| 5 | `sent_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `referral_invite_pkey` → `PRIMARY KEY (id)`
- `UNIQUE` `referral_invite_referrer_id_email_key` → `UNIQUE (referrer_id, email)`
- `FK` `referral_invite_referrer_id_fkey` → `FOREIGN KEY (referrer_id) REFERENCES app.doctor(id) ON DELETE CASCADE`

**İndeksler:**

- `referral_invite_email_idx`
  ```sql
  CREATE INDEX referral_invite_email_idx ON app.referral_invite USING btree (lower(email))
  ```
- `referral_invite_pkey`
  ```sql
  CREATE UNIQUE INDEX referral_invite_pkey ON app.referral_invite USING btree (id)
  ```
- `referral_invite_referrer_id_email_key`
  ```sql
  CREATE UNIQUE INDEX referral_invite_referrer_id_email_key ON app.referral_invite USING btree (referrer_id, email)
  ```
- `referral_invite_referrer_idx`
  ```sql
  CREATE INDEX referral_invite_referrer_idx ON app.referral_invite USING btree (referrer_id)
  ```

## `app.reservation_settled`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `rid` | `bigint` | **hayır** | |
| 2 | `doctor_id` | `bigint` | evet | |
| 3 | `kind` | `text` | **hayır** | |
| 4 | `topup` | `integer` | **hayır** | `0` |
| 5 | `gift` | `integer` | **hayır** | `0` |
| 6 | `ts` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `reservation_settled_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `reservation_settled_pkey` → `PRIMARY KEY (rid)`

**İndeksler:**

- `reservation_settled_pkey`
  ```sql
  CREATE UNIQUE INDEX reservation_settled_pkey ON app.reservation_settled USING btree (rid)
  ```
- `reservation_settled_ts_idx`
  ```sql
  CREATE INDEX reservation_settled_ts_idx ON app.reservation_settled USING btree (ts)
  ```

## `app.schema_migration`

> Deyim-bazlı migrasyon takibi — kararlı durumda migrate SIFIR DDL çalıştırır (deploy kilidi freni).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `file` | `text` | **hayır** | |
| 2 | `stmt_hash` | `text` | **hayır** | |
| 3 | `ord` | `integer` | evet | |
| 4 | `snippet` | `text` | evet | |
| 5 | `applied_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `schema_migration_pkey` → `PRIMARY KEY (file, stmt_hash)`

**İndeksler:**

- `schema_migration_pkey`
  ```sql
  CREATE UNIQUE INDEX schema_migration_pkey ON app.schema_migration USING btree (file, stmt_hash)
  ```

## `app.session`

> Sunucu-taraflı oturum. Çerez (`klv_session`) yalnız rastgele token taşır; kimlik burada.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `token` | `text` | **hayır** | |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 4 | `expires_at` | `timestamp with time zone` | **hayır** | |
| 5 | `last_seen` | `timestamp with time zone` | **hayır** | `now()` |
| 6 | `login_event_id` | `bigint` | evet | |
| 7 | `cihaz_id` | `bigint` | evet | |

**Kısıtlar:**

- `FK` `session_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `session_pkey` → `PRIMARY KEY (token)`

**İndeksler:**

- `session_doctor_idx`
  ```sql
  CREATE INDEX session_doctor_idx ON app.session USING btree (doctor_id)
  ```
- `session_pkey`
  ```sql
  CREATE UNIQUE INDEX session_pkey ON app.session USING btree (token)
  ```

## `app.setting`

> Anahtar-değer. ⚠ `trial_salt` burada: **TEK NOKTA ARIZASI** — kaybolursa deneme freni düşer.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `key` | `text` | **hayır** | |
| 2 | `value` | `text` | **hayır** | |
| 3 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `setting_pkey` → `PRIMARY KEY (key)`

**İndeksler:**

- `setting_pkey`
  ```sql
  CREATE UNIQUE INDEX setting_pkey ON app.setting USING btree (key)
  ```

## `app.thread`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.thread_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `title` | `text` | evet | |
| 4 | `case_id` | `bigint` | evet | |
| 5 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |
| 6 | `title_auto` | `boolean` | evet | |

**Kısıtlar:**

- `FK` `thread_case_fk` → `FOREIGN KEY (case_id) REFERENCES app.case_note(id) ON DELETE SET NULL`
- `FK` `thread_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `thread_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `thread_doctor_created_idx`
  ```sql
  CREATE INDEX thread_doctor_created_idx ON app.thread USING btree (doctor_id, created_at DESC)
  ```
- `thread_doctor_idx`
  ```sql
  CREATE INDEX thread_doctor_idx ON app.thread USING btree (doctor_id)
  ```
- `thread_pkey`
  ```sql
  CREATE UNIQUE INDEX thread_pkey ON app.thread USING btree (id)
  ```

## `app.topup_grant`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `session_id` | `text` | **hayır** | |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `credits` | `integer` | **hayır** | |
| 4 | `ts` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `topup_grant_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `topup_grant_pkey` → `PRIMARY KEY (session_id)`

**İndeksler:**

- `topup_grant_pkey`
  ```sql
  CREATE UNIQUE INDEX topup_grant_pkey ON app.topup_grant USING btree (session_id)
  ```

## `app.tr_cache`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `src_hash` | `text` | **hayır** | |
| 2 | `lang` | `text` | **hayır** | |
| 3 | `translated` | `text` | **hayır** | |
| 4 | `model` | `text` | evet | |
| 5 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `tr_cache_pkey` → `PRIMARY KEY (src_hash, lang)`

**İndeksler:**

- `tr_cache_pkey`
  ```sql
  CREATE UNIQUE INDEX tr_cache_pkey ON app.tr_cache USING btree (src_hash, lang)
  ```

## `app.trial_used`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `email_hash` | `text` | **hayır** | |
| 2 | `claimed_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `trial_used_pkey` → `PRIMARY KEY (email_hash)`

**İndeksler:**

- `trial_used_pkey`
  ```sql
  CREATE UNIQUE INDEX trial_used_pkey ON app.trial_used USING btree (email_hash)
  ```

## `app.usage`

> Maliyetin **ve** günlük frenin TEK kaynağı. ⚠ Akış iptalinde de yazılır — yazılmazsa kaçak ÖLÇÜLEMEZ.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('app.usage_id_seq'::regclass)` |
| 2 | `doctor_id` | `bigint` | **hayır** | |
| 3 | `ts` | `timestamp with time zone` | **hayır** | `now()` |
| 4 | `mode` | `text` | evet | |
| 5 | `input_tokens` | `integer` | evet | `0` |
| 6 | `output_tokens` | `integer` | evet | `0` |
| 7 | `cache_read` | `integer` | evet | `0` |
| 8 | `cache_write` | `integer` | evet | `0` |
| 9 | `usd` | `numeric` | evet | `0` |
| 10 | `credits` | `smallint` | **hayır** | `1` |
| 11 | `topup` | `smallint` | **hayır** | `0` |
| 12 | `gift` | `smallint` | **hayır** | `0` |

**Kısıtlar:**

- `FK` `usage_doctor_id_fkey` → `FOREIGN KEY (doctor_id) REFERENCES app.doctor(id) ON DELETE CASCADE`
- `PK` `usage_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `usage_doctor_ts_idx`
  ```sql
  CREATE INDEX usage_doctor_ts_idx ON app.usage USING btree (doctor_id, ts)
  ```
- `usage_pkey`
  ```sql
  CREATE UNIQUE INDEX usage_pkey ON app.usage USING btree (id)
  ```
- `usage_ts_idx`
  ```sql
  CREATE INDEX usage_ts_idx ON app.usage USING btree (ts)
  ```

## `app.wall_stat`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `gun` | `date` | **hayır** | |
| 2 | `olay` | `text` | **hayır** | |
| 3 | `yuzey` | `text` | **hayır** | |
| 4 | `n` | `bigint` | **hayır** | `0` |

**Kısıtlar:**

- `PK` `wall_stat_pkey` → `PRIMARY KEY (gun, olay, yuzey)`

**İndeksler:**

- `wall_stat_pkey`
  ```sql
  CREATE UNIQUE INDEX wall_stat_pkey ON app.wall_stat USING btree (gun, olay, yuzey)
  ```

# Şema `core`

Bilgi tabanı (KB) — ilaç, hastalık, korpus, yan etki, kodlar. **Kişisel veri YOKTUR.**

## `core.adverse_event`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `generic_name` | `text` | **hayır** | |
| 2 | `reaction` | `text` | **hayır** | |
| 3 | `report_count` | `integer` | evet | |

**Kısıtlar:**

- `PK` `adverse_event_pkey` → `PRIMARY KEY (generic_name, reaction)`

**İndeksler:**

- `adverse_event_generic_trgm`
  ```sql
  CREATE INDEX adverse_event_generic_trgm ON core.adverse_event USING gin (generic_name gin_trgm_ops)
  ```
- `adverse_event_pkey`
  ```sql
  CREATE UNIQUE INDEX adverse_event_pkey ON core.adverse_event USING btree (generic_name, reaction)
  ```
- `idx_ae_generic`
  ```sql
  CREATE INDEX idx_ae_generic ON core.adverse_event USING btree (generic_name)
  ```

## `core.code_map`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `entity_type` | `text` | **hayır** | |
| 3 | `entity_id` | `bigint` | **hayır** | |
| 4 | `system` | `text` | **hayır** | |
| 5 | `code` | `text` | **hayır** | |

**Kısıtlar:**

- `UNIQUE` `code_map_entity_type_entity_id_system_code_key` → `UNIQUE (entity_type, entity_id, system, code)`
- `PK` `code_map_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `code_map_entity_type_entity_id_system_code_key`
  ```sql
  CREATE UNIQUE INDEX code_map_entity_type_entity_id_system_code_key ON core.code_map USING btree (entity_type, entity_id, system, code)
  ```
- `code_map_pkey`
  ```sql
  CREATE UNIQUE INDEX code_map_pkey ON core.code_map USING btree (id)
  ```
- `idx_codemap_lookup`
  ```sql
  CREATE INDEX idx_codemap_lookup ON core.code_map USING btree (system, code)
  ```

## `core.corpus`

> Klinik literatür korpusu + `tsv` GIN indeksi (FTS). ⚠ `license` alanı kaynağın KENDİ sayfasından BİREBİR yazılır.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `source` | `text` | **hayır** | |
| 3 | `doc_id` | `text` | **hayır** | |
| 4 | `title` | `text` | evet | |
| 5 | `body` | `text` | evet | |
| 6 | `license` | `text` | evet | |
| 7 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |
| 8 | `tsv` | `tsvector` | evet | |
| 9 | `retraction_status` | `text` | evet | |
| 10 | `retraction_checked_at` | `timestamp with time zone` | evet | |
| 11 | `retraction_source` | `text` | evet | |

**Kısıtlar:**

- `PK` `corpus_pkey` → `PRIMARY KEY (id)`
- `CHECK` `corpus_retraction_status_chk` → `CHECK (((retraction_status IS NULL) OR (retraction_status = ANY (ARRAY['retracted'::text, 'unknown'::text, 'not_applicable'::text]))))`
- `UNIQUE` `corpus_source_doc_id_key` → `UNIQUE (source, doc_id)`

**İndeksler:**

- `corpus_pkey`
  ```sql
  CREATE UNIQUE INDEX corpus_pkey ON core.corpus USING btree (id)
  ```
- `corpus_source_doc_id_key`
  ```sql
  CREATE UNIQUE INDEX corpus_source_doc_id_key ON core.corpus USING btree (source, doc_id)
  ```
- `corpus_tsv_gin`
  ```sql
  CREATE INDEX corpus_tsv_gin ON core.corpus USING gin (tsv)
  ```
- `idx_corpus_retracted`
  ```sql
  CREATE INDEX idx_corpus_retracted ON core.corpus USING btree (retraction_status) WHERE (retraction_status = 'retracted'::text)
  ```
- `idx_corpus_title_fts`
  ```sql
  CREATE INDEX idx_corpus_title_fts ON core.corpus USING gin (to_tsvector('english'::regconfig, COALESCE(title, ''::text)))
  ```

## `core.corpus_disease`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `corpus_id` | `bigint` | **hayır** | |
| 2 | `disease_id` | `bigint` | **hayır** | |
| 3 | `method` | `text` | **hayır** | |

**Kısıtlar:**

- `FK` `corpus_disease_corpus_id_fkey` → `FOREIGN KEY (corpus_id) REFERENCES core.corpus(id) ON DELETE CASCADE`
- `FK` `corpus_disease_disease_id_fkey` → `FOREIGN KEY (disease_id) REFERENCES core.disease(id) ON DELETE CASCADE`
- `PK` `corpus_disease_pkey` → `PRIMARY KEY (corpus_id, disease_id, method)`

**İndeksler:**

- `corpus_disease_pkey`
  ```sql
  CREATE UNIQUE INDEX corpus_disease_pkey ON core.corpus_disease USING btree (corpus_id, disease_id, method)
  ```
- `idx_cd_disease`
  ```sql
  CREATE INDEX idx_cd_disease ON core.corpus_disease USING btree (disease_id)
  ```

## `core.corpus_pilot_yedek`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `corpus_id` | `bigint` | **hayır** | |
| 2 | `doc_id` | `text` | **hayır** | |
| 3 | `body` | `text` | evet | |
| 4 | `license` | `text` | evet | |
| 5 | `alindi_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `corpus_pilot_yedek_pkey` → `PRIMARY KEY (corpus_id)`

**İndeksler:**

- `corpus_pilot_yedek_pkey`
  ```sql
  CREATE UNIQUE INDEX corpus_pilot_yedek_pkey ON core.corpus_pilot_yedek USING btree (corpus_id)
  ```

## `core.disease`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `icd11_code` | `text` | evet | |
| 3 | `title` | `text` | **hayır** | |
| 4 | `definition` | `text` | evet | |
| 5 | `parent_uri` | `text` | evet | |
| 6 | `uri` | `text` | evet | |
| 7 | `source_id` | `text` | **hayır** | |
| 8 | `source_key` | `text` | **hayır** | |
| 9 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |
| 10 | `title_tr` | `text` | evet | |
| 11 | `class_kind` | `text` | evet | |
| 12 | `parent_id` | `bigint` | evet | |
| 13 | `synonyms` | `ARRAY` | evet | |

**Kısıtlar:**

- `PK` `disease_pkey` → `PRIMARY KEY (id)`
- `UNIQUE` `disease_source_id_source_key_key` → `UNIQUE (source_id, source_key)`
- `UNIQUE` `disease_uri_key` → `UNIQUE (uri)`

**İndeksler:**

- `disease_pkey`
  ```sql
  CREATE UNIQUE INDEX disease_pkey ON core.disease USING btree (id)
  ```
- `disease_source_id_source_key_key`
  ```sql
  CREATE UNIQUE INDEX disease_source_id_source_key_key ON core.disease USING btree (source_id, source_key)
  ```
- `disease_uri_key`
  ```sql
  CREATE UNIQUE INDEX disease_uri_key ON core.disease USING btree (uri)
  ```
- `idx_disease_code`
  ```sql
  CREATE INDEX idx_disease_code ON core.disease USING btree (icd11_code)
  ```
- `idx_disease_parent`
  ```sql
  CREATE INDEX idx_disease_parent ON core.disease USING btree (parent_id)
  ```
- `idx_disease_title`
  ```sql
  CREATE INDEX idx_disease_title ON core.disease USING btree (lower(title))
  ```

## `core.drug`

> İlaç kartı verisi. `find_drugs` sıralaması = **hasta güvenliği** (yanlış sıralama yanlış ilaç kartı gösterir).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `generic_name` | `text` | evet | |
| 3 | `brand_names` | `ARRAY` | evet | |
| 4 | `manufacturer` | `text` | evet | |
| 5 | `indications` | `text` | evet | |
| 6 | `warnings` | `text` | evet | |
| 7 | `source_id` | `text` | **hayır** | |
| 8 | `source_key` | `text` | **hayır** | |
| 9 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |
| 10 | `interactions` | `text` | evet | |
| 11 | `contraindications` | `text` | evet | |
| 12 | `dosage` | `text` | evet | |
| 13 | `boxed_warning` | `text` | evet | |
| 14 | `adverse_reactions` | `text` | evet | |
| 15 | `label_date` | `date` | evet | |
| 16 | `label_date_kind` | `text` | evet | |
| 17 | `label_version` | `integer` | evet | |

**Kısıtlar:**

- `CHECK` `drug_label_date_kind_ck` → `CHECK ((((label_date IS NULL) = (label_date_kind IS NULL)) AND ((label_date_kind IS NULL) OR (label_date_kind = ANY (ARRAY['yururluk'::text, 'yayim'::text, 'onay'::text]))))) NOT VALID`
- `PK` `drug_pkey` → `PRIMARY KEY (id)`
- `UNIQUE` `drug_source_id_source_key_key` → `UNIQUE (source_id, source_key)`

**İndeksler:**

- `drug_brands_trgm`
  ```sql
  CREATE INDEX drug_brands_trgm ON core.drug USING gin (core.brands_text(brand_names) gin_trgm_ops)
  ```
- `drug_brands_trgm_ccnew`
  ```sql
  CREATE INDEX drug_brands_trgm_ccnew ON core.drug USING gin (core.brands_text(brand_names) gin_trgm_ops)
  ```
- `drug_generic_trgm`
  ```sql
  CREATE INDEX drug_generic_trgm ON core.drug USING gin (generic_name gin_trgm_ops)
  ```
- `drug_pkey`
  ```sql
  CREATE UNIQUE INDEX drug_pkey ON core.drug USING btree (id)
  ```
- `drug_source_id_source_key_key`
  ```sql
  CREATE UNIQUE INDEX drug_source_id_source_key_key ON core.drug USING btree (source_id, source_key)
  ```
- `idx_drug_generic`
  ```sql
  CREATE INDEX idx_drug_generic ON core.drug USING btree (lower(generic_name))
  ```
- `idx_drug_ind_fts`
  ```sql
  CREATE INDEX idx_drug_ind_fts ON core.drug USING gin (to_tsvector('english'::regconfig, COALESCE(indications, ''::text)))
  ```

## `core.drug_class_cache`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `query_norm` | `text` | **hayır** | |
| 2 | `atc` | `ARRAY` | evet | |
| 3 | `moa` | `ARRAY` | evet | |
| 4 | `rxcui` | `text` | evet | |
| 5 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `drug_class_cache_pkey` → `PRIMARY KEY (query_norm)`

**İndeksler:**

- `drug_class_cache_pkey`
  ```sql
  CREATE UNIQUE INDEX drug_class_cache_pkey ON core.drug_class_cache USING btree (query_norm)
  ```

## `core.drug_disease`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `drug_id` | `bigint` | **hayır** | |
| 2 | `disease_id` | `bigint` | **hayır** | |
| 3 | `method` | `text` | **hayır** | |
| 4 | `confidence` | `real` | evet | |
| 5 | `created_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `FK` `drug_disease_disease_id_fkey` → `FOREIGN KEY (disease_id) REFERENCES core.disease(id) ON DELETE CASCADE`
- `FK` `drug_disease_drug_id_fkey` → `FOREIGN KEY (drug_id) REFERENCES core.drug(id) ON DELETE CASCADE`
- `PK` `drug_disease_pkey` → `PRIMARY KEY (drug_id, disease_id, method)`

**İndeksler:**

- `drug_disease_pkey`
  ```sql
  CREATE UNIQUE INDEX drug_disease_pkey ON core.drug_disease USING btree (drug_id, disease_id, method)
  ```
- `idx_dd_disease`
  ```sql
  CREATE INDEX idx_dd_disease ON core.drug_disease USING btree (disease_id)
  ```

## `core.drug_recall`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `recall_number` | `text` | **hayır** | |
| 2 | `generic_name` | `text` | evet | |
| 3 | `brand_name` | `text` | evet | |
| 4 | `product_desc` | `text` | evet | |
| 5 | `reason` | `text` | evet | |
| 6 | `classification` | `text` | evet | |
| 7 | `status` | `text` | evet | |
| 8 | `recalling_firm` | `text` | evet | |
| 9 | `initiated_date` | `text` | evet | |
| 10 | `source_id` | `text` | **hayır** | `'enforcement'::text` |
| 11 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `drug_recall_pkey` → `PRIMARY KEY (recall_number)`

**İndeksler:**

- `drug_recall_pkey`
  ```sql
  CREATE UNIQUE INDEX drug_recall_pkey ON core.drug_recall USING btree (recall_number)
  ```
- `idx_recall_generic`
  ```sql
  CREATE INDEX idx_recall_generic ON core.drug_recall USING btree (lower(generic_name))
  ```

## `core.faers_fetch`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `generic_name` | `text` | **hayır** | |
| 2 | `status` | `text` | **hayır** | |
| 3 | `n_reactions` | `integer` | **hayır** | `0` |
| 4 | `fetched_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `faers_fetch_pkey` → `PRIMARY KEY (generic_name)`

**İndeksler:**

- `faers_fetch_pkey`
  ```sql
  CREATE UNIQUE INDEX faers_fetch_pkey ON core.faers_fetch USING btree (generic_name)
  ```

## `core.ingest_log`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `source_id` | `text` | **hayır** | |
| 3 | `started_at` | `timestamp with time zone` | **hayır** | `now()` |
| 4 | `finished_at` | `timestamp with time zone` | evet | |
| 5 | `status` | `text` | evet | |
| 6 | `records` | `integer` | **hayır** | `0` |
| 7 | `cursor_before` | `text` | evet | |
| 8 | `cursor_after` | `text` | evet | |
| 9 | `error` | `text` | evet | |

**Kısıtlar:**

- `PK` `ingest_log_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `idx_ingest_log_source`
  ```sql
  CREATE INDEX idx_ingest_log_source ON core.ingest_log USING btree (source_id, started_at DESC)
  ```
- `ingest_log_pkey`
  ```sql
  CREATE UNIQUE INDEX ingest_log_pkey ON core.ingest_log USING btree (id)
  ```

## `core.kub_extract`

> TİTCK KÜB yapılandırılmış çıkarımı. `onay` = **klinik yüzey kapısı**; `hata=NULL` arafı ayrı durumdur.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `generic_key` | `text` | **hayır** | |
| 2 | `generic_name` | `text` | **hayır** | |
| 3 | `drug_id` | `bigint` | evet | |
| 4 | `kub_url` | `text` | **hayır** | |
| 5 | `endikasyon` | `text` | evet | |
| 6 | `doz` | `text` | evet | |
| 7 | `kontrendikasyon` | `text` | evet | |
| 8 | `etkilesim` | `text` | evet | |
| 9 | `uyari` | `text` | evet | |
| 10 | `onay` | `text` | **hayır** | `'pending'::text` |
| 11 | `onaylayan` | `text` | evet | |
| 12 | `onay_at` | `timestamp with time zone` | evet | |
| 13 | `model` | `text` | evet | |
| 14 | `tokens_in` | `integer` | evet | |
| 15 | `tokens_out` | `integer` | evet | |
| 16 | `pdf_chars` | `integer` | evet | |
| 17 | `hata` | `text` | evet | |
| 18 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |
| 19 | `ham_cikti` | `text` | evet | |

**Kısıtlar:**

- `PK` `kub_extract_pkey` → `PRIMARY KEY (generic_key)`

**İndeksler:**

- `idx_kub_hata`
  ```sql
  CREATE INDEX idx_kub_hata ON core.kub_extract USING btree (hata) WHERE (hata IS NOT NULL)
  ```
- `idx_kub_onay`
  ```sql
  CREATE INDEX idx_kub_onay ON core.kub_extract USING btree (onay)
  ```
- `kub_extract_pkey`
  ```sql
  CREATE UNIQUE INDEX kub_extract_pkey ON core.kub_extract USING btree (generic_key)
  ```

## `core.name_alias`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `query_norm` | `text` | **hayır** | |
| 2 | `ingredient` | `text` | evet | |
| 3 | `rxcui` | `text` | evet | |
| 4 | `source` | `text` | **hayır** | `'rxnorm'::text` |
| 5 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `name_alias_pkey` → `PRIMARY KEY (query_norm)`

**İndeksler:**

- `name_alias_pkey`
  ```sql
  CREATE UNIQUE INDEX name_alias_pkey ON core.name_alias USING btree (query_norm)
  ```

## `core.retracted_pub`

> Geri çekilme hattı. ⚠ Ad eşleşmesi DEĞER kanıtı değildir (bir kez %100 yanlış pozitif verdi).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `src_key` | `text` | **hayır** | |
| 3 | `pmcid` | `text` | evet | |
| 4 | `pmid` | `text` | evet | |
| 5 | `doi` | `text` | evet | |
| 6 | `title` | `text` | evet | |
| 7 | `pub_year` | `integer` | evet | |
| 8 | `source` | `text` | **hayır** | `'europepmc'::text` |
| 9 | `first_seen` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `retracted_pub_pkey` → `PRIMARY KEY (id)`
- `UNIQUE` `retracted_pub_source_src_key_key` → `UNIQUE (source, src_key)`

**İndeksler:**

- `idx_retracted_pmcid`
  ```sql
  CREATE INDEX idx_retracted_pmcid ON core.retracted_pub USING btree (pmcid) WHERE (pmcid IS NOT NULL)
  ```
- `retracted_pub_pkey`
  ```sql
  CREATE UNIQUE INDEX retracted_pub_pkey ON core.retracted_pub USING btree (id)
  ```
- `retracted_pub_source_src_key_key`
  ```sql
  CREATE UNIQUE INDEX retracted_pub_source_src_key_key ON core.retracted_pub USING btree (source, src_key)
  ```

## `core.sgk_odeme`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `barkod` | `text` | **hayır** | |
| 2 | `kamu_no` | `text` | evet | |
| 3 | `ilac_adi` | `text` | **hayır** | |
| 4 | `esdeger_grup` | `text` | evet | |
| 5 | `terapotik_grup` | `text` | evet | |
| 6 | `liste_giris` | `text` | evet | |
| 7 | `aktiflenme` | `text` | evet | |
| 8 | `pasiflenme` | `text` | evet | |
| 9 | `surum_id` | `bigint` | **hayır** | |

**Kısıtlar:**

- `PK` `sgk_odeme_pkey` → `PRIMARY KEY (barkod)`
- `FK` `sgk_odeme_surum_id_fkey` → `FOREIGN KEY (surum_id) REFERENCES core.sgk_odeme_surum(id) ON DELETE CASCADE`

**İndeksler:**

- `idx_sgk_odeme_ad`
  ```sql
  CREATE INDEX idx_sgk_odeme_ad ON core.sgk_odeme USING btree (lower(ilac_adi))
  ```
- `idx_sgk_odeme_esdeger`
  ```sql
  CREATE INDEX idx_sgk_odeme_esdeger ON core.sgk_odeme USING btree (esdeger_grup)
  ```
- `sgk_odeme_pkey`
  ```sql
  CREATE UNIQUE INDEX sgk_odeme_pkey ON core.sgk_odeme USING btree (barkod)
  ```

## `core.sgk_odeme_surum`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | `nextval('core.sgk_odeme_surum_id_seq'::regclass)` |
| 2 | `duyuru_url` | `text` | **hayır** | |
| 3 | `duyuru_tarihi` | `date` | evet | |
| 4 | `dosya_adi` | `text` | evet | |
| 5 | `satir_sayisi` | `integer` | evet | |
| 6 | `cekildi_at` | `timestamp with time zone` | **hayır** | `now()` |
| 7 | `aktif` | `boolean` | **hayır** | `false` |

**Kısıtlar:**

- `PK` `sgk_odeme_surum_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `idx_sgk_surum_tek_aktif`
  ```sql
  CREATE UNIQUE INDEX idx_sgk_surum_tek_aktif ON core.sgk_odeme_surum USING btree (aktif) WHERE aktif
  ```
- `sgk_odeme_surum_pkey`
  ```sql
  CREATE UNIQUE INDEX sgk_odeme_surum_pkey ON core.sgk_odeme_surum USING btree (id)
  ```

## `core.symptom`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `hpo_id` | `text` | **hayır** | |
| 2 | `label` | `text` | **hayır** | |
| 3 | `definition` | `text` | evet | |

**Kısıtlar:**

- `PK` `symptom_pkey` → `PRIMARY KEY (hpo_id)`

**İndeksler:**

- `idx_symptom_label`
  ```sql
  CREATE INDEX idx_symptom_label ON core.symptom USING btree (lower(label))
  ```
- `symptom_pkey`
  ```sql
  CREATE UNIQUE INDEX symptom_pkey ON core.symptom USING btree (hpo_id)
  ```

## `core.symptom_disease`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `hpo_id` | `text` | **hayır** | |
| 2 | `disease_ref` | `text` | **hayır** | |
| 3 | `disease_name` | `text` | evet | |
| 4 | `frequency` | `text` | evet | |
| 5 | `icd_disease_id` | `bigint` | evet | |

**Kısıtlar:**

- `FK` `symptom_disease_icd_disease_id_fkey` → `FOREIGN KEY (icd_disease_id) REFERENCES core.disease(id) ON DELETE SET NULL`
- `PK` `symptom_disease_pkey` → `PRIMARY KEY (hpo_id, disease_ref)`

**İndeksler:**

- `idx_sd_hpo`
  ```sql
  CREATE INDEX idx_sd_hpo ON core.symptom_disease USING btree (hpo_id)
  ```
- `idx_sd_icd`
  ```sql
  CREATE INDEX idx_sd_icd ON core.symptom_disease USING btree (icd_disease_id)
  ```
- `symptom_disease_pkey`
  ```sql
  CREATE UNIQUE INDEX symptom_disease_pkey ON core.symptom_disease USING btree (hpo_id, disease_ref)
  ```

## `core.sync_state`

> Sürekli besleme watermark'ı — cursor **OPAK** (yorumlanmaz).

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `source_id` | `text` | **hayır** | |
| 2 | `cursor` | `text` | evet | |
| 3 | `last_key` | `text` | evet | |
| 4 | `records_total` | `bigint` | **hayır** | `0` |
| 5 | `last_run_at` | `timestamp with time zone` | evet | |
| 6 | `last_success_at` | `timestamp with time zone` | evet | |
| 7 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `sync_state_pkey` → `PRIMARY KEY (source_id)`

**İndeksler:**

- `sync_state_pkey`
  ```sql
  CREATE UNIQUE INDEX sync_state_pkey ON core.sync_state USING btree (source_id)
  ```

## `core.trial`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `nct_id` | `text` | evet | |
| 3 | `title` | `text` | evet | |
| 4 | `status` | `text` | evet | |
| 5 | `conditions` | `ARRAY` | evet | |
| 6 | `source_id` | `text` | **hayır** | |
| 7 | `updated_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `UNIQUE` `trial_nct_id_key` → `UNIQUE (nct_id)`
- `PK` `trial_pkey` → `PRIMARY KEY (id)`

**İndeksler:**

- `trial_nct_id_key`
  ```sql
  CREATE UNIQUE INDEX trial_nct_id_key ON core.trial USING btree (nct_id)
  ```
- `trial_pkey`
  ```sql
  CREATE UNIQUE INDEX trial_pkey ON core.trial USING btree (id)
  ```

## `core.trial_disease`

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `trial_id` | `bigint` | **hayır** | |
| 2 | `disease_id` | `bigint` | **hayır** | |
| 3 | `method` | `text` | **hayır** | |

**Kısıtlar:**

- `FK` `trial_disease_disease_id_fkey` → `FOREIGN KEY (disease_id) REFERENCES core.disease(id) ON DELETE CASCADE`
- `PK` `trial_disease_pkey` → `PRIMARY KEY (trial_id, disease_id, method)`
- `FK` `trial_disease_trial_id_fkey` → `FOREIGN KEY (trial_id) REFERENCES core.trial(id) ON DELETE CASCADE`

**İndeksler:**

- `idx_td_disease`
  ```sql
  CREATE INDEX idx_td_disease ON core.trial_disease USING btree (disease_id)
  ```
- `trial_disease_pkey`
  ```sql
  CREATE UNIQUE INDEX trial_disease_pkey ON core.trial_disease USING btree (trial_id, disease_id, method)
  ```

# Şema `raw`

Ara-staging (ham connector çıktısı, ~4 GB). **Prod'a ALINMAZ.**

## `raw.records`

> Connector'ların ham JSON payload'ı. Yeniden işlenebilirlik için tutulur.

| # | Kolon | Tip | Null | Varsayılan |
|---|---|---|---|---|
| 1 | `id` | `bigint` | **hayır** | |
| 2 | `source_id` | `text` | **hayır** | |
| 3 | `source_key` | `text` | **hayır** | |
| 4 | `payload` | `jsonb` | **hayır** | |
| 5 | `fetched_at` | `timestamp with time zone` | **hayır** | `now()` |

**Kısıtlar:**

- `PK` `records_pkey` → `PRIMARY KEY (id)`
- `UNIQUE` `records_source_id_source_key_key` → `UNIQUE (source_id, source_key)`

**İndeksler:**

- `idx_raw_source`
  ```sql
  CREATE INDEX idx_raw_source ON raw.records USING btree (source_id)
  ```
- `records_pkey`
  ```sql
  CREATE UNIQUE INDEX records_pkey ON raw.records USING btree (id)
  ```
- `records_source_id_source_key_key`
  ```sql
  CREATE UNIQUE INDEX records_source_id_source_key_key ON raw.records USING btree (source_id, source_key)
  ```

