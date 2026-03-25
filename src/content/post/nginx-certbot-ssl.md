---
title: "使用 nginx + certbot 快速整合 SSL 至自架 HTTP 網站"
description: "使用 nginx 與 certbot 快速為自架的 http 網站整合 Let's Encrypt SSL 憑證，含 docker compose 設定步驟。"
publishDate: "2023-12-03"
tags: ["Web"]
coverImage:
  src: "https://files.sakana.tw/blog/https-with-nginx-and-certbot/Title.png"
  alt: "cover image"
---



近期想搞一個用來管理圖片的服務，找到了 [immich](https://github.com/immich-app/immich) 這款 self hosted 的圖片管理工具，在 localhost 試用了一小段時間，感覺很不錯，不管是 UI 還是搜尋功能都沒得挑剔，唯一可惜的是搜尋功能沒有整合 OCR。

直到當我想把他放上雲端，方便從外網存取時，才發現一個大問題，這個通過 docker compose 架起來的 server，只支援 http，雪特，這樣我不敢登入 server 的 web 介面耶...，於是開始了這工具的 https 升級之旅。

雖然這是我設定 immich 的記錄，但大部分的 http server 應該都管用。

## 事前評估
首先做個整理，方便收斂出實際的做法

- 手上的資源
    - 從 gandi 購買，目前由 cloudflare DNS 託管的 domain - sakana.tw。

- 取得憑證的幾個方法
    1. 跟 gandi 買萬用憑證，人工塞憑證到 server 上：太貴了
    2. 從 cloudflare 抓萬用憑證，人工塞憑證到 server 上：可行
    3. 從 lets encrypt 申請憑證，網路上有一堆自動化更新憑證的教學資源：可行

- 將 http 的 server 升級成 https
    1. 掛一個 nginx，將收到的 https 流量導向 http 的 immich server。
       要解決 mitm 的問題，這應該算是最簡單的解法，

## 作法簡介
最後我決定試試自動化更新憑證的方式，做法如下
1. 掛 nginx
    - immich server 的 docker-compose.yml 加入 nginx
    - nginx.conf 設定將 https 導向 immich server 
2. 掛 certbot 自動跟 lets encrypt 申請憑證
    - 掛載 share volume 指向 nginx 的 ssl 資料夾以及 certbot 產生憑證的資料夾
3. 開始下指令抓憑證下來


## 步驟
### 準備 DNS record
> 這部分每個人應略有不同，個人是使用 cloudflare DNS 託管服務

於 cloudflare DNS 頁面設定 immich.sakana.tw 的 A record 為雲端主機的 ip 即可。

### 準備相關設定檔

#### 將 nginx 以及 certbot 加入至 `./docker-compose.yml` 中

``` yaml
# Definition of immich compoents
# ...
# ...
# nginx and certbot settings
  nginx:
    container_name: immich-nginx
    image: nginx:1.22
    restart: always
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certbot/www:/var/www/certbot/:ro
      - ./certbot/conf/:/etc/nginx/ssl/:ro
    ports:
      - 80:80
      - 443:443
      - 2080:2080

  certbot:
    image: certbot/certbot:latest
    volumes:
      - ./certbot/www/:/var/www/certbot/:rw
      - ./certbot/conf/:/etc/letsencrypt/:rw
# ...
# ...
```

#### 設定 `nginx.conf`
``` nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
  worker_connections  1024;
}

http {
  server {
    listen 80;
    listen [::]:80;

    server_name immich.sakana.tw
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://immich.sakana.tw$request_uri;
    }
  }
  # Remove the comment sign # after getting the ssl cert from let's encrypt
  # server {
  #     listen 443 default_server ssl http2;
  #     listen [::]:443 ssl http2;

  #     server_name immich.sakana.tw;

  #     ssl_certificate /etc/nginx/ssl/live/immich.sakana.tw/fullchain.pem;
  #     ssl_certificate_key /etc/nginx/ssl/live/immich.sakana.tw/privkey.pem;

  #     location / {
  #       proxy_pass http://immich-server:3001;
  #     }
  # }
}

```
### 執行指令

Step 1: 執行以下指令
``` bash
# 更新、下載 image
docker compose pull

# 將 server 跑起來
docker compose up -d

# 測試是否能向 let's encrypt 申請、下載憑證 (dry run)
docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d immich.sakana.tw

# 實際向 let's encrypt 申請、下載憑證
docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ -d immich.sakana.tw

```
Step 2: 調整 nginx 設定並重啟服務
``` bash
# 調整 nginx 設定，將原先註解掉的 https 設定取消註解
vim nginx.conf

# 重啟服務
docker compose stop
docker compose up -d
# docker composer restart 似乎是不會吃到編輯過後的 docker-compose.yml 檔案，導致我先前測試亂試時 debug 很久...，所以我推薦用上面兩個指令來重啟。

```

至此完成 immich 的 https 服務設定了，可喜可賀。

## 參考資料
- [HTTPS using Nginx and Let's encrypt in Docker](https://mindsers.blog/en/post/https-using-nginx-certbot-docker/)
    - 基本上是參考這一篇設定的
- [Docker 一分鐘完成自動更新 SSL 証書的 nginx-proxy 設置](https://blog.256pages.com/docker-nginx-proxy-ssl-companion/)
    - 這篇的設定感覺稍微複雜了一點，而且 nginx-proxy 這工具我沒搞很懂，就沒仔細看了。


