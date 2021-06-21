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

## Activate Unaccente case insensitive
Add the line `django.contrib.postgres` in INSTALLED_APPS (settings.py)
Activate the 'unaccent' extension  :
create am empty migration (./manage.py makemigrations --empty your.application.name)
and edit the generated file with the following content (source) and, finally, running ./manage.py migrate:

```
from django.contrib.postgres.operations import UnaccentExtension

class Migration(migrations.Migration):

dependencies = [
    (<snip>)
]

operations = [
    UnaccentExtension()
]
```

Be careful, postgres user need to be SUPERUSER :
`sudo su - postgres`
`psql`
`ALTER ROLE guillaume SUPERUSER;``
