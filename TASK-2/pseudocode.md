BAŞLA

// --- AÇIKLAMA: VERİ YAPILARI ---
// Bu bölümde, sistemimizde kullanacağımız temel veri türlerini tanımlıyoruz.
// Urun yapısı, mağazadaki her bir ürünü; SepetUrunu ise kullanıcının sepetine eklediği bir ürünü temsil eder.

YAPI Urun:
ID: SAYI
Ad: METİN
Fiyat: ONDALIKLI SAYI
Stok: SAYI
IndirimVarMi: MANTIKSAL // EVET veya HAYIR
BİTİR YAPI

YAPI SepetUrunu:
Urun: Urun NESNESİ
Miktar: SAYI
BİTİR YAPI

// --- AÇIKLAMA: SİSTEM BAŞLANGIÇ VERİLERİ ---
// Algoritmanın çalışmasını test etmek için sembolik bir ürün veritabanı ve boş bir kullanıcı sepeti oluşturuyoruz.
UrunVeritabani = [
{ID:1, Ad:"Akıllı Telefon", Fiyat:15000.0, Stok:10, IndirimVarMi:EVET},
{ID:2, Ad:"Kablosuz Kulaklık", Fiyat:1200.0, Stok:25, IndirimVarMi:HAYIR},
{ID:3, Ad:"Laptop Çantası", Fiyat:750.0, Stok:5, IndirimVarMi:EVET}
]
KullaniciSepeti = BOŞ LİSTE (SepetUrunu nesneleri içerecek)

// ====================================================================
// ANA KISIM - KULLANICI ETKİLEŞİM DÖNGÜSÜ
// ====================================================================
// --- AÇIKLAMA: Bu ana döngü, kullanıcı çıkış yapana kadar sürekli çalışır ve ana menüyü gösterir.
ANA KISIM:
Calisiyor = EVET
SÜRECE Calisiyor == EVET:
EKRANA_YAZ("\n--- ONLINE MAĞAZA MENÜSÜ ---")
EKRANA_YAZ("1. Ürünleri Listele")
EKRANA_YAZ("2. Sepete Ürün Ekle")
EKRANA_YAZ("3. Sepeti Görüntüle")
EKRANA_YAZ("4. Ödeme Yap")
EKRANA_YAZ("5. Çıkış")
Secim = KULLANICIDAN_AL_SAYI("Seçiminiz: ")

  EĞER Secim == 1 İSE UrunleriListele(UrunVeritabani)
  EĞER Secim == 2 İSE SepeteEkle(UrunVeritabani, KullaniciSepeti)
  EĞER Secim == 3 İSE SepetiGoster(KullaniciSepeti)
  EĞER Secim == 4 İSE OdemeYap(KullaniciSepeti)
  EĞER Secim == 5 İSE Calisiyor = HAYIR
BİTİR SÜRECE
EKRANA_YAZ("Mağazamızdan ayrıldınız. İyi günler!")
BİTİR ANA KISIM

// ====================================================================
// PROSEDÜRLER - İŞLEM MANTIKLARI
// ====================================================================

// --- AÇIKLAMA: Bu prosedür, veritabanındaki tüm ürünleri bir döngü kullanarak ekrana listeler.
PROSEDÜR UrunleriListele(Veritabani):
EKRANA_YAZ("\n--- MAĞAZADAKİ ÜRÜNLER ---")
HER urun İÇİN Veritabani:
EKRANA_YAZ("ID: " + urun.ID + " | Ad: " + urun.Ad + " | Fiyat: " + urun.Fiyat + " TL | Stok: " + urun.Stok)
BİTİR HER
BİTİR PROSEDÜR

// --- AÇIKLAMA: Bu prosedür, kullanıcının seçtiği ürünü, stok kontrolü gibi karar yapıları kullanarak sepete ekler.
PROSEDÜR SepeteEkle(Veritabani, Sepet):
UrunID = KULLANICIDAN_AL_SAYI("Eklemek istediğiniz ürünün ID'sini girin: ")
SecilenUrun = null
HER urun İÇİN Veritabani:
EĞER urun.ID == UrunID İSE SecilenUrun = urun
BİTİR HER

EĞER SecilenUrun != null İSE
  EĞER SecilenUrun.Stok > 0 İSE
    Miktar = KULLANICIDAN_AL_SAYI("Kaç adet eklemek istersiniz?: ")
    EĞER Miktar <= SecilenUrun.Stok İSE
      YeniSepetUrunu = YENİ SepetUrunu
      YeniSepetUrunu.Urun = SecilenUrun
      YeniSepetUrunu.Miktar = Miktar
      Sepet.EKLE(YeniSepetUrunu)
      SecilenUrun.Stok = SecilenUrun.Stok - Miktar // Stoktan düş
      EKRANA_YAZ("'" + SecilenUrun.Ad + "' sepete eklendi.")
    İSE
      EKRANA_YAZ("Hata: Stokta yeterli miktarda ürün yok.")
    BİTİR EĞER
  İSE
    EKRANA_YAZ("Hata: Bu ürünün stoğu tükenmiştir.")
  BİTİR EĞER
İSE
  EKRANA_YAZ("Hata: Bu ID'ye sahip bir ürün bulunamadı.")
BİTİR EĞER
BİTİR PROSEDÜR

// --- AÇIKLAMA: Bu prosedür, sepette ürün olup olmadığını bir karar yapısıyla kontrol eder ve varsa bir döngü ile listeler.
PROSEDÜR SepetiGoster(Sepet):
EKRANA_YAZ("\n--- ALIŞVERİŞ SEPETİNİZ ---")
EĞER Sepet BOŞ İSE
EKRANA_YAZ("Sepetiniz şu anda boş.")
İSE
HER sepetUrunu İÇİN Sepet:
EKRANA_YAZ(sepetUrunu.Miktar + " x " + sepetUrunu.Urun.Ad + " - " + (sepetUrunu.Miktar * sepetUrunu.Urun.Fiyat) + " TL")
BİTİR HER
BİTİR EĞER
BİTİR PROSEDÜR

// --- AÇIKLAMA: Bu prosedür, sepetin toplamını hesaplar. Bu sırada bir döngü içinde her ürün için indirim olup olmadığını bir karar yapısıyla kontrol eder.
PROSEDÜR OdemeYap(Sepet):
EĞER Sepet BOŞ İSE
EKRANA_YAZ("Ödeme yapmak için sepetinizde ürün bulunmalıdır.")
RETURN
İSE
AraToplam = 0.0
IndirimToplami = 0.0
HER sepetUrunu İÇİN Sepet:
SatirToplami = sepetUrunu.Urun.Fiyat * sepetUrunu.Miktar
AraToplam = AraToplam + SatirToplami

    // **İNDİRİM KONTROLÜ KARAR YAPISI**
    EĞER sepetUrunu.Urun.IndirimVarMi == EVET İSE
      IndirimMiktari = SatirToplami * 0.10 // %10 indirim uygula
      IndirimToplami = IndirimToplami + IndirimMiktari
      EKRANA_YAZ("'" + sepetUrunu.Urun.Ad + "' için " + IndirimMiktari + " TL indirim uygulandı!")
    BİTİR EĞER
  BİTİR HER
  
  GenelToplam = AraToplam - IndirimToplami
  EKRANA_YAZ("\n--- ÖDEME ÖZETİ ---")
  EKRANA_YAZ("Ara Toplam: " + AraToplam + " TL")
  EKRANA_YAZ("Uygulanan İndirim: " + IndirimToplami + " TL")
  EKRANA_YAZ("Ödenecek Tutar: " + GenelToplam + " TL")
  
  Onay = KULLANICIDAN_AL("Ödemeyi onaylıyor musunuz? (Evet/Hayır): ")
  EĞER Onay == "Evet" İSE
    EKRANA_YAZ("Ödeme başarıyla tamamlandı. Teşekkür ederiz!")
    Sepet.TEMİZLE() // Sepeti boşalt
  İSE
    EKRANA_YAZ("Ödeme işlemi iptal edildi.")
  BİTİR EĞER
BİTİR EĞER
BİTİR PROSEDÜR

BİTİR
