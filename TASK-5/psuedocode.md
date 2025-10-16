BAŞLA

// --- AÇIKLAMA: VERİ YAPILARI VE DURUM DEĞİŞKENLERİ --- // Sistemin anlık durumunu (aktif mi, ev sahibi evde mi, alarm çalıyor mu) // ve anlık sensör verilerini tutan temel yapılar. YAPI SistemDurumu: AktifMi: MANTIKSAL EvSahibiEvdeMi: MANTIKSAL AlarmCalıyorMu: MANTIKSAL BİTİR YAPI

YAPI SensorVerileri: HareketAlgilandi: MANTIKSAL KapiPencereAcildi: MANTIKSAL BİTİR YAPI

// --- AÇIKLAMA: SİSTEM BAŞLANGIÇ DURUMU --- // Sistemin ilk başladığında hangi durumda olacağını tanımlar. MevcutDurum = {AktifMi:EVET, EvSahibiEvdeMi:HAYIR, AlarmCalıyorMu:HAYIR}

// ==================================================================== // ANA KISIM - SÜREKLİ İZLEME DÖNGÜSÜ (SİSTEMİN KALP ATIŞI) // ==================================================================== // --- AÇIKLAMA: Bu ana döngü, sistemin enerjisi kesilene kadar sonsuza dek çalışır. // Ana görevi, sistemi sürekli olarak izlemek ve bir olay olduğunda ilgili prosedürü tetiklemektir. ANA KISIM: SÜRECE DOĞRU: EĞER MevcutDurum.AktifMi == EVET İSE // Sensörleri sürekli olarak oku OkunanVeri = SensorleriOku()

    // Karar: Herhangi bir sensör tetiklendi mi?
    EĞER OkunanVeri.HareketAlgilandi == EVET VEYA OkunanVeri.KapiPencereAcildi == EVET İSE
      // Karar: Alarm zaten çalmıyorsa, karışıklığı önlemek için alarm sürecini başlat
      EĞER MevcutDurum.AlarmCalıyorMu == HAYIR İSE
        AlarmSureciniBaslat(OkunanVeri)
      BİTİR EĞER
    BİTİR EĞER
  İSE
    // Sistem 'Kurulu Değil' durumundaysa, işlem gücü harcamamak için kısa bir süre bekler.
    BEKLE(5 saniye)
  BİTİR EĞER
BİTİR SÜRECE
BİTİR ANA KISIM

// ==================================================================== // PROSEDÜRLER - İŞLEM MANTIKLARI // ====================================================================

// --- AÇIKLAMA: Alarm tetiklendiğinde çalışan, tüm kontrolleri zincirleme olarak yapan ana prosedür. PROSEDÜR AlarmSureciniBaslat(Veri): EKRANA_YAZ("Sensör tetiklendi. Kontroller başlatılıyor...")

// Karar 1: Yanlış Alarm Kontrolü (Ev Sahibi Evde mi?)
EĞER MevcutDurum.EvSahibiEvdeMi == HAYIR İSE
  EKRANA_YAZ("Ev sahibi evde değil. Alarm süreci devam ediyor.")
  KameralariAktiveEt()
  
  // Bekle ve Tekrar Kontrol Et Döngüsü
  EKRANA_YAZ("Nihai teyit için 15 saniye bekleniyor...")
  BEKLE(15 saniye)
  YeniOkunanVeri = SensorleriOku()
  
  // Karar 2: Tehdit Hala Devam Ediyor mu? (Nihai Teyit)
  EĞER YeniOkunanVeri.HareketAlgilandi == EVET VEYA YeniOkunanVeri.KapiPencereAcildi == EVET İSE
    EKRANA_YAZ("Tehdit teyit edildi. Alarm ve bildirimler başlatılıyor.")
    MevcutDurum.AlarmCalıyorMu = EVET
    
    // Karar 3: Alarm Seviyesi Belirleme
    AlarmSeviyesi = 0
    EĞER Veri.KapiPencereAcildi == EVET İSE
      AlarmSeviyesi = 3 // Yüksek Öncelik
    İSE EĞER Veri.HareketAlgilandi == EVET İSE
      AlarmSeviyesi = 2 // Orta Öncelik
    BİTİR EĞER
    
    AlarmiCal(AlarmSeviyesi)
    BildirimGonder("SMS", AlarmSeviyesi)
    BildirimGonder("Uygulama Bildirimi", AlarmSeviyesi)
    BildirimGonder("E-posta", AlarmSeviyesi)
    
    // Alarm Sıfırlama veya Devam Ettirme Döngüsü
    SÜRECE MevcutDurum.AlarmCalıyorMu == EVET:
      KullaniciKomutu = KomutBekle() // Ev sahibinden gelen 'Sıfırla' komutunu dinle
      EĞER KullaniciKomutu == "Sıfırla" İSE
        MevcutDurum.AlarmCalıyorMu = HAYIR
        AlarmiDurdur()
        EKRANA_YAZ("Alarm ev sahibi tarafından sıfırlandı.")
      BİTİR EĞER
    BİTİR SÜRECE
    
  İSE
    EKRANA_YAZ("İlk tetikleme teyit edilmedi. Anlık bir durum olabilir. Alarm iptal edildi.")
  BİTİR EĞER
  
İSE
  EKRANA_YAZ("Ev sahibi evde. Yanlış alarm. Olay sadece kaydedildi.")
BİTİR EĞER
BİTİR PROSEDÜR

// --- AÇIKLAMA: YARDIMCI PROSEDÜRLER (Donanım ve Servisleri Temsil Eder) ---

PROSEDÜR SensorleriOku(): // Bu prosedür, fiziksel sensörlerden (PIR, manyetik kontak vb.) anlık veri okur. // Okunan verilere göre bir SensorVerileri nesnesi oluşturup döndürür. RETURN {HareketAlgilandi:HAYIR, KapiPencereAcildi:HAYIR} // Örnek dönüş BİTİR PROSEDÜR

PROSEDÜR KameralariAktiveEt(): EKRANA_YAZ("Kameralar aktive edildi ve kayıt başlatıldı.") BİTİR PROSEDÜR

PROSEDÜR AlarmiCal(Seviye): EKRANA_YAZ("Alarm Seviye " + Seviye + " ile çalıştırıldı!") BİTİR PROSEDÜR

PROSEDÜR AlarmiDurdur(): EKRANA_YAZ("Alarm susturuldu.") BİTİR PROSEDÜR

PROSEDÜR BildirimGonder(Kanal, Seviye): EKRANA_YAZ("'" + Kanal + "' üzerinden Seviye " + Seviye + " alarm bildirimi gönderildi.") BİTİR PROSEDÜR

PROSEDÜR KomutBekle(): // Bu prosedür, ev sahibinin mobil uygulamasından veya başka bir arayüzden // gelen komutları (örn: "Sıfırla") dinler. RETURN "" // Komut yoksa boş döner BİTİR PROSEDÜR

BİTİR
