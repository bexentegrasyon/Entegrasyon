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


###Mobil İşyeri Uygulamasından Yönlendirme

Mobil uygulamalar tarayıcıları POST metoduyla açamamaktadır. Bu sebeple yukarıda anlatılan
yönlendirme gerçekleştirilememektedir. Bunun yerine üye işyerinin mobil uygulaması, işyeri
sunucusundan aldığı token değerini BKM Mobil uygulamasının önceden kendilerine iletilmiş
olan URL’i ile birleştirerek açmalıdır.

1- Kullanıcı mobil işyeri uygulamasından ödeme yapma seçeneğine gelir.

2- Bu noktada BKM Express ile ödeme seçeneğini seçer.

3- İşyeri mobil uygulaması, kendi sunucusuna istekte bulunur ve ödemenin BKM Express ile
yapılacağını belirtir.

4- İşyeri sunucusu, aldığı istek doğrultusunda BKM Express’e işlem başlatma isteği
gönderir.(InitPayment) Bu istekte, optional alanlar doldurulur.

5- BKM Express işlem isteğini işler, optional alanları yorumlayarak, işlemi kendi
veritabanında mobil işlem olarak işaretler.

6- Cevabı alan sunucu, kendi uygulamasına işlemin tokenını iletir.
 
7- Üye işyeri uygulaması , aldığı token ile birlikte kendisine önceden iletilmiş olan BKM
Mobil uygulaması URL’ini (bkmurl://payment.token?XXX
) birleştirerek uygulamanın
açılıp açılamayacağını kontrol eder. Uygulamayı açabiliyorsa açar (bu durumda 8. Ve 9.
adımlara gelinmez, işlem bitiminde mobil success veya cancel url’e dönülür), açamıyorsa
Safari tarayıcısında BKM Express sunucusunun yönlendirmeyi yapacak olan sayfasını açar.
Bu sayfaya tokenı gönderir.

8- BKM Express sayfası token ile birlikte işlem bilgilerine erişir ve gerekli yönlendirmeyi
yapar.

9- BKM Express gelen tokendan işlem tipine bakar. Mobil uygulamadan veya Mobil
tarayıcıdan gelmiş ise, BKM Mobil uygulamasını açmaya çalışır, açamazsa uygulama
indirme linkine yönlendirir.


Bundan sonra akış BKM Express'e geçer. Burada kart seçimi sonrası OTP doğrulaması
yapıldıktan sonra akış tekrar merchant tarafına web servis çağrısı olarak geçer. Burada diğer
entegrasyon dökümanında anlatıldığı gibi requestMerchInfo web servisi çağırılır. BKM
Expresse verilen service url inde RequestMerchInfoServer.php ın yer aldığı teyit edilmelidir.
Bu dosyadaki RequestMerchInfoServiceImpl sınıfı gerekli metodu handle etmektedir.


Ayrıca bu dosyada yer alan RequestMerchInfoService_latest.wsdl
dosyası da web servisin konumuna göre güncellenmelidir. Bu web servis sınıfında bundan başka dikket
edilmesi gereken iki nokta var:

1- Seçilen bankaya göre belirlenecek sanal pos ve mpi bilgileri<br>
Burada kullanılan sınıfın adı MerchantInfoActionImpl. Bu sınıfın<br>
public function getVirtualPos($bankId){ <br>
imzalı metodu sanal pos bilgisini dönüyor. Bu metod şu anda dummy datalar
dönüyor.

<b>Önemli Not: Her merchantın bu metodu kendi yapısına göre düzenlemesi gerekiyor.</b>


2- BKM Express ten gelen isteğin imzasının doğrulanması

<b>Önemli Not: imzalama ve doğrulama metodlarının çalışabilmesi için PHP nin
Openssl desteğinin açık olması gerekmektedir. Ayrıca son versiyonların kullanılmasına dikkat
edilmesi gerekli, eski versiyon php ve openssllerde hash metodlarının çalışmadığı tespit
edilmiştir.</b>


BKM tarafından gerekli public keyler elde edilene kadar test işlemlerine
devam edebilmek için imza doğrulanması kısmı comment out edilmiş durumda. Bunun da
uygun koşullar oluşunca commentlerinin kaldırılması gerekiyor.

Sonuçların dönmesi konusunda, ilk başta initPayment çağrısında gönderilen success
url ve cancel urller geçerli. Bunlara karşılık olarak kütüphanede success.php ve fail.php
olarak iki dosya oluşturuldu. Bu sayfalarda ilk olarak gelen tüm parametreler ekrana
var_dump yapılıyor. Daha sonra da ilgili IncomingResultModel sınıfının nesnesine çevriliyor.
Buradan sonrasını da her merchant kendi isteğine göre kodlamaya devam edebilir.

###Confirmation URL

BKM Express sisteminde, sanal pos işlemlerinde başarılı sonucun işyerine ulaştırılması
amacıyla 2 yöntem paralel kullanılmaktadır. Bunlardan akışa göre ilki, işyeri tarafından
sağlanan confirmation URL’ye BKM Express sistemi tarafından bildirim yapılmasıdır.
İşyerlerinin bu bildirimi alabilmek üzere confirmationURL çalışmasını yapması, ilk
entegrasyonlarda isteğe bağlı bırakılmakla birlikte, bu dokümanın 2.8 sürümü ve 19 Kasım
2013 tarihi itibariyle tüm işyerleri için <b>mecburi</b> hale getirilmiştir.

BKM, işyeri tarafından açılacak confirmationURL adresine success URL’ye dönüş
yapacağı alanları aynı formatta <b>post edecektir.</b>

İşyeri, confirmationURL’ye yapılan bildirimleri, BKM Express’ten geşdiğini succes URL
tarafı ile aynı şekilde kontrol edecektir (imza kontrolü v.b.). Eğer bildirim gerçekten BKM
Express tarafından yapılıyor ve SanalPOS’tan onay alındığını bildiriyorsa, bu adımda işyerinin
ürünü müştriye sağlayacak adımları başlatması gerekir (müşterinin email ile bilgilendirmesi,
işlem statü güncellemesi, stok hareketi v.b.)

Eğer bir sonraki adımda açıklanacak şekilde kullanıcı redirect ile işyerine giderken bir
sorun yaşarsa, zaten confirmationURL adımında ürün sağlama başlayacağı için, kullanıcı

1- İşyeri sayfasına yeniden girip “Siparişlerim” adımına baktığında, siparişinin
onaylandığını görmelidir

2- Aynı şekilde işyeri email ile bildirim de yapıyorsa, kullanıcı email ile siparişin başarılı
gerçekleştiğini görebiliyor olmalıdır.


###İşyerine Başarılı İşlem Sonucu Yönlendirme

Express ödeme akışının başarıyla tamamlandığı durumlarda express sisteminin işyerine
yaptığı geri yönlendirmedir. “initializePayment İstek Parametreleri” içinde alınan sUrl’e
aşağıdaki bilgiler http formu içerisinde hidden alan olarak post edilecektir.

Eğer işlem mobil uygulamadan geliyor ise mobil uygulamanın successURL’ine sadece token
iletilecek uygulama da bu token ile kendi serverına daha önceden iletilmiş (İşyerinin
confirmationURL adresine bir önceki adımda yapılan bildirimdir) olan işlem sonucunu
sorabilecektir. Eğer işlem web üzerinden yapılıyorsa, confirmationURL adresine de bildirilmiş
olan standart tüm alanlar, redirect ile de işyerine tekrar bildirilmektedir.

Bilgiler “Güvenlik” bölümünde ve aşağıda belirtilen imzalama yöntemi dikkate alınarak
imzalanacaktır. Ödeme akışının herhangi bir nedenle başarısız olması sonucunda kullanıcı,
“İşyerine İptal için Yönlendirme” yöntemi ile işyeri’ne yönlendirilir.

Önemli Not: confirmationURL adresine BKM Express tarafından yapılan bildirim başarılı ve
imza kontrolünden geçmiş ise , redirect ile yapılan bildirimde imzanın tutmaması, sipariş iptali
için yeterli neden değildir. Ancak bu tip bir redirect’te işyeri sayfasında hata gösterilmesi
önerilir.


###BKM – İşyeri Arasındaki Mesaj İmzalama Ve İmza Doğrulama

1- Satış başlatma web servis çağrısı için imzalama<br>

Bu özelliğin kullanılması için temel olarak utilities.php dosyasındaki sign
metodu kullanılır.

Bu metodun imzası :<br>
<b>function sign($data, $privateKey)</b>

Burada $data parametresi imzalanacak olan veri, $privateKey ise merchantın
kendine ait olan rsa şifresinin private kısmıdır. Projede örnek olarak mykem.pem dosyası
bulunmaktadır. Bu dosyanın içeriği gene utililties.php de yeralan<br>
<b>function readKeyFromFile($filename){ </b> <br>
metodu ile okunabilir.

Bilindiği gibi işlem başlatma web servis çağrısının imzalanması için parametrelerinin de sırayla
birleştirilmesi gerekiyor.Bu işlem için de ayrıca InitPaymentInterface.php dosyasındaki
initPaymentActionImpl sınıfının prepareHash metodundan yararlanılabilir.

public function prepareHash($params)

2- Sanal pos bilgi web servisi için imzalama ve doğrulama<br>

İmzalama özelliğinin kullanılabilmesi için RequestMerchInfoInterface.php
dosyasında yer alan MerchantInfoActıonImpl örnek sınıfı kullanılmalıdır. Bu sınıftaki
public function signResponse($response){ <br>
metoduna parametre olarak response nesnesi geçirildiğinde geriye imzalanmış ve base64
için encode edilmiş string dönülmektedir.

Dogrulama içinse aynı sınıfın<br>
public function verifyRequest($signature, $requestObj){ 

imzalı metodu kullanılmaldır. Burada metodun bkm.pub isimli yine pem formatında BKM
Express in public keyini içeren bir dosya okuduğuna dikkat edilmelidir. Dilendiği takdirde
dosyanın adı değiştirilebilir.

###Projedeki Dosyaların Listesi Ve Kullanım Amaçları

1- BkmExpressPaymentService.php : initPayment web servis çağrısında kullanılması gereken
    tüm sınıfların bulunduğu dosya.
    
2- InitPaymentInterface.php : initPayment web servisinin çağrılması için gerekli operasyon
    sınıfının bulunduğu dosya.
    
3- RedirectData.php : initPayment çağrısı sonrasında gelen dataların, hidden form şeklinde
    post edilmesi için kullanılan ara data sınıfının olduğu dosya. Bu dosyada ayrıca işlem
    sonucunda gelen sonuç datasının da tutulacağı sınıfın tanımı yer alıyor.
    
4- RequestMerchInfoInterface.php : Bu dosyada VirtualPos yardımcı sınıfı tanımlanmıs.
    Ayrıca BKM den geri dönüş kodları için de tanımlamalar mevcut. Ayrıca imzalama ve
    doğrulama işlerini yapan ve her merchant tarafından pos bilgilerini aldığı metodu yeniden
    yazılması gereken MerchantInfoActionImpl sınıfı da burada yer alıyor.
    
5- RequestMerchInfoServer.php : requestMerchInfo web servis metodunun handle edildiği
    dosya
    
6- RequestMerchInfoService.php : requestMerchInfo web servisi için gerekli tüm sınıfların
    tanımlandığı dosya
    
7- fail.php : başarısız sonucun döndüğü dosya

8- success.php : başarılı sonucun döndüğü dosya

9- initPurchaseRedirect.php : hidden form ile işlemin BKM Expresse aktarıldığı dosya

10-requestMerchInfo_client_test.php : requestMerchInfo servisini dogrudan test etmek için hazırlanmış test dosyası

11- test_lib.php : Tüm işlem akışının testinin başlatıldığı dosya

12- utilities.php : imzalama, doğrulama, anahtar okuma gibi işlevlerin yer aldığı dosya.

###İşlem Bilgileri Sorgulama

İşyeri, işlem için iletilen token bilgisini kullanarak, işlemin durumunu ve gerçekleşmişse
ödeme bilgilerini, BKM Express’in bu servisi aracılığıyla sorgulayabilir
Bu dokümanın 2.8 sürümü ve 19 Kasım 2013 tarihi itibariyle işyerlerinin İşlem Bilgileri
Servisini kullanabilmeleri <b>mecburidir.</b>

Sorgulama servisi, özellikle işyerlerinin geçici olarak yaşayabilecekleri sorun sonrasında,
aslında ödemesi gerçekleşen ancak ürün sağlanmayan işlemlerin tespiti için kullanılacaktır.
İşyeri sorgulama yaptığında ödemesinin alındığını tespit ettiği işlemler için ilgili ürünleri
müşteriye sağlamak. ile yükümlüdür.

Mevcut InitPayment servisi içerisindeki “QueryPaymentResult” metodu yardımıyla
İşyeri, istekte bulunarak işlem sonuç bilgilerine ulaşılabilmektedir.

Gönderilecek değerler aşağıdaki şekildedir.<br>
mId= merchantID,<br>
s= imza { mId , t , ts }<br>
t= token,<br>
ts= timestamp<br>

<b>** Bu alanlardan imza alanının diğer alanların birbiri ardına eklenmesi ve SHA256
ile oluşturulan hash değerinin mac verify yöntemi ile doğrulanması gerekmektedir.
Bu işlem için üye işyerleri BKM Express in RSA public keyini kullanmalıdır.</b>

Servisin cevap parametreleri ise aşağıdaki şekildedir.<br>
t= token<br>
ts= timestamp<br>
s= imza {t, success, ts}<br>
success = işlem sonucu , boolean,<br>

<b>** Bu alanlardan imza alanının diğer alanların birbiri ardına eklenmesi ve SHA256
ile oluşturulan hash değerinin mac verify yöntemi ile doğrulanması gerekmektedir.
Bu işlem için üye işyerleri kendi RSA public keyini kullanmalıdır.</b>


##Sıkça Sorulan Sorular

<b>Site üzerinden BKM’ye yönlenmeye çalıştığımda istek mesajım BKM’ye ulaşmıyor.
Ne yapmalıyım?</b>

İstek mesajları, bize aşağıdaki nedenlerden ulaşmıyor olabilir

- “telnet preprod.bkmexpress.com.tr 9620” yapabildiğinizi
- Web.config dosyasında doğru endpoint adresini verdiğinizi
- BKM’ye ilettiğiniz public key ile sizin referans verdiğiniz public key değerinin aynı
olduğunu<br>
  Web.config dosyalarında key path’lerinin doğru verildiğini
- Web.config ‘in bulunduğu dosyaya okuma hakkınızın olduğunu
kontrol etmelisiniz.

<b>Preprod ve Production için endpoint adresleri nelerdir?</b>

- Preprod için :<br>
https://preprod.bkmexpress.com.tr:9620/BKMExpress/BkmExpressPaymentService.ws<br>
- Prod için :<br>
https://www.bkmexpress.com.tr:9620/BKMExpress/BkmExpressPaymentService.ws<br>

<b>Public key uzunluğu ne olmalıdır?</b>

Public key uzunluğunuz 1024 yada 2048 olmalıdır.

<b>Doğrulama kodu cep telefonuma gelmiyor. Ne yapmalıyım?</b>

Preprod ortamı için doğrulama kodu alanına, “123456” girebilirsiniz.

<b>3D Secure üyesi değilim. Webserviste MPI bilgilerini ne göndermeliyim.</b>

3D Secure desteklemiyor iseniz, webservis çağrısında bu alanları bize boş göndermelisiniz.

<b>Web servis çağrısı yapıyorum fakat “Unknown Error” hatası ile karşılaşıyorum.</b>

Burada kontrol etmeniz gereken bir kaç nokta mevcut : <br>
- Merchant olarak kaydınızı BKM’ye yaptırdınız mı? Bu kaydın sonunda size bir
“Merchant Id” tanımlanmış olmalı, yaptığınız istekte bu alanı kullanıyor musunuz?

- Dökümanın başında yazılı talimatlardan olan private-public key çiftinizi ürettiniz mi?<br>
&nbsp; &nbsp;  o Ürettiyseniz public keyinizi BKM ile paylaştınız mı?<br>
&nbsp; &nbsp;  o Public keyinizi paylaştıysanız, BKM’nin keyinizi kaydettiğini doğrular mısınız?<br>

- Yukarıdaki iki seçenekte de sorun yoksa bu sefer BKM’den, sizin için hata loglarını
incelemelerini istemelisiniz.

<b>Web servis çağrısı yapıyorum ama “Request Not Synchronized” hatası alıyorum.</b>

Bu hatayı alıyor olmanızın sebebi; BKM’nin sunucuları ile sizin sunucularınız
arasındaki zaman farkının kabul edilebilir düzeyin üstünde olmasıdır.<br>
Bu hatayı önlemek için; sunucu zaman ayarınızı NTP sunucularından almanız gerekmektedir.<br>
Örnek olarak; asia.pool.ntp.org üzerinden alabilirsiniz.

<b>Key üretmede sıkıntı yaşıyorum</b>

Windows bazlı, .Net geliştirme ortamına sahip sistem kullanıcıları, BKM’nin, sizlere
sağladığı “Key Generator” uygulamasını kullanabilirsiniz.

Unix bazlı sistemler için daha önce size iletilen komutları kullanabilirsiniz.
