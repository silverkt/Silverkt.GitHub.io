### 安装

git clone [https://github.com/letsencrypt/letsencrypt](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fletsencrypt%2Fletsencrypt)

### 执行

```bash
cd letsencrypt
./certbot-auto certonly  -d *.你的域名 --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

这里用了泛域名证书

### 执行命令

如果上面报错

```bash
OSError: Command /opt/eff.org/certbot/venv/bin/python2.7 - setuptools pip wheel failed with error code 1
```

### 解决问题

```bash
卸载virtualenv： pip  uninstall  virtualenv

再安装virtualenv ： pip  install  virtualenv==15.1.0
```

### 输出

```bash
ackage gcc-4.8.5-36.el7_6.2.x86_64 already installed and latest version
Package augeas-libs-1.4.0-6.el7_6.1.x86_64 already installed and latest version
Package 1:openssl-1.0.2k-16.el7_6.1.x86_64 already installed and latest version
Package 1:openssl-devel-1.0.2k-16.el7_6.1.x86_64 already installed and latest version
Package libffi-devel-3.0.13-18.el7.x86_64 already installed and latest version
Package redhat-rpm-config-9.1.0-87.el7.centos.noarch already installed and latest version
Package ca-certificates-2018.2.22-70.0.el7_5.noarch already installed and latest version
Package python-devel-2.7.5-77.el7_6.x86_64 already installed and latest version
Package python-virtualenv-15.1.0-2.el7.noarch already installed and latest version
Package python-tools-2.7.5-77.el7_6.x86_64 already installed and latest version
Package python2-pip-8.1.2-8.el7.noarch already installed and latest version
Nothing to do
Creating virtual environment...
Installing Python packages...
Installation succeeded.
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): 531833XXX@qq.com
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
```

是否通用协议，选择是，那就是'A'

```bash
A
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
```

是否分享你的邮箱，否

```bash
N
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.
```

询问是否对域名和机器（IP）进行绑定=>需要同意

```bash
Y
```
需要按照要求(指定key：value)在dns设置一条txt记录，在dns设置完成之前不能回车
```bash
Please deploy a DNS TXT record under the name
_acme-challenge.ippnetwork.com with the following value:
sFT0ouhU4sktQl8_D8XPySBKUnLfmoGe_Ief4Kr7P1I
Before continuing, verify the record is deployed.
```
此处key设置注意，去掉域名后缀，按上方例子，key值为_acme-challenge
回车后将会验证，成功后提示如下：
```bash
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/ippnetwork.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/ippnetwork.com/privkey.pem
   Your cert will expire on 2020-03-02. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```


### 证书续签

注:证书在到期前30天才会续签成功,但为了确保证书在运行过程中不过期,官方建议每天自动执行续签两次;
 使用crontab自动续期

 ```bash
 crontab -e // 编辑定时任务
 0 */12 * * * certbot renew --quiet --renew-hook "/etc/init.d/nginx reload"
 ```

### 证书保存的路径[配置nginx需要用到的]

```bash
/etc/letsencrypt/live/you.cn/fullchain.pem
/etc/letsencrypt/live/you.cn/privkey.pem

nginx配置样例：
server {
listen 80;
server_name ippnetwork.com;
root /usr/share/nginx/www;
}

server {
listen *:443 ssl http2;
root /usr/share/nginx/www;
server_name ippnetwork.com;
ssl_certificate /etc/letsencrypt/live/ippnetwork.com/fullchain.pem; 
ssl_certificate_key  /etc/letsencrypt/live/ippnetwork.com/privkey.pem;
}
```

### 取消证书

可以使用一下命令取消刚刚生成的密匙，也就是以上的反操作：

```bash
certbot revoke --cert-path /etc/letsencrypt/live/you.cn/cert.pem
certbot delete --cert-name you.cn
```


### 扩展阅读
#### 如何申请 Let’s Encrypt 通配符证书

为了实现通配符证书，Let’s Encrypt 对 ACME 协议的实现进行了升级，只有 v2 协议才能支持通配符证书。

也就是说任何客户端只要支持 ACME v2 版本，就可以申请通配符证书了，是不是很激动。

读者可以查看下自己惯用的客户端是不是支持 ACME v2 版本，官方介绍 Certbot 0.22.0 版本支持新的协议版本，我立刻进行了升级：

```bash
./certbot-auto -V
Upgrading certbot-auto 0.21.1 to 0.22.0...
Replacing certbot-auto... 

./certbot-auto -V
certbot 0.22.0 
```

在了解该协议之前有几个注意点：

1）客户在申请 Let’s Encrypt 证书的时候，需要校验域名的所有权，证明操作者有权利为该域名申请证书，目前支持三种验证方式：

- dns-01：给域名添加一个 DNS TXT 记录。
- http-01：在域名对应的 Web 服务器下放置一个 HTTP well-known URL 资源文件。
- tls-sni-01：在域名对应的 Web 服务器下放置一个 HTTPS well-known URL 资源文件。

而申请通配符证书，只能使用 dns-01 的方式

2）ACME v2 和 v1 协议是互相不兼容的，为了使用 v2 版本，客户端需要创建另外一个账户（代表客户端操作者），以 Certbot 客户端为例，大家可以查看：

```bash
$ tree /etc/letsencrypt/accounts 
.
├── acme-staging.api.letsencrypt.org
├── acme-v01.api.letsencrypt.org
└── acme-v02.api.letsencrypt.org
```

3）Enumerable Orders 和限制

为了实现通配符证书，Let's Encrypt  在申请者身份校验上做了很大的改变。

- 有了订单 ID 的概念，主要是为了追踪通配符域名。
- 申请限制，在 V1 版本，Let's Encrypt 为了避免滥操作，对申请证书有一些限制（很难学习，但是正常使用不会遇到该限制）。而 v2 版本，对于通配符证书，多了一个限制，New Orders per Account（每个证书订单数限制）。

这两个细节，后续再仔细研究。

#### 实践

我迫不及待想使用 Certbot 申请通配符证书，升级 Certbot 版本运行下列命令：

```bash
 ./certbot-auto certonly  -d *.newyingyong.cn --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory 
```

介绍下相关参数：

- certonly，表示安装模式，Certbot 有安装模式和验证模式两种类型的插件。
- --manual  表示手动安装插件，Certbot 有很多插件，不同的插件都可以申请证书，用户可以根据需要自行选择
- -d 为那些主机申请证书，如果是通配符，输入 *.newyingyong.cn（可以替换为你自己的域名）
- --preferred-challenges dns，使用 DNS 方式校验域名所有权
- --server，Let's Encrypt ACME v2 版本使用的服务器不同于 v1 版本，需要显示指定。

接下去就是命令行的输出：

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): ywdblog@gmail.com

-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A

Plugins selected: Authenticator manual, Installer None
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for newyingyong.cn

-------------------------------------------------------------------------------
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
-------------------------------------------------------------------------------
(Y)es/(N)o: y
```

上述有两个交互式的提示：

- 是否同意 Let's Encrypt 协议要求
- 询问是否对域名和机器（IP）进行绑定

确认同意才能继续。

继续查看命令行的输出，非常关键：

```bash
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.newyingyong.cn with the following value:

2_8KBE_jXH8nYZ2unEViIbW52LhIqxkg6i9mcwsRvhQ

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
Waiting for verification...
Cleaning up challenges 
```

要求配置 DNS TXT 记录，从而校验域名所有权，也就是判断证书申请者是否有域名的所有权。

上面输出要求给 _acme-challenge.newyingyong.cn  配置一条 TXT 记录，在没有确认 TXT 记录生效之前不要回车执行。

我使用的是阿里云的域名服务器，登录控制台操作如下图：

![img](https:////upload-images.jianshu.io/upload_images/234392-69d92733b0c8f97e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

https.png

然后输入下列命令确认 TXT 记录是否生效：

```bash
$ dig  -t txt  _acme-challenge.newyingyong.cn @8.8.8.8    

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;_acme-challenge.newyingyong.cn.        IN      TXT

;; ANSWER SECTION:
_acme-challenge.newyingyong.cn. 599 IN  TXT     "2_8KBE_jXH8nYZ2unEViIbW52LhIqxkg6i9mcwsRvhQ"
```

确认生效后，回车执行，输出如下：

```bash
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/newyingyong.cn/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/newyingyong.cn/privkey.pem
   Your cert will expire on 2018-06-12. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

恭喜您，证书申请成功，证书和密钥保存在下列目录：

```bash
$ tree /etc/letsencrypt/archive/newyingyong.cn 
.
├── cert1.pem
├── chain1.pem
├── fullchain1.pem
└── privkey1.pem
```

然后校验证书信息，输入如下命令：

```bash
openssl x509 -in  /etc/letsencrypt/archive/newyingyong.cn/cert1.pem -noout -text 
```

关键输出如下：

```bash
X509v3 Subject Alternative Name: 
    DNS:*.newyingyong.cn
```

