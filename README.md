Entegrasyon
===========

#Genel Bilgiler

Express Hizmeti ile kart sahiplerinin e-ticaret işlemlerinde önceden kayıt ettikleri kartları ile kolay, hızlı ve güvenli bir akış sağlamak amaçlanmıştır.

#DOKÜMANIN AMACI VE İÇERİĞİ

Bu dokümanın amacı; BKM Express projesi kapsamında gerçekleşecek alışveriş akışlarında, Üye İşyerleri tarafından gerçekleştirilecek entegrasyona ilişkin hazırlanmış olan webservislerin detaylı açıklamalarının hazırlanmasıdır.
Alışveriş Ödeme Akışına ilişkin detaylı bilgi bu dökümanda yer almaktadır.

#KAPSAM
BKM Express Projesi Alışveriş Ödeme akışları, kullanıcı işyeri sitesinde alışveriş yaparken, adres ve kargo bilgilerini belirledikten sonra kredi kartı bilgisi girerek ödemeyi gerçekleştirecekleri adımda kullanıcı, “BKM Express ile ödeme yapmak istiyorum” seçtiğinde çalıştırılacaktır. Amaç Express’e üye olmuş kullanıcıların alışveriş esnasında Express’e kayıt etmiş oldukları bilgileri kullanarak güvenli ve hızlı bir şekilde ödemelerini gerçekleştirmeleridir. Alışveriş akışlarında webservisler ve redirect yöntemi bir arada kullanılacaktır. Akışlar detaylı olarak bu dökümanda anlatılmaktadır.

#TANIMLAR
BKM: Bankalararası Kart Merkezi
Banka: Issuer Banka
Express: Express Websitesi
OTP: Tek Kullanımlık Şifre (TKŞ)
İşyeri(Merchant): Express’e üye işyerini tanımlar.
Kuştüyü : İşyerinden Sanal Pos (Virtual Pos) parametrelerinin talep edilmediği, bu parametrelerin BKM Express sistemine tanımlanarak “requestMerchInfo” web servisine gerek duyulmayan entegrasyon modelidir. Detaylı bilgiyi dokümanın EK E - Kuştüyü Entegrasyon Modeli bölümünde bulabilirsiniz.
Default Sanal Pos: İşyerinin seçilen karta ait sanal posu bulunmuyorsa işlem tek çekim olarak bu sanal postan gerçekleşir.

#MÜŞTERİ DENEYİMİ
Express hizmetinden yararlanmak isteyen kart sahiplerinin öncelikle sistemde hesap
açmaları ve kart eklemeleri gerekmektedir. Kullanıcılar, Express’i destekleyen issuer
bankalara ait kartlarını ekleyebilmektedir.
BKM, kart bilgilerinin tamamını saklamamakta sadece müşteri adı/soyadı/TCKN’si, kartının
ilk 6/son 4 hanesini saklamaktadır.
Kart sahipleri alışveriş esnasında “Express” ile ödeme seçeneğini seçtikten sonra daha önce
eklediği kartını seçerek veya işlem anında yeni bir kart ekleyerek işlemini yapabilmektedir.
BKM alışveriş sırasında, müşterinin seçtiği karta ait bankaya bağlanarak kart bilgilerinin
tamamını alır. Bu arada banka müşteriyi kendisinde kayıtlı cep telefonuna doğrulama mesajı
(tek kullanımlık şifre) göndererek doğrular. Doğrulamayı başarıyla geçen kart sahibine ait
bilgiler sanal POS’a aktırılır ve işlem sonucu alınır. İşlem sonucu BKM tarafından işyeri ile
paylaşılır.
