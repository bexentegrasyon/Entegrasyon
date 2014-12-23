#EK C - XML İle Paket Kullanılmadan Entegrasyon
##Entegrasyon Başlangıcı<br>
Entegrasyona başlamanız için ilk olarak işyeri kaydınızın yapılması ve mid değerinin size ulaşmış olması gerekmektedir.
Aynı zamanda 1024 veya 2048 boyutunda bir public key yaratıp bunu BKM’ye iletmeniz gerekmektedir.
Public key yaratmak için, Unix ortamında, Openssl ile;<br>
openssl genrsa -out mykey.pem 1024 <br>
openssl rsa -in mykey.pem -pubout > mykey.pub<br>  komutlarını kullanabilirsiniz.

##Webservis Tanımları <br>
initializePayment Servisi için hazırlanan webservis tanımları aşağıda yer almaktadır.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5504655/146c6c74-8790-11e4-964e-4002e2abc3e0.png)

##İşyeri Ödeme
Kullanıcının express ekranına yönlendirilerek ödeme yapabilmesi için işyeri tarafından gerekli verilerin iletildiği adımdır.

###initializePayment Adımı
Kullanıcının işyerinden express sistemine yönlendirilirken işyeri tarafından çağırılacak olan metoddur. İşlemin başarılı, başarısız gerçekleşme durumunda yönlendirilecek URL bilgileri ile son tutarı ve tüm taksit seçeneklerini içermelidir.

Dönüş parametreleri içerisinde, BKM tarafından üretilecek olan token ve işyerinin express’e yönlendirme yapacağı URL bilgisi iletilir.
Taksit seçeneği yer almayacak ve tek ödeme yapılacaksa;

- Her banka için taksit sayısı nofInst = 1 olarak iletilecektir.

Bankalar için ayrı ayrı bilgi gönderilmeyecek ve tüm bankalar için tek tutar iletilecekse;
- id = 9999 olarak iletilecektir.
Tüm tutarlar, Türk Lirası (TRY) cinsinden ve virgülden sonra 2 basamak kuruş eklenerek (Örn:
“105,99”, “85,00") iletilecektir.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5515955/4693d3f2-888c-11e4-981c-d33782b643cc.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5515943/043f1dae-888c-11e4-9c82-e84f5d8bb9a3.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5515976/11385394-888d-11e4-8afb-086022472692.jpg)
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:bkm="http://www.bkmexpress.com.tr">
  <soapenv:Header/>
  <soapenv:Body>
    <bkm:initializePayment>
      <bkm:initializePaymentWSRequest>
        <mId>7f85e133-3cd2-4821-9bf7-eb0bcb55cbb3</mId>
        <sUrl>http://.../odemeBasarili.jsp</sUrl>
        <cUrl>http://.../odemeBasarisiz.jsp</cUrl>
        <sAmount>78,99</sAmount>
        <cAmount>5,00</cAmount>
        <instOpts>
          <!--1 or more repetitions:-->
          <bank>
            <id>9999</id>
            <name>BankaIsmi</name>
            <expBank>Banka firsatlarini kaçirmayin!</expBank>
            <bins>
              <!--1 or more repetitions:-->
              <bin>
                <value>444411</value>
                <insts>
                  <!--1 or more repetitions:-->
                  <inst>
                    <nofInst>1</nofInst>
                    <amountInst>80,99</amountInst>
                    <cAmount>5,00</cAmount>
                    <tAmount>80,99</tAmount>
                    <cPaid1stInst>true</cPaid1stInst>
                    <expInst>Pesin ödemelerde 3 TL indirim</expInst>
                  </inst>
                  <inst>
                    <nofInst>2</nofInst>
                    <amountInst>42,00</amountInst>
                    <cAmount>5,00</cAmount>
                    <tAmount>84,00</tAmount>
                    <cPaid1stInst>false</cPaid1stInst>
                    <expInst/>
                  </inst>
                </insts>
              </bin>
            </bins>
          </bank>
        </instOpts>
        <ts>20110729-15:05:23</ts>
        <s>IWijxQjUrcXBYoCei4QxjWo9Kg8D3p9tlWoT4t0/gyTE96639In0FZFY2/rvP+/bMJ01EArmKZsR5VW3rwoPxw=</s>
      </bkm:initializePaymentWSRequest>
    </bkm:initializePayment>
  </soapenv:Body>
</soapenv:Envelope>

```
<b>-Şekil 6 Örnek İstek-<b/>

```xml
İmza oluşturmak için alanların birleştirilmesi örneği (Java Kodu)
StringBuffer sb = new StringBuffer();‬
sb.append(mId).append(sUrl).append(cUrl).append(sAmount).append(cAmount);‬
for(Bank aBank : bank)
{‬
    sb.append(aBank.getId()).append(aBank.getName())
                            .append(aBank.getExpBank());‬
for(Bin bin : aBank.getBin())
{‬
    sb.append(bin.getValue());‬
for(Installment inst : bin.getInst())
{‬
sb.append(inst.getNofInst()).append(inst.getAmountInst())
                            .append(inst.getcAmount())‬
                            .append(inst.gettAmount())
                            .append(inst.iscPaid1stInst())
                            .append(inst.getExpInst());‬
}‬
}‬
}‬
sb.append(ts);‬
return sb.toString();
```

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516081/e16dca22-8891-11e4-9025-c8351cf28faa.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516103/eac877c4-8892-11e4-9332-31b54ec00dbd.jpg)

&lt;S:Envelope xmlns:soapenv=&quot;http://schemas.xmlsoap.org/soap/envelope/&quot;&gt;<br>
&lt;S:Body&gt;<br>
&lt;ns2:initializePaymentResponse xmlns:ws=&quot;http://merchant.ws.expressbkm.com.tr/&quot;&gt;<br>
&lt;InitializePaymentWSResponse&gt;<br>
&lt;t&gt;186005784745383&lt;/t&gt;<br>
&lt;url&gt;http://bkmexpress/islemBasarili.bkm&lt;/url&gt;<br>
&lt;ts&gt;20110729-15:05:23&lt;/ts&gt;<br>
&lt;s&gt;PHijxQjUrcXBYoCei4QxjWo9Kg8D3p9tlWoT4t0/gyTE96639In0FZFY2/rvP+/bMsd1EArmKZsR5VW3rwoPxw=&lt;/s&gt;<br>
&lt;res&gt;<br>
&lt;resCode&gt;0&lt;/resCode&gt;<br>
&lt;resMsg&gt;Success&lt;/resMsg&gt;<br>
&lt;resDet/&gt;<br>
&lt;/res&gt;<br>
&lt;/InitializePaymentWSResponse&gt;<br>
&lt;/ns2:initializePaymentResponse&gt;<br>
&lt;/S:Body&gt;<br>
&lt;/S:Envelope&gt;<br>
<b>-Şekil 7 Örnek Dönüş-</b>

Bu adım sonrasında kullanıcı Express sistemine yönlendirilir (redirect yapılmalıdır, <b>iFrame desteklenmemektedir</b>)
“initializePayment Dönüş parametreleri” içinde alınan URL’e, token ile “Güvenlik” bölümünde
belirtilen imzalama yöntemi dikkate alınarak gerçekleştirilir.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516134/be71bd8c-8894-11e4-8ef8-82df7937b6d3.jpg)

###Mobil İşyeri Uygulamasından Yönlendirme
Mobil uygulamalar tarayıcıları POST metoduyla açamamaktadır. Bu sebeple yukarıda anlatılan
yönlendirme gerçekleştirilememektedir. Bunun yerine üye işyerinin mobil uygulaması, işyeri
sunucusundan aldığı token değerini BKM Mobil uygulamasının önceden kendilerine iletilmiş
olan URL’i ile birleştirerek açmalıdır.<br>

1-Kullanıcı mobil işyeri uygulamasından ödeme yapma seçeneğine gelir.<br>

2- Bu noktada BKM Express ile ödeme seçeneğini seçer.<br>

3- İşyeri mobil uygulaması, kendi sunucusuna istekte bulunur ve ödemenin BKM Express ile
yapılacağını belirtir.<br>

4- İşyeri sunucusu, aldığı istek doğrultusunda BKM Express’e işlem başlatma isteği
gönderir.(InitPayment) Bu istekte, optional alanlar doldurulur.<br>

5- BKM Express işlem isteğini işler, optional alanları yorumlayarak, işlemi kendi
veritabanında mobil işlem olarak işaretler.<br>

6- Cevabı alan sunucu, kendi uygulamasına işlemin tokenını iletir.<br>

7- Üye işyeri uygulaması , aldığı token ile birlikte kendisine önceden iletilmiş olan BKM
Mobil uygulaması URL’ini (bkmurl://payment.token?XXX
) birleştirerek uygulamanın açılıp açılamayacağını kontrol eder. Uygulamayı açabiliyorsa açar (bu durumda 8. Ve 9.
adımlara gelinmez, işlem bitiminde mobil success veya cancel url’e dönülür), açamıyorsa
Safari tarayıcısında BKM Express sunucusunun yönlendirmeyi yapacak olan sayfasını açar.
Bu sayfaya tokenı gönderir.<br>

8- BKM Express sayfası token ile birlikte işlem bilgilerine erişir ve gerekli yönlendirmeyi
yapar.<br>

9- BKM Express gelen tokendan işlem tipine bakar. Mobil uygulamadan veya Mobil
tarayıcıdan gelmiş ise, BKM Mobil uygulamasını açmaya çalışır, açamazsa uygulama
indirme linkine yönlendirir.<br>

###requestMerchInfo

Ödeme yapacağı kartı ve ödeme şeklini (taksit, peşin, taksit atlatmalı vb.) belirleyen
kullanıcının ödemesini gerçekleştirebilmesi için işyeri tarafından iletilecek POS ve MPI
verilerinin express sistemi tarafından işyerinden istendiği adımdır.

İşyeri, bu adımda bir web servis ile BKM’den gelen isteği karşılayıp cevap dönecektir.
İşyerinin, hazırladığı web servisinin kullanılabilirliğini test etmesi gerekmektedir.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516183/41323c68-8897-11e4-8c00-2406ed0cbd44.jpg)

###Confirmation URL

BKM Express sisteminde, sanal pos işlemlerinde başarılı sonucun işyerine ulaşmaması
ve kullanıcının işyerine son aşamada dönüşündeki sorunları en aza indirgemek amacıyla
confirmation URL yapısı kullanılmaktadır.

İşyeri, BKM’ye, işlem sonucunun success URL öncesi teyid amaçlı işyerine dönülmesi
için bir URL bilgisi vermelidir. BKM, bu URL e, success URL e dönüş yapmadan önce aynı
parametrelerle post yapmaktadır.

&lt;soapenv:Envelope xmlns:soapenv=&quot;http://schemas.xmlsoap.org/soap/envelope/&quot; xmlns:ws=&quot;http://merchant.ws.expressbkm.com.tr/&quot;&gt;<br>
&lt;soapenv:Header/&gt;<br>
&lt;soapenv:Body&gt;<br>
&lt;bkm:requestMerchInfo&gt;<br>
&lt;requestMerchInfoWSRequest&gt;<br>
&lt;t&gt;186005784745383&lt;/t&gt;<br>
&lt;bid&gt;123&lt;/bid&gt;<br>
&lt;bName&gt;BankaIsmi&lt;/bName&gt;<br>
&lt;cBin&gt;458652&lt;/cBin&gt;<br>
&lt;nofInst&gt;2&lt;/nofInst&gt;<br>
&lt;ts&gt;20110729-15:05:23&lt;/ts&gt;<br>
&lt;s&gt;PHijxQjUrcXBYoCei4QxjWo9Kg8D3p9tlWoT4t0/gyTE96639In0FZFY2/rvP+/bMsd1EArmKZsR5VW3rwoPxw=&lt;/s&gt;<br>
&lt;/requestMerchInfoWSRequest&gt;<br>
&lt;/bkm:requestMerchInfo&gt;<br>
&lt;/soapenv:Body&gt;<br>
&lt;/soapenv:Envelope&gt;<br>
<b>-Şekil 8 Örnek İstek-</b>

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516187/7c991d44-8897-11e4-9568-994a955d288a.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516196/26f5655e-8898-11e4-8a2d-3c5e6a242df7.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516201/651a85b2-8898-11e4-80d1-c028c649bb84.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516204/a7b45588-8898-11e4-8e50-a9c58816682a.jpg)


&lt;S:Envelope xmlns:soapenv=&quot;http://schemas.xmlsoap.org/soap/envelope/&quot;&gt;<br>
&lt;S:Body&gt;<br>
&lt;ns2:requestMerchInfoResponse xmlns:ws=&quot;http://merchant.ws.expressbkm.com.tr/&quot;&gt;<br>
&lt;RequestMerchInfoWSResponse&gt;<br>
&lt;t&gt;186005784745383&lt;/t&gt;<br>
&lt;posUrl&gt;https://posUrl.jsp&lt;/posUrl&gt;<br>
&lt;posUid&gt;posUsername&lt;/posUid&gt;<br>
&lt;posPwd&gt;posPassword&lt;/posPwd&gt;<br>
&lt;s3DAuth&gt;0&lt;/s3DAuth&gt;<br>
&lt;mpiUrl&gt;https://mpiUrl.jsp&lt;/mpiUrl&gt;<br>
&lt;mpiUid&gt;mpiUsername&lt;/mpiUid&gt;<br>
&lt;mpiPwd&gt;mpiPassword&lt;/mpiUid&gt;<br>
&lt;md&gt;23456&lt;/md&gt;<br>
&lt;xid&gt;NjQ5MThjYmQ2MTFmNWE1MGRiNDg=&lt;/xid&gt;<br>
&lt;s3DFDec&gt;0&lt;/s3DFDec&gt;<br>
&lt;cIp&gt;88.247.126.211&lt;/cIp&gt;<br>
&lt;extra&gt;{&quot;posno&quot;:&quot;P000007&quot;,&quot;xcip&quot;:&quot;EDF2556SA&quot;}&lt;/extra&gt;<br>
&lt;ts&gt;20110729-15:05:23&lt;/ts&gt;<br>
&lt;s&gt;PHijxQjUrcXBYoCei4QxjWo9Kg8D3p9tlWoT4t0/gyTE96639In0FZFY2/rvP+/bMsd1EArmKZsR5VW3rwoPxw=&lt;/s&gt;<br>
&lt;res&gt;<br>
&lt;resCode&gt;0&lt;/resCode&gt;<br>
&lt;resMsg&gt;Success&lt;/resMsg&gt;<br>
&lt;resDet/&gt;<br>
&lt;/res&gt;<br>
&lt;/RequestMerchInfoWSResponse&gt;<br>
&lt;/ns2:requestMerchInfoResponse&gt;<br>
&lt;/S:Body&gt;<br>
&lt;/S:Envelope&gt;<br>
<b>-Şekil 9 Örnek Dönüş-</b>

###İmza oluşturmak için alanların birleştirilmesi örneği(Java Kodu)<br>

StringBuffer sb = new StringBuffer();‬<br>

sb.append(t).append(posUrl)<br>
  &nbsp;  &nbsp;   .append(posUid).append(posPwd)<br>
  &nbsp;   &nbsp;  .append(s3Dauth).append(mpiUrl)<br>
  &nbsp;   &nbsp;  .append(mpiUid)‬ append(mpiPwd).<br>
  &nbsp;   &nbsp;  .append(md).append(xid)<br>
  &nbsp;   &nbsp; .append(s3DFDec).append(cIp)<br>
  &nbsp;   &nbsp;  .append(extra).append(ts);‬<br>
  
return sb.toString();‬<br>

###requestMerchInfo Dönüş Kodları: <br>

Yalnızca requestMerchInfo adımında, sistemden dönebilecek kod listesidir. Servisin tüm
dönüş kodları listesi EK D’ de yer almaktadır.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516241/bd9249c6-889a-11e4-82a8-ee431c3245da.jpg)

###Confirmation URL

BKM Express sisteminde, sanal pos işlemlerinde başarılı sonucun işyerine ulaştırılması
amacıyla 2 yöntem paralel kullanılmaktadır. Bunlardan akışa göre ilki, işyeri tarafından
sağlanan confirmation URL’ye BKM Express sistemi tarafından bildirim yapılmasıdır.
İşyerlerinin bu bildirimi alabilmek üzere confirmationURL çalışmasını yapması, ilk
entegrasyonlarda isteğe bağlı bırakılmakla birlikte, bu dokümanın 2.8 sürümü ve 19 Kasım
2013 tarihi itibariyle tüm işyerleri için mecburi hale getirilmiştir.

BKM, işyeri tarafından açılacak confirmationURL adresine success URL’ye dönüş
yapacağı alanları aynı formatta post edecektir.

İşyeri, confirmationURL’ye yapılan bildirimleri, BKM Express’ten geşdiğini succes URL
tarafı ile aynı şekilde kontrol edecektir (imza kontrolü v.b.). Eğer bildirim gerçekten BKM
Express tarafından yapılıyor ve SanalPOS’tan onay alındığını bildiriyorsa, bu adımda işyerinin
ürünü müştriye sağlayacak adımları başlatması gerekir (müşterinin email ile bilgilendirmesi,
işlem statü güncellemesi, stok hareketi v.b.)

Eğer bir sonraki adımda açıklanacak şekilde kullanıcı redirect ile işyerine giderken bir
sorun yaşarsa, zaten confirmationURL adımında ürün sağlama başlayacağı için, kullanıcı

 &nbsp;   &nbsp; 1- İşyeri sayfasına yeniden girip “Siparişlerim” adımına baktığında, siparişinin
onaylandığını görmelidir.

 &nbsp;   &nbsp; 2- Aynı şekilde işyeri email ile bildirim de yapıyorsa, kullanıcı email ile siparişin başarılı
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

Express ödeme akışının başarıyla tamamlandığı durumlarda express sisteminin işyerine
yaptığı geri yönlendirmedir. “initializePayment İstek Parametreleri” içinde alınan sUrl’e
aşağıdaki bilgiler http formu içerisinde hidden alan olarak post edilecektir. Bilgiler “Güvenlik”
bölümünde ve aşağıda belirtilen imzalama yöntemi dikkate alınarak imzalanacaktır. Ödeme
akışının herhangi bir nedenle başarısız olması sonucunda kullanıcı, “İşyerine İptal için
Yönlendirme” yöntemi ile işyeri’ne yönlendirilir.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516264/8cf5caf8-889b-11e4-811f-497f3706b03d.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516271/d820b088-889b-11e4-9d13-0399c38fda1e.jpg)

Kullanılacak Sanal POS’a göre posMessage içerisinde iletilen bilgiler değişecektir. POS’tan
dönen cevap içerisinde, Kredi Kartı ile ilgili bilgiler (PAN, CVC/CVV2, Son Kullanma Tarihi, Kart
Sahibi İsmi) çıkartılacak ve bunun dışında kalan bilgiler JSON formatında iletilecektir. Sanal
POS’tan dönen cevap için iletilecek posMessage örneği aşağıda yer almaktadır.

&lt;PosResponse xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br>
xmlns:xsd=&quot;http://www.w3.org/2001/XMLSchema&quot;&gt;<br>
&lt;Host&gt;<br>
&lt;AuthCode&gt;123456&lt;/AuthCode&gt;<br>
&lt;RRN&gt;126ABA7DFB72&lt;/RRN&gt;<br>
&lt;Date&gt;1234&lt;/Date&gt;<br>
&lt;Time&gt;010106&lt;/Time&gt;<br>
&lt;RC&gt;00&lt;/RC&gt;<br>
&lt;/Host&gt;<br>
&lt;Result&gt;<br>
&lt;Code&gt;0&lt;/Code&gt;<br>
&lt;Text /&gt;<br>
&lt;ErrorMessage /&gt;<br>
&lt;/Result&gt;<br>
&lt;TrnxID&gt;830601d3-1808-4c73-8d64-39fae20644b7&lt;/TrnxID&gt;<br>
&lt;TrnxType&gt;Sale&lt;/TrnxType&gt;<br>
&lt;CustomData /&gt;<br>
&lt;/PosResponse<br>
<b>-Şekil 10 Gerçek POS mesajı-</b>


{"PosResponse":{"Host":{"AuthCode":"123456","RRN":"126ABA7DFB72","Date":"1234","Time":"010
106","RC":"00"},"Result":{"Code":"0","Text":"","ErrorMessage":""},"TrnxID":"830601d3-1808-4c73-
8d64-39fae20644b7","TrnxType":"Sale","CustomData":""}}<br>
<b>-Şekil 11 POS mesajının JSON formatına çevrilmiş hali-</b>


###İşyerine İptal için Yönlendirme<br>

Express sisteminin, ödeme akışının tamamlanamadığı durumlarda herhangi bir adımda
işyerine yaptığı geri yönlendirmedir. “initializePayment İstek Parametreleri” içinde alınan cUrl
adresine, “initializePayment Dönüş Parametreleri”inde yer alan token ile “Güvenlik”
bölümünde belirtilen imzalama yöntemi dikkate alınarak gerçekleştirilir.

Aşağıdaki bilgiler redirect edilecek URL’e GET parametresi olarak eklenecektir.

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516281/a80f9e62-889c-11e4-8119-ac4d2ac2591f.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516288/eab66e26-889c-11e4-952c-048bb8cf0874.jpg)

&lt;PosResponse xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;<br>
xmlns:xsd=&quot;http://www.w3.org/2001/XMLSchema&quot;&gt;<br>
&lt;Host&gt;<br>
&lt;AuthCode&gt;123456&lt;/AuthCode&gt;<br>
&lt;RRN&gt;126ABA7DFB72&lt;/RRN&gt;<br>
&lt;Date&gt;1234&lt;/Date&gt;<br>
&lt;Time&gt;010106&lt;/Time&gt;<br>
&lt;RC&gt;00&lt;/RC&gt;<br>
&lt;/Host&gt;<br>
&lt;Result&gt;<br>
&lt;Code&gt;0&lt;/Code&gt;<br>
&lt;Text /&gt;<br>
&lt;ErrorMessage /&gt;<br>
&lt;/Result&gt;<br>
&lt;TrnxID&gt;830601d3-1808-4c73-8d64-39fae20644b7&lt;/TrnxID&gt;<br>
&lt;TrnxType&gt;Sale&lt;/TrnxType&gt;<br>
&lt;CustomData /&gt;<br>
&lt;/PosResponse<br>
<b>-Şekil 12 Gerçek POS mesajı-</b>

{"PosResponse":{"Host":{"AuthCode":"123456","RRN":"126ABA7DFB72","Date":"1234","Time":"010
106","RC":"00"},"Result":{"Code":"0","Text":"","ErrorMessage":""},"TrnxID":"830601d3-1808-4c73-
8d64-39fae20644b7","TrnxType":"Sale","CustomData":""}}<br>
<b>-Şekil 13 POS mesajının JSON formatına çevrilmiş hali-</b>

###İşlem Bilgileri Sorgulama

İşyeri, işlem için iletilen token bilgisini kullanarak, işlemin durumunu ve gerçekleşmişse
ödeme bilgilerini, BKM Express’in bu servisi aracılığıyla sorgulayabilir.

Bu dokümanın 2.8 sürümü ve 19 Kasım 2013 tarihi itibariyle işyerlerinin İşlem Bilgileri
Servisini kullanabilmeleri mecburidir.

Sorgulama servisi, özellikle işyerlerinin geçici olarak yaşayabilecekleri sorun sonrasında,
aslında ödemesi gerçekleşen ancak ürün sağlanmayan işlemlerin tespiti için kullanılacaktır.

İşyeri sorgulama yaptığında ödemesinin alındığını tespit ettiği işlemler için ilgili ürünleri
müşteriye sağlamak ile yükümlüdür.

Mevcut InitPayment servisi içerisindeki “QueryPaymentResult” metodu yardımıyla
İşyeri, istekte bulunarak işlem sonuç bilgilerine ulaşılabilmektedir.

Gönderilecek değerler aşağıdaki şekildedir.

mId= merchantID,
s= imza { mId , t , ts }
t= token,
ts= timestamp

<b>** Bu alanlardan imza alanının diğer alanların birbiri ardına eklenmesi ve SHA256
ile oluşturulan hash değerinin mac verify yöntemi ile doğrulanması gerekmektedir.
Bu işlem için üye işyerleri BKM Express in RSA public keyini kullanmalıdır.</b>

Servisin cevap parametreleri ise aşağıdaki şekildedir.

t= token
ts= timestamp
s= imza {t, success, ts}
success = işlem sonucu , boolean,

<b>** Bu alanlardan imza alanının diğer alanların birbiri ardına eklenmesi ve SHA256
ile oluşturulan hash değerinin mac verify yöntemi ile doğrulanması gerekmektedir.
Bu işlem için üye işyerleri kendi RSA public keyini kullanmalıdır.</b>


###Sıkça Sorulan Sorular

<b>Site üzerinden BKM’ye yönlenmeye çalıştığımda istek mesajım BKM’ye ulaşmıyor.
Ne yapmalıyım?</b>

- İstek mesajları, bize aşağıdaki nedenlerden ulaşmıyor olabilir.
- “telnet preprod.bkmexpress.com.tr 9620” yapabildiğinizi
- Web.config dosyasında doğru endpoint adresini verdiğinizi
- BKM’ye ilettiğiniz public key ile sizin referans verdiğiniz public key değerinin aynı
olduğunu
- Web.config dosyalarında key path’lerinin doğru verildiğini
- Web.config ‘in bulunduğu dosyaya okuma hakkınızın olduğunu
kontrol etmelisiniz.


<b>Preprod ve Production için endpoint adresleri nelerdir?</b>

- Preprod için :
https://preprod.bkmexpress.com.tr:9620/BKMExpress/BkmExpressPaymentService.ws
- Prod için :
https://www.bkmexpress.com.tr:9620/BKMExpress/BkmExpressPaymentService.ws

<b>Public key uzunluğu ne olmalıdır?</b>

Public key uzunluğunuz 1024 yada 2048 olmalıdır.

<b>Doğrulama kodu cep telefonuma gelmiyor. Ne yapmalıyım?</b>

PreProd ortamı için doğrulama kodu alanına, “123456” girebilirsiniz.

<b>3D Secure üyesi değilim. Webserviste MPI bilgilerini ne göndermeliyim.</b>

3D Secure desteklemiyor iseniz, webservis çağrısında bu alanları bize boş göndermelisiniz.

<b>Web servis çağrısı yapıyorum fakat “Unknown Error” hatası ile karşılaşıyorum.</b>

Burada kontrol etmeniz gereken bir kaç nokta mevcut :

- Merchant olarak kaydınızı BKM’ye yaptırdınız mı? Bu kaydın sonunda size bir
- “Merchant Id” tanımlanmış olmalı, yaptığınız istekte bu alanı kullanıyor musunuz?
- Dökümanın başında yazılı talimatlardan olan private-public key çiftinizi ürettiniz mi?<br>
&nbsp;   &nbsp; o Ürettiyseniz public keyinizi BKM ile paylaştınız mı?<br>
&nbsp;   &nbsp; o Public keyinizi paylaştıysanız, BKM’nin keyinizi kaydettiğini doğrular mısınız?<br>
- Yukarıdaki iki seçenekte de sorun yoksa bu sefer BKM’den, sizin için hata loglarını
incelemelerini istemelisiniz.

<b>Web servis çağrısı yapıyorum ama “Request Not Synchronized” hatası alıyorum.</b>

Bu hatayı alıyor olmanızın sebebi; BKM’nin sunucuları ile sizin sunucularınız
arasındaki zaman farkının kabul edilebilir düzeyin üstünde olmasıdır.

Bu hatayı önlemek için; sunucu zaman ayarınızı NTP sunucularından almanız gerekmektedir.
Örnek olarak; asia.pool.ntp.org üzerinden alabilirsiniz.

<b>Key üretmede sıkıntı yaşıyorum</b>
Windows bazlı, .Net geliştirme ortamına sahip sistem kullanıcıları, BKM’nin, sizlere
sağladığı “Key Generator” uygulamasını kullanabilirsiniz.

Unix bazlı sistemler için daha önce size iletilen komutları kullanabilirsiniz.

<b>BKM’den aldığım wsdl dosyalarını çalıştırdığımda bağlantı hatası alıyorum.</b>

Bkm’den aldığınız wsdl dosyasının endpointlerini kontrol edin. Eğer endpointlerde
geçen değer localhost veya 127.0.0.1 ile başlıyorsa bağlanamamanız normal. BKM’den,
bağlanma izniniz olan bir endpoint adresi isteyebilirsiniz.

Bunların dışında bir adres görüp de yine de bağlanamamanız durumunda BKM’den konu ile
ilgili yardım istemeniz gerekmektedir.

<b>İmzalama ile ilgili sorunlar</b>

İmzalama konusunda bir kaç temel problem yaşanabiliyor :<br>

- İmza, istendiği şekilde alanların sırayla eklenmesi ile oluşturulmamış.<br>

&nbsp; &nbsp; &nbsp; o Bu durum özellikle initialize payment metodunda yaşanıyor. Gönderdiğiniz
her banka ve her taksit seçeneği için birer inner loop yaparak imza
oluşturacağınız stringi oluşturmanız gerekiyor. Detay için, ilgili web
servislerin açıklamalarındaki Java örneğine bakabilirsiniz.<br>

- Null alanlar imzaya boş string olarak eklenmiş.<br>

&nbsp; &nbsp; &nbsp; o Null olan alanlarınızı “null” olarak imzalarınıza eklediğinizden emin olun.
Boolean değerleri imzanıza ve mesajınıza aynı şekilde eklediğinizden emin olun.<br>

&nbsp; &nbsp; &nbsp; o Mesajda “true” gelip imzanızda “1” yazdığınızda, imzalar uyuşmayacağından
“MAC verification fail” hatası alacaksınız.

<b>RequestMerchInfo web servisini hazırlayamıyorum</b>

Bu web servisi, işyerlerine gönderilen “requestMerchInfoService.wsdl” ile uyumlu
olarak yazmanız gerekiyor. Aksi takdirde BKM, sizin web servisinize geldiğinde büyük
ihtimalle namespace e dayalı bir hata oluşacaktır.

Bu servisleri wsdl e uygun halde hazırlamak için,Java ve .NET ortamlarında web servis
kabukları üreten araçlar mevcut, aynı zamanda BKM’nin sunduğu PHP paketinde de örnek bir
web servis bulunuyor.





