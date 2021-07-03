---
layout: post
title: Django Azure MS SQL Databae
date: 2021-07-03 16:32:45 +0200
# intro: This short post will explain how to use an Azure database with Django
categories: [Django, Python, Cloud, Azure]
---

I have been playing around with Django. This is a great framework that is based on Python.

One reason why I have develop a great love for this framwork is its built in auth system and admin dashboard system. 

This framework generates you database table after you created your models and also an admin dashboard where you can easily manipulate the data in your database. Usually I spend hours on adding the feature for work and sometime there is not enough time and then I neglect this feature and rather manipulate data directly in the data and thats playing with fire!

Today I have tried to connect am Azure MS SQL database engine onto a Django app and it took me a while to figure out as the pip library does not support Django 3.2 and only up to 3.0.

## Prerequisites:

This post assumes that you already have the following:
1. Django 
2. pip 
3. You have already created a database on Azure and you have your hostname, username & password

If you have not already install the following, then follow the following the tutorial on the official [Django](https://www.djangoproject.com/) website.

## Let's Begin!:

### Setup
Create a new Django project with `python-admin startproject django_azure` or with any other name.

First verify that your Django project works..

Run: 

```console
$ python managt.py runserver
```

Then open `localhost:8000`, you will be greeted by a welcome page with a rocket animation. This confirms that your Django project works.

![Django Welcome](https://user-images.githubusercontent.com/17809351/124359381-f575bf00-dc24-11eb-82b5-921aa507330c.png)

### Setup for Azure

Now we have to connect our Django project with our Azure Database. 

If you are running Linux, then you need to install the OBDC driver. I am running Manjaro, thus I can easily install it from AUR by running the follwoing: 

```sh
$ pamac build msodbcsql
```

Otherwise of you use Ubuntu or a distro similar then run:

```sh
$ sudo su
$ curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
$ curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
$ exit
$ sudo apt-get update
$ sudo ACCEPT_EULA=Y apt-get install msodbcsql=13.1.4.0-1 mssql-tools-14.0.3.0-1 unixodbc-dev
$ echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
$ echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
$ source ~/.bashrc
```

After the intsallation finished, check if it is valid by running:

```sh
$ odbcinst -d -q
```

At the time of writing this post you will see the following:

```sh
[ODBC Driver 17 for SQL Server]
```

You might see a different version if a newer one was ever released.

### Configure your Django project for Azure

Now we need to install the engine that Django will use to communicate with Azure:

```sh
$ pip install django-mssql-backend
```

In your Django project, open the settings.py file and locate the DATABASES section. Remove the existing default database and add in the following:

```python
DATABASES = {
    'default': {
         'ENGINE': 'sql_server.pyodbc',
         'NAME': '<Database Name>',
         'USER': '<Username>',
         'PASSWORD': '<Password>',
         'HOST': '<Server Name>',
         'PORT': '',
         'OPTIONS': {
             'driver': 'ODBC Driver 17 for SQL Server'
        }
    }
}
```

**PLEASE NOTE:** The < Username > & < Password > are the credentials you have entered when you initially created the database.

### Migrate Tables

At this time of writing this post, the django-mssql-backend library only support up to Django 3.0. When you run `python manage.py migrate`, then the command will return an error and the migration task failed.

In order to overcome this issue, you only need to downgrade Django temporarily for the initial migration and after that you can use the latest version of Django again.

To downgrade Django to version 3.0, run the following:

```sh
$ pip install "Django~=3.0.0"
```

After this command finished, you can run the following to confirm the version:

```sh
$ python
Python 3.9.5 (default, May 24 2021, 12:50:35) 
[GCC 11.1.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> import django
>>> print (django.get_version())
3.0.0
>>>
```

Return to your project directory and run:

```console
$ python manage.py migrate
```

You will see similar output as below:

```sh
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
```

After the migration completed successfully, run the following command to upgrade your Django back to the latest version.

```sh
$ python -m pip install -U Django
```

You can check and verify again your Django version.

Run `python manage.py migrate` again to update your tables with the latest changes from the 2 different versions.

## That's It!

After the above step, you will be able to start using an Azure database with Django. You can start creating your apps and your models to add more tables to your database.