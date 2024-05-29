# Bank-Account-Management-System-Project

import sqlite3

class Urun:
    def __init__(self, tablo_adi):
        self.tablo_adi = tablo_adi
        self.conn = sqlite3.connect('restaurant.db')
        self.cursor = self.conn.cursor()
        self.create_table()

    def create_table(self):
        self.cursor.execute(f'''
            CREATE TABLE IF NOT EXISTS {self.tablo_adi} (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                isim TEXT NOT NULL,
                fiyat REAL NOT NULL
            )
        ''')
        self.conn.commit()

    def urun_ekle(self, isim, fiyat):
        self.cursor.execute(f"INSERT INTO {self.tablo_adi} (isim, fiyat) VALUES (?, ?)", (isim, fiyat))
        self.conn.commit()
        print(f"{isim} eklendi.")

    def urun_sil(self, urun_id):
        self.cursor.execute(f"DELETE FROM {self.tablo_adi} WHERE id=?", (urun_id,))
        self.conn.commit()
        print(f"{urun_id} ID'li ürün silindi.")

    def urun_guncelle(self, urun_id, isim, fiyat):
        self.cursor.execute(f"UPDATE {self.tablo_adi} SET isim=?, fiyat=? WHERE id=?", (isim, fiyat, urun_id))
        self.conn.commit()
        print(f"{urun_id} ID'li ürün güncellendi.")

    def urunleri_listele(self):
        self.cursor.execute(f"SELECT id, isim, fiyat FROM {self.tablo_adi}")
        urunler = self.cursor.fetchall()
        print(f"\n***** {self.tablo_adi.upper()} *****")
        for urun in urunler:
            print(f"ID: {urun[0]}, İsim: {urun[1]}, Fiyat: {urun[2]} TL")


class Yemek(Urun):
    def __init__(self):
        super().__init__('yemekler')


class Icecek(Urun):
    def __init__(self):
        super().__init__('icecekler')


class Siparisler:
    def __init__(self):
        self.conn = sqlite3.connect('restaurant.db')
        self.cursor = self.conn.cursor()
        self.create_table()

    def create_table(self):
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS siparisler (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                yemek_id INTEGER NOT NULL,
                icecek_id INTEGER NOT NULL,
                yemek_adet INTEGER NOT NULL,
                icecek_adet INTEGER NOT NULL,
                toplam_fiyat REAL NOT NULL,
                FOREIGN KEY (yemek_id) REFERENCES yemekler(id),
                FOREIGN KEY (icecek_id) REFERENCES icecekler(id)
            )
        ''')
        self.conn.commit()

    def siparis_ver(self, yemek_id, icecek_id, yemek_adet, icecek_adet):
        yemek_fiyat = self.get_urun_fiyati('yemekler', yemek_id)
        icecek_fiyat = self.get_urun_fiyati('icecekler', icecek_id)
        toplam_fiyat = (yemek_fiyat * yemek_adet) + (icecek_fiyat * icecek_adet)

        self.cursor.execute("INSERT INTO siparisler (yemek_id, icecek_id, yemek_adet, icecek_adet, toplam_fiyat) VALUES (?, ?, ?, ?, ?)",
                            (yemek_id, icecek_id, yemek_adet, icecek_adet, toplam_fiyat))
        self.conn.commit()

        print(f"{yemek_adet} adet {self.get_urun_adi('yemekler', yemek_id)} ve {icecek_adet} adet {self.get_urun_adi('icecekler', icecek_id)} "
              f"siparişi verildi. Toplam Tutar: {toplam_fiyat} TL")

    def siparisleri_goster(self):
        self.cursor.execute('''
            SELECT siparisler.id, yemekler.isim AS yemek, icecekler.isim AS icecek, 
            siparisler.yemek_adet, siparisler.icecek_adet, siparisler.toplam_fiyat
            FROM siparisler
            INNER JOIN yemekler ON siparisler.yemek_id = yemekler.id
            INNER JOIN icecekler ON siparisler.icecek_id = icecekler.id
        ''')
        siparisler = self.cursor.fetchall()

        if not siparisler:
            print("Henüz sipariş bulunmamaktadır.")
        else:
            print("\n***** SİPARİŞLER *****")
            for siparis in siparisler:
                print(f"ID: {siparis[0]}, Yemek: {siparis[1]}, İçecek: {siparis[2]}, "
                      f"Yemek Adet: {siparis[3]}, İçecek Adet: {siparis[4]}, Toplam Fiyat: {siparis[5]} TL")

    def get_urun_fiyati(self, tablo, urun_id):
        self.cursor.execute(f"SELECT fiyat FROM {tablo} WHERE id=?", (urun_id,))
        fiyat = self.cursor.fetchone()
        if fiyat:
            return fiyat[0]
        else:
            print("Ürün bulunamadı.")
            return 0

    def get_urun_adi(self, tablo, urun_id):
        self.cursor.execute(f"SELECT isim FROM {tablo} WHERE id=?", (urun_id,))
        isim = self.cursor.fetchone()
        if isim:
            return isim[0]
        else:
            print("Ürün bulunamadı.")
            return ""


class RestaurantYonetimSistemi:
    def __init__(self):
        self.yemekler = Yemek()
        self.icecekler = Icecek()
        self.siparisler = Siparisler()
        
    def siparis_ver(self):
        self.yemekler.urunleri_listele()
        yemek_id = int(input("Sipariş vermek istediğiniz yemeğin ID'sini girin: "))
        yemek_adet = int(input("Yemek adetini girin: "))

        self.icecekler.urunleri_listele()
        icecek_id = int(input("Sipariş vermek istediğiniz içeceğin ID'sini girin: "))
        icecek_adet = int(input("İçecek adetini girin: "))

        self.siparisler.siparis_ver(yemek_id, icecek_id, yemek_adet, icecek_adet)

    def menu_goster(self):
        while True:
            print("\n***** RESTAURANT MENÜ *****")
            print("1. Yemek Ekle")
            print("2. İçecek Ekle")
            print("3. Yemek Sil")
            print("4. İçecek Sil")
            print("5. Yemek Güncelle")
            print("6. İçecek Güncelle")
            print("7. Yemekleri Göster")
            print("8. İçecekleri Göster")
            print("9. Siparişleri Göster")
            print("10. Sipariş Ver")
            print("0. Çıkış")

            secim = input("Lütfen bir seçenek girin: ")

            if secim == "1":
                self.yemekler.urun_ekle(input("Yemek ismi: "), float(input("Yemek fiyatı: ")))
            elif secim == "2":
                self.icecekler.urun_ekle(input("İçecek ismi: "), float(input("İçecek fiyatı: ")))
            elif secim == "3":
                self.yemekler.urunleri_listele()
                self.yemekler.urun_sil(int(input("Silmek istediğiniz yemeğin ID'sini girin: ")))
            elif secim == "4":
                self.icecekler.urunleri_listele()
                self.icecekler.urun_sil(int(input("Silmek istediğiniz içeceğin ID'sini girin: ")))
            elif secim == "5":
                self.yemekler.urunleri_listele()
                urun_id = int(input("Güncellemek istediğiniz yemeğin ID'sini girin: "))
                self.yemekler.urun_guncelle(urun_id, input("Yeni yemek ismi: "), float(input("Yeni yemek fiyatı: ")))
            elif secim == "6":
                self.icecekler.urunleri_listele()
                urun_id = int(input("Güncellemek istediğiniz içeceğin ID'sini girin: "))
                self.icecekler.urun_guncelle(urun_id, input("Yeni içecek ismi: "), float(input("Yeni içecek fiyatı: ")))
            elif secim == "7":
                self.yemekler.urunleri_listele()
            elif secim == "8":
                self.icecekler.urunleri_listele()
            elif secim == "9":
                self.siparisler.siparisleri_goster()
            elif secim == "10":
                self.siparis_ver()
            elif secim == "0":
                break
            else:
                print("Geçersiz seçenek! Tekrar deneyin.")


if __name__ == "__main__":
    restaurant = RestaurantYonetimSistemi()
    restaurant.menu_goster()
