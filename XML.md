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
