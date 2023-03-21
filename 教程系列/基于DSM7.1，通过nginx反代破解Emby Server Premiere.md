# 基于DSM7.1，通过nginx反代破解Emby Server Premiere

## 一、前言

**<font color="red">！！！破解教程仅供学习参考，务必支持正版软件，构建良好的软件版权生态！！！</font>**

Emby Server是一个开源的流媒体中心软件，可以非常方便的管理和维护电影，电视剧集等媒体库，并生成强大美观的海报墙。并且全平台的客户端，使其跨设备观看变得非常便利。但是服务端的部分插件功能，以及视频转码功能，需要会员才能解锁。针对尝鲜的个人用户来说，这个会员着实不便宜，经过搜寻相关资料后，确定其会员验证是通过与`mb3admin.com`这个网站通信完成，因此本教程将通过nginx的反向代理+hosts进行域名劫持来实现会员的"破解"。

## 二、准备工具

* DMS7.1群晖系统（本质上任意一个nginx服务都可以，DSM本身的web就是通过nginx实现的，因此不需要单独在开一个nginx服务了）
* 文本编辑软件（Notepad++，subline text，vs code均可）
* ssh工具（Xshell，MobaXterm，putty，finalshell等）

## 三、破解步骤

### 1、申请ssl证书

由于Emby Server需要和`mb3admin.com`进行https通信，因此我们需要针对该域名申请ssl证书。

推荐 GMCert.org https://www.gmcert.org/subForm。按以下申请步骤进行：

![](https://user-images.githubusercontent.com/35327600/201810752-d921d035-c76d-439c-9160-e320a36af37d.jpg)

* CA证书如果此前PC端已安装过，则可不用重复下载安装；如果是全新安装，则尽量安装一下，双击证书，点击安装，然后手动选择证书存储，将证书安装到“受信任的根证书颁发机构”即可。
* 主题名称为此次所需的二级域名`mb3admin.com`

点开下方的“**高级选项**”，并按照以下进行配置：

![](https://user-images.githubusercontent.com/35327600/201813309-cc83b055-df1d-4b61-bda9-103939c7cc2c.png)

* 在主题备用名称中，填入上图内容，将泛域名也填充进去，保证任意三级域名都处于ssl证书授权范围
* 密钥用途和扩展密钥用途按图中红框勾选即可
* 点击“**签发证书**”，会下载一个包含密钥和证书的压缩包，解压并保存

### 2、上传密钥和证书

将上一步中解压获取的证书和密钥文件，上传至群晖中

![](https://user-images.githubusercontent.com/35327600/201814624-8bc428eb-1947-430f-a27a-2301ce7379d8.png)

### 3、创建nginx代理配置

新建文件`emby_crack_nginx.conf`，粘贴以下内容：

```nginx
server {
     listen 443 ssl;
     listen [::]:443 ssl;
     server_name mb3admin.com;
     ssl_certificate /volume1/web/mb3admin.com/mb3admin.com.cert.pem;
     ssl_certificate_key /volume1/web/mb3admin.com/mb3admin.com.key.pem;
     ssl_session_timeout 5m;
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
     ssl_prefer_server_ciphers on;
     location = /webdefault/images/logo.jpg {
     alias /usr/syno/share/nginx/logo.jpg;
            }
     location @error_page {
     root /usr/syno/share/nginx;
     rewrite (.*) /error.html break;
            }
     location ^~ /.well-known/acme-challenge {
     root /var/lib/letsencrypt;
     default_type text/plain;
            }
     location / {
     rewrite ^ / redirect;
            }
     location ~ ^/$ {
     rewrite / https://$host:5001/ redirect;
            }
     add_header Access-Control-Allow-Origin *;
     add_header Access-Control-Allow-Headers *;
     add_header Access-Control-Allow-Method *;
     add_header Access-Control-Allow-Credentials true;
     location /admin/service/registration/validateDevice {
     default_type application/json;
     return 200 '{"cacheExpirationDays": 365,"message": "Device Valid","resultCode": "GOOD"}';
    }
     location /admin/service/registration/validate {
     default_type application/json;
     return 200 '{"featId":"","registered":true,"expDate":"2099-01-01","key":""}';
    }
     location /admin/service/registration/getStatus {
     default_type application/json;
     return 200 '{"deviceStatus":"0","planType":"Lifetime","subscriptions":{}}';
    }
}
```

注意：

* 由于证书和密钥的文件路径可能各有不同，在上述代码块中修改成自定义路径

  ```nginx
  # ssl_certificate /volume1/web/mb3admin.com/mb3admin.com.cert.pem;
  ssl_certificate /change/to/your/own/cert/file/path;
  # ssl_certificate_key /volume1/web/mb3admin.com/mb3admin.com.key.pem;
  ssl_certificate_key /change/to/your/own/key/file/path;
  ```

将配置文件`emby_crack_nginx.conf`拷贝至群晖的系统目录`/etc/nginx/sites-enabled`目录下。


### 4、修改hosts文件

由于Emby Server和Emby Client在验证会员时，向`mb3admin.com`进行post请求，因此需要在服务器或者客户端发出请求时，劫持到我们自行构建的nginx服务上，通过nginx发送假的验证通过的消息，实现会员资格验证成功。

因此需要在Emby Server和Emby Client所在的设备上修改hosts文件，将mb3admin.com域名直接指向群晖的IP。

* 如果家中有能修改hosts的路由设备，可在路由器中直接修改，这样就不需要在每一个子设备中进行修改了；如果没有则在以下设备中的hosts文件中加入代码块中内容；
* Emby Server所在服务端设备；
* windows系统PC端；

```bash
# hosts
# <群晖IP> mb3admin.com
# 假如群晖IP是：192.168.1.100，则如下所示
192.168.1.100 mb3admin.com
```

### 5、向Emby服务端的证书库中导入CA证书

在日志中发现会存在无法建立SsL连接的情况，爬贴后发现，是因为自签名证书不被Emby信任导致。这时候就需要我们将当初申请证书时，获取到的CA证书导入到Emby的可信任证书库中。docker 版本的话，需要先确认根证书文件是否由 host 端导入。整体操作流程，无论是套件版本还是 docker 版本，都大同小异。下面以套件版本为例进行说明。

> 一般根证书文件存储在 `/etc` 目录下，因此需要 root 权限才能完成。以下操作均在 root 用户下进行。

1. 进入证书存储的目录，以上文为例，执行命令： `cd /volume1/web/mb3admin.com` 
2. 打印证书内容，观察格式是否正确：`cat mb3admin.com.cert.pem`
    按下回车键后，屏幕将输出形如一下内容，确认文件以 `-----BEGIN CERTIFICATE-----` 开头，`-----END CERTIFICATE-----` 结尾：
    ![](https://img-blog.csdnimg.cn/img_convert/61aeee6cc39dee935294d56e3bd929a8.jpeg)
3. 将证书拷贝至对应目录，并重命名。这里以群晖7.1为例，执行以下命令：
~~~bash
sudo mkdir -p /usr/syno/etc/security-profile/ca-bundle-profile/ca-certificates/;
cp /volume1/web/mb3admin.com/mb3admin.com.cert.pem /usr/syno/etc/security-profile/ca-bundle-profile/ca-certificates/mb3admin.com.crt;
~~~

>  /usr/syno/etc/security-profile/ca-bundle-profile/ca-certificates/ 这个路径是从update-ca-certificates.sh中获取的，不同系统的路径可能不同，建议先执行 `sodu find / -name "update-ca-certificates` ，查找这个文件。例如群晖7.1中，该文件位于 /usr/syno/bin 中。使用 cat 命令查看文件内容，找到 `USERCERTSDIR=xxxxxxx` 行，"xxxxxxxx" 就是对应的路径；Ubuntu系统下，找到 LOCALCERTSDIR=xxxxx 行即可。

4. 更新根证书，执行命令：`update-ca-certificates.sh`
