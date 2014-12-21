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

<b> ÖNEMLİ NOT: Imzalama ve doğrulama metodlarının çalışabilmesi için </b>

<b>1. Kendi private-public key çiftinizi üretmiş olmanız ve public keyinizi pem formatında BKM ile
   paylasmış olmanız. </b><br>
<b>2. BKM nin public keyini pem formatında elde etmiş olmanız gereklidir.</b><br>

<b>Bu anahtarlar kütüphanede mykey.pem → private key, bkm.pub → bkmnin public keyi
olarak geçmektedir. bkm.pub sonrada projeye eklenmelidir.</b>

Bu nesnenin işlemi başlatması için initPayment metodunun çağrılması gereklidir. Bu
metoda işlem için gerekli wsdl dosyasının bulunduğu yer veya linki ve bankalara ve kartlara
yapılacak taksitlendirme için gerekli olan “banka dizisi” gönderilmelidir. Bu dizi için gerekli
olan sınıflar “BkmExpressPaymentService.php” dosyasında bulunmaktadır.

Daha sonra gelen cevaptan RedirectModel nesnesi oluşturulur ve redirection
sayfasına yönlendirilir. Redirection yapacak olan sayfa “initPurchaseRedirect.php” dir.
Redirection yapılacak olan sayfanın adresi bunun dışında merchant tarafından
güncellenmelidir. Bu değer test_lib.php dosyasının en altında header metoduna
verilmektedir. (redirect yapılmalıdır, <b>iFrame desteklenmemektedir</b>)


##Mobil İşyeri Uygulamasından Yönlendirme

Mobil uygulamalar tarayıcıları POST metoduyla açamamaktadır. Bu sebeple yukarıda anlatılan
yönlendirme gerçekleştirilememektedir. Bunun yerine üye işyerinin mobil uygulaması, işyeri
sunucusundan aldığı token değerini BKM Mobil uygulamasının önceden kendilerine iletilmiş
olan URL’i ile birleştirerek açmalıdır.

1. Kullanıcı mobil işyeri uygulamasından ödeme yapma seçeneğine gelir.

2. Bu noktada BKM Express ile ödeme seçeneğini seçer.

3. İşyeri mobil uygulaması, kendi sunucusuna istekte bulunur ve ödemenin BKM Express ile
yapılacağını belirtir.

4. İşyeri sunucusu, aldığı istek doğrultusunda BKM Express’e işlem başlatma isteği
gönderir.(InitPayment) Bu istekte, optional alanlar doldurulur.

5. BKM Express işlem isteğini işler, optional alanları yorumlayarak, işlemi kendi
veritabanında mobil işlem olarak işaretler.

6. Cevabı alan sunucu, kendi uygulamasına işlemin tokenını iletir.
7. 
7. Üye işyeri uygulaması , aldığı token ile birlikte kendisine önceden iletilmiş olan BKM
Mobil uygulaması URL’ini (bkmurl://payment.token?XXX
) birleştirerek uygulamanın
açılıp açılamayacağını kontrol eder. Uygulamayı açabiliyorsa açar (bu durumda 8. Ve 9.
adımlara gelinmez, işlem bitiminde mobil success veya cancel url’e dönülür), açamıyorsa
Safari tarayıcısında BKM Express sunucusunun yönlendirmeyi yapacak olan sayfasını açar.
Bu sayfaya tokenı gönderir.

8. BKM Express sayfası token ile birlikte işlem bilgilerine erişir ve gerekli yönlendirmeyi
yapar.

9. BKM Express gelen tokendan işlem tipine bakar. Mobil uygulamadan veya Mobil
tarayıcıdan gelmiş ise, BKM Mobil uygulamasını açmaya çalışır, açamazsa uygulama
indirme linkine yönlendirir.

