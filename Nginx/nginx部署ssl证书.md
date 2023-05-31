## Nginx部署ssl证书

1、使用远程登录工具，登录Nginx服务器。

2、执行以下命令，在Nginx安装目录（默认为/usr/local/nginx/conf）下创建一个用于存放证书的目录

```
cd /usr/local/nginx/conf  #进入Nginx默认安装目录。如果您修改过默认安装目录，请根据实际配置调整。
mkdir cert  #创建证书目录，命名为cert。
```

3、使用远程登录工具附带的本地文件上传功能，将证书文件和私钥文件上传到Nginx服务器的证书目录（示例中为/usr/local/nginx/conf/cert）。

4、编辑Nginx配置文件nginx.conf，修改与证书相关的配置。

> 执行以下命令，打开配置文件。

```
vim /usr/local/nginx/conf/nginx.conf
```

**重要** nginx.conf默认保存在/usr/local/nginx/conf目录下。如果您修改过nginx.conf的位置，请将`/usr/local/nginx/conf/nginx.conf`替换成修改后的位置。您可以执行`nginx -t`，查看nginx的配置文件路径。

>按**i**键进入编辑模式

>在nginx.conf中定位到HTTP协议代码片段（`http{}`），并在HTTP协议代码里，按照以下代码示例，修改server属性配置。

```
#以下属性中，以ssl开头的属性表示与证书配置有关。
server {
    #配置HTTPS的默认访问端口为443。
    #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
    #如果您使用Nginx 1.15.0及以上版本，请使用listen 443 ssl代替listen 443和ssl on。
    listen 443 ssl;

    #填写证书绑定的域名
    server_name <yourdomain>;
    root html;
    index index.html index.htm;

    #填写证书文件名称
    ssl_certificate cert/<cert-file-name>.pem;
    #填写证书私钥文件名称  
    ssl_certificate_key cert/<cert-file-name>.key;

    ssl_session_timeout 5m;
    #表示使用的加密套件的类型
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的TLS协议的类型，您需要自行评估是否配置TLSv1.1协议。
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;

    ssl_prefer_server_ciphers on;
    location / {
        #Web网站程序存放目录
        root html;
        index index.html index.htm;
    }
}
```

>**可选：**设置HTTP请求自动跳转HTTPS。

如果您希望所有的HTTP访问自动跳转到HTTPS页面，则可以在需要跳转的HTTP站点下添加以下`rewrite`语句。

**重要** 以下代码片段需要放置在nginx.conf文件中`server {}`代码段后面，即设置HTTP请求自动跳转HTTPS后，nginx.conf文件中会存在两个`server {}`代码段。

```
server {
    listen 80;
    #填写证书绑定的域名
    server_name <yourdomain>;
    #将所有HTTP请求通过rewrite指令重定向到HTTPS。
    rewrite ^(.*)$ https://$host$1;
    location / {
        index index.html index.htm;
    }
}
```

>修改完成后，按**Esc**键、输入**:wq!**并按**Enter**键，保存修改后的配置文件并退出编辑模式

5、执行以下命令，重启Nginx服务

```
cd /usr/local/nginx/sbin  #进入Nginx服务的可执行目录。
./nginx -s reload  #重新载入配置文件。
```

如果重启Nginx服务时收到报错，您可以使用以下方法进行排查：

- 报错`the "ssl" parameter requires ngx_http_ssl_module`：您需要重新编译Nginx并在编译安装的时候加上`--with-http_ssl_module`配置。
- 报错`"/cert/3970497_demo.aliyundoc.com.pem":BIO_new_file() failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/cert/3970497_demo.aliyundoc.com.pem','r') error:2006D080:BIO routines:BIO_new_file:no such file)`：您需要去掉证书相对路径最前面的`/`。例如，您需要去掉`/cert/cert-file-name.pem`最前面的`/`，使用正确的相对路径`cert/cert-file-name.pem`。