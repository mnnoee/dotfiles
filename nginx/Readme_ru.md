
<p align="center">
  <a href="Readme.md">English</a> • 
  <a href="Readme_ru.md">Русский</a>
</p>

# Виртуальный хостинг как искусство

## Минимальный конфиг для возврата 444 ошибки

Самый простой конфиг Nginx, который всегда возвращает 444 ошибку, когда идёт обращение к сайту по ip-адресу:

```nginx
~ → cat /etc/nginx/http.d/default.conf

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    ssl_certificate /etc/nginx/fake.crt;
    ssl_certificate_key /etc/nginx/fake.key;

    return 444;
}
```

**Зачем это нужно?**  
Можно вывести логи в отдельный файл → подключить fail2ban → Банить всех по IP-адресам.

---

## Пример конфига для домена смешнявки.рф

```nginx
~ # cat /etc/nginx/http.d/smeshnyavki.conf

server {  
    listen 443 ssl;  
    listen [::]:443 ssl;  

    server_name .смешнявки.рф .xn--b1afkhhhz2d6c.xn--p1ai;  

    ssl_certificate /etc/letsencrypt/live/xn--b1afkhhhz2d6c.xn--p1ai/fullchain.pem;  
    ssl_certificate_key /etc/letsencrypt/live/xn--b1afkhhhz2d6c.xn--p1ai/privkey.pem;  
    ssl_protocols TLSv1.2 TLSv1.3;  

    if ($host !~* ^(смешнявки\.рф|xn--b1afkhhhz2d6c\.xn--p1ai)$) {  
        return 301 https://смешнявки.рф$request_uri;  
    }  

    root /PATH/YOU/ROOT_HOME/WEBSITE; # <-- необходимо указать свой путь  
    index index.html;  

    ## Для логов тоже нужно указать свой путь
    access_log /PATH/YOU/ROOT_HOME/logs/access.log;
    error_log /PATH/YOU/ROOT_HOME/logs/error.log;

    gzip on;  
    gzip_vary on;  
    gzip_proxied any;  
    gzip_comp_level 6;  
    gzip_min_length 256;  
    gzip_types  
        text/plain  
        text/css  
        text/javascript  
        application/javascript  
        application/x-javascript  
        application/json  
        application/xml  
        application/xml+rss  
        image/svg+xml  
        application/x-web-app-manifest+json;  
    gzip_disable "msie6";  # <-- Почему бы и нет?

    location ~ /\. {  
        deny all;  
        return 403;  
    }  

    # Кеширование - компромисс между версионностью и кешем раз в год
    location ~* \.(?:css|html?|ico|js|md|png|sample|txt|webmanifest|webp|woff2|xml)$ {  
        expires 30d;  
        add_header Cache-Control "public, no-transform";  

        # Отключаем сжатие для уже сжатых файлов
        if ($request_uri ~* \.(webp|woff2|gz)$) {  
            gzip off;  
        }  
        try_files $uri $uri/ =404;  
    }  

    location / {  
        try_files $uri $uri/ =404;  
    }  
}  

server {  
    listen 80;  
    listen [::]:80;  
    server_name .смешнявки.рф .xn--b1afkhhhz2d6c.xn--p1ai;  
    return 301 https://смешнявки.рф$request_uri;
}
```

Где `/PATH/YOU/ROOT_HOME` - путь к корневой директории моего сайта.

В данном конфиге используется:
- Редирект с поддоменов
- HTTPS принудительно
- Отдельные логи для анализа ошибок (в том числе 404)
- Оптимизация сжатия и кеширования
- Защита скрытых файлов (например для .git)
