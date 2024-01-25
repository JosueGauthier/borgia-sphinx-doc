# Major update fixes

## Introduction

> This part is dedicated to perform a major update of the Borgia app.

This includes : 
- Django and libs updates
- Functionnalities addon
- Template update
- New Django app 

This guide is intended to help you sort out any errors that may occur.

## Preparation 

Before attempting any update, make a backup of your VM, container or any other solution hosting your application.

## Update Borgia while preserving the DB

If you want to move from one VM to another or start from scratch, you have several options. 

The one I'm going to detail has the advantage of being robust and adaptable to all situations.

> This installation is based solely on a production setup (link)

### On the machine where the DB is located

>Make an SQL dump of the DB 
> `pg_dump -h localhost -p 5432 -U mon_utilisateur -d ma_base_de_donnees -f sauvegarde.sql`


On the machine to which you wish to import the DB or update Borgia

### Update the server:

- `apt-get update`
- `apt-get upgrade`

### Remove Apache if installed:

- `apt-get purge apache2`

### Install necessary packages for the rest of the installation:

- `apt-get install curl apt-transport-https`

### Install all necessary packages:

- `sudo apt install build-essential libssl-dev libffi-dev libjpeg-dev python-venv python-dev libpq-dev postgresql postgresql-contrib nginx curl git`

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

### Import the DB

In the empty DB, import the backup dump : 

`sudo -u postgres psql borgia < /home/josue/Downloads/backup_P3.sql`


### Installation of Yarn 

See (link)

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

- `python -m venv venv`
- `source venv/bin/activate`

## Installation of packages necessary for the application

In `/borgia-app/Borgia` and in the virtual environment install the `prod.txt` requirement file.

- `pip3 install -r requirements/prod.txt`

And finally, outside the virtual environment:

- `yarn global add less`

> If error  `lessc` : `npm install -g less`


## Software configuration

See (link)

## Database migration

This is where your problems are likely to start. 

Here's the kind of error you may encounter:

```sh
(borgiaenv) root@borgia-p3:/borgia-app/Borgia/borgia# python manage.py makemigrations modules

...

  File "/borgia-app/borgiaenv/lib/python.9/site-packages/django/db/models/options.py", line 224, in contribute_to_class
    raise TypeError(
TypeError: 'class Meta' got invalid attribute(s): ask_password
...
```

```sh
...
django.db.migrations.exceptions.InconsistentMigrationHistory: Migration users.0001_initial is applied before its dependency auth.0012_alter_user_first_name_max_length on database 'default'.
```


First of all, try a classic migration:


In `/borgia-app/Borgia/borgia` and in the virtual environment:

- `python manage.py makemigrations configurations users shops finances events modules sales stocks`
- `python manage.py migrate`
- `python manage.py collectstatic --clear` by accepting the alert


If you get an error, it means that a link has been broken between your DB and your Django. You'll need to repair it.


Firstly, in the `/borgia-app/Borgia/borgia` delete all the current migrations files :


```sh
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
find . -path "*/migrations/*.pyc"  -delete

```

Then inside the PostgreSQL database you will delete the Django migrations too.

```sh

python manage.py dbshell

borgia=> DELETE FROM django_migrations;

```

Then you will retry to build the migrations : 
In `/borgia-app/Borgia/borgia` and in the virtual environment:

- `python manage.py makemigrations configurations users shops finances events modules sales stocks`
- `python manage.py migrate`

If you still get an error you need to fake at least initial migrations [Django fake migrations](https://docs.djangoproject.com/fr/5.0/topics/migrations/#initial-migrations). 


- `python manage.py migrate --fake-initial`

If after this you still get an error fake the corresponding app with the `--fake` attribute. For example here if you get an error with the `contenttypes` app do :

- `python manage.py migrate contenttypes --fake`

> Be careful with this command beacause after that you will need to redo the migrations from scratch.

Then, enter the password for the administrator account (which will be deactivated later):

Changing the password of the first user `AE_ENSAM`:

- `python manage.py shell`
- `from users.models import User`
- `u = User.objects.get(pk=1)`
- `u.set_password("NEW_PASSWORD")`
- `u.save()`
- `exit()`


After all these commands you will be able to run a server : 


### UFW configuration

If you followed the initial server setup guide, you should have a UFW firewall protecting your server. In order to test the development server, you need to allow access to the port you’ll be using.

Create exception for port 8000 : 

- `sudo ufw allow 8000`

### Run the server

The command in the virtual environment `python3 manage.py runserver 0.0.0.0:8000` should start the server and should not report any errors. If this is the case, continue to the rest and end of the installation guide.

In your web browser, visit your server’s domain name or IP address followed by port 8000 : http://server_domain_or_IP:8000 


If everythings go well you can normally continue the installation.
Else if you get an error like : 

```
django.db.utils.ProgrammingError: column shops_shop.correcting_factor_activated does not exist
LINE 1: ..."shops_shop"."description", "shops_shop"."color", "shops_sho...

```

It means that due to fake migrations the corresponding fields does not exist in the DB so you need to make a fix migration by hand.

To do this indificate the corresponding fields that have a problem, in the given example it is : `correcting_factor_activated` from `shops` app. 

So in the shops migration folder, go to the file that create the `correcting_factor_activated` field and remove it. 


```py

# Generated by Django 4.0.4 on 2024-01-22 21:08

from decimal import Decimal
import django.core.validators
from django.db import migrations, models
import django.db.models.deletion


class Migration(migrations.Migration):

    initial = True

    dependencies = [
    ]

    operations = [
        migrations.CreateModel(
            name='Shop',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('name', models.CharField(max_length=255, validators=[django.core.validators.RegexValidator(message='Ne doit contenir que des lettres\n                                minuscules, sans espace ni caractère\n                                spécial.', regex='^[a-z]+$')], verbose_name='Code')),
                ('description', models.TextField(verbose_name='Description')),
                ('color', models.CharField(max_length=255, validators=[django.core.validators.RegexValidator(message='Doit être dans le format #F4FA58', regex='^#[A-Za-z0-9]{6}')], verbose_name='Couleur')),
                #('correcting_factor_activated', models.BooleanField(default=True, verbose_name='Activation du facteur de correction')),
            ],
        ),


...

```


Then create a new file inside the migration folder like `0002_FIX_correcting_factor_activated.py` and inside put the following content : 

```py



from django.db import migrations, models


class Migration(migrations.Migration):


    dependencies = [
        ('shops', '0001_initial'),
    ]

    operations = [
        migrations.AddField(  
            model_name='Shop',
            name='correcting_factor_activated',
            field=models.BooleanField(default=True, verbose_name='Activation du facteur de correction'),
        ),
       
    ]


```
What it does ? It does basically add a field inside the Shop model.

After that do again :

- `python manage.py makemigrations`
- `python manage.py migrate`


And repeat for each missing field. 

After that, you should be able to run a Django server.