# 伺服器--Nginx

# **nginx 基礎設定**

**安裝 nginx**
ubuntu 使用 apt-get

    apt-get update
    apt-get install nginx

如果是 osx，請使用內建的套件管理工具 Homebrew 來安裝

    brew install nginx

完成之後就可使用 nginx

    # 啟動 nginx
    nginx
    # 加上 -s option 來下指令
    # 停止 nginx
    nginx -s stop
    # 重新讀取設定檔
    nginx -s reload 
## **自動啟動**

ubuntu 環境下，使用 `apt-get` 安裝的話，會自動把 nginx 啟動的 bash 檔放到 /etc/init.d/ 下方，也就是重開機後會自動執行 nginx。
osx 環境下，請參考[這篇](http://derickbailey.com/2014/12/27/how-to-start-nginx-on-port-80-at-mac-osx-boot-up-log-in/)
**設定檔路徑**
nginx 的設定檔名為 `ngix.conf`，會依據安裝方式導致被放置的路徑不同，可以透過 `nginx -t` 來查詢

    nginx -t
    # nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
    # nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
    # 表示設定檔路徑為 /usr/local/etc/nginx/nginx.conf

**ngix.conf**
要先了解 conf 檔案的架構及一些名詞。
config 檔是由一連串的 **directive** 所組成的。directive 針對特定的部分作設定，分為兩種：simple directive 及 block directive。
simple directive 要以分號 `;` 結尾，而 block directive 會有一組大括號 `{}`，包著其他的 directive（simple 或是 block）。
因此設定檔中顯眼的 `http`、`server` 及 `location` 都是 block directive。它們有著從屬關係

    http {
    # server 一定在 http 裡面
    server {
    # location 一定在 server 裡面
    location {}
    }
    }

而最底層的 block directive 只會有兩種：`http` 及 `event`，稱之為 **main context**。
simple directive 會對把它包起來的 block directive 作設定，舉最簡單的例子：

    
    # listen 是個 simple directive
    # 對 server 這個 block directive 作設定
    # 指定它監聽 port 80
    server {
    listen 80;
    }
## **設定 proxy server**

如果 server app 是透過某個 port 作為介面（例如：[express.js](http://expressjs.com/en/index.html)）。
這時可用 nginx 來作為 proxy server。
情境：
app 使用的 port 為 2368，要讓當連到 `www.example.com` 時，交由這個 app 來處理。
先修改 `/etc/hosts`（需 root 權限），讓 `www.example.com` 指向本機

    # /etc/hosts
    127.0.0.1 www.example.com

再到 `nginx.conf` 把 www.example.com 指到 2368 這個 port

    server {
    server_name www.example.com;
    location / {
    proxy_pass 127.0.0.1:2368;
    }
    }
## **設定 load balancer**

使用 upstream 這個 directive。
假設 2 個 ip：`192.168.1.1` 及 `192.168.1.2` 的 2368 port 都有相同的 app。

    http {
    upstream my_upstream {
    server 192.168.1.1:2368;
    server 192.168.1.2:2368;
    }
    server {
    server_name www.example.com;
    location / {
    proxy_pass http://my_upstream; # 指到設定的 upstream 及 protocol
    }
    }
    }

這樣子 nginx 會使用預設的 round-robin 原則來作 load balance，另外還有兩種 `least-connected` 及 `ip-hash` 原則可用，詳情請見[官方文件](http://nginx.org/en/docs/http/load_balancing.html)的說明。

## **小結**

照上述的方式，打開瀏覽器連到 www.example.com，就能夠連到 port 2368 的 app 了。另外還有一些進階的 directive，包括 `expires`、`gzip`、`proxy_cache` 等等，在之後介紹。


# [**設定 Let's Encrypt HTTPS nginx certbot 自動更新 教學**](https://blog.hellojcc.tw/2018/05/02/setup-https-with-letsencrypt-on-nginx/)

