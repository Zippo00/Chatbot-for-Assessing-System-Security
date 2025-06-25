# Chatbot-for-Assessing-System-Security
Codebase for Chatbot for Assessing System Security with GPT-3.5

# Installation
This documentation assumes you already have a virtual machine set up for hosting the application.
## 1)	Install Python

Connect to the virtual machine via SSH and update the system packages with:
```sh
sudo apt update -y;sudo apt upgrade -y 
```

Clone `pyenv` repository via git:
```sh
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```

For `pyenv` to work, `.bashrc` file needs to be updated. Run the following commands to update it and reload the shell:
```sh
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc
exec "$SHELL"
```

Install required `Python` build dependencies with:
```sh
sudo apt-get update; sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
``` 

Install `Python 3.10.6` with:
```sh
pyenv install 3.10.6
``` 

You should see:
```sh
Downloading Python-3.10.6.tar.xz...
-> https://www.python.org/ftp/python/3.10.6/Python-3.10.6.tar.xz
Installing Python-3.10.6...
``` 

Make sure the correct `Python` version is installed with:
```sh
pyenv versions
``` 

And set the system `Python`:
```sh
pyenv global 3.10.6
``` 

## 2) Clone or download this repository to the virtual machine.



## 3)	Install required `Python` packages
Navigate to the parent `public_chatbot_backend` directory:
```sh
cd public_chatbot_backend
```

Upgrade `pip` and install requirements:
```sh
pip install --upgrade pip
pip install -r requirements.txt
```

## 4)	Install NGINX
NGINX is used for handling `HTTP` traffic.
Install NGNIX with:
```sh
sudo apt install nginx
```

Adjust the UFW firewall to enable ports used by NGNIX.
Allow SSH connections, so your connection to the VM will not get interrupted (assuming you are connected via SSH):
```sh
sudo ufw allow 22
```

Enable UFW:
```sh
sudo ufw enable
```

UFW can see if NGNIX is installed and you can check this with:
```sh
sudo ufw app list
```

You should see something like this:
```sh
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
``` 

Depending on configurations of the application and your virtual machine, you can choose to allow `HTTP` and/or `HTTPS` traffic. To allow both (default), type:
```sh
sudo ufw allow 'Nginx Full'
``` 

Make sure the rule has been applied with:
```sh
sudo ufw status
``` 

You should see:
```sh
Status: active
To                         Action      From
--                         ------      ----
22			   ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
22 (v6)			   ALLOW       Anywhere (v6)                 
Nginx Full (v6)            ALLOW       Anywhere (v6) 

``` 


You can make sure NGNIX is set up properly by heading to the IP address of your virtual machine in a browser:
```sh
http://<YOUR IP ADDRESS>
```

And you should see the default NGNIX welcome page.

Check with `systemd` that NGNIX is running:
```sh
systemctl status nginx
```

You should see:
```sh
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-03-21 12:09:59 UTC; 6min ago
     Docs: man:nginx(8)
  Process: 5761 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  Process: 5749 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 5764 (nginx)
    Tasks: 2 (limit: 667)
   CGroup: /system.slice/nginx.service
           ├─5764 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─5767 nginx: worker process
``` 


## 5)	Configure NGNIX
Create a new server block:
```sh
sudo nano /etc/nginx/sites-available/app
```

And add following to the file:
```sh
server {
    listen 80;
    server_name <YOUR IP ADDRESS>;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/<YOUR USERNAME>/public_chatbot_backend/app.sock;
    }
}
server {
    listen 5000;
    server_name <YOUR IP ADDRESS>;
    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/<YOUR USERNAME>/public_chatbot_backend/app.sock;
    }
}
``` 

For custom domain name, replace `<YOUR IP ADDRESS>` with the domain name:
```sh
server_name example.com www.example.com;
``` 

Link the created server block to `sites-enabled`:
```sh
sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled
``` 

Check NGNIX for syntax errors with:
```sh
sudo nginx -t
``` 

You should see:
```sh
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
``` 

Make sure Nginx has permission to access the backend directory:
```sh
sudo chmod 755 /home/<YOUR USERNAME>
``` 

Finally, restart the NGNIX service:
```sh
sudo systemctl restart nginx
``` 

## 6)	Create a system unit file to run the backend automatically

Open `nano` with:
```sh
sudo nano /etc/systemd/system/app.service
``` 

Add following to the file:
```sh
[Unit]
Description=Backend of Alice
After=network.target

[Service]
User=<YOUR USERNAME HERE>
Group=www-data
WorkingDirectory=/home/<YOUR USERNAME HERE>/public_chatbot_backend
ExecStart=/home/<YOUR USERNAME HERE>/.pyenv/shims/uwsgi --ini app.ini

[Install]
WantedBy=multi-user.target
``` 

Save and close the file.
Start and enable the service with:
```sh
sudo systemctl start app
sudo systemctl enable app
``` 

Check the service status:
```sh
sudo systemctl status app
``` 

You should see:
```sh
● app.service – Backend of Alice
   Loaded: loaded (/etc/systemd/system/app.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2023-06-14 17:11:44 UTC; 2min 56s ago
 Main PID: 3680 (uwsgi)
      Tasks: 26 (limit: 1087)
     Memory: 44.1M
        CPU: 715ms
   CGroup: /system.slice/app.service
           ├─33617 /home/ubuntu/.pyenv/versions/3.10.6/bin/uwsgi --ini app.ini
           ├─33657 /home/ubuntu/.pyenv/versions/3.10.6/bin/uwsgi --ini app.ini
           ├─33658 /home/ubuntu/.pyenv/versions/3.10.6/bin/uwsgi --ini app.ini
           ├─33659 /home/ubuntu/.pyenv/versions/3.10.6/bin/uwsgi --ini app.ini
           ├─33660 /home/ubuntu/.pyenv/versions/3.10.6/bin/uwsgi --ini app.ini
           └─33661 /home/ubuntu/.pyenv/versions/3.10.6/bin/uwsgi --ini app.ini
``` 

**Now backend of the application should be up and running. If you already have frontend of the application set up, you should see response messages from Alice when you type messages into the chat box. If not, head into the Chatbot-for-Assessing-System-Security/app/frontend directory and follow the Chatbot-for-Assessing-System-Security/app/frontend/README.md to set up the frontend.**