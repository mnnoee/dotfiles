<p align="center">
  <a href="README.md">English</a> • 
  <a href="README_ru.md">Русский</a>
</p>
# Virtual Hosting as an Art Form

## Minimal Configuration for Returning 444 Error

The simplest Nginx configuration that always returns a 444 error when accessing the site by IP address:


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

**Why is this needed?**  
You can direct logs to a separate file → connect fail2ban → Ban everyone by IP addresses.

---

## Example Configuration for the Domain смешнявки.рф (smeshnyavki.rf)

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

    root /PATH/YOU/ROOT_HOME/WEBSITE; # <-- you need to specify your own path  
    index index.html;  

    ## You also need to specify your own path for logs
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
    gzip_disable "msie6";  # <-- Why not?

    location ~ /\. {  
        deny all;  
        return 403;  
    }  

    # Caching - a compromise between versioning and caching once a year
    location ~* \.(?:css|html?|ico|js|md|png|sample|txt|webmanifest|webp|woff2|xml)$ {  
        expires 30d;  
        add_header Cache-Control "public, no-transform";  

        # Disable compression for already compressed files
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

Where `/PATH/YOU/ROOT_HOME` is the path to my website's root directory.

This configuration uses:
- Redirect from subdomains
- Forced HTTPS
- Separate logs for error analysis (including 404s)
- Compression and caching optimization
- Protection of hidden files (e.g., for .git)
