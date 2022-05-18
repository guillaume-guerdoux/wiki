**Deploy Flask to VPS**
----
_This tutorial shows how to deploy a flask / nginx project to an ovh vps_

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

Then create the .env file into app/main with :
```
ENV_CONFIG=prod

LOCAL_DATABASE_URL=postgresql:///PROJECT_ranking

DATABASE_URL=postgresql:///PROJECT_ranking
```

ENV_CONFIG=        prod
LOCAL_DATABASE_URL=postgresql:///NAME_DATABASE


## 3. Flask configuration
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
### Flask Installation
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

Then remove the migrations folder and run
```
python3 manage.py db init

python3 manage.py db migrate --message "initial database migration"

python3 manage.py db upgrade

```
Then copy data into sql database
`python app/data/copy_data.py`

Create the users
`python create_user.py USERNAME PASSWORD`
```

### Deploy Flask with nginx and gunicorn

#### Uwsgi deployment
Verfiy that gunicore can work by running :
```
gunicorn manage:app --timeout 120
```

`deactivate`
`sudo nano /etc/systemd/system/myproject.service`

Add guillaume to www-data group `sudo usermod -a -G www-data guillaume`


And write
```
[Unit]
Description=Gunicorn instance to serve PROJECT
After=network.target

[Service]
User=guillaume
Group=www-data
WorkingDirectory=/home/guillaume/krunch-PROJECT-deploy
Environment="PATH=/home/guillaume/krunch-PROJECT-deploy/.venv/bin"
ExecStart=/home/guillaume/krunch-PROJECT-deploy/.venv/bin/gunicorn --workers 3 --bind unix:krunch_PROJECT.sock -m 007 manage:app --timeout 120

[Install]
WantedBy=multi-user.target
```

Run `sudo systemctl daemon-reload` and `sudo systemctl start myproject`
And check if everything works : `sudo systemctl status myproject`
Enable uwsgi en startup : `sudo systemctl enable myproject`



#### Nginx deployment
Install nginx `sudo apt-get install nginx`
Remove the default file by doing `sudo rm /etc/nginx/sites-enabled/default`

Create nginx file
`sudo nano /etc/nginx/sites-available/krunch_PROJECT`

Put that in the file
```
server {
    listen 80;
    server_name PROJECT.krunch-kol.com;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/guillaume/krunch-PROJECT-deploy/krunch_PROJECT.sock;
    }
}

```

`sudo ln -s /etc/nginx/sites-available/krunch_PROJECT /etc/nginx/sites-enabled/`

#### Securing the app
`sudo snap install --classic certbot`
`sudo ln -s /snap/bin/certbot /usr/bin/certbot`

Get the SSL certificate (be careful, you should have nginx running for your domain name)
`sudo certbot certonly --nginx`


Change nginx file
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name XX.XXX.XX.XX PROJECT.krunch-kol.com;
    return 301 https://PROJECT.krunch-kol.com$request_uri;
}

server {
    listen 443 ssl;
    server_name mici.krunch-kol.com;
    ssl_certificate       /etc/letsencrypt/live/PROJECT.krunch-kol.com/fullchain.pem;
    ssl_certificate_key   /etc/letsencrypt/live/PROJECT.krunch-kol.com/privkey.pem;  
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/guillaume/krunch-mici-deploy/krunch_mici.sock;
    }
}
```
