# Klivance — Sistem Envanteri ve Mimari Dokümantasyonu

**Klivance**, Türk hekimler için klinik karar-destek (CDS) SaaS'ıdır. Açık tıbbi literatür ve
kurumsal veriyi tarar, hekime **atıflı ve denetlenebilir** yanıt verir. **Tanı koymaz.**

Bu depo, ürünün geliştirme ve yönetiminin **hiveteams.ai** (Agentic AI SDLC) tarafından
devralınması için hazırlanmış devir paketidir.

---

## 👉 Başlangıç noktası: [`INDEX.md`](INDEX.md)

Tüm dokümanlar oradan ilişkilendirilmiştir. İlk okuma sırası:

1. [`00-URUN-VE-KAPSAM.md`](00-URUN-VE-KAPSAM.md) — ne yapar, ne yapmaz, hangi sınırlar
2. [`01-MIMARI.md`](01-MIMARI.md) — katmanlar, istek yaşam döngüsü
3. [`16-ISLETME-KURALLARI-VE-KARARLAR.md`](16-ISLETME-KURALLARI-VE-KARARLAR.md) — "neden böyle"
4. [`17-BILINEN-TUZAKLAR-VE-BORCLAR.md`](17-BILINEN-TUZAKLAR-VE-BORCLAR.md) — ne bozuk
5. [`18-DEVIR-NOTLARI.md`](18-DEVIR-NOTLARI.md) — nasıl devralınır

---

## Ölçülen büyüklükler

Aşağıdaki sayılar **2026-09-17'de, commit `cb16758d` üzerinde ölçüldü** — tahmin değil.

| Ne | Sayı | Nasıl |
|---|---:|---|
| Python modülü | 140 | `ast` |
| Python satırı | ~69.000 | `wc -l` |
| Fonksiyon (modül düzeyi) | 1.514 | `ast` |
| HTTP rotası | 142 | canlı `app.routes` |
| Veritabanı tablosu | 55 | `information_schema` |
| CI doğrulama kapısı | 320 | `test.yml` |
| Operasyon scripti | 177 | `ast` |
| Docstring hacmi | ~278.000 karakter | `ast.get_docstring` |

---

## Bu dokümantasyonu okurken

- **`⚠⚠` bir sözleşmedir.** Her biri yaşanmış bir hatadan doğdu ve genellikle bir CI
  kapısıyla çivilidir.
- **Sayılar bayatlar.** Karar vereceksen yeniden ölç; her dosya reçetesini verir.
  Fiyat, deneme süresi, günlük tavan ve KB satır sayıları **bilinçli yazılmadı** — tek doğru
  kaynak koddur.
- **Üretilmiş dosyaları elle düzenleme:** `03`, `04`, `15a/b/c` ve `veri/*.json` mekanik
  olarak üretildi (reçete: `18-DEVIR-NOTLARI.md` §6).
- **Bu paketin tavanı var.** Derin alan bilgisi ve ölçüm anlatıları ana depoda kalır.

---

## Makine-okunur ekler

[`veri/`](veri/) altında rota, modül, şema ve middleware envanterleri JSON olarak durur —
agentic araçların doğrudan tüketebileceği biçimde.

---

*İçerik: mimari, rota/şema/fonksiyon envanteri, iş kuralları, açık borçlar ve devir notları.
Sır değeri ve kişisel veri içermez.*
