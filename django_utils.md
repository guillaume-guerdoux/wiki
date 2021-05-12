# Django utils
_Django util functions_

## Create a new django secret key
`python -c 'import random; result = "".join([random.choice("abcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*(-_=+)") for i in range(50)]); print(result)'`

## Log UWSGI to a file
Add `logto = /var/log/uwsgi/%n.log` to the uwsgi.ini file

```
sudo mkdir -p /var/log/uwsgi
sudo chown -R guillaume:guillaume /var/log/uwsgi
```
