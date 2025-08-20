# Flask-waitress-Nginx

## Installation

The steps are taken on a Ubuntu box.

```shell
sudo apt update
sudo apt install nginx-full \
		python3-flask python3-flask-restful \
		python3-waitress
```

## Configuration

#### Nginx

File: /etc/nginx/etc/nginx/sites-enabled/default

```nginx
http {
   server_tokens off;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       443 ssl http2 default_server;
        #listen       [::]:443 ssl http2 default_server;
        server_name  git-repos.iaas.local;
        root         /data/git-repos;

        ssl_certificate "/etc/nginx/nginx.crt";
        ssl_certificate_key "/etc/nginx/nginx.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        client_max_body_size 0;

        location ~ ^/findmac {
            auth_basic "Restricted";
            auth_basic_user_file /etc/nginx/basic-auth.user;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Host $host:$server_port;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_pass http://127.0.0.1:10808;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```

#### waitress-serve starting script

File: ./start-server.sh

```shell
#!/bin/bash

BASE_DIR=$(cd $(dirname $0); pwd);
cd $BASE_DIR/app;
waitress-serve --listen='127.0.0.1:10808' wsgi:app > $BASE_DIR/app.log 2>&1 &
```

### Write a Flask app

File: ./app/wsgi.py

```python
import flask as fk

app = fk.Flask(__name__)

@app.route('/')
def index():
    return 'Hello, world!\n'
```

