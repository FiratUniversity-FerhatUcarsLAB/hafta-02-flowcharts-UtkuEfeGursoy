BAŞLA

// --- AÇIKLAMA: VERİ YAPILARI --- // Sistemin çalışması için gerekli olan hasta, doktor ve tahlil gibi temel veri yapıları. YAPI Hasta: TCNo: METİN AdSoyad: METİN BİTİR YAPI

YAPI Tahlil: HastaTC: METİN SonucHazirMi: MANTIKSAL // EVET veya HAYIR SonucDetayi: METİN BİTİR YAPI

// --- AÇIKLAMA: SİSTEM BAŞLANGIÇ VERİLERİ --- // Algoritmanın çalışmasını test etmek için sembolik veritabanları HastaVeritabani = [{TCNo:"11122233344", AdSoyad:"Ali Veli"}] TahlilVeritabani = [{HastaTC:"11122233344", SonucHazirMi:EVET, SonucDetayi:"Kan değerleriniz referans aralığındadır."}]

// ==================================================================== // ANA KISIM - HASTANE OTOMASYON DÖNGÜSÜ // ==================================================================== // --- AÇIKLAMA: Bu ana döngü, ATM/Kiosk gibi, bir sonraki hasta için sürekli olarak çalışır durumda kalır. ANA KISIM: SÜRECE DOĞRU: HastaTC = KULLANICIDAN_AL("Lütfen TC Kimlik Numaranızı giriniz: ") AktifHasta = KimlikDogrula(HastaTC, HastaVeritabani)

  EĞER AktifHasta != null İSE
    EKRANA_YAZ("Hoş geldiniz, " + AktifHasta.AdSoyad)
    IslemMenusu(AktifHasta) // Hastanın kendi işlem döngüsünü başlat
  İSE
    EKRANA_YAZ("Hata: Bu TC Kimlik Numarasına sahip bir hasta bulunamadı.")
  BİTİR EĞER
BİTİR SÜRECE
BİTİR ANA KISIM

// ==================================================================== // PROSEDÜRLER - İŞLEM MANTIKLARI // ====================================================================

// --- AÇIKLAMA: Verilen TC No'yu veritabanında arar. Bulursa hasta bilgilerini, bulamazsa 'null' döndürür. PROSEDÜR KimlikDogrula(TCNo, Veritabani): HER hasta İÇİN Veritabani: EĞER hasta.TCNo == TCNo İSE RETURN hasta // Hasta nesnesini döndür BİTİR EĞER BİTİR HER RETURN null // Bulunamazsa boş döndür BİTİR PROSEDÜR

// --- AÇIKLAMA: Kullanıcının oturumunu yönetir. Kullanıcı çıkış yapana kadar ana menüyü gösterir. PROSEDÜR IslemMenusu(Hasta): OturumAcik = EVET SÜRECE OturumAcik == EVET: EKRANA_YAZ("\n--- ANA MENÜ ---") EKRANA_YAZ("1. Randevu İşlemleri") EKRANA_YAZ("2. Tahlil Sonucu Görüntüle") EKRANA_YAZ("3. Çıkış Yap") Secim = KULLANICIDAN_AL_SAYI("Seçiminiz: ")

  EĞER Secim == 1 İSE RandevuModulu(Hasta)
  EĞER Secim == 2 İSE TahlilModulu(Hasta)
  EĞER Secim == 3 İSE OturumAcik = HAYIR
BİTİR SÜRECE
EKRANA_YAZ("Oturum kapatıldı. İyi günler dileriz.")
BİTİR PROSEDÜR

// --- AÇIKLAMA: MODÜL 1 - RANDEVU ALMA --- // Kullanıcının poliklinik, doktor ve saat seçerek randevu almasını sağlar. PROSEDÜR RandevuModulu(Hasta): EKRANA_YAZ("\n--- RANDEVU ALMA SİSTEMİ ---") Poliklinik = KULLANICIDAN_AL("Lütfen poliklinik seçin (Örn: Dahiliye): ") Doktor = KULLANICIDAN_AL("Lütfen doktor seçin (Örn: Dr. Ahmet Yılmaz): ") Saat = KULLANICIDAN_AL("Lütfen uygun bir saat seçin (Örn: 14:30): ")

EKRANA_YAZ(Poliklinik + " bölümünde, " + Doktor + " için saat " + Saat + "'a randevu alınacaktır.")
Onay = KULLANICIDAN_AL("Onaylıyor musunuz? (Evet/Hayır): ")

EĞER Onay == "Evet" İSE
  // Burada randevu veritabanına kaydedilir.
  EKRANA_YAZ("Randevunuz başarıyla oluşturuldu.")
  EKRANA_YAZ("Bilgilendirme SMS'i telefonunuza gönderildi.")
İSE
  EKRANA_YAZ("Randevu alma işlemi iptal edildi.")
BİTİR EĞER
BİTİR PROSEDÜR

// --- AÇIKLAMA: MODÜL 2 - TAHLİL SONUCU GÖRÜNTÜLEME --- // Hastanın tahlilinin olup olmadığını ve hazır olup olmadığını kontrol eder, sonucu gösterir. PROSEDÜR TahlilModulu(Hasta): EKRANA_YAZ("\n--- TAHLİL SONUÇ SİSTEMİ ---") HastaninTahlili = null HER tahlil İÇİN TahlilVeritabani: EĞER tahlil.HastaTC == Hasta.TCNo İSE HastaninTahlili = tahlil BİTİR EĞER BİTİR HER

EĞER HastaninTahlili != null İSE // Karar: Tahlil var mı?
  EĞER HastaninTahlili.SonucHazirMi == EVET İSE // Karar: Sonuç hazır mı?
    EKRANA_YAZ("--- Tahlil Sonucunuz ---")
    EKRANA_YAZ(HastaninTahlili.SonucDetayi)
    IndirmeOnayi = KULLANICIDAN_AL("Sonucu PDF olarak indirmek ister misiniz? (Evet/Hayır): ")
    EĞER IndirmeOnayi == "Evet" İSE
      EKRANA_YAZ("PDF dosyası indiriliyor...")
    BİTİR EĞER
  İSE
    EKRANA_YAZ("Tahlil sonucunuz henüz hazırlanmamıştır. Lütfen daha sonra tekrar kontrol ediniz.")
  BİTİR EĞER
İSE
  EKRANA_YAZ("Sistemde kayıtlı bir tahlil sonucunuz bulunmamaktadır.")
BİTİR EĞER
BİTİR PROSEDÜR

BİTİR
