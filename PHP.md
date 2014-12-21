#EK B - PHP Paketi İle Entegrasyon

##Entegrasyon Başlangıcı

Entegrasyona başlamanız için ilk olarak işyeri kaydınızın yapılması ve mid değerinin size
ulaşmış olması gerekmektedir.<br>

Aynı zamanda 1024 veya 2048 boyutunda bir public key yaratıp bunu BKM’ye iletmeniz
gerekmektedir.<br>

Public key yaratmak için, Unix ortamında, Openssl ile;<br>

&nbsp; &nbsp; openssl genrsa -out mykey.pem 1024<br>
&nbsp; &nbsp; openssl rsa -in mykey.pem -pubout > mykey.pub<br>

komutlarını kullanabilirsiniz.

##Entegrasyon Kütüphanesi Kullanımı

###Baştan Sona Entegrasyon
Baştan sonra entegrasyon işlemi kütüphanede örnekle gösterilmektedir. test_lib.php
dosyasında bir test işlemi başlatılması yazılmıştır. Burada initPaymentActionImpl sınıfından
bir nesne üretilmiş ve bu nesne gerekli değerler ile construct edilmiştir. Bu değerler
işlemden işleme ve merchanttan merchanta değişeceği için, kütüphaneyi kullanan
merchantlar tarafından güncellenmelidir.

<b> ÖNEMLİ NOT: Imzalama ve doğrulama metodlarının çalışabilmesi için 

1. Kendi private-public key çiftinizi üretmiş olmanız ve public keyinizi pem formatında BKM ile
   paylasmış olmanız <br>
2. BKM nin public keyini pem formatında elde etmiş olmanız gereklidir.<br>

Bu anahtarlar kütüphanede mykey.pem → private key, bkm.pub → bkmnin public keyi
olarak geçmektedir. bkm.pub sonrada projeye eklenmelidir.</b>

Bu nesnenin işlemi başlatması için initPayment metodunun çağrılması gereklidir. Bu
metoda işlem için gerekli wsdl dosyasının bulunduğu yer veya linki ve bankalara ve kartlara
yapılacak taksitlendirme için gerekli olan “banka dizisi” gönderilmelidir. Bu dizi için gerekli
olan sınıflar “BkmExpressPaymentService.php” dosyasında bulunmaktadır.
