**Add S3 storage for django application**
This tutorial gives you all the step to add s3 storage to django app
[Link 1](https://simpleisbetterthancomplex.com/tutorial/2017/08/01/how-to-setup-amazon-s3-in-a-django-project.html)

## Installation
`pip install django-storages`

## Settings
Copy this instead of al static media config in settings.py

```
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)
STATIC_ROOT = os.path.join(BASE_DIR, 'public')

STATIC_URL = '/static/'

# S3
#AWS_PUBLIC_MEDIA_LOCATION = 'media/public'
AWS_PRIVATE_MEDIA_LOCATION = 'media/private'

if DEBUG:
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    MEDIA_URL = '/media/'
    DEFAULT_FILE_STORAGE = 'django.core.files.storage.FileSystemStorage'
    #PRIVATE_FILE_STORAGE = 'django.core.files.storage.FileSystemStorage'
else:
    # AWS S3 STORAGE
    AWS_ACCESS_KEY_ID = localsettings_dict['AWS_ACCESS_KEY_ID']
    AWS_SECRET_ACCESS_KEY = localsettings_dict['AWS_SECRET_ACCESS_KEY']
    AWS_STORAGE_BUCKET_NAME = localsettings_dict['AWS_STORAGE_BUCKET_NAME']

    AWS_S3_CUSTOM_DOMAIN = '%s.s3.amazonaws.com' % AWS_STORAGE_BUCKET_NAME

    AWS_S3_OBJECT_PARAMETERS = {
        'CacheControl': 'max-age=86400',
    }

    DEFAULT_FILE_STORAGE = 'audiensMinistereBackEnd.storage_backends.PrivateMediaStorage'

    AWS_DEFAULT_ACL = None
```

Ceate a file `storage_backends.py` next to settings with

```
from django.conf import settings
from django.core.files.storage import get_storage_class
from storages.backends.s3boto3 import S3Boto3Storage

#class PublicMediaStorage(S3Boto3Storage):
#    location = settings.AWS_PUBLIC_MEDIA_LOCATION
#    file_overwrite = False

class PrivateMediaStorage(S3Boto3Storage):
    location = settings.AWS_PRIVATE_MEDIA_LOCATION
    default_acl = 'private'
    file_overwrite = False
    custom_domain = False
```

Add in localsettings.py :
```
'AWS_ACCESS_KEY_ID': 'XXXX',
'AWS_SECRET_ACCESS_KEY': 'XXXX',
'AWS_STORAGE_BUCKET_NAME': 'XXX'
```
