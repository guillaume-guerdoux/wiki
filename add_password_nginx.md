**Add password to nginx**
----
_This tutorial shows how to add password to nginx_
[See this link](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-nginx-on-ubuntu-14-04)


## Tutorial
First add the user
```
sudo sh -c "echo -n 'guillaume:' >> /etc/nginx/.htpasswd"
```

Then add the password
```
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

Finally add the authentification to the nginx file
```
location / {
    try_files $uri $uri/ =404;
    auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```