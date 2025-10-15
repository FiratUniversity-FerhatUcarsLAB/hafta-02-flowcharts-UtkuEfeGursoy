// ========================================================================
// ALGORİTMA: AltinIslemli_ATM_Sistemi (4 Deneme Haklı)
// AMAÇ: Standart bankacılık işlemlerine ek olarak, kullanıcının
//       hesabından anlık kur ile altın satın alabilmesini sağlayan
//       güvenli bir ATM yazılımı mantığı oluşturmak.
// ========================================================================

BAŞLA

  // --------------------------------------------------------------------
  // 1. TEMEL VERİ YAPILARI
  // --------------------------------------------------------------------
  YAPI MusteriHesabi:
    KartID: METİN
    SifreHash: METİN // Güvenlik için şifre açık metin olarak tutulmaz
    Bakiye: ONDALIKLI SAYI
  BİTİR YAPI

  // Sembolik bir veritabanı
  Veritabani = [ MusteriHesabi1, MusteriHesabi2, ... ]

  // --------------------------------------------------------------------
  // 2. ANA SİSTEM DÖNGÜSÜ
  // --------------------------------------------------------------------
  ANA KISIM:
    SÜRECE TRUE: // ATM sürekli çalışır durumda
      EKRANA_YAZ("Lütfen kartınızı takınız.")
      GirilenKartID = KART_OKUYUCUDAN_OKU()
      
      Hesap = VeritabanindanHesapBul(GirilenKartID)
      
      EĞER Hesap BULUNDUYSA:
        DogrulamaBasarili = KimlikDogrula(Hesap)
        EĞER DogrulamaBasarili == TRUE:
          IslemMenusu(Hesap) // Kullanıcıyı işlem menüsüne yönlendir
        BİTİR EĞER
      DEĞİLSE:
        EKRANA_YAZ("Geçersiz kart. Kart iade ediliyor.")
      BİTİR EĞER
      
      EKRANA_YAZ("İyi günler dileriz. Lütfen kartınızı alınız.")
    BİTİR SÜRECE
  BİTİR ANA KISIM

  // --------------------------------------------------------------------
  // 3. YARDIMCI PROSEDÜRLER
  // --------------------------------------------------------------------

  PROSEDÜR KimlikDogrula(Hesap):
    SifreDenemeHakki = 4 // Deneme hakkı 4 olarak ayarlandı.
    SÜRECE SifreDenemeHakki > 0:
      GirilenSifre = KULLANICIDAN_AL("Lütfen 4 haneli şifrenizi girin: ")
      GirilenSifreHash = HashFonksiyonu(GirilenSifre) // Girilen şifreyi hash'le
      
      EĞER GirilenSifreHash == Hesap.SifreHash:
        RETURN TRUE // Başarılı giriş
      DEĞİLSE:
        SifreDenemeHakki = SifreDenemeHakki - 1
        EKRANA_YAZ("Hatalı şifre. Kalan deneme hakkı: " + SifreDenemeHakki)
      BİTİR EĞER
    BİTİR SÜRECE
    
    EKRANA_YAZ("Şifre 4 kez hatalı girildi. Güvenlik nedeniyle kartınız bloke edilmiştir.")
    RETURN FALSE // Başarısız giriş
  BİTİR PROSEDÜR

  PROSEDÜR IslemMenusu(Hesap):
    OturumAcik = TRUE
    SÜRECE OturumAcik:
      EKRANA_YAZ("\n--- ANA MENÜ ---")
      EKRANA_YAZ("1. Bakiye Görüntüle")
      EKRANA_YAZ("2. Para Çek")
      EKRANA_YAZ("3. Para Yatır")
      EKRANA_YAZ("4. Altın Satın Al")
      EKRANA_YAZ("5. Çıkış")
      Secim = KULLANICIDAN_AL_SAYI("Lütfen yapmak istediğiniz işlemi seçin: ")

      EĞER Secim == 1: BakiyeGoster(Hesap)
      DEĞİLSE EĞER Secim == 2: ParaCek(Hesap)
      DEĞİLSE EĞER Secim == 3: ParaYatir(Hesap)
      DEĞİLSE EĞER Secim == 4: AltinSatinAl(Hesap)
      DEĞİLSE EĞER Secim == 5:
        OturumAcik = FALSE // Döngüyü sonlandır
      DEĞİLSE:
        EKRANA_YAZ("Geçersiz seçim.")
      BİTİR EĞER
    BİTİR SÜRECE
  BİTİR PROSEDÜR

  PROSEDÜR BakiyeGoster(Hesap):
    EKRANA_YAZ("Mevcut Bakiyeniz: " + Hesap.Bakiye + " TL")
  BİTİR PROSEDÜR

  PROSEDÜR ParaCek(Hesap):
    Miktar = KULLANICIDAN_AL_SAYI("Çekmek istediğiniz tutarı girin: ")
    EĞER Miktar <= Hesap.Bakiye:
      Hesap.Bakiye = Hesap.Bakiye - Miktar
      PARA_BOLMESINDEN_VER(Miktar)
      EKRANA_YAZ("İşlem başarılı. Yeni bakiyeniz: " + Hesap.Bakiye + " TL")
    DEĞİLSE:
      EKRANA_YAZ("Yetersiz bakiye.")
    BİTİR EĞER
  BİTİR PROSEDÜR
  
  PROSEDÜR ParaYatir(Hesap):
      Miktar = PARA_YATIRMA_BOLMESINDEN_OKU()
      Hesap.Bakiye = Hesap.Bakiye + Miktar
      EKRANA_YAZ("İşlem başarılı. Yeni bakiyeniz: " + Hesap.Bakiye + " TL")
  BİTİR PROSEDÜR

  PROSEDÜR AltinSatinAl(Hesap):
    AnlikGramFiyati = DIŞ_SERVİSTEN_ALTIN_FİYATI_GETİR()
    EKRANA_YAZ("Anlık Altın Fiyatı (1 Gram): " + AnlikGramFiyati + " TL")
    
    MiktarGram = KULLANICIDAN_AL_SAYI("Kaç gram altın almak istersiniz?: ")
    ToplamTutar = MiktarGram * AnlikGramFiyati
    
    EKRANA_YAZ(MiktarGram + " gram altın için ödenecek tutar: " + ToplamTutar + " TL. Onaylıyor musunuz? (E/H)")
    Onay = KULLANICIDAN_AL()
    
    EĞER Onay == "E" VEYA Onay == "e":
      EĞER ToplamTutar <= Hesap.Bakiye:
        Hesap.Bakiye = Hesap.Bakiye - ToplamTutar
        ALTIN_BOLMESINDEN_VER(MiktarGram)
        EKRANA_YAZ("Altın alım işlemi başarılı. Yeni bakiyeniz: " + Hesap.Bakiye + " TL")
      DEĞİLSE:
        EKRANA_YAZ("Yetersiz bakiye. İşlem iptal edildi.")
      BİTİR EĞER
    DEĞİLSE:
      EKRANA_YAZ("İşlem iptal edildi.")
    BİTİR EĞER
  BİTİR PROSEDÜR

BİTİR
