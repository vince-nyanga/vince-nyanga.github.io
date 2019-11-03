---
title: Building a carpool web API with Django (part 1)
date: 2019-11-03
tags: [Django, Python]
---

In this series we are going to create a web API using Django and `django-restframework`library.

## What is Django
Django is a Python Web framework that allows developers to build websites rapidly and cleanly. It provides developers with a complete suite of tools that make it easy to go from an idea to something working in a very short space of time. There are many sites that are built with Django but the most popular one is Instagram and I believe you've heard about it.

## What are we going to build?
We are going to create a web API for an imaginary carpooling service called **KaPool** (I'm very bad at naming things). This is going to be a very minimalistic and simple system and is going to have the following features:
- A user with a car can add a `Trip` from point A to pont B.
- Users who need a ride can browse available trips.
- If a user finds a driver they like they will place a reservation which the driver can either accept or deny.
- User can view all their trips both as a driver and as a passenger.
- More features might be added as we go.

I'm by no means a pro when it comes to Django and will be learning as well during this series and hopefully we will gain some knowledge of this wonderful framework together. 

## Let's get started
Before we proceed make sure you have Python 3+ is installed on your machine and you have Pipenv installed as well.

In this part of the series we are going to add a `User` model that will contain properties for our users. To start let us create our django project, but first, we need to create the folder that is going to contain our code. I'm gonna put mine in a `desktop/code/kapool`. While you are in your `kapool` folder install django by running the following command:
```bash
pipenv install django==2.2.5
```
Now let's spawn a shell in a virtual environment and get going:
```bash
pipenv shell
```

With our virtual environment now active, let's create the django project:
```bash
django-admin startproject kapool_project .
```
Don't forget the `.` at the end so that our directory structures are similar. After you enter that command django will create a project for you and you will be good to go. To check if everything is working run the following command:
```bash
python manage.py runserver
```

Go to `http://localhost:8000/` and you will see a page that looks like the one below.
<figure>
<img src="{{ site.baseurl }}/images/kapool/kapool-1.png" alt="Django welcome page">
<figcaption>Welcome page</figcaption>
</figure>
If you see this then you are on the right track.

### `Users` app
A django project constists of individual apps that perform various tasks, you can have a `users` app, a `blog` app, or `products` app inside one project which each app depending on other apps in a way. In our case we are going to create a `users` app that will contain logic related to our users. Let's create the app by running the following command:
```bash
python manage.py startapp users
```
We need to add the app we have just created to our project. Go to `kapool_project/settings.py` and add the following inside the `INSTALLED_APPS` list:

```python
# kapool_project/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'users.apps.UsersConfig', # add this...
]
```
Now that we have added our `users` app to our project let's create our `User` model inside `users/models.py`. Our `User` model will inherit from the `AbstractUser` that django provides. We will use this custom `User` model to replace the one django provides because we want to add more properties to it (the defaut model).

```python
# users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models
from django.utils.translation import ugettext_lazy as _

GENDER_OPTIONS = (
    ('female', 'Female'),
    ('male', 'Male'),
    ('wont-say', "Won't say"),
)

class User(AbstractUser):
    """
    Custom user model to replace the one provided by django
    """
    gender = models.CharField(
        null=True,
        blank=True,
        max_length=30,
        choices=GENDER_OPTIONS,
        default='wont-say',
        verbose_name=_('Gender'),
        help_text=_("User's gender")
    )

    birth_date = models.DateField(
        null=True,
        blank=True,
        verbose_name=_('Date of birth'),
        help_text=_('Date of birth')
    )

    def __str__(self):
        return self.email

    class Meta:
        verbose_name = _('User')
        verbose_name_plural = _('Users')

```

Let us now tell django that we want to replace the default `User` model with our own model. Add the following to `kapool_project/settings.py`:

```python
# kapool_project/settings.py
# ...
AUTH_USER_MODEL = 'users.User'
```

Our custom user model needs to be registered to the admin site so that we may add new users through the admin page. Fistly let's create forms that will be used for adding and updating our `User` model:
```bash
touch users/forms.py
```
Inside the `users/forms.py` file add the following code:

```python
# users/forms.py
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import (UserCreationForm, 
UserChangeForm)

class CustomUserCreationForm(UserCreationForm):
    """
    Custom user creation form
    """

    class Meta:
        model = get_user_model()
        fields = (
            'username', 
            'email', 
            'first_name', 
            'last_name',
             'birth_date', 
             'gender'
        )

class CustomUserChangeForm(UserChangeForm):
    """
    Custom user change form
    """

    class Meta:
        model = get_user_model()
        fields = (
            'username', 
            'email', 
            'first_name', 
            'last_name',
             'birth_date', 
             'gender'
        )

```
Lastly, lets register our custom `User` to the admin page. Go to `users/admin.py` and add the following:
```python
# users/admin.py
from django.contrib import admin
from django.contrib.auth import get_user_model
from django.contrib.auth.admin import UserAdmin

from .forms import CustomUserChangeForm, CustomUserCreationForm

CustomUser = get_user_model()

class CustomUserAdmin(UserAdmin):
    """
    Custom user admin
    """

    add_form = CustomUserCreationForm
    form = CustomUserChangeForm
    list_display = [
        'username',
        'first_name', 
        'last_name', 
        'email', 
        'gender', 
        'birth_date'
    ]

    add_fieldsets = (
            (
                'User info',
                {
                    'classes': ('wide',),
                    'fields': (
                        'username',
                        'email', 
                        'password1', 
                        'password2', 
                        'gender', 
                        'birth_date'
                    ),
                },
            ),
    )

    fieldsets = (
            (
                'User info',
                {
                    'classes': ('wide',),
                    'fields': (
                        'username',
                        'email', 
                        'gender', 
                        'birth_date'
                    ),
                },
            ),
    )

admin.site.register(CustomUser, CustomUserAdmin) 

```
Let's now make migrations that will add our `User` model to our database by running the following commands:
```bash
python manage.py makemigrations
```
```bash
python manage.py migrate
```
Let's create a super user that we will log into the admin page with to see if everything is working as planned. Run the following command and follow the prompts that follow:
```
python manage.py createsuperuser
```
You should get this message is everything went well: `Superuser created successfully.`

Let's run our server (`python manage.py runserver`), go to `http://localhost:8000/admin` and login as the user we created. Once you log into your admin page you will be able to add and edit users. That will be all for today.

## What did we do?
We created our **KaPool** project and added a custom `User` model to replace the one django provides. This enables us to add new properties to our model -- `birth_date` and `gender` in this instance. We are going to add more features to our model as we build the API. We also registered our custom user to the django admin page which enables us to add more users as well as edit users in the admin page.

Once again thanks for taking your time to read. It has been a learning experience for me as well -- there are some things that I did not know how to do that I now know and I hope you learned something too. 

In the next post we are going add some validation to our `User` model as well as unit tests so stay tuned.