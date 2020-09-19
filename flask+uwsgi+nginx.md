# Flask+uWSGI+Nginx

## Installation

The steps are taken on a Ubuntu 20.04.

```shell
sudo apt update
sudo apt install nginx-full \
		python3-flask python3-flask flask-restful \
		uwsgi uwsgi-plugin-python3
```

## Configuration

#### Nginx

File: /etc/nginx/etc/nginx/sites-enabled/default

```nginx
server {
    # ...
    
    # Add a new location block
    # uWSGI will receive a request to URL: /api/...
    location /api/ {
		include uwsgi_params;
		# Refer to uWSGI doc for the correct socket path 
        uwsgi_pass unix:/run/uwsgi/app/api/socket;
	}
    
    # ...
}
```

#### uWSGI

File: /etc/uwsgi/apps-available/api.ini

```ini
[uwsgi]
wsgi-file = /path/to/wsgi.py
plugin = python3
enable-threads = true
```

### Write a Flask app

File: /path/to/wsgi.py

```python
#!/usr/bin/python3

import flask as fk
import flask_restful as fr

# There must be a callable named application
application = fk.Flask(__name__)
api = fr.Api(app)
# more code ...
```

