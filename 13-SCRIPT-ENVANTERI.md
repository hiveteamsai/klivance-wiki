# 13 — SCRIPT ENVANTERİ (`scripts/`)

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> **177 Python scripti.** "Ne yapar" sütunu **scriptin kendi docstring'inin ilk satırıdır**.

---

## 1. Script sınıfları

| Sınıf | Önek | Ne |
|---|---|---|
| **Veri alımı** | `ingest_*` | Kaynaktan `core.*`'a besleme |
| **Prod'a taşıma** | `migrate_*` | Yerel → prod KB transferi (**dry-run varsayılan**) |
| **Reklam operasyonu** | `meta_*` · `gads_*` · `fb_*` · `ig_*` | Meta Marketing API + Google Ads API |
| **Çıkarım** | `extract_*` | LLM ile yapılandırılmış çıkarım (KÜB, openFDA alanları) |
| **Ayrıştırma** | `parse_*` | PDF/HTML → korpus |
| **Teşhis / kontrol** | `diag_*` · `check_*` | Tek seferlik tanı araçları |
| **Operasyon** | `sync_feed` · `purge_retention` · `yedek_al` · `status` · `verify_db` | Cron ve bakım |
| **Mail** | `uye_aktivasyon_mail` · `ilk_soru_hatirlatma` · `yenileme_hatirlatma` · `telafi_bilgilendirme` · `tuzak_vaka_maili` · `mail_karsilastirma_gonder` | ⚠ Founder onayı gerektirir |

---

## 2. ⚠⚠ ÇALIŞTIRMADAN ÖNCE — DÖRT KURAL

### K1 — `--apply` olmadan koşan script çoğunlukla **dry-run**'dır
**101 script `--apply` bayrağı taşır.** Bayraksız koşum genelde yalnız rapor üretir.
⚠ Ama **bu bir garanti değildir** — her scriptin kendi başlığını oku.
Aşağıdaki tabloda `⚠` işareti = script `--apply`/`apply=` deseni içeriyor.

### K2 — PARA HARCAYAN / DIŞARI ÇIKAN İŞLEM **FOUNDER ONAYI** İSTER
Reklam scriptleri (`meta_*`, `gads_*`) **canlı reklam hesabına yazar**. Mail scriptleri
**gerçek hekimlere gönderir**. `migrate_*` **prod DB'ye yazar**.

### K3 — `scripts/*.py`: `__main__` kapısı ŞART
Çıplak `sys.exit(main())` **import edeni öldürür** = yeşil basan araç.
**+ hata gövdesine ÜYELİKLE bak** (`"HATA" in d`; `.get()` boş sözlükte fail-open).
İkisi de canlıda yakalandı → commit `8c0232e`.

> **Ölçülmüş durum (2026-09-17):** 177 scriptten **3'ü** modül düzeyinde çıplak `sys.exit`
> taşıyor **ve** `__main__` kapısı yok:
> `meta_kesim_20260812.py` · `meta_rmk_kitle_olc.py` · `meta_rmk_otomatik_ac.py`.
> Üçü de tek seferlik Meta reklam operasyon scriptidir ve hiçbir yerden import edilmez →
> bugün zarar vermiyor, ama deseni **kopyalama**.

### K4 — ⚠ Toplu `--apply` YASAKLARI
| Script | Yasak |
|---|---|
| `scratchpad/kvkk_takma_kod_temizle.py` | **Toplu `--apply` YASAK** — gerçek ad artık meşru; kapsamsız apply'ı kod zaten reddeder. Yalnız tek hesaplık KVKK m.11 talebi için |
| `mailer` künyesi boşken | Mail betikleri `--apply`ı **REDDEDER** (`mailer._MERSIS`) |

---

## 3. SIK KULLANILAN REÇETELER

```bash
# ── Durum ──────────────────────────────────────────────────
./.venv/Scripts/python.exe scripts/status.py        # besleme watermark + son koşu
./.venv/Scripts/python.exe scripts/verify_db.py     # tablo bazında satır sayısı
./.venv/Scripts/python.exe scripts/smoke_test.py    # hızlı duman testi

# ── Besleme ────────────────────────────────────────────────
./.venv/Scripts/python.exe scripts/sync_feed.py openfda --batch 500
./.venv/Scripts/python.exe scripts/run_ingest.py <source>

# ── TİTCK KÜB (YALNIZ YERELDEN — bulut-IP engeli) ──────────
./.venv/Scripts/python.exe scripts/extract_kub_batch.py
./.venv/Scripts/python.exe scratchpad/kub_onay_oncesi_kontrol.py   # ⚠ ONAYDAN ÖNCE ŞART
./.venv/Scripts/python.exe scripts/extract_kub_batch.py --onayla

# ── Prod'a taşıma (dry-run VARSAYILAN, founder onayı ister) ─
./.venv/Scripts/python.exe scripts/migrate_kb_delta.py
./.venv/Scripts/python.exe scripts/migrate_corpus.py

# ── Bakım ──────────────────────────────────────────────────
./.venv/Scripts/python.exe scripts/yedek_al.py
./.venv/Scripts/python.exe scripts/purge_retention.py
./.venv/Scripts/python.exe scripts/rotate_file_key.py      # Fernet rotasyonu

# ── Reklam raporu (salt-okuma) ─────────────────────────────
./.venv/Scripts/python.exe scripts/reklam_rapor.py
./.venv/Scripts/python.exe scripts/ga4_report.py
```

⚠ `requirements-ads.txt` **ayrıdır** — `google-ads` (ağır `grpcio`/`protobuf`) app runtime'a
girmez, prod imajını şişirmesin. Reklam scriptleri ve CI onu kurar.

---

## 4. ENVANTER

### VERİ ALIMI (ingest)  (12)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `ingest_asi_virus_sayfa.py` |  | Kamu malı KURUM SAYFALARI — aşı / virüs / parazit / enjeksiyon (#764c-İNDİR, 2026-09-11, Derya). |
| `ingest_bakteri_sayfa.py` |  | ECDC A-Z bakteriyel konu sayfaları (`ecdc_azlist`) + CDC HCP klinik sayfaları (`cdc_hcp`) |
| `ingest_dergipark.py` |  | DergiPark (TR) OAI-PMH harvester — Türkçe açık tıp dergilerinden makale özetleri. |
| `ingest_europepmc.py` |  | Europe PMC konu-hedefli literatür — klinik aktif hastalıklar için son yıl |
| `ingest_europepmc_derin.py` |  | EUROPE PMC DERİN KAZI — mevcut dar ingest'in kapsamını genişletir + HAM veriyi diske yazar. |
| `ingest_europepmc_konu.py` |  | EUROPE PMC KONU-ANAHTARLI İNGEST — insan vücudundaki bakteriler (mikrobiyom · patojen · AMR). |
| `ingest_europepmc_rads.py` | ⚠ | EUROPE PMC — RADYOLOJİ SINIFLAMA SİSTEMLERİ (IP3 / görüntüleme YOL 1; kararlar B.Q5 EVET). |
| `ingest_faers.py` |  | openFDA FAERS advers olay sıklığı → `core.adverse_event`. |
| `ingest_greenbook_bolum.py` |  | UKHSA Green Book PDF bölümleri → core.corpus `ukhsa_greenbook` (#764c-İNDİR, 2026-09-11, Derya). |
| `ingest_hpo.py` |  | HPO semptom ontolojisi + fenotip-hastalık anotasyonu → core.symptom / core.symptom_disease. |
| `ingest_kamu_sayfa.py` |  | Kamu malı KURUM SAYFALARI → core.corpus (#763c-İNDİR, 2026-09-11, Derya). |
| `ingest_pdf_bolum.py` |  | PDF → core.corpus BÖLÜMLÜ İNGEST — kamu malı / CC BY tek belgeler (CDC MMWR · ECDC/WHO AMR). |

### PROD'A TAŞIMA (migrate)  (5)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `migrate_corpus.py` | ⚠ | Yerel `core.corpus` (+ `core.corpus_disease` bağları) → PROD. |
| `migrate_faers.py` | ⚠ | Yerel `core.adverse_event` (openFDA FAERS) verisini PROD'a taşır. |
| `migrate_kb_delta.py` |  | Yerel KB'deki Orphanet (nadir hastalık) + enforcement (recall) verisini PROD'a taşır. |
| `migrate_kub_extract.py` | ⚠ | Yerel `core.kub_extract` (onaylı TİTCK KÜB çıkarımı) → PROD taşıma. |
| `migrate_retraction.py` | ⚠ | Geri çekilme (retraction) DURUM ALANLARINI yerelden PROD'a taşır (#67). |

### META (Facebook/Instagram) REKLAM  (66)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `meta_abd_pilot.py` | ⚠ | ABD PİLOTU — kampanya + ad set (+ görsel gelince reklam) KUR, **PAUSED BIRAK**. |
| `meta_abd_pilot_olc.py` |  | ABD pilotunun SONUCUNU ölçer ve TR tabanıyla kıyaslar (SALT OKUMA). |
| `meta_abd_yayina_al.py` | ⚠ | ABD pilot kampanyasını YAYINA ALIR (kampanya + ad set + reklam → ACTIVE). |
| `meta_acil_fren.py` | ⚠ | META ACİL FREN — denetim bulgusu sonrası teslim düzeltmesi (2026-07-29). |
| `meta_an_test_kapat.py` | ⚠ | META — `KLV-TR-AN-TEST` KAPATMA (founder talimatı 2026-08-10: "an testi kapat"). |
| `meta_an_test_kur.py` |  | Audience Network SINIRLI TESTİ — ayrı kampanya + AN-only ad set, 100 TL/gün. |
| `meta_an_yayina_al.py` |  | AN sınırlı testini YAYINA ALIR: kreatif + reklam + kampanya/ad set ACTIVE. |
| `meta_audience_network_kapat.py` |  | Meta AUDIENCE NETWORK yerleşimini kapatır (ölçülmüş çöp trafik). |
| `meta_aylik_30k_20260901.py` | ⚠ | META — AYLIK 30.000 TL DÜZENİ (founder 2026-09-01 23:5x: "aylık 30000 yap. diğerlerini de |
| `meta_brans_yayina_al.py` |  | BRANŞ kampanyası: psikiyatri + psikolog soru-cevap reklamları, 100'er TL/gün. |
| `meta_butce_120.py` | ⚠ | Meta — TÜM AKTİF ad set'lerin günlük bütçesini 120 TL'ye çeker (founder 2026-08-09). |
| `meta_butce_350.py` | ⚠ | AKTİF ad set'lerin GÜNLÜK BÜTÇESİNİ 350 TL'ye eşitler (founder 2026-08-06). |
| `meta_callout_kimlik_duzelt.py` | ⚠ | GOOGLE ADS — #134b: «Hasta kimliği işlenmez» callout'unu düzelt. ⚠⚠ VARSAYILAN DRY-RUN. |
| `meta_denetle_orijinal_sayi_duzelt.py` | ⚠ | `KLV-TOF-denetle-orijinal` gövde metnini düzelt — YENİ kreatif + YENİ reklam. |
| `meta_fb_kredi_dili_temizle.py` | ⚠ | FACEBOOK sayfa gönderilerinde KREDİ DİLİNİ düzelt (#396b). VARSAYILAN DRY-RUN. |
| `meta_gorsel_tazele.py` | ⚠ | META canlı kreatiflerinde GÖRSEL TAZELEME (StatPearls lisans P0 temizliği). |
| `meta_hedef_hekim.py` |  | Meta hedeflemesini SALT HEKİM'e kilitler (founder kuralı: "hedefleme her |
| `meta_inis_anasayfa.py` | ⚠ | META İNİŞ SAYFASI: V6 ARAÇ reklamları → ANA SAYFA (`/`) — founder kararı 2026-08-06. |
| `meta_inis_chatgpt.py` | ⚠ | META İNİŞ SAYFASI → `/chatgpt` (founder talimatı 2026-07-30). |
| `meta_kart_ifadesi_kaldir.py` | ⚠ | #401b — CANLI 4 REKLAMDAN «kart gerekmez» KALDIR. VARSAYILAN DRY-RUN. |
| `meta_kesim_20260812.py` | ⚠ | Meta — ONAYLANMIŞ REKLAM KESİMİ (founder onayı 2026-08-12: "başla hepsini yap"). |
| `meta_kesim_gb_ae_20260818.py` | ⚠ | Meta — GB + AE KOLLARINI DURDURMA ÖNERİSİ (D18 denetimi, 2026-08-18). |
| `meta_kirli_kreatif_sil.py` | ⚠ | Meta kreatif deposundaki KİRLİ + BAĞLI-OLMAYAN kreatifleri sil (founder onayı 2026-08-01). |
| `meta_klv_kayit_donusum_onar.py` | ⚠ | `klv_kayit` ÖZEL DÖNÜŞÜMÜNÜ ONAR — atıf katmanı ölçülemiyor (#255b). |
| `meta_lpv_ac.py` |  | LPV kampanyasını 250 TL/gün ile yeniden aç (founder onayı 2026-07-29). |
| `meta_metin_tazele.py` | ⚠ | META reklam METİNLERİNDE (gövde/başlık/açıklama) toplu metin ikamesi. |
| `meta_organik_statpearls_temizle.py` | ⚠ | Organik sosyal gönderilerden StatPearls adını çıkar (lisans P0, CC BY-NC-ND). |
| `meta_ozel_donusum_olustur.py` |  | Meta ÖZEL DÖNÜŞÜM: `klv_kayit` (FİLTRESİZ kayıt) — founder onayı 2026-07-29. |
| `meta_post.py` |  | Klivance — Meta (Facebook Sayfa + Instagram) ORGANİK gönderi yayınlayıcı. |
| `meta_reels_kapat.py` | ⚠ | META REELS KAPATMA — founder kararı 2026-08-01. ⚠⚠ VARSAYILAN DRY-RUN. |
| `meta_remarketing_kur.py` | ⚠ | Klivance — Meta REMARKETING kurulumu (founder kararı 2026-08-09). |
| `meta_rmk_alan_duzelt.py` | ⚠ | META — İKİ ONAYLI DÜZELTME (founder onayı 2026-08-10). ⚠⚠ VARSAYILAN DRY-RUN. |
| `meta_rmk_dislama_ekle.py` | ⚠ | META — RMK ad set'ine KAYITLI/ALICI haric-tutmasi ekle. Founder onayi 2026-08-18. |
| `meta_rmk_donusum_gecisi.py` | ⚠ | RMK AD SET'İNİ DÖNÜŞÜME GEÇİR — `KLV-TR-RMK-ZIYARETCI-01`. |
| `meta_rmk_kapat_20260812.py` | ⚠ | META — `KLV-TR-RMK-ZIYARETCI-01` ad set'ini DURDUR. Founder onayı 2026-08-12. |
| `meta_rmk_kitle_olc.py` |  | Remarketing kitlesi 1.000 eşiğini GEÇTİ Mİ — tek soruyu ölçer, tek cevap verir. |
| `meta_rmk_kreatif_degistir.py` | ⚠ | REMARKETING kreatifini v10-RMK karşılaştırma görseliyle DEĞİŞTİRİR. DRY-RUN varsayılan. |
| `meta_rmk_otomatik_ac.py` | ⚠ | Kitle eşiği geçtiyse remarketing kampanyasını OTOMATİK AÇ (founder kararı 2026-08-09). |
| `meta_spend_cap.py` | ⚠ | Meta reklam hesabının HARCAMA TAVANINI (`spend_cap`) okur / günceller. |
| `meta_sure_uzat_20260901.py` | ⚠ | META — `KLV-META-V10-VAKA` / `KLV-TR-V10-01` SÜRE UZATMA (founder talimatı 2026-09-01). |
| `meta_v10_adset_ac_20260901.py` | ⚠ | META — `KLV-TR-V10-01` ad set'ini yeniden AÇ (founder 2026-09-01 23:3x: "v14y leri aç"). |
| `meta_v10_gb_kolu.py` | ⚠ | v10 VAKA — İNGİLTERE (GB) kolu ekler. DRY-RUN VARSAYILAN. |
| `meta_v10_vaka_ac.py` | ⚠ | v10 VAKA TR reklamını KREDİ/SÜRE/KART DİLİ OLMADAN yeniden açar. DRY-RUN VARSAYILAN. |
| `meta_v10_vaka_yayin.py` | ⚠ | v10 VAKA kampanyası — TR + BAE, 3 dilli kreatif. DRY-RUN VARSAYILAN. |
| `meta_v12_yayinla.py` | ⚠ | V12 — iki yeni kreatifi MEVCUT ad set'e EK REKLAM olarak ekle. VARSAYILAN DRY-RUN. |
| `meta_v15_video_gece_nobeti.py` | ⚠ | META — V15 «gece nöbeti» VİDEO deneme kolu (#712b). ⚠⚠ VARSAYILAN DRY-RUN. |
| `meta_v5_ac.py` | ⚠ | v5 kampanyasını ACTIVE'e alır (founder onayı 2026-08-02, Ömer iletti). |
| `meta_v5_hedef_18_60.py` | ⚠ | v5 ad set hedeflemesi: yaş 18-60 (+ opsiyonel bileşim takası) — founder kararı 2026-08-03. |
| `meta_v5_hedef_sadelestir.py` | ⚠ | v5 ad set hedeflemesini sadeleştir — founder kararı 2026-08-02 (Ömer iletti). |
| `meta_v5_kampanya.py` | ⚠ | v5 SORU-CEVAP → yeni Meta kampanyası (founder onayı 2026-08-02, Ömer iletti). |
| `meta_v6_arac_sure_uzat.py` | ⚠ | META — `KLV-TR-V6-ARAC-01` SÜRE UZATMA +7 GÜN (founder talimatı 2026-08-10). |
| `meta_v6_butce_tavani.py` | ⚠ | v6 ad set'ine YAPISAL harcama tavanı yazar: 700 TL ömür-boyu bütçe + 7 günlük pencere. |
| `meta_v6_kirli_reklam_durdur.py` | ⚠ | V6'NIN IKI REKLAMINI DURDUR — VARSAYILAN DRY-RUN. |
| `meta_v6_link_gorsele_uydur_20260818.py` | ⚠ | Meta — GÖRSEL-LİNK UYUŞMAZLIĞI ONARIMI (founder onayı 2026-08-18: "linki görsele uydur"). |
| `meta_v6_metin_hizala.py` | ⚠ | V6 reklamlarının GÖVDE METNİNİ iniş sayfasıyla hizalar (founder 2026-08-06). |
| `meta_v6_mgkg_derin_baglanti_20260818.py` | ⚠ | Meta — mg/kg HESAPLAYICI REKLAMI: `?c=` DERİN BAĞLANTISI + GÖRSEL METNİ (founder onayı 2026-08-18). |
| `meta_v6_yayina_al.py` | ⚠ | v6 ad set'ini YAYINA ALIR: `status` → ACTIVE. **Yazdığı TEK alan budur.** |
| `meta_v7_baslik_uyumlu_ac.py` | ⚠ | V7-AKUT-VAKA'YI 28.08 BAŞLIK KURALINA UYGUN OLARAK YENİDEN AÇ — VARSAYILAN DRY-RUN. |
| `meta_v7_yayina_al.py` | ⚠ | v7 AKUT VAKA kreatifini YAYINA ALIR ve V5 reklamlarını DURDURUR (founder 2026-08-06). |
| `meta_v8_carousel_hazirla.py` | ⚠ | KLV-V8 ADSIZ CAROUSEL — mevcut `KLV-TR-V5-01` ad set'ine **PAUSED** yeni reklam. |
| `meta_v9_reklam_yenile.py` | ⚠ | V9 B kolu — REKLAMLARI YENİLER (Fırat denetimi P1 A+B + P2 kesme düzeltmesi). |
| `meta_vschat_b_ekle.py` | ⚠ | VSCHAT ad set'ine `vs-chatgpt` mesajının B VARYANTINI ekle. |
| `meta_vschatgpt_kampanya.py` | ⚠ | META — `vs-chatgpt` kreatifi için AYRI kampanya (aç kalan varlığa gerçek teslimat). |
| `meta_yerlesim_beyaz_liste.py` |  | YERLEŞİM BEYAZ LİSTESİ — aktif ad set'leri Feed+Stories'e kısıtlar. |
| `meta_yerlesim_duzelt.py` | ⚠ | META YERLEŞİM ONARIMI — Feed + Story geri açılır, Reels KALIR. |
| `meta_yerlesim_kesim_20260901.py` | ⚠ | META YERLEŞİM KESİMİ + V5 DURDURMA — founder onayı 2026-09-01 ("hepsini uygula"). |

### GOOGLE ADS  (21)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `gads_ag_hesaplayici_durdur.py` | ⚠ | `AG-Hesaplayici` reklam grubunu DURDUR + gerekçeyi platformda ETİKETLE (founder onayı 2026-08-01). |
| `gads_atif_ayarla.py` | ⚠ | Google Ads ATIF BOŞLUĞU onarımı — `final_url_suffix` (utm_*) hesap + kampanya düzeyinde. |
| `gads_butce_kararlari_20260801.py` | ⚠ | Öncelik 3 — Google bütçe/durum kararları (founder onaylı, 2026-08-01). |
| `gads_callout_bagi_kaldir.py` | ⚠ | GOOGLE ADS — KIRLI VARLIK BAGLARINI KALDIR (CALLOUT + SITELINK).  (#512b-A) |
| `gads_cevrimdisi_donusum.py` | ⚠ | Google Ads ÇEVRİMDIŞI DÖNÜŞÜM YÜKLEME — DB'deki gclid'i Ads'e geri gönderir. |
| `gads_cta_soru.py` | ⚠ | Google RSA: CTA birimi "15 kredi" → "15 ücretsiz soru" (founder kararı 2026-07-31). |
| `gads_display_cpc_tavan.py` |  | Display remarketing'in GERÇEKTE bağlayıcı olan CPC tavanını amaçlanan değere çeker. |
| `gads_display_kreatif_v2.py` |  | Display remarketing kreatifini PANEL KARARLARIYLA yeniler (2026-08-07). |
| `gads_display_reklam_tamamla.py` |  | Display remarketing kurulumunun SON adımı: duyarlı display reklamını oluşturur. |
| `gads_display_remarketing.py` |  | Google DISPLAY REMARKETING kampanyasını kurar ve AÇAR (founder 2026-08-06). |
| `gads_display_yerlesim_temizle.py` |  | Display remarketing'de ÇÖP ENVANTERİ hariç tutar (mobil uygulama + içerik etiketi + |
| `gads_etkilesim_lp_reklam_durdur.py` | ⚠ | `AG-Etkilesim-LP` reklamını duraklat + gerekçeyi PLATFORMA etiketle (Fırat tur-2 bulgusu). |
| `gads_etkilesim_url_duzelt.py` |  | AG-Etkilesim'i '/' anasayfadan '/etkilesim' ARACINA bağla. |
| `gads_hesaplayici_url_duzelt.py` | ⚠ | Google Ads: hesaplayıcı kelimelerinin ÖLÜ `#fragment` inişini `?c=` ile değiştirir. |
| `gads_negatif_ekle.py` | ⚠ | Google Ads NEGATİF ANAHTAR KELİME ekleme — arama terimi sızıntısı onarımı. |
| `gads_rda_sure_vaadi_duzelt.py` | ⚠ | #511b — CANLI GOOGLE REKLAMINDA SURE VAADINI KALDIR.  (TEK REKLAM, TEK BASLIK) |
| `gads_red_kaldir_butce.py` | ⚠ | GOOGLE: reddedilen reklamı devreden çıkar + CDS bütçesini 250 TL/gün yap. |
| `gads_rmk_baslik_ve_durdur.py` | ⚠ | #513b — GOOGLE ADS UYGULAMA: (A) RMK basligini duzelt  (B) RMK kampanyasini DURDUR. |
| `gads_rsa_onayli_dile_cek.py` | ⚠ | GOOGLE ADS — RSA BASLIK/ACIKLAMALARINI ONAYLI DILE CEK.  (#512b-B) |
| `gads_statpearls_kaldir.py` | ⚠ | Google RSA açıklamalarından StatPearls adını çıkar (lisans P0). |
| `gads_statpearls_sitelink_bayat_sayi.py` | ⚠ | Google Ads: StatPearls'ü CANLI sitelink'ten çıkar + bayat sayıları düzelt. |

### FACEBOOK SAYFA/GÖNDERİ  (4)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `fb_duyuru_yayinla.py` | ⚠ | #396b — FACEBOOK sayfa gonderisi: "yeni sisteme gectik" duyurusu. VARSAYILAN DRY-RUN. |
| `fb_gorsel_kredi_sil.py` | ⚠ | #396b — FB'de GORSELINDE SINIF-A kredi vaadi kalan gonderileri sil. VARSAYILAN DRY-RUN. |
| `fb_kalan_yedek_al.py` |  | #396b — FB'de KALAN 10 gonderinin TAM yedegi (silme ONCESI, silme YAPMAZ). |
| `fb_kirli_gonderi_sil.py` | ⚠ | #396b — FACEBOOK'ta SINIF-A (yanlis beyan) gonderileri SIL. VARSAYILAN DRY-RUN. |

### INSTAGRAM  (2)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `ig_statpearls_gonderi_sil.py` | ⚠ | Instagram'da StatPearls geçen gönderileri SİL (lisans P0). ⚠⚠ GERİ ALINAMAZ. |
| `ig_story_yayinla.py` | ⚠ | #396b — INSTAGRAM STORY: "yeni sisteme gectik" duyurusu. VARSAYILAN DRY-RUN. |

### ÇIKARIM (LLM)  (3)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `extract_kub.py` |  | KÜB (Kısa Ürün Bilgisi / SmPC) yapılandırılmış çıkarım — PoC / TEMEL (L, kök çözüm). |
| `extract_kub_batch.py` |  | KÜB TOPLU ÇIKARIM — TR ilaç kartlarının klinik boşluğunu kapatan hat. |
| `extract_openfda_fields.py` |  | openFDA ham payload'larından hekim-kritik klinik alanları core.drug'a çıkarır. |

### AYRIŞTIRMA  (4)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `parse_bookshelf.py` |  | NCBI Bookshelf NXML paketini core.corpus'a yükler (LactMed, LiverTox, ...). |
| `parse_bookshelf_html.py` | ⚠ | NCBI Bookshelf **HTML/TXT** paketini core.corpus'a yükler (NXML olmayan kazımalar için). |
| `parse_bookshelf_sec.py` | ⚠ | NCBI Bookshelf NXML paketini **BÖLÜM İÇİ `<sec>` düzeyinde** core.corpus'a yükler. |
| `parse_statpearls.py` |  | StatPearls NXML makalelerini metne çevirip core.corpus'a yükler. |

### TEŞHİS  (4)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `diag_ct.py` |  | ClinicalTrials 403 teshisi: farkli header kombinasyonlari. |
| `diag_rxnav.py` |  | TR etkin madde -> RxNorm eslesme fizibilitesi (RxNav API, anahtarsiz). |
| `diag_rxnav2.py` |  | Uretim eslesme mantigini izole test et — skor + hata gorunur. |
| `diag_titck.py` |  | TITCK KUB/KT endpoint dogrulama: session + _token + tam DataTables payload. |

### KONTROL  (3)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `check_disease.py` |  | ICD-11 hastalik kayitlarini UTF-8 dosyaya yazar. |
| `check_titck.py` |  | TITCK kayitlarini UTF-8 dosyaya yazar (konsol encoding sorununu atlatmak icin). |
| `check_tm.py` |  | ICD-11 verisinde Geleneksel Tip (Traditional Medicine) bolumu var mi? |

### KÖPRÜ  (2)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `bridge_tr.py` |  | TR ilaç → hastalık köprüsü (etkin madde ADI üzerinden — düzeltilmiş). |
| `bridge_trials.py` |  | Klinik denemeleri grafiğe bağlar: deneme → hastalık (condition metni × ICD başlığı) |

### İNŞA  (2)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `build_bridge.py` |  | İlaç ↔ Hastalık köprüsü (v1 sezgisel). |
| `build_hierarchy.py` |  | ICD-11 hastalık hiyerarşisini sorgulanabilir hale getirir (parent_id). |

### ZENGİNLEŞTİRME  (2)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `enrich_classkind.py` |  | ICD-11 classKind'i core.disease'e ekler ve ham payload'dan backfill eder. |
| `enrich_titck_rxnorm.py` |  | TİTCK etkin maddelerini RxNorm'a bağlar (RxNav approximateTerm, anahtarsız). |

### PİLOT ÖLÇÜM  (5)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `pilot_guvenlik_slot_olc.py` |  | KABUL ÖLÇÜTÜ (3) — tam metin GÜVENLİK KAYNAKLARINI eliyor mu? |
| `pilot_kabul_olc.py` |  | PİLOT KABUL ÖLÇÜTÜ — tabanla KARŞILAŞTIR, hükmü SCRIPT versin. |
| `pilot_taban_olc.py` |  | PİLOT TABAN ÖLÇÜMÜ — özet→tam metin pilotunun "ÖNCESİ" durumu. |
| `pilot_tam_metin.py` | ⚠ | PİLOT — europepmc ÖZET → TAM METİN yükseltmesi (5.000 makale, YALNIZ YEREL). |
| `pilot_tsrank_norm_olc.py` |  | ts_rank NORMALİZASYONU — hangisi? ÖLÇ, TAHMİN ETME. (SALT-OKUMA) |

### REKLAM RAPOR/BÜTÇE  (4)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `reklam_butce_dagit.py` | ⚠ | GÜNLÜK 1.000 TL BÜTÇE DAĞITIMI (founder kararı 2026-07-29) — Meta + Google. |
| `reklam_rapor.py` |  | GÜNLÜK REKLAM RAPORU — "reklam kontrol" dendiğinde çalışan tek komut. |
| `reklam_v5_tumunu_durdur.py` | ⚠ | TÜM REKLAMLARI DURDUR — Meta + Google (founder kararı 2026-08-02). |
| `reklam_yapisi_20260802.py` | ⚠ | REKLAM YAPISI DEĞİŞİKLİĞİ — founder onaylı (2026-08-01, Ömer iletti). |

### DİĞER / OPERASYON  (38)

| Script | `--apply` | Ne yapar |
|---|:-:|---|
| `backfill_label_date.py` | ⚠ | #70 — `core.drug.label_date / label_date_kind / label_version` GERİYE DÖNÜK DOLDURMA. |
| `bulk_ingest_openfda.py` |  | openFDA bulk etiket loader — indirilen 14 ZIP'ten tüm ABD etiketlerini ingest eder. |
| `cds.py` |  | Hekim karar-destek prototipi — RETRIEVAL motoru. |
| `churn_otopsi.py` |  | AYRILIK OTOPSİSİ — silme talebi veren hekim NEDEN gitti? (SALT-OKUR) |
| `cop_generic_temizle.py` | ⚠ | `core.drug.generic_name` ÇÖP KİMLİKLERİNİ temizle (yerel ve/veya PROD). |
| `fetch_retractions.py` | ⚠ | GERİ ÇEKİLME (retraction) YETKİLİ LİSTESİ — Europe PMC → `core.retracted_pub` (#67). |
| `ga4_report.py` |  | GA4 raporlama aracı (servis hesabıyla — kullanıcı OAuth'una DOKUNMAZ). |
| `geo_veri_uret.py` |  | Kayıt formu ÜLKE + ŞEHİR verisini üretir (2026-08-15, founder: "bütün dünyadaki ülkeler ve |
| `google_ads_test.py` |  | Google Ads bağlantı + refresh token FONKSİYONEL testi + para birimi/saat dilimi TEYİDİ. |
| `hpo_capa_uret.py` |  | HPO ÇAPA KÜMESİ ÜRETECİ — `saglik/cds/fenotip_capa.py` dosyasını yazar. |
| `hpo_tr_uret.py` |  | HPO TÜRKÇE ETİKET KÖPRÜSÜ ÜRETECİ — `saglik/cds/fenotip_tr.json` dosyasını yazar. |
| `ilk_soru_hatirlatma.py` | ⚠ | İLK-SORU HATIRLATMASI — kaydolup hiç soru sormamış hekime tek seferlik mail. |
| `indir_greenbook_pdf.py` |  | UKHSA Green Book ("Immunisation against infectious disease") bölüm PDF'lerini İNDİRİR |
| `init_db.py` |  | Şemayı PostgreSQL'e yükler:  python scripts/init_db.py |
| `klivance_query.py` |  | Klivance anahtarsız motor — CLI test. |
| `kredi_dili_canli_duzelt.py` | ⚠ | #396b — CANLI 3 reklamda kredi dilini duzelt. VARSAYILAN DRY-RUN. |
| `kub_url_tazele.py` |  | ÖLÜ KÜB LİNKLERİNİ AYNI ETKEN MADDENİN BAŞKA MARKASINDAN TAZELE. |
| `link_corpus_diseases.py` |  | Korpus (StatPearls) makalelerini hastalıklara bağlar — başlık eşleşmesiyle. |
| `mail_karsilastirma_gonder.py` | ⚠ | «Aynı soru, dört sistem» karşılaştırma maili — TÜM ÜYELERE. DRY-RUN VARSAYILAN. |
| `prod_geri_cekilen_sil.py` | ⚠ | PROD `core.corpus`'tan GERİ ÇEKİLMİŞ makaleleri sil (+ `corpus_disease` bağları). |
| `purge_retention.py` |  | KVKK retention purge — grace süresi dolan silme-talepli hesapları + eski soft-delete |
| `raw_bozuk_tara.py` |  | raw.records'taki BOZUK (DataCorrupted/TOAST) satırları bul — ikili bölmeyle. |
| `reset_icd11.py` |  | Eski ICD-11 verisini temizler (surum/dil degisikligi oncesi). |
| `rotate_file_key.py` |  | Hasta dosyası şifreleme anahtarı rotasyonu — tüm aktif case_file.content_enc'i BİRİNCİL |
| `run_ingest.py` |  | Bir kaynağı çeker ve bilgi tabanına yazar. |
| `showcase.py` |  | Bilgi tabanının nihai vitrini: bilingual ilaç→hastalık köprüsü + istatistikler. |
| `smoke_test.py` |  | DB'siz smoke test: importlar + canlı openFDA fetch. |
| `sosyal_gunluk.py` | ⚠ | GÜNLÜK ORGANİK SOSYAL YAYIN — IG besleme + IG hikâye + FB sayfa. VARSAYILAN DRY-RUN. |
| `status.py` |  | Bilgi tabanı tam durum anlık görüntüsü → UTF-8 dosya (konsol encoding'ini atlar). |
| `sut_ek4a_cek.py` | ⚠ | SGK Ek-4/A "Bedeli Ödenecek İlaçlar Listesi" → `core.sgk_odeme` (#354b). |
| `sync_feed.py` |  | Sürekli besleme hattı CLI — bir kaynağı artımlı/idempotent çeker (watermark + koşu logu). |
| `telafi_bilgilendirme.py` | ⚠ | #396b-K TELAFİ BİLGİLENDİRME MAİLİ — kredi kesiminde erişimi kapanan hekimlere. |
| `tuzak_vaka_maili.py` | ⚠ | ÖRNEK VAKA MAİLİ — «zor bir vaka» ile aktivasyon (founder isteği 2026-08-31). |
| `uye_aktivasyon_mail.py` | ⚠ | ÜYE AKTİVASYON E-POSTASI — hiç soru sormamış üyelere tek seferlik hatırlatma. |
| `verify_db.py` |  | Bilgi tabanının doldugunu dogrular. |
| `yedek_al.py` | ⚠ | PROD VERİTABANI YEDEĞİ — tek komutla, doğrulanmış, tekrarlanabilir. |
| `yeni_sistem_duyuru.py` | ⚠ | #406b YENİ SİSTEM DUYURUSU — kredi sisteminin kalktığını hekimlere bildirir. |
| `yenileme_hatirlatma.py` | ⚠ | YENİLEME HATIRLATMASI ("C KÖPRÜSÜ") — erişim dönemi bitmek üzere / bitti bildirimi. |


---

## 5. ARŞİV

`scripts/arsiv-olu-kampanya/` — kullanımdan kalkmış reklam kampanya scriptleri.
`scratchpad/arsiv-kredi/` — kredi sistemi kaldırıldığında (2026-08-21) emekliye ayrılan
doğrulama ağları.

⚠ Arşivdeki bir scripti geri getirmeden önce **neden emekliye ayrıldığına** bak — çoğu
ölçülerek reddedilmiş bir yaklaşımı temsil eder.

---

## 6. REKLAM SCRIPTLERİ — devralan ekip için not

Reklam operasyonu **ürün kodunun bir parçası değildir**; ayrı bir disiplindir ve kendi
kural seti `.claude/agents/hasan.md`'dedir. Devralan ekip **ürün** geliştirmesini
devralıyorsa bu scriptlere dokunmasına gerek yoktur.

Bilinmesi gereken üç şey:

1. ⚠⚠ **Canlı reklam inişleri kırılmaz.** `/etkilesim` (eski `?a=&b=` formatı dahil),
   `/hesaplayicilar` ve `/` ücretli reklam inişidir. URL/parametre değiştirmek **para
   yakar** (kıran P0 sınıfı).
2. ⚠⚠ **Geri alma dalı, alma dalının AYNASI DEĞİLDİR.** Ölçülmüş ders: durdururken
   "hepsi" doğru, açarken "hepsi" **yanlış** (bilerek kapatılmış kampanyalar açılacaktı).
   Kilit: **beyaz liste + damga**.
3. ⚠⚠ **`effective_status=ACTIVE` teslimat GARANTİ ETMEZ** — `end_time` geçmişteyse
   API **söylemez** (`issues_info`/`recommendations` boş), ama reklam teslimat almaz.
   Hüküm ampirik kurulur.

---

**Sonraki:** [`14-MASAUSTU-VE-EKLENTI.md`](14-MASAUSTU-VE-EKLENTI.md)
