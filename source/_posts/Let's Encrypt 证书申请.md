---
title: 学会造一个句子
date: 2024-06-112 16:15:40
tags:
---

Let's Encrypt 提供免费的 SSL/TLS 证书，可以通过`Certbot`工具轻松申请和续签。

### 安装
在大多数 Linux 发行版上，`Certbot` 可以通过包管理器安装。
```
sudo apt update
sudo apt install certbot
```

### 申请证书
通常情况下，Certbot的证书需要在域名解析到的服务器上申请，因为Let’s Encrypt CA需要验证您对该域名的控制权。具体来说，CA会向您的域名发送验证请求，这个请求需要由您的服务器处理并返回正确的响应。

#### 验证方法
1. HTTP-01 验证
    * 这是最常用的验证方法，Certbot会在您的服务器上创建一个临时文件，然后Let’s Encrypt CA会尝试访问这个文件以验证您对域名的控制权。这个请求会发送到域名解析到的服务器，因此必须在该服务器上运行Certbot。(意思就是certbot得安装在server上)
2. DNS-01 验证
    * 这种方法通过在您的DNS记录中添加一个特定的TXT记录来验证您对域名的控制权。这种方法不需要在域名解析到的服务器上运行Certbot，可以在任意服务器上操作，只要您能修改域名的DNS记录。
3. TLS-ALPN-01 验证
    * 这种方法通过在特定端口（通常是443）上进行验证。它也需要在域名解析到的服务器上运行Certbot。

##### 使用DNS-01方式验证
1. 申请证书
    ```
    certbot certonly --manual --preferred-challenges dns -d example.com
    ```
    * `certbot certonly`：这是 Certbot 命令，用于生成证书（certificates only），不会配置 Web 服务器。
    * `--manual`：使用手动模式。这意味着 Certbot 不会自动配置或验证域名，而是需要用户手动完成 DNS 验证步骤。
    * `--preferred-challenges dns`：指定首选的挑战类型为 DNS。Certbot 将通过添加 DNS TXT 记录来验证你对域名的控制权。
    * `-d example.com`：指定你想要生成证书的域名（在这个例子中是 example.com）。你可以用逗号分隔的方式指定多个域名。
2. 登录云平台域名解析添加TXT记录
    * 当上面命令回车后，会收到类似如下提示，要求添加 TXT 解析记录：
    ```
    Please deploy a DNS TXT record under the name
    _acme-challenge.example.com with the following value:

    667drNmQL3vX6bu8YZlgy0wKNBlCny8yrjF1lSaUndc

    Once this is deployed,
    Press ENTER to continue
    ```
    * 根据上面提示，登录云商后台（比如阿里云、腾讯云等等），添加的 TXT 记录.
    * 由于 DNS 记录不会马上生效，所以等配置好，通过`dig +short -t txt _acme-challenge.example.com`命令验证DNS生效后再按回车
3. 查看ssl证书文件
    * 当上面命令回车后，ssl文件将生成在目录`/etc/letsencrypt/live/example.com/`下
    ```
    IMPORTANT NOTES:
        - Congratulations! Your certificate and chain have been saved at:
        /etc/letsencrypt/live/example.com/fullchain.pem
        Your key file has been saved at:
        /etc/letsencrypt/live/example.com/privkey.pem
        Your cert will expire on 2020-03-11. To obtain a new or tweaked
        version of this certificate in the future, simply run certbot
        again. To non-interactively renew *all* of your certificates, run
        "certbot renew"
        - If you like Certbot, please consider supporting our work by:

    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
    Donating to EFF:                    https://eff.org/donate-le
    ```
    > Certbot 生成的这四个文件分别是:
    * cert.pem：这是域名的 SSL 证书文件。它包含你的域名经过签名的公钥证书。
    * chain.pem：这是中间证书文件。它包含中间证书颁发机构的公钥证书，用于建立信任链。
    * fullchain.pem：这是完整的证书链文件，包含了你的域名证书和中间证书。(nginx使用)
    * privkey.pem：这是私钥文件，用于加密和解密数据，必须保密。(nginx使用)
4. nginx配置域名ssl
    ```
    server {
        listen 80;
        server_name example.com;

        location / {
            return 301 https://$server_name$request_uri;
        }
    }

    server {
        listen 443 ssl;
        server_name example.com;

        ssl_certificate /mnt/disk1/ssl/example.com/fullchain.pem;
        ssl_certificate_key /mnt/disk1/ssl/example.com/privkey.pem;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        ssl_session_cache shared:SSL:10m;

        # 这里是反向代理，酌情处理
        location / {
            proxy_pass http://localhost:8089;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        access_log /var/log/nginx/example_com.access.log;
        error_log /var/log/nginx/example_com.error.log;
    }
    ```

    ### 证书续期