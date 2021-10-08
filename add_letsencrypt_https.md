**Add free lets encrypt certificate**
----
_This tutorial shows how to add a free lets encrypt ssl certificate_
[See this link](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)


## Tutorial
First install snapd (should already be installed on ubuntu)
Ensure that snapd version is up to date
```
sudo snap install core; sudo snap refresh core
```

Remove existing certbot
```
sudo apt-get remove certbot
```

Then install certbot
```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Get the SSL certificate (be careful, you should have nginx running for your domain name)
`sudo certbot certonly --nginx`


Normally the certificates will always be renewed, test it
`sudo certbot renew --dry-run`

Modify the nginx configurations (back and front) with the lets encrypt certificates
```
/etc/letsencrypt/live/garde-enfant-fonpeps-audiens.org/fullchain.pem
/etc/letsencrypt/live/garde-enfant-fonpeps-audiens.org/privkey.pem
```

Restart nginx
`sudo systemctl restart nginx`
