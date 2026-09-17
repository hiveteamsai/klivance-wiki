# 14 — MASAÜSTÜ PROGRAM VE e-NABIZ EKLENTİSİ

> Ölçüm anı: 2026-09-17 · commit `cb16758d`.
> Dizin: `Klivance_software v101/`
>
> ⚠⚠ **DURUM: DENEME.** Kod + otomatik kapılar yeşil, **uçtan uca test yapılmadı**.
> **Canlıya dokunmaz** ve bu bir kapıyla çivilidir (§2).

---

## 1. Ne yapar

Hastanın **kendi** açık e-Nabız oturumundan sağlık kayıtlarını okur →
`Downloads\enabiz\` altına indirir → hekimin **Klivance hasta kartına** aktarır.

İki bileşen:

| Bileşen | Teknoloji | Yol |
|---|---|---|
| **Chrome eklentisi** (v0.3.0) | Manifest V3 | `Klivance_software v101/eklenti/` |
| **Masaüstü program** | Electron 44 | `Klivance_software v101/src/` |

---

## 2. ⚠⚠ DENEME KİLİDİ — eklenti CANLIYA DOKUNMAZ

**Founder kararı 2026-08-17:** *"hasta kartı butonunu canlıya alma, deneme yapmamız lazım."*

Manifest kapsamı **yalnız yerel sunucu**:

```json
"content_scripts": [{
  "matches": ["http://127.0.0.1:8000/cases/*", "http://localhost:8000/cases/*"],
  "js": ["src/harita.js", "src/eslestir.js", "src/klivance.js"]
}]
```

| Yasak | Neden |
|---|---|
| `klivance.com` | Canlı |
| **`klivance.onrender.com`** | ⚠ **TEK Render servisi var** → "staging" adı **canlı veritabanına bakıyor olabilir**. *"Staging gibi duruyor"* yeterli güvence **değil** |

- ⚠ **Kod değişikliği gerekmez, KAPSAM yeterlidir**: content script yüklüyken düğme canlı
  kartta da çıkar ve **gerçek hastaya PDF yükleyebilirdi**.
- **Klivance kaynak kodunda hiçbir değişiklik YOK** — düğmeyi **eklenti enjekte eder**,
  deploy edilecek bir şey yoktur, hekimler bu düğmeyi **görmez**.
- **Karar yorumla değil KAPIYLA çivili:** `test/yol_kapisi_test.js` bölüm 5 manifest'i okur;
  canlı/geniş kapsam eklenirse **KIRMIZI** olur (tatbikatla doğrulandı). Aynı bölüm
  e-Nabız'a **otomatik** content script enjeksiyonunu da yasaklar.
- Başka portta çalışıyorsan manifest'e o portu ekle (`8000` sabit yazılı).

---

## 3. ⚠⚠ BEYAZ LİSTE İLKESİ — link TAKİP EDİLMEZ

**Kara liste yetmez.** İki ölçülmüş kaza ve yolların tam tablosu:
`Klivance_software v101/enabiz-navigasyon.md`.

Eklenti yalnız **önceden bilinen** e-Nabız yollarına gider; sayfadaki linkleri **izlemez**.

---

## 4. ⚠ RIZA KAPISI — kaynağın kendisinde ve SÜREKLİ

Hasta e-Devlet'ten *"doktor bilgilerimi görebilir"* dediği **sürece** hekim görür;
**her an geri alınabilir.**

⚠ **Açık kalem (#442b):** alınan **kopya geri alınamıyor** (KVKK m.7). Rıza geri çekilse
bile hasta kartındaki dosya durur. Memory: `klivance-enabiz-eklentisi`.

---

## 5. ⚠⚠ 2026-09-03 FOUNDER KURALI: PDF KAYNAK DEĞİL, AKORDİYON KAYNAK (#668b)

Founder kendi hesabında **gösterdi**: e-Nabız'ın ürettiği tahlil PDF'inde **bazı değerler
YOK**; akordiyon açılınca satırda (Sonuç / Birim / Referans) **yazıyor**.

> **PDF eksik kopyadır; sayfa HTML'i tam veridir.**

**Kural:**
- Tahlil değerleri **daima HTML'den** (`cikarici.tahlil`) alınır
- PDF **yalnız belge olarak** yüklenir
- PDF'ten değer çıkaran her katman (kokpit lab parser, Haiku çıkarımı) **HTML paketiyle
  kıyaslanır**
- **PDF'te olmayan parametre "yok" sayılmaz**
- ⚠ Hangi parametrelerin eksik olduğu **henüz ÖLÇÜLMEDİ**

### 5.1 Uygulama (2026-09-03, Kamil)

```
klivance.js  PDF yükledikten sonra
   → tahlilPaketiYukle  (cikarici.tahlil çıktısı; ham/pdf/_serbest GÖNDERİLMEZ)
   → POST /api/cases/{cid}/enabiz-tahlil        (saglik/app/enabiz_tahlil.py)
   → normalize (grup başlığı düşer · kart-içi yinelenen dörtlü tek kopya · en yeni kart önce)
   → ŞİFRELİ "belge" olarak app.case_file        (enabiz-tahlil-<gün>.json, application/json)
   → sha256 tekilliği (#619b)
   → extracted_text ÖNCEDEN DOLU → analiz MAP'i ATLAR (0 LLM)
   → REDUCE paketi <dosya_ozetleri>'nde görür → kokpit oradan çizer
```

**Panelde satır:** *"Tahlil paketi (HTML → karta)"* — düşerse **KIRMIZI**.

**Ölçüm:** PDF tabanlı analiz kokpitte **15** lab parametresi çizerken, HTML paketiyle **24**.

⚠ Form/kaydet sözleşmesi **değişmedi** (paket forma yazılmaz, **belge** olarak girer;
hekim silebilir).
⚠ **Yeni çekim eskisini SİLMEZ** — e-Nabız'ın gösterdiği tarih penceresi ölçülmedi;
"üst küme" varsayımı kurulmadı.

Kapı: `scratchpad/enabiz_tahlil_paketi_verify.py`.

---

## 6. Eklenti dosya yapısı

| Dosya | Ne |
|---|---|
| `manifest.json` | MV3 · **kapsam kilidi burada** |
| `panel.html` · `panel.js` | Eklenti popup'ı |
| `src/sw.js` | Service worker (background) |
| `src/content.js` | e-Nabız sayfası content script |
| `src/kesif.js` | Sayfa keşfi (beyaz liste yolları) |
| `src/cikarici.js` | **Veri çıkarımı** — `cikarici.tahlil` burada |
| `src/belge.js` | Belge/PDF indirme |
| `src/harita.js` | Alan haritası |
| `src/eslestir.js` | Klivance hasta kartı eşleştirme |
| `src/klivance.js` | Klivance tarafına yükleme (content script, **yalnız 127.0.0.1:8000**) |

**İzinler:** `activeTab` · `scripting` · `downloads` · `tabs` · `storage`
**Host izinleri:** `https://enabiz.gov.tr/*` · `http://127.0.0.1:8000/*` · `http://localhost:8000/*`

---

## 7. Masaüstü program (Electron)

```json
{ "name": "klivance-enabiz-aktarici", "main": "src/ana/main.js",
  "devDependencies": { "electron": "^44.0.0" } }
```

| Yol | Ne |
|---|---|
| `src/ana/main.js` | Electron ana süreç |
| `src/arayuz/index.html` · `arayuz.js` | Arayüz |
| `Desktop program interface design/` | Tasarım dosyaları + tasarım sistemi (`_ds/`) |

**Kimlik:** masaüstü program Klivance'a **cihaz eşleştirme** (#596b) ile bağlanır →
`POST /api/cihaz/istek` → `POST /api/cihaz/bekle` → `GET /cihaz`
(`saglik/app/cihaz_routes.py`, tablo `app.cihaz_eslesme`).

---

## 8. KAPILAR — `JS_SUITES` (Node ile koşar)

```bash
node scratchpad/enabiz_yol_kapisi_verify.js          # beyaz liste + manifest kapsam kilidi
node scratchpad/enabiz_eslestir_verify.js            # hasta kartı eşleştirme
node scratchpad/enabiz_indirme_verify.js             # belge indirme
node scratchpad/enabiz_cekirdek_tasinabilir_verify.js
node scratchpad/enabiz_kopru_arguman_verify.js
node scratchpad/enabiz_masaustu_guvenlik_verify.js
node scratchpad/enabiz_derin_baglanti_verify.js
node scratchpad/enabiz_giris_liste_koprusu_verify.js
node scratchpad/enabiz_belge_yukleme_verify.js
node scratchpad/enabiz_tahlil_grup_verify.js
node scratchpad/enabiz_grup_sizinti_verify.js
node scratchpad/cihaz_enjekte_js_verify.js
```

Paket içi kısayol:
```bash
cd "Klivance_software v101" && npm run kapi
```

---

## 9. Ayrıntı dosyaları

| Dosya | İçerik |
|---|---|
| `Klivance_software v101/eklenti/README.md` | **Ana referans** — kurulum, ölçüm kaydı, tuzaklar |
| `Klivance_software v101/enabiz-navigasyon.md` | **Navigasyon ölçümü** — yolların tam tablosu, iki kaza |
| `Klivance_software v101/enabiz-masaustu-mimari.md` | Masaüstü mimari |
| `Klivance_software v101/mimari-2-hekim-oturumu.md` | Hekim oturumu mimarisi |
| `Klivance_software v101/eklenti-raporu-2026-08-31.md` | Denetim raporu |
| `Klivance_software v101/uc-envanteri.md` | Uç envanteri |
| memory `klivance-enabiz-eklentisi` | Karar kaydı |

---

## 10. Devralan ekip için — açık riskler

| # | Risk |
|---|---|
| 1 | **Uçtan uca test yapılmadı.** Kod + kapılar yeşil, gerçek akış denenmedi |
| 2 | ⚠ **Kopya geri alınamıyor** (KVKK m.7, #442b) — rıza geri çekilse bile dosya kartta durur |
| 3 | **Chrome Web Store ret vektörleri** değerlendirilmedi (memory'de not var) |
| 4 | **PDF'te eksik olan parametreler ÖLÇÜLMEDİ** — yalnız "PDF eksik" olgusu kanıtlı |
| 5 | e-Nabız'ın **tarih penceresi ölçülmedi** → "üst küme" varsayımı kurulamaz |
| 6 | Deneme kilidi kaldırılırsa **gerçek hastaya yazma riski** doğar — kapıyı önce oku |

---

**Sonraki:** [`15a-FONKSIYON-REFERANSI-APP.md`](15a-FONKSIYON-REFERANSI-APP.md)
