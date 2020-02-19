---
title: 為 xshogi 官網的第二個 domain 加入 ssl
published: true
---

<!-- wp:paragraph -->
<p>實驗域名：xshogi.com</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>已經有了上次的<a href="https://hackmd.io/6boNOfNTTWy7wxlGIUxqGg">經驗</a>後，要為另一個 xshogi.com 加入 ssl 憑證，過程記錄如下。</p>
<!-- /wp:paragraph -->

## 第一階段：購買 SSL 憑證

<!-- wp:list {"ordered":true} -->
<ol><li><code>openssl req -nodes -newkey rsa:2048 -keyout xshogi.com.key -out xshogi.com.csr</code></li></ol>
<!-- /wp:list -->

<!-- wp:code -->
<pre class="wp-block-code"><code>
↪ openssl req -nodes -newkey rsa:2048 -sha256 -keyout xshogi.com.key -out xshogi.com.csr
Generating a 2048 bit RSA private key
............................+++
............+++
writing new private key to 'xshogi.com.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) &#91;]:TW
State or Province Name (full name) &#91;]:Taipei
Locality Name (eg, city) &#91;]:city
Organization Name (eg, company) &#91;]:xshogi
Organizational Unit Name (eg, section) &#91;]:
Common Name (eg, fully qualified host name) &#91;]:www.xshogi.com
Email Address &#91;]:xshogi.it@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password &#91;]:
</code></pre>
<!-- /wp:code -->

到 Gandi SSL 購買介面，選擇類型為 "基本" 和 "單一位址"，然後到貼上 CSR 的地方

![](assets/20181025/20181025-1.png)

這邊如果把上次的貼上(in .ssl/xshogi_heroku.csr)，會解析簽署名稱(CN)成 www.xshogi.com.tw，所以要貼上新的 xshogi.com.csr

![](assets/20181025/20181025-2.png)

<!-- wp:list {"ordered":true} -->
<ol><li>下一步以後付完款，接著就是等待 域名所有權驗證<br>
</li></ol>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>會收到一封信說，有三種方法可以驗證您擁有域名。您可以在憑證控制面板中的 "編輯" 選擇要驗證的方式。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><em>經由DNS 認證</em><br>
必須將此介面提供的 CNAME 記錄加入 xshogi.com 的 DNS 區域檔。若為多重域名憑證，則每個域名都必須完成此操作。第一次驗證的時間將會在三小時之後開始。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><em>經由電子郵件認證(最快)</em><br>
重要：您必須建立一個電子郵件地址 admin@xshogi.com (若為多重域名憑證，每個域名都需要一個電子郵件地址) 以接收認證訊息 (內含需點擊的連結，會連線到 Comod 的網站)。注意：若電子郵件地址建立速度太慢，可能會漏收郵件：也可以於此介面重新發送認證電子郵件。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><em>檔案認證</em><br>
您必須到憑證詳細資訊中取得認證的 .TXT 檔並且把它放置到需驗證網站的根目錄。若為多重域名憑證，則上述的每個域名與網站都必須完成此操作。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>若完全不需進行這些操作，這表示您是使用免費的憑證並且由 Gandi 自動驗證。</p>
<!-- /wp:paragraph -->

![](assets/20181025/20181025-3.png)

我選擇使用 DNS：
![](assets/20181025/20181025-4.png)

<!-- wp:paragraph -->
<p>切到區域檔紀錄，會發現已經自動幫我們加入一筆新的 CNAME 了<br>
</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>過一陣子後驗證就會自動完成並啟用了。</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li>下載 .crt 和 .pem 並將他們連接在一起 <code>cat GandiStandardSSLCA2.pem www.xshogi.com.crt &gt; all_xshogi.com.crt</code></li></ol>
<!-- /wp:list -->

![](assets/20181025/20181025-5.png)

## 第二階段：設定 Heroku

<!-- wp:list {"ordered":true} -->
<ol><li>新增 certification 和 key 到 heroku</li></ol>
<!-- /wp:list -->

<!-- wp:code -->
<pre class="wp-block-code"><code>
↪ heroku certs:add all_xshogi.com.crt xshogi.com.key --app xshogi
 ›   Warning: heroku update available from 7.16.4 to 7.16.6
Resolving trust chain... done
Adding SSL certificate to ⬢ xshogi... !
 ▸    Only one SNI endpoint is allowed per app (try certs:update instead).
 </code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Oops! 因為已經有一組了，只能用 update 來替換掉原本的</p>
<!-- /wp:paragraph -->

<pre class="wp-block-code"><code>
↪ heroku certs:update all_xshogi.com.crt xshogi.com.key --app xshogi
› Warning: heroku update available from 7.16.4 to 7.16.6
Resolving trust chain... done
▸ Potentially Destructive Action
▸ This command will change the certificate of endpoint ceratosaurus-25252 from ⬢ xshogi.
▸ To proceed, type xshogi or re-run this command with --confirm xshogi

> xshogi
> Updating SSL certificate ceratosaurus-25252 for ⬢ xshogi... done
> Updated certificate details:
> Common Name(s): www.xshogi.com

                xshogi.com

Expires At: 2019-10-26 07:59 UTC
Issuer: /C=FR/ST=Paris/L=Paris/O=Gandi/CN=Gandi Standard SSL CA 2
Starts At: 2018-10-25 08:00 UTC
Subject: /OU=Domain Control Validated/OU=Gandi Standard SSL/CN=www.xshogi.com
SSL certificate is verified by a root authority.
</code></pre>

還要在 add 一次：

<pre class="wp-block-code"><code>
↪ heroku certs:add all_xshogi.com.crt xshogi.com.key --app xshogi
› Warning: heroku update available from 7.16.4 to 7.16.6
Resolving trust chain... done
Adding SSL certificate to ⬢ xshogi... done
Certificate details:
Common Name(s): www.xshogi.com
xshogi.com
Expires At: 2019-10-26 07:59 UTC
Issuer: /C=FR/ST=Paris/L=Paris/O=Gandi/CN=Gandi Standard SSL CA 2
Starts At: 2018-10-25 08:00 UTC
Subject: /OU=Domain Control Validated/OU=Gandi Standard SSL/CN=www.xshogi.com
SSL certificate is verified by a root authority.

=== The following common names already have domain entries
www.xshogi.com
xshogi.com

=== Your certificate has been added successfully. Update your application's DNS settings as follows
Domain Record Type DNS Target
───────────────── ─────────── ──────────────────────────────────────────────────────────────
www.xshogi.com CNAME marine-procompsognathus-l8xr054mis0egvrf5l7q8pks.herokudns.com
www.山角行.com CNAME www.xn--rhto43ho7a.com.herokudns.com
xshogi.com.tw ALIAS/ANAME xshogi.com.tw.herokudns.com
www.xshogi.com.tw CNAME www.xshogi.com.tw.herokudns.com
山角行.com ALIAS/ANAME xn--rhto43ho7a.com.herokudns.com
xshogi.com ALIAS/ANAME opaque-owl-6ikxb7mtjg47hg8u6mcu5g2x.herokudns.com
</code></pre>

檢查憑證

<pre class="wp-block-code"><code>
↪ heroku certs:info -a xshogi
› Warning: heroku update available from 7.16.4 to 7.16.6
Fetching SSL certificate velociraptor-96982 info for ⬢ xshogi... done
Certificate details:
Common Name(s): www.xshogi.com
xshogi.com
Expires At: 2019-10-26 07:59 UTC
Issuer: /C=FR/ST=Paris/L=Paris/O=Gandi/CN=Gandi Standard SSL CA 2
Starts At: 2018-10-25 08:00 UTC
Subject: /OU=Domain Control Validated/OU=Gandi Standard SSL/CN=www.xshogi.com
SSL certificate is verified by a root authority.
</code></pre>

修改 Gandi 的網頁轉址使用 https

## 成果

![](assets/20181025/20181025-6.png)
