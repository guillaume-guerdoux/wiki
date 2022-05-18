**Deploy django / vue to VPS**
----
_This tutorial shows how to deploy a django / nginx project to an ovh vps_

## 1. Server configuration
[Link 1](https://docs.ovh.com/fr/vps/conseils-securisation-vps/)
[Link 2](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart)

### Server connection and upgrade
First connect to the server via ubuntu user `ssh ubuntu@XX.XX.XX.XX`

Then update and upgrade the server
```
sudo apt-get update
sudo apt-get upgrade
```

Install Fail2ban `sudo apt-get install fail2ban`

### SSH modification
Modify the ssh listening port
```
sudo nano /etc/ssh/sshd_config
```
Replace the 22 port by another port
```
# What ports, IPs and protocols we listen for
Port 22
```
Restart ssh
```
systemctl restart sshd
```
Go back to local and add the Host and the port to `~/.ssh/config` file
Need to add at the end of the file
```
Host YOUR IP
	Port YOUR PORT
```

### User configuration
Modify the ubuntu password by typing `passwd`
Add a new user by typing : `sudo adduser guillaume`
Give sudo role to guillaume : `sudo usermod -aG sudo guillaume`

### Connect with guillaume
Copy the ssh key : `ssh-copy-id guillaume@XX.XX.XX.XX`
Connect with guillaume `ssh guillaume@XX.XX.XX.XX`

## 2. Git linking
[Link 1](https://gist.github.com/noelboss/3fe13927025b89757f8fb12e9066f2fa)
[Link 2](https://medium.com/@francoisromain/vps-deploy-with-git-fea605f1303b)

In your user home folder :
```
mkdir ~/your-project-deploy-folder
git init --bare ~/your-project.git
```

Then go to `~/your-project.git/hooks/post-receive` and copy paste this file :
```
#!/bin/bash
TARGET="/home/guillaume/name-deploy"
GIT_DIR="/home/guillaume/name-project.git"
BRANCH="master"

while read oldrev newrev ref
do
	# only checking out the master (or whatever branch you would like to deploy)
	if [[ $ref = refs/heads/$BRANCH ]];
	then
		echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
		git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f
	else
		echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
	fi
done
```
Make the post-receive executable by typing `sudo chmod +x post-receive`
Go back to your local git folder and add the remote `git remote add production guillaume@XX.XX.XX.XX:name-project.git`
Then each time you want to push to your remote server, run `git push production master`

For preprod, if you have the error message `remote: fatal: You are on a branch yet to be born`
Got to server in the git projet and in the HEAD file, replace master by the name of your branch

Then transfer to your server the files `localsettings.py` into the same folder as settings.py (for django)
and `localVariables.js` for Vue.js
Adapt these two files to your case

## 3. Django configuration
### Postgres Installation
Install postgres `sudo apt install -y postgresql postgresql-contrib`
Use postgres user `sudo -u postgres psql`

Create the database and the user
```
CREATE DATABASE nom_db;
CREATE USER guillaume WITH PASSWORD 'P@ssw0rd';
ALTER ROLE guillaume SET client_encoding TO 'utf8';
ALTER ROLE guillaume SET default_transaction_isolation TO 'read committed';
ALTER ROLE guillaume SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE nom_db TO guillaume;
```
### Django Installation
Go to your deploy folder
```
sudo apt-get update
export LC_ALL="en_US.UTF-8"
sudo dpkg-reconfigure locales
sudo apt-get upgrade
sudo apt-get install python3-dev
sudo apt-get install python3-pip
sudo apt-get install virtualenv
virtualenv .venv -p python3
```
Install les requirements et configure django db
```
source .venv/bin/activate
pip install -r requirements.txt
cd DjangoBackEnd/
mkdir static
python manage.py collectstatic
python manage.py migrate
```

### Deploy Django with nginx and uwsgi

#### Uwsgi deployment
Check if uwsgi ins installed in virtualenv
`source .venv/bin/active`
`uwsgi --version`

Add guillaume to www-data group `sudo usermod -a -G www-data guillaume`

Go to your backend folder and create `nano uwsgi_params` with :
```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```
Create a folder `sudo mkdir -p /etc/uwsgi/sites`
Create the file `sudo nano /etc/uwsgi/sites/project_uwsgi.ini`
The file is :
```
[uwsgi]
project = projectBackEnd
uid = guillaume
base = /home/%(uid)

chdir = /home/guillaume/project-deploy/projectBackEnd
home = /home/guillaume/project-deploy/.venv
module = projectBackEnd.wsgi

master = true
processes = 10

socket = /run/uwsgi/projectBackEnd.sock
chown-socket = guillaume:www-data
chmod-socket = 660
vacuum = true
```


Create a systemd Unit File for uWSGI `sudo nano /etc/systemd/system/uwsgi.service`
And add :
```
[Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown guillaume:www-data /run/uwsgi'
ExecStart=/home/guillaume/project-deploy/.venv/bin/uwsgi --emperor /etc/uwsgi/sites
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

Run `sudo systemctl daemon-reload` and `sudo systemctl start uwsgi`
And check if everything works : `sudo systemctl status uwsgi`
Enable uwsgi en startup : `sudo systemctl enable uwsgi`

#### Nginx deployment
Install nginx `sudo apt-get install nginx`
Remove the default file by doing `sudo rm /etc/nginx/sites-enabled/default`
Create file `nano project_django_nginx.conf`
Add and adapt to the file
```
# project_django_nginx.conf
# the upstream component nginx needs to connect to
upstream django {
    server unix:/run/uwsgi/projectBackEnd.sock;
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8443 ssl;
    # the domain name it will serve for
    server_name XX.XX.XX.XX domain.fr;
    charset     utf-8;

    ssl_certificate       /etc/ssl/domain.fr.chained.crt;
    ssl_certificate_key   /etc/ssl/doman_fr.key;    

    # max upload size
    client_max_body_size 10M;   # adjust to taste

    # Django media
    location /media  {
        alias /home/guillaume/project-deploy/projectBackEnd/media/public/;  # your Django project's media files - amend as required
    }

    location /static {
        alias /home/guillaume/project-deploy/projectBackEnd/public; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /home/guillaume/project-deploy/projectBackEnd/uwsgi_params;
        proxy_redirect    off;
        proxy_set_header  Host              $http_host;
        proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto https;
    }

}
```

Add a symlink : `sudo ln -s /home/guillaume/project-deploy/projectBackend/project_django_nginx.conf /etc/nginx/sites-enabled/`
Restart nginx : `sudo systemctl restart nginx` and check `sudo systemctl status nginx`
Enable nginx on startup : `sudo systemctl enable nginx`
And run `sudo systemctl daemon-reload`

Now do `sudo visudo` and add `guillaume ALL=NOPASSWD: /bin/systemctl restart uwsgi.service`
Now in the post-receive, you need to add
```
echo "Python migrations"
cd /home/guillaume/project-deploy/
source .venv/bin/activate
echo "Install requirements"
pip install -r requirements.txt
echo "Requirements installed"
cd projectBackEnd
python manage.py migrate
echo "Migration done"
echo "Python collectstatic"
python manage.py collectstatic --noinput
echo "Collectstatic done"
echo "Redeploy uwsgi"
sudo /bin/systemctl restart uwsgi.service
```

Verify if it's working !


## 4. Vue configuration
[Link 1](https://github.com/nodesource/distributions/blob/master/README.md)
[Link 2](https://medium.com/@thucnc/deploy-a-vuejs-web-app-with-nginx-on-ubuntu-18-04-f93860219030)

Install NodeJs LTS
```
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```
And install the vue cli `sudo npm install -g @vue/cli`
Go into your frontend folder and run `npm install` then `npm run build`

Then create `project_vue_nginx.conf` file and add in it :
```
# Redirect all non-encrypted to encrypted
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name XX.XXX.XX.XX domain.fr;
    return 301 https://domain.fr$request_uri;
}

server {
    # the port your site will be served on
    # the port your site will be served on
    listen      443 ssl http2;
    listen [::]:443 ssl http2;

    root /home/guillaume/project-deploy/project-front-end;
    index index.html index.htm;

    server_name XX.XX.XX.XX domain.fr;

    charset     utf-8;

    error_page 404 /index.html;

    ssl_certificate      /etc/ssl/domain.fr.chained.crt;
    ssl_certificate_key  /etc/ssl/domain_fr.key;

    # max upload size
    client_max_body_size 10M;   # adjust to taste

    #gzip_static on;

    location / {
	root /home/guillaume/project-deploy/project-front-end/dist;

	expires -1;
        add_header Pragma "no-cache";
        add_header Cache-Control "no-store, no-cache, must-revalidate, post-check=0, pre-check=0";

	try_files $uri /index.html;
    }
}
```

Then run `sudo ln -s /home/guillaume/project-deploy/webapp/project_nginx.conf /etc/nginx/sites-enabled/`
Restart nginx : `sudo systemctl restart nginx` and check `sudo systemctl status nginx`

Now in the post-receive, you need to add
```
echo "Install all dependencies"
npm install --prefix /home/guillaume/project-deploy/project-front-end/
echo "Dependencies installed"
echo "Build the vue project"
npm run build --prefix /home/guillaume/project-deploy/project-front-end/
echo "Vue project built"

```
## 5. Celery configuration
[Link 1](https://dev.to/idrisrampurawala/deploying-django-with-celery-and-redis-on-ubuntu-3fo6)
Install redis `sudo apt-get install redis-server` and enable it `sudo systemctl enable redis-server.service`

Install supervisor `sudo apt-get install supervisor`

Create the celery logs file `mkdir /home/guillaume/project-deploy/projectBackEnd/celery_logs`
Create a celery conf with `sudo nano /etc/supervisor/conf.d/celery_worker.conf` and add into the file
```
; ==================================
;  celery worker supervisor
; ==================================

[program:celery]
directory=/home/guillaume/project-deploy/projectBackEnd
command=/home/guillaume/project-deploy/.venv/bin/celery worker -A projectBackEnd.celery --loglevel=INFO

user=guillaume
numprocs=1
stdout_logfile=/home/guillaume/project-deploy/projectBackEnd/celery_logs/worker-access.log
stderr_logfile=/home/guillaume/project-deploy/projectBackEnd/celery_logs/worker-error.log
stdout_logfile_maxbytes=50
stderr_logfile_maxbytes=50
stdout_logfile_backups=10
stderr_logfile_backups=10
autostart=true
autorestart=true
startsecs=10

; Need to wait for currently executing tasks to finish at shutdown.
; Increase this if you have very long running tasks.
stopwaitsecs = 600

; Causes supervisor to send the termination signal (SIGTERM) to the whole process group.
stopasgroup=true

; Set Celery priority higher than default (999)
; so, if rabbitmq is supervised, it will start first.
priority=1000
```

Launch it by typing :
```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start all
```
Enable supervisor to start on reboot : `sudo systemctl enable supervisor`
Add the celery in visudo `sudo visudo` and add `guillaume ALL=NOPASSWD: /usr/bin/supervisorctl restart celerydefault`

Add the celery restart on hook post-receive by adding `sudo /usr/bin/supervisorctl restart celerydefault`

## 6. Flower configuration
Create a flower logs folder in backend folder : `mkdir flower_logs`
Add a flower supervisor by typing `sudo nano /etc/supervisor/conf.d/flower.conf`
Copy this :
```
; ==================================
;  flower supervisor
; ==================================

[program:flower]
environment =
    LC_ALL=en_US.UTF-8,
    LANG=en_US.UTF-8
directory=/home/guillaume/projet-deploy/projectBackEnd
command=/home/guillaume/projet-deploy/.venv/bin/flower -A projectBackEnd --url_prefix=flower

user=guillaume
numprocs=1
stdout_logfile=/home/guillaume/projet-deploy/projectBackEnd/flower_logs/flower-access.log
stderr_logfile=/home/guillaume/projet-deploy/projectBackEnd/flower_logs/flower-error.log
stdout_logfile_maxbytes=50
stderr_logfile_maxbytes=50
stdout_logfile_backups=10
stderr_logfile_backups=10
autostart=true
autorestart=true
startsecs=10

; Need to wait for currently executing tasks to finish at shutdown.
; Increase this if you have very long running tasks.
stopwaitsecs = 600

; Causes supervisor to send the termination signal (SIGTERM) to the whole process group.
stopasgroup=true
```

```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start flower
```

Create a new user flower :
```
sudo sh -c "echo -n 'guillaume:' >> /etc/nginx/.htpasswd"
```
Then add the password
```
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

Add this to your backend nginx file :
```
location /flower {
    proxy_pass http://localhost:5555;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
	auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;    
}
```
