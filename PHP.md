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
<b>function readKeyFromFile($filename){</b>


