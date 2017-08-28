

**邮件服务器DNS设置**



[http://www.samhelp.cn/bbs/ShowPost.asp?PostID=28611](http://www.samhelp.cn/bbs/ShowPost.asp?PostID=28611)

DNS记录，需要你到你的域名托管商那里进行设置或者你自己管理DNS服务器。不少域名托管商不支持txt记录或者不支持DKIM记录，所以你就无法使用SPF和DKIM的功能。  
 DNS的修改，需要48小时以上才能生效。  
国内的万网是不支持DKIM，目前新网是支持SPF和DKIM。  
  
 1.MX记录  
邮件的MX记录最好是指向机器A记录，尽量不要直接指向IP地址（不符合规范）。  
**1.1添加A记录**  
 mail.example.com 192.168.1.100  
**1.2添加MX记录**  
 example.com mail.example.com  
**1.3测试MX记录**  
 \# host exmple.com  
 example.com mail is handled by 10 mail.example.com.  
 \#nslookup mail.example.com  
 Name:mail.example.com  
 Address:192.168.1.100  
  
 2.SPF记录  
 SPF是指Sender Policy Framework，是为了防范垃圾邮件而提出来的一种DNS记录类型，SPF是一种TXT类型的记录。SPF记录的本质，就是向收件人宣告：本域名的邮件从清单上所列IP发出的都是合法邮件，并非冒充的垃圾邮件。设置好SPF是正确设置邮件发送的域名记录和STMP的非常重要的一步。  
例如：  
 SPF记录指向A主机记录  
 example.com. 3600 IN TXT "v=spf1 mx mx:mail.example.com ~all"  
 SPF记录指向IP地址  
 example.com. 3600 IN TXT "v=spf1 ip4:192.168.1.100 ~all"  
  
**2.1如何查看SPF**  
 Windows下进入DOS模式后用以下命令:  
 nslookup -type=txt域名  
 Unix操作系统下用：  
 \# dig -t txt域名  
  
**2.2 SPF的简单说明如下：**  
 v=spf1表示spf1的版本  
 IP4代表IPv4进行验证\(IP6代表用IPv6进行验证\),注意“ip4:”和“IP”之间是没有空格的  
 ~all代表结束  
  
**2.3 SPF记录例释**  
我们看这条SPF:  
 yourdomain.com "v=spf1 a mx mx:mail.jefflei.com ip4:202.96.88.88 ~all"  
这条SPF记录具体的说明了允许发送@yourdomain.com的IP地址是：a（这个a是指yourdomain.com解析出来的IP地址，若没有配置应取消）  
 mx（yourdomain.com对应的mx，即mail.yourdomain.com的A记录所对应的ip）  
 mx:mail.jefflei.com（如果没有配置过mail.jefflei.com这条MX记录也应取消）  
 ip4:202.96.88.88（直接就是202.152.186.85这个IP地址）  
其他还有些语法如下：  
 - Fail,表示没有其他任何匹配发生  
 ~代表软失败，通常用于测试中  
 ?代表忽略  
  
如果外发的ip不止一个，那么必须要包含多个  
 v=spf1 ip4:202.96.88.88 ip4:202.96.88.87 ~all  
  
**2.4测试SPF设置结果**  
设置好DNS中的SPF记录后，发送邮件给自己的gmail，然后查看邮件的源文件，应该能看到类似的邮件头，其中有pass表示设置成功。  
 Received-SPF: pass \(google.com: domain of [test@jefflei.com](mailto:test@jefflei.com) designates  
 202.96.88.87 as permitted sender\) client-ip=202.96.88.87;  
需要注意的是，服务器的IP若有更改，需要同时修改SPF！！！  
  
**2.5利用SPF记录防止垃圾邮件**  
在Unix下可以安装配置SpamAssassin之类的插件来防止垃圾邮件和钓鱼邮件\(Phishing\)  
 3.DKIM记录  
 DKIM技术通过在每封电子邮件上增加加密的数字标志，然后与合法的互联网地址数据库中的记录进行比较。当收到电子邮件后，只有加密信息与数据库中记录匹配的电子邮件，才能够进入用户的收件箱。它还能检查邮件的完整性，避免\*\*\*\*等未授权者的修改。DKIM的基本工作原理同样是基于传统的密钥认证方式，他会产生两组钥匙，公钥\(public key\)和私钥\(private key\)，公钥将会存放在DNS中，而私钥会存放在寄信服务器中。私钥会自动产生，并依附在邮件头中，发送到寄信者的服务器里。公钥则放在DNS服务器上，供自动获得。收信的服务器，将会收到夹带在邮件头中的私钥和在DNS上自己获取公钥，然后进行比对，比较寄信者的域名是否合法，如果不合法，则判定为垃圾邮件。由于数字签名是无法仿造的，因此这项技术对于垃圾邮件制造者将是一次致命的打击，他们很难再像过去一样，通过盗用发件人姓名、改变附件属性等小伎俩达到目的。在此之前，垃圾邮件制造者通过把文本转换为图像等方式逃避邮件过滤，并且使得一度逐渐下降的垃圾邮件数目再度抬头。  
注意：Amavisd-new只有2.6.0及以上版本集成了DKIM功能。  
  
**3.1这里可以通过iredmail.tips获得域名的DKIM，也可以在命令行下输入**  
 \# amavisd-new showkeys  
 ; key\#1, domain example.com, /var/lib/dkim/example.com.pem  
 dkim.\_domainkey.example.com. 3600 TXT \(  
"v=DKIM1; p="  
"MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDGNVMuQRKqYeySIzqTGTm3xRzF"  
"/ZzhmMnpZkEcVVjFAk+t7E388oFGu/knyh6KBKwpZxHUN5HoOYVjMudqaR2FcSvK"  
"z+joFj8Vh3rXoTLa1zHDyfD7hICzxdEgmQZ8MJM5rjPPrRGZXnPowNYDsd6nDJ86"  
"N38iFYU+jALBYDLBwQIDAQAB"\)  
  
**3.2把上面记录添加到ISP的DNS记录**  
 dkim.\_domainkey.example.com. v=DKIM1; p=MIGfMA0....（省略）DLBwQIDAQAB  
  
**3.3添加完DNS记录后，如果记录生效，可以通过运行命令检测**  
 \# amavisd-new testkeys  
 TESTING: dkim.\_domainkey.example.com =&gt; pass  
  
检查DNS设置  
  
  
下面有几种方法，可以帮助你检测DNS是否设置生效和正常工作：  
**1.nslookup**  
 \#nslookup  
 Default Server: unknown  
 Address: 192.168.1.1  
&gt; server 4.2.2.1  
 Default Server: vnsc-pri.sys.gtei.net  
 Address: 4.2.2.1  
&gt; set type=mx  
&gt; example.com  
 Server: vnsc-pri.sys.gtei.net  
 Address: 4.2.2.1  
 Non-authoritative answer:  
 example.com MX preference = 20, mail exchanger = mail.example.com  
&gt; set type=txt  
&gt; example.com  
 Server: vnsc-pri.sys.gtei.net  
 Address: 4.2.2.1  
 Non-authoritative answer:  
 example.com text =  
"v=spf1 ip4:192.168.1.100 -all"  
&gt; dkim.\_domainkey.example.com  
 Server: vnsc-pri.sys.gtei.net  
 Address: 4.2.2.1  
 Non-authoritative answer:  
 dkim.\_domainkey.example.com text =  
"v=DKIM1; p= MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCsgZaIvYHAos2jbp3CHW0  
 AwrTnAEwV1p4EaZP/JuF8t1BETBVg6WJr3YWN5ijCpi9vnw96nmf/u5MgtbLwZ+AzDBkbOY7Jbb/hIO+  
 mpmmfdJAY3w8KoXLCuQKDysXOys45YtfJEj66s51EHH3W+iXPYw3I/NWHjY3a5/mXnk4XJQIDAQAB"  
  
**2.linux dig**  
 MX记录  
 \# host exmple.com  
 example.com mail is handled by 10 mail.example.com.  
  
 SPF记录  
 \# dig txt hotmail.com  
 ; &lt;&lt;&gt;&gt; DiG 9.4.2-P2 &lt;&lt;&gt;&gt; txt hotmail.com  
 ;; global options: printcmd  
 ;; Got answer:  
 ;; -&gt;&gt;HEADER&lt;&lt;- opcode: QUERY, status: NOERROR, id: 43130  
 ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0  
  
 ;; QUESTION SECTION:  
 ;hotmail.com. IN TXT  
  
 ;; ANSWER SECTION:  
 hotmail.com. 3600 IN TXT "v=spf1 include:spf-a.hotmail.com include:spf-b.hotmail.com include:spf-c.hotmail.com include:spf-d.hotmail.com ~all"  
  
 ;; Query time: 176 msec  
 ;; SERVER: 64.71.161.8\#53\(64.71.161.8\)  
 ;; WHEN: Sat Dec 5 08:43:51 2009  
 ;; MSG SIZE rcvd: 157  
  
 DKIM记录  
 \#dig txt dkim.\_domainkey.example.com

