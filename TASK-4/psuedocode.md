BAŞLA

// --- AÇIKLAMA: GELİŞMİŞ VERİ YAPILARI --- // Sistemin tüm gereksinimlerini (şifre, GPA, ön koşul, zamanlama vb.) karşılamak üzere güncellenmiş yapılar. YAPI Ogrenci: OgrenciNo: METİN SifreHash: METİN AdSoyad: METİN GPA: ONDALIKLI SAYI MaksimumKredi: SAYI // Sabit: 35 AldigiKrediler: SAYI TamamlananDersler: LİSTE // Örn: ["MAT101", "FZK101"] KayitliDersler: LİSTE // DersKodu ve Durum içerir. Örn: [{DersKodu:"BLG101", Durum:"Onaylandı"}] BİTİR YAPI

YAPI Ders: DersKodu: METİN DersAdi: METİN Kredi: SAYI Kontenjan: SAYI OnKosullar: LİSTE // Örn: ["MAT101"] Zamanlama: METİN // Örn: "Pzt 10:00-12:00" BİTİR YAPI

// --- AÇIKLAMA: SİSTEM BAŞLANGIÇ VERİLERİ --- // Algoritmanın çalışmasını test etmek için sembolik veritabanları OgrenciVeritabani = [{OgrenciNo:"20251101", SifreHash:"abc123hash", AdSoyad:"Ayşe Yılmaz", GPA:2.4, MaksimumKredi:35, AldigiKrediler:0, TamamlananDersler:["MAT101"], KayitliDersler:[]}] DersVeritabani = [ {DersKodu:"BLG201", DersAdi:"Veri Yapıları", Kredi:5, Kontenjan:50, OnKosullar:["BLG101"], Zamanlama:"Salı 13:00-15:00"}, {DersKodu:"BLG101", DersAdi:"Programlamaya Giriş", Kredi:5, Kontenjan:1, OnKosullar:[], Zamanlama:"Pzt 10:00-12:00"}, {DersKodu:"FZK102", DersAdi:"Fizik II", Kredi:4, Kontenjan:75, OnKosullar:["FZK101"], Zamanlama:"Pzt 10:00-12:00"} ]

// ==================================================================== // ANA KISIM - SİSTEM GİRİŞ DÖNGÜSÜ // ==================================================================== ANA KISIM: OgrenciNo = KULLANICIDAN_AL("Öğrenci No: ") Sifre = KULLANICIDAN_AL("Şifre: ") AktifOgrenci = KimlikDogrula(OgrenciNo, Sifre, OgrenciVeritabani)

EĞER AktifOgrenci != null İSE
  EKRANA_YAZ("Hoş geldiniz, " + AktifOgrenci.AdSoyad)
  IslemMenusu(AktifOgrenci)
İSE
  EKRANA_YAZ("Hata: Öğrenci No veya Şifre hatalı.")
BİTİR EĞER
BİTİR ANA KISIM

// ==================================================================== // PROSEDÜRLER // ====================================================================

// --- AÇIKLAMA: Öğrenci No ve şifreyi doğrular. Başarılı ise öğrenci nesnesini, değilse 'null' döndürür. PROSEDÜR KimlikDogrula(No, GirilenSifre, Veritabani): HER ogrenci İÇİN Veritabani: EĞER ogrenci.OgrenciNo == No VE Hash(GirilenSifre) == ogrenci.SifreHash İSE RETURN ogrenci BİTİR EĞER BİTİR HER RETURN null BİTİR PROSEDÜR

// --- AÇIKLAMA: Öğrencinin oturumunu yönetir. Çıkış yapana kadar ana menüyü sunar. PROSEDÜR IslemMenusu(Ogrenci): OturumAcik = EVET SÜRECE OturumAcik == EVET: EKRANA_YAZ("\n--- DERS KAYIT SİSTEMİ ---") EKRANA_YAZ("1. Açılan Dersleri Listele") EKRANA_YAZ("2. Derse Kayıt Ol") EKRANA_YAZ("3. Kayıtlı Derslerimi Görüntüle") EKRANA_YAZ("4. Ders Bırak") EKRANA_YAZ("5. Çıkış Yap") Secim = KULLANICIDAN_AL_SAYI("Seçiminiz: ")

  EĞER Secim == 1 İSE DersleriListele(DersVeritabani)
  EĞER Secim == 2 İSE DerseKayitOl(Ogrenci, DersVeritabani)
  EĞER Secim == 3 İSE KayitliDersleriGoster(Ogrenci)
  EĞER Secim == 4 İSE DersBirak(Ogrenci)
  EĞER Secim == 5 İSE OturumAcik = HAYIR
BİTİR SÜRECE
EKRANA_YAZ("Oturum kapatıldı.")
BİTİR PROSEDÜR

// --- AÇIKLAMA: Veritabanındaki tüm dersleri, kalan kontenjan bilgisiyle birlikte listeler. PROSEDÜR DersleriListele(Veritabani): EKRANA_YAZ("\n--- AÇILAN DERSLER ---") HER ders İÇİN Veritabani: EKRANA_YAZ(ders.DersKodu + " - " + ders.DersAdi + " | Kredi: " + ders.Kredi + " | Kalan Kontenjan: " + KalanKontenjanHesapla(ders)) BİTİR HER BİTİR PROSEDÜR

// --- AÇIKLAMA: Derse kayıt olma işleminin tüm kontrollerini zincirleme olarak yapan ana prosedür. PROSEDÜR DerseKayitOl(Ogrenci, Dersler): DersKodu = KULLANICIDAN_AL("Kayıt olmak istediğiniz dersin kodunu girin: ") SecilenDers = VeritabanindanDersBul(DersKodu, Dersler)

// --- ONAYLAMA ŞELALESİ BAŞLANGICI ---
EĞER SecilenDers != null İSE // Karar 1: Ders mevcut mu?
  EĞER OgrenciBuDerseKayitliMi(Ogrenci, SecilenDers) == HAYIR İSE // Karar 2: Zaten kayıtlı mı?
    EĞER OnkosullariSaglıyorMu(Ogrenci, SecilenDers) == EVET İSE // Karar 3: Ön koşullar tamam mı?
      EĞER ZamanCakismasiVarMi(Ogrenci, SecilenDers) == HAYIR İSE // Karar 4: Zaman çakışması var mı?
        EĞER KontenjanVarMi(SecilenDers) == EVET İSE // Karar 5: Kontenjan var mı?
          EĞER KrediLimitiYeterliMi(Ogrenci, SecilenDers) == EVET İSE // Karar 6: Kredi limiti yeterli mi?
            
            // --- FİNAL KARARI: DANIŞMAN ONAYI GEREKLİLİĞİ ---
            EĞER Ogrenci.GPA < 2.5 İSE // Karar 7: GPA 2.5'ten küçük mü?
              Ogrenci.KayitliDersler.EKLE({DersKodu: SecilenDers.DersKodu, Durum:"Onay Bekliyor"})
              EKRANA_YAZ("Başarılı! GPA'nız 2.5 altında olduğu için ders danışman onayına gönderildi.")
            İSE
              Ogrenci.KayitliDersler.EKLE({DersKodu: SecilenDers.DersKodu, Durum:"Onaylandı"})
              Ogrenci.AldigiKrediler = Ogrenci.AldigiKrediler + SecilenDers.Kredi
              EKRANA_YAZ("'" + SecilenDers.DersAdi + "' dersine kayıt başarıyla tamamlandı.")
            BİTİR EĞER

          İSE EKRANA_YAZ("Hata: Kredi limitiniz (35) bu dersi almaya yetmiyor.")
          BİTİR EĞER
        İSE EKRANA_YAZ("Hata: Dersin kontenjanı doludur.")
        BİTİR EĞER
      İSE EKRANA_YAZ("Hata: Seçtiğiniz dersin zamanı, kayıtlı olduğunuz başka bir dersle çakışıyor.")
      BİTİR EĞER
    İSE EKRANA_YAZ("Hata: Bu dersin ön koşul derslerini tamamlamamışsınız.")
    BİTİR EĞER
  İSE EKRANA_YAZ("Hata: Bu derse zaten kayıtlısınız veya dersiniz onay bekliyor.")
  BİTİR EĞER
İSE EKRANA_YAZ("Hata: Bu koda sahip bir ders bulunamadı.")
BİTİR EĞER
BİTİR PROSEDÜR

// --- AÇIKLAMA: Öğrencinin kayıtlı olduğu dersleri ve durumlarını listeler. PROSEDÜR KayitliDersleriGoster(Ogrenci): EKRANA_YAZ("\n--- KAYITLI DERSLERİNİZ ---") EĞER Ogrenci.KayitliDersler BOŞ İSE EKRANA_YAZ("Şu anda kayıtlı olduğunuz bir ders bulunmuyor.") İSE HER kayitliDers İÇİN Ogrenci.KayitliDersler: EKRANA_YAZ(kayitliDers.DersKodu + " - Durum: " + kayitliDers.Durum) BİTİR HER BİTİR EĞER BİTİR PROSEDÜR

// --- AÇIKLAMA: Öğrencinin seçtiği bir dersi kayıtlı dersler listesinden çıkarır. PROSEDÜR DersBirak(Ogrenci): DersKodu = KULLANICIDAN_AL("Bırakmak istediğiniz dersin kodunu girin: ") // ... Ders bırakma kontrol ve silme mantığı buraya gelir ... EKRANA_YAZ("'" + DersKodu + "' dersi bırakma işlemi tamamlandı.") BİTİR PROSEDÜR

BİTİR
