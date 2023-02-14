#### server
server {
    listen 80;
    server_name localhost;
    
    access_log /var/log/nginx/domain.access.log main;
    error_log /var/log/nginx/domain.error.log error;

    fastcgi_ignore_client_abort     on;
    proxy_ignore_client_abort       on;
    fastcgi_connect_timeout 300;
  
    fastcgi_read_timeout 600;
    fastcgi_send_timeout 600;
    proxy_connect_timeout 600;
    proxy_read_timeout 600;

    location ~ .*\.(html|htm)$ {
        add_header Cache-Control "private, no-store, no-cache, must-revalidate, proxy-revalidate";
    }

    location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css|json|svg|ttf)$ {
        expires      7d;
    }

    location /api/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:9000;
    }

    location /child/post {
        index  index.html index.htm;
        try_files $uri $uri/ /child/post/index.html;
    }
    location ^~/weiboimg/ {
        if ($request_uri ~ ^\/weiboimg\/(.*?)\/(.*)) {
            set $img_tuchuang_host $1;
        }

        proxy_set_header referer https://weibo.com;
        proxy_set_header Origin 'https://weibo.com';
        rewrite ^/weiboimg/(.*?)/(.*) /$2 break;
        proxy_pass https://$img_tuchuang_host;
    }
}

示例：
`curl localhost:9081/weiboimg/qyapi.weixin.qq.com/cgi-bin/user/get?access_token=oh`，`$1`返回`qyapi.weixin.qq.com`。

#### cors
add_header Access-Control-Allow-Origin * always;
add_header Access-Control-Allow-Credentials true always;
add_header Access-Control-Allow-Methods "GET,POST,OPTIONS,PUT,DELETE" always;
add_header Access-Control-Allow-Headers "DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,auth-signature" always;

#### deny
allow 127.0.0.1;
allow 192.168.0.0/24;
deny all;
