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
