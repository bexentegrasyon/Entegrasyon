#EK C - XML İle Paket Kullanılmadan Entegrasyon
Entegrasyon Başlangıcı<br>
Entegrasyona başlamanız için ilk olarak işyeri kaydınızın yapılması ve mid değerinin size
ulaşmış olması gerekmektedir.
Aynı zamanda 1024 veya 2048 boyutunda bir public key yaratıp bunu BKM’ye iletmeniz
gerekmektedir.
Public key yaratmak için, Unix ortamında, Openssl ile;
openssl genrsa -out mykey.pem 1024
openssl rsa -in mykey.pem -pubout > mykey.pub
komutlarını kullanabilirsiniz.

Webservis Tanımları
initializePayment Servisi için hazırlanan webservis tanımları aşağıda yer almaktadır.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5504655/146c6c74-8790-11e4-964e-4002e2abc3e0.png)



