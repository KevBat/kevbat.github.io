---
layout: post
title:  "Installing Taiga Docker container on an Azure VM"
date:   2023-08-12 14:12:25 -0500
categories: azure docker taiga
---

This work was done quite awhile ago so I might be missing some of the details. But this page should cover most of the steps required to get Taiga running and serving on 443.

Basic requirements:
- Azure Ubuntu VM created
- Public IP attached to VM

### Creating SSL cert on Azure VM
We will create the Let's Encrypt cert on the server. This will be mapped in the Docker container in the next steps.

Follow the steps [here](https://certbot.eff.org/instructions?ws=other&os=ubuntufocal) to setup certbot to handle the Let's Encrypt certificate (and automatic renewal).

### Modifying docker-compose.yml to use Let's Encrypt cert

Github repo is located [here](https://github.com/taigaio/taiga-docker), specifically the [docker-compose.yml](https://github.com/taigaio/taiga-docker/blob/master/docker-compose.yml) is important and what we will be modifying.

I made the following changes in `docker-compose.yml` to mount the modified nginx config and the let's encrypt certificates created on the server. I also removed the port 80 mapping. 

```
  taiga-gateway:
    image: nginx:1.19-alpine
    container_name: nginx
    ports:
      - "443:443"
    volumes:
      - ./taiga-gateway/taiga.conf:/etc/nginx/conf.d/default.conf
      - taiga-static-data:/taiga/static
      - taiga-media-data:/taiga/media
      - /etc/letsencrypt/live/taiga.example.com/fullchain.pem:/etc/nginx/certs/fullchain.pem
      - /etc/letsencrypt/live/taiga.example.com/privkey.pem:/etc/nginx/certs/privkey.pem
    networks:
      - taiga
    depends_on:
      - taiga-front
      - taiga-back
      - taiga-events
```

### Adding additional configuration in .env

I only need to change the top few lines of the `.env` file for Docker. But the rest can be changed too if you wish to connect to an external database or set up email.

```
# Taiga's URLs - Variables to define where Taiga should be served
TAIGA_SCHEME=https  # serve Taiga using "http" or "https" (secured) connection
TAIGA_DOMAIN=taiga.example.com:443 # Taiga's base URL
SUBPATH="" # it'll be appended to the TAIGA_DOMAIN (use either "" or a "/subpath")
WEBSOCKETS_SCHEME=wss  # events connection protocol (use either "ws" or "wss")

...

```


### Adding additional configuration in nginx

Taiga will run on https with the certificate we mapped in the `volumes` section above.

Here is what my modified config looked like:

```
#server {
#    listen 80;
#    server_name taiga.example.com;
#    return 301 https://$server_name$request_uri;
#}

server {
    listen 443 ssl;

    server_name taiga.example.com;

    ssl_certificate /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    #include /config/nginx/ssl.conf;


    client_max_body_size 100M;
    charset utf-8;

    # Frontend
    location / {
        proxy_pass http://taiga-front/;
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
    }

    # API
    location /api/ {
        proxy_pass http://taiga-back:8000/api/;
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
    }

    # Admin
    location /admin/ {
        proxy_pass http://taiga-back:8000/admin/;
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
    }

    # Static
    location /static/ {
        alias /taiga/static/;
    }

    # Media
    location /_protected/ {
        internal;
        alias /taiga/media/;
        add_header Content-disposition "attachment";
    }

    # Unprotected section
    location /media/exports/ {
        alias /taiga/media/exports/;
        add_header Content-disposition "attachment";
    }

    location /media/ {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://taiga-protected:8003/;
        proxy_redirect off;
    }

    # Events
    location /events {
        proxy_pass http://taiga-events:8888/events;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
    }

}
```

### Final thoughts

Taiga documentation isn't terrible. It will tell you how to create the initial super user, start the containers with docker compose, etc. 

The main thing after this is mapping your DNS records to the VM IP if using public IPs.
