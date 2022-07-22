## ai2.glis 域名設定
1. 新增nginx設定檔案
在host上面（**不需要**到container裡面）這個路徑裡面`/data/nginx/user_conf.d`新增一個檔案叫做`xxdomain.com.tw.conf`，注意`.conf`不可省略，沒有加的話不會被套用到nginx上面。
檔案內容範例如下，把`xxdomain.com.tw`改成你的域名，並把port改成你的服務在本機端上的port
```
server {
    listen [::]:443 ssl;
    listen 443 ssl;

    # Domain names this server should respond to.
    server_name xxdomain.com.tw; # 修改這個域名

    location / {
        proxy_pass http://127.0.0.1:1234; # 修改這個port
        client_max_body_size 50M;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # Load the certificate files.
    ssl_certificate         /etc/letsencrypt/live/xxdomain.com.tw/fullchain.pem; # 修改這個域名
    ssl_certificate_key     /etc/letsencrypt/live/xxdomain.com.tw/privkey.pem; # 修改這個域名
    ssl_trusted_certificate /etc/letsencrypt/live/xxdomain.com.tw/chain.pem; # 修改這個域名

    # Load the Diffie-Hellman parameter.
    ssl_dhparam /etc/letsencrypt/dhparams/dhparam.pem;
}
```

2. 新增好檔案之後，重新啟動docker即可套用新的設定
```bash
docker restart nginx-certbot
```

3. 如果有新的域名與服務要放上去，重複以上步驟即可，記得將域名以及port修改至符合你需求