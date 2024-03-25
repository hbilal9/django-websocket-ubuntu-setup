# django-websocket-ubuntu-setup

## upgrading python

### step 01 - check python version

```
python3 -V
```
output:- 3.10
suppose here im upgrading to 3.11
Install Python 3.11
### step 02 - Add the repository and update
```
sudo add-apt-repository ppa:deadsnakes/ppa
```
```
sudo apt-get update
```
```
apt-get update
```
Verify the updated Python packages list using this command.
```
apt list | grep python3.11
```
### step 03 - Install the Python 3.11 package using apt-get
```
sudo apt-get install python3.11
```
### step 04 - Add Python 3.10 & Python 3.11 to update-alternatives
```
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
```
```
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 2
```

### step 05 -  Update Python 3 for point to Python 3.11
```
sudo update-alternatives --config python3
```
```
python3 -V
```
You should get Python 3.11 as an output.


## Step 1 — Installing the Packages from the Ubuntu Repositories

```
sudo apt update
```
```
sudo apt install python3-venv libpq-dev postgresql postgresql-contrib nginx curl
```

## Step 2 — Creating a Python Virtual Environment for your Project
```
python3 -m venv venv
```

```
source venv/bin/activate
```

install dependences
```
pip install django gunicorn uvicorn psycopg2-binary
```
sometime uvicorn not workng try this
```
pip install 'uvicorn[standard]'
```

## Step 3 — Configuring Django Project
```
sudo ufw allow 8000
```

```
gunicorn --bind 0.0.0.0:8000 project.asgi -w 4 -k uvicorn.workers.UvicornWorker
```

```
uvicorn project.asgi:application --host 0.0.0.0 --port 8000
```

```
deactivate
```

## Step 4 — Creating systemd Socket and Service Files for Gunicorn
```
sudo nano /etc/systemd/system/gunicorn.socket
```
Paste it inside gunicorn.socket
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

Next, create and open a systemd service file for Gunicorn with sudo privileges in your text editor. The service filename should match the socket filename exception for the extension:
```
sudo nano /etc/systemd/system/gunicorn.service
```
Paste it inside gunicorn.service
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/var/www/backend/core
ExecStart=/var/www/backend/venv/bin/gunicorn \
          --access-logfile - \
          -k uvicorn.workers.UvicornWorker \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          core.asgi:application

[Install]
WantedBy=multi-user.target
```

You can now start and enable the Gunicorn socket. This will create the socket file at /run/gunicorn.sock now and at boot. When a connection is made to that socket, systemd will automatically start the gunicorn.service to handle it:
```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

## Step 5 — Checking for the Gunicorn Socket File
```
sudo systemctl status gunicorn.socket
```

```
Output
● gunicorn.socket - gunicorn socket
     Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor prese>
     Active: active (listening) since Fri 2020-06-26 17:53:10 UTC; 14s ago
   Triggers: ● gunicorn.service
     Listen: /run/gunicorn.sock (Stream)
      Tasks: 0 (limit: 1137)
     Memory: 0B
     CGroup: /system.slice/gunicorn.socket
```

Reload Servers
```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

Step 6 — Configure Nginx to Proxy Pass to Gunicorn
```
sudo nano /etc/nginx/sites-available/projectname
```

Paste it into it
```
server {
    server_name api.your-site.com;
    client_max_body_size 100M;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /var/www/backend/core;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
       # proxy_pass http://localhost:8000;
    }

    location /ws/ {
        #proxy_pass http://localhost:8000;
        proxy_pass http://unix:/run/gunicorn.sock;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable nginx configuration
```
sudo ln -s /etc/nginx/sites-available/projectname /etc/nginx/sites-enabled
```
Test nginx
```
sudo nginx -t
```
Restart Nginx
```
sudo systemctl restart nginx
```

Finally, you’ll need to open the firewall to normal traffic on port 80. Since you no longer need access to the development server, you can remove the rule to open port 8000 as well:

```
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
```

restart gunicorn
```
sudo systemctl restart gunicorn
```
If you change Gunicorn socket or service files, reload the daemon and restart the process by typing:
```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn.socket gunicorn.service
```

## Conclusion
In this guide, you’ve set up an ASGI Django project in its own virtual environment. 
