# Postgres errors
List of postgres errors
[Link 1](https://tecadmin.net/setup-selenium-chromedriver-on-ubuntu/)

## Database ownership
Error `When django.db.utils.ProgrammingError: permission denied for relation django_migrations`

```
psql mydatabase -c "GRANT ALL ON ALL TABLES IN SCHEMA public to dbuser;"
psql mydatabase -c "GRANT ALL ON ALL SEQUENCES IN SCHEMA public to dbuser;"
psql mydatabase -c "GRANT ALL ON ALL FUNCTIONS IN SCHEMA public to dbuser;"
```
