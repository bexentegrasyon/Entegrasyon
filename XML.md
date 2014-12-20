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

&lt;soapenv:Envelope xmlns:soapenv=&quot;http://schemas.xmlsoap.org/soap/envelope/&quot; xmlns:bkm=&quot;http://www.bkmexpress.com.tr&quot;&gt;<br>
&lt;soapenv:Header/&gt;<br>
&lt;soapenv:Body&gt;<br>
&lt;bkm:initializePayment&gt;<br>
&lt;bkm:initializePaymentWSRequest&gt;<br>
&lt;mId&gt;7f85e133-3cd2-4821-9bf7-eb0bcb55cbb3&lt;/mId&gt;<br>
&lt;sUrl&gt;http://.../odemeBasarili.jsp&lt;/sUrl&gt;<br>
&lt;cUrl&gt;http://.../odemeBasarisiz.jsp&lt;/cUrl&gt;<br>
&lt;sAmount&gt;78,99&lt;/sAmount&gt;<br>
&lt;cAmount&gt;5,00&lt;/cAmount&gt;<br>
&lt;instOpts&gt;<br>
&lt;!--1 or more repetitions:--&gt;<br>
&lt;bank&gt;<br>
&lt;id&gt;9999&lt;/id&gt;<br>
&lt;name&gt;BankaIsmi&lt;/name&gt;<br>
&lt;expBank&gt;Banka firsatlarini ka&ccedil;irmayin!&lt;/expBank&gt;<br>
&lt;bins&gt;<br>
&lt;!--1 or more repetitions:--&gt;<br>
&lt;bin&gt;<br>
&lt;value&gt;444411&lt;/value&gt;<br>
&lt;insts&gt;<br>
&lt;!--1 or more repetitions:--&gt;<br>
&lt;inst&gt;<br>
&lt;nofInst&gt;1&lt;/nofInst&gt;<br>
&lt;amountInst&gt;80,99&lt;/amountInst&gt;<br>
&lt;cAmount&gt;5,00&lt;/cAmount&gt;<br>
&lt;tAmount&gt;80,99&lt;/tAmount&gt;<br>
&lt;cPaid1stInst&gt;true&lt;/cPaid1stInst&gt;<br>
&lt;expInst&gt;Pesin &ouml;demelerde 3 TL indirim&lt;/expInst&gt;<br>
&lt;/inst&gt;<br>
&lt;inst&gt;<br>
&lt;nofInst&gt;2&lt;/nofInst&gt;<br>
&lt;amountInst&gt;42,00&lt;/amountInst&gt;<br>
&lt;cAmount&gt;5,00&lt;/cAmount&gt;<br>
&lt;tAmount&gt;84,00&lt;/tAmount&gt;<br>
&lt;cPaid1stInst&gt;false&lt;/cPaid1stInst&gt;<br>
&lt;expInst/&gt;<br>
&lt;/inst&gt;<br>
&lt;/insts&gt;<br>
&lt;/bin&gt;<br>
&lt;/bins&gt;<br>
&lt;/bank&gt;<br>
&lt;/instOpts&gt;<br>
&lt;ts&gt;20110729-15:05:23&lt;/ts&gt;<br>
&lt;s&gt;IWijxQjUrcXBYoCei4QxjWo9Kg8D3p9tlWoT4t0/gyTE96639In0FZFY2/rvP+/bMJ01EArmKZsR5VW3rwoPxw=&lt;/s&gt;<br>
&lt;/bkm:initializePaymentWSRequest&gt;<br>
&lt;/bkm:initializePayment&gt;<br>
&lt;/soapenv:Body&gt;<br>
&lt;/soapenv:Envelope&gt;<br>
<b>-Şekil 6 Örnek İstek-<b/>

##&#304;mza olu&#351;turmak i&ccedil;in alanlar&#305;n birle&#351;tirilmesi &ouml;rne&#287;i (Java Kodu)
<p>
  StringBuffer sb = new StringBuffer();&#8236;<br>
  sb.append(mId).append(sUrl).append(cUrl).append(sAmount).append(cAmount);&#8236;<br>
  for(Bank aBank : bank)<br>
  {&#8236;<br>
  sb.append(aBank.getId()).append(aBank.getName())<br>
  .append(aBank.getExpBank());&#8236;<br>
  for(Bin bin : aBank.getBin())<br>
  {&#8236;<br>
  sb.append(bin.getValue());&#8236;<br>
  for(Installment inst : bin.getInst())<br>
  {&#8236;<br>
  sb.append(inst.getNofInst()).append(inst.getAmountInst())<br>
  .append(inst.getcAmount())&#8236;<br>
  .append(inst.gettAmount())<br>
  .append(inst.iscPaid1stInst())<br>
  .append(inst.getExpInst());&#8236;<br>
  }&#8236;<br>
  }&#8236;<br>
  }&#8236;<br>
  sb.append(ts);&#8236;<br>
  return sb.toString();<br>
</p>

![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516081/e16dca22-8891-11e4-9025-c8351cf28faa.jpg)
![GitHub Logo](https://cloud.githubusercontent.com/assets/10204757/5516103/eac877c4-8892-11e4-9332-31b54ec00dbd.jpg)

<table cellspacing="0" cellpadding="0">
  <tr>
    <td valign="top" align="left"><p>&lt;S:Envelope xmlns:soapenv=&quot;http://schemas.xmlsoap.org/soap/envelope/&quot;&gt; </p>
        <p>&lt;S:Body&gt; </p>
        <p>&lt;ns2:initializePaymentResponse xmlns:ws=&quot;http://merchant.ws.expressbkm.com.tr/&quot;&gt; </p>
        <p>&lt;InitializePaymentWSResponse&gt; </p>
        <p>&lt;t&gt;186005784745383&lt;/t&gt; </p>
        <p>&lt;url&gt;http://bkmexpress/islemBasarili.bkm&lt;/url&gt; </p>
        <p>&lt;ts&gt;20110729-15:05:23&lt;/ts&gt; </p>
        <p>&lt;s&gt;PHijxQjUrcXBYoCei4QxjWo9Kg8D3p9tlWoT4t0/gyTE96639In0FZFY2/rvP+/bMsd1EArmKZsR5VW3rwoPxw=&lt;/s&gt; </p>
        <p>&lt;res&gt; </p>
        <p>&lt;resCode&gt;0&lt;/resCode&gt; </p>
        <p>&lt;resMsg&gt;Success&lt;/resMsg&gt; </p>
        <p>&lt;resDet/&gt; </p>
        <p>&lt;/res&gt; </p>
        <p>&lt;/InitializePaymentWSResponse&gt; </p>
        <p>&lt;/ns2:initializePaymentResponse&gt; </p>
        <p>&lt;/S:Body&gt; </p>
        <p>&lt;/S:Envelope&gt; </p></td>
  </tr>
</table>
