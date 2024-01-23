# Major update fixes

## Documentation - Installation

## Introduction

> This part is dedicated to perform a major update of the Borgia app 

This includes : 
- Django and libs updates
- Functionnalities addon
- Mise a jour mettant en jeu les modèles
- Nouvelles app Django 

Ce guide a pour but de vous aidez à vous dépatouiller si une erreur venait à se presenter


## Preparation 

## Erreur sur les migrations :

```sh

find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
find . -path "*/migrations/*.pyc"  -delete


```

```bash


(borgiaenv) root@borgia-p3:/borgia-app/Borgia/borgia# python3 manage.py makemigrations modules

    ...

  File "/borgia-app/borgiaenv/lib/python3.9/site-packages/django/db/models/options.py", line 224, in contribute_to_class
    raise TypeError(
TypeError: 'class Meta' got invalid attribute(s): ask_password

```


```

django.db.migrations.exceptions.InconsistentMigrationHistory: Migration users.0001_initial is applied before its dependency auth.0012_alter_user_first_name_max_length on database 'default'.

```


une fois la bdd borgia creer faire : sudo -u postgres psql borgia  < /home/josue/Downloads/backup_P3.sql


La methode à suivre si vous souhaitez transvaser votre base de données

## Update Borgia en conservant la BDD

Si vous souhaitez passer d'une VM à une autre ou de repartir sur une base propre vous avez plusieurs options. 

Celle que je vais detailler à l'avantage d'etre robuste et de s'adapter à toutes situations

> Cette installation se base uniquement pour un setup de production (link)

### Sur la machine ou se trouve la BDD

>Faire une sauvegarde (un dump) SQL de la BDD 
> `pg_dump -h localhost -p 5432 -U mon_utilisateur -d ma_base_de_donnees -f sauvegarde.sql`

### Sur la machine vers laquelle vous souhaitez importer la BDD ou mettre a jour le Borgia
### Update the server:

- `apt-get update`
- `apt-get upgrade`

### Remove Apache if installed:

- `apt-get purge apache2`

### Install necessary packages for the rest of the installation:

- `apt-get install curl apt-transport-https`

### Install all necessary packages:

- `sudo apt install build-essential libssl-dev libffi-dev libjpeg-dev python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl git`

### Creating the PostgreSQL Database and User

- `sudo -u postgres psql`

Inside the `postgres=#` terminal : 

```
CREATE DATABASE borgia;
CREATE USER borgiauser WITH PASSWORD 'password';
ALTER ROLE borgiauser SET client_encoding TO 'utf8';
ALTER ROLE borgiauser SET default_transaction_isolation TO 'read committed';
ALTER ROLE borgiauser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE borgia TO borgiauser;
\q

```


> Dans la BDD vide, importer le dump de la sauvegarde
> `sudo -u postgres psql borgia  < /home/josue/Downloads/backup_P3.sql`


### Installation of Yarn 

Explicit case of Debian, otherwise see [here](https://yarnpkg.com/lang/en/docs/install/)

- `curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
- `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
- `apt-get update && sudo apt-get install yarn`


### Create the Borgia root folder:

- `mkdir /borgia-app`
- `cd /borgia-app`

## Copy of Borgia

In `/borgia-app`:

- `git clone https://github.com/borgia-app/Borgia.git`

Then in `/borgia-app/Borgia`:

- `git checkout tags/RELEASE_TO_UTILISER`
- `git checkout -b production_RELEASE_TO_USE`

### Creating a Python Virtual Environment for your Project

- `python3 -m venv venv`
- `source venv/bin/activate`

## Installation of packages necessary for the application

In `/borgia-app/Borgia` and in the virtual environment install the `prod.txt` requirement file.

- `pip3 install -r requirements/prod.txt`

And finally, outside the virtual environment:

- `yarn global add less`

> Si erreur  `lessc` : `npm install -g less`


## Software configuration

### Vital parameters

Copy the file `/borgia-app/Borgia/contrib/production/settings.py` to `/borgia-app/Borgia/borgia/borgia/settings.py` and:

- Modify the `SECRET_KEY =` line by indicating a random private key. For example, [this site](https://randomkeygen.com/) allows you to generate keys, choose at least "CodeIgniter Encryption Keys", for example: `SECRET_KEY = 'AAHHBxi0qHiVWWk6J1bVWCMdF45p6X9t'`.

- Make sure `DEBUG = False`.

- Modify the `ALLOWED_HOSTS =` line by indicating the domains or subdomains accepted by the application. For example: `ALLOWED_HOSTS = ['sibers.borgia-app.com', 'borgia-me.ueam.net']`.


### Database

In the `/borgia-app/Borgia/borgia/borgia/settings.py` file, modify the part:

```python
DATABASES = {
...
}
```

by indicating the name of the database, the user name and the password defined during the configuration of the latter. For example :

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'borgia',
        'USER': 'borgiauser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

### Mail server


Voir (link)

### Administrators


Voir (link)


## Database migration

In `/borgia-app/Borgia/borgia` and in the virtual environment:

- `python3 manage.py makemigrations configurations users shops finances events modules sales stocks`
- `python3 manage.py migrate`
- `python3 manage.py loaddata initial`
- `python3 manage.py collectstatic --clear` by accepting the alert

Then, enter the password for the administrator account (which will be deactivated later):

Changing the password of the first user `AE_ENSAM`:

- `python manage.py shell`
- `from users.models import User`
- `u = User.objects.get(pk=1)`
- `u.set_password("NEW_PASSWORD")`
- `u.save()`
- `exit()`


## Intermediate test

### UFW configuration

If you followed the initial server setup guide, you should have a UFW firewall protecting your server. In order to test the development server, you need to allow access to the port you’ll be using.

Create exception for port 8000 : 

- `sudo ufw allow 8000`

### Run the server

The command in the virtual environment `python3 manage.py runserver 0.0.0.0:8000` should start the server and should not report any errors. If this is the case, continue to the rest and end of the installation guide.

In your web browser, visit your server’s domain name or IP address followed by port 8000 : http://server_domain_or_IP:8000 




A tester : faire la maj en place cad juste update les fichiers django



## Cas pratique 

## Autre erreurs rencontrées



SI erreur lessc : npm install -g less




---




This guide allows you to install, configure and operate Borgia locally for **development**.

All of the following is independent of the operating system used. It works on Windows, MacOS and Linux.

Be careful, if Python 2 and 3 coexist, python 3 will be called with `python3`. In all cases, check that Python 3 is used for the following commands, by doing: `python --version` or `python3 --version`. Same for `pip` and `pip3` if necessary.

## Installing dependencies

- Python packages: `pip install -r requirements/dev.txt`
- Less: `yarn global add less` or `npm install -g less`

## Configuring `settings.py`

- Copy/paste the `settings.py` file located in `/contrib/development` into `/borgia/borgia`
- Optional : Modify all the variables that must be done by browsing the file.
- Optional : In the case of configuring a Gmail email, with email `GMAIL_EMAIL` and password `GMAIL_PASSWORD`, use:
```python
DEFAULT_FROM_EMAIL = 'GMAIL_EMAIL'
SERVER_EMAIL = 'GMAIL_EMAIL'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_HOST_USER = 'GMAIL_EMAIL'
EMAIL_HOST_PASSWORD = 'GMAIL_PASSWORD'
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
```
- Optional: Don't forget to configure Gmail to accept "less secure applications".
  
## Migrations and initial data

The following commands must be executed in the `/borgia` application folder.

- `python manage.py makemigrations configurations users shops finances events modules sales stocks`
- `python manage.py migrate`
- `python manage.py loaddata initial`
- `python manage.py collectstatic --clear` indicating "yes" on validation.

Initial data for simulation and development.

- `python manage.py loaddata tests_data`

Changing the password of the first user `AE_ENSAM`:

- `python manage.py shell`
- `from users.models import User`
- `u = User.objects.get(pk=1)`
- `u.set_password("NEW_PASSWORD")`
- `u.save()`
- `exit()`

## Run local server

You can then launch a local development server:

- `python manage.py runserver`

To launch it on a specific port address use:

- `python manage.py runserver localhost:8081`

## Tests

Unit tests are run by `python manage.py test` or `python manage.py test APPLICATION_NAME` to test a particular application.

They must be executed, without errors before each push.