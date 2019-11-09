---
title: Building a carpool web API with Django (part 3)
date: 2019-11-09
tags: [Django, Python]
---
In [part 2]({{ site.baseurl }}/building-a-carpooling-api-with-django-part-2/) of the series we added some validation to our custom `User` model as well as unit tests. In this post we are going to create more models -- `Vehicle`, `Place` and `Trip`. 

## Vehicle
Let's start by adding a `Vehicle` model that will hold information about a vehicle such as make, model, image etcetera.  Let's go to our `kapool` directory and activate our virtual environment by running `pipenv shell`. You should see the prefix `(kapool)` on your terminal to indicate that the virtual environment is now active. Let's install [pillow](https://python-pillow.org/) an image processing library that Django uses. Run the following command:
```
pipenv install pillow==6.2.0
```
With pillow installed let's create a new app called `vehicles` by running this command:
```
python manage.py startapp vehicles
```
Before we forget we need to register our `vehicles` app to our project by adding it to the list of `INSTALLED_APPS` in our `kapool_project/settings.py`:
```python
# kapool_project/settings.py
INSTALLED_APPS = [
    # ...
    'vehicles.apps.VehiclesConfig', # new
]
```
While we are still in the `settings` module we need to tell Django where our media files will be stored. Let's add `MEDIA_ROOT` and `MEDIA_URL` configurations to our settings.

```python
# kapool_project/settings.py
MEDIA_URL = '/media/' # new
MEDIA_ROOT = os.path.join(BASE_DIR, 'media') # new
```
Now let's go to the `vehicles/models.py` file and create or `Vehicle` model:
```python
# vehicles/models.py
from django.contrib.auth import get_user_model
from django.db import models
from django.utils.translation import ugettext_lazy as _


class Vehicle(models.Model):
    user = models.ForeignKey(
        get_user_model(),
        on_delete=models.CASCADE,
        related_name='vehicles',
        verbose_name=_('Owner')
    )

    make = models.CharField(
        max_length=30,
        verbose_name=_('Make'),
        null=False,
        blank=False
    )

    model = models.CharField(
        max_length=30,
        verbose_name=_('Model'),
        null=False,
        blank=False
    )

    reg_number = models.CharField(
        max_length=30,
        verbose_name=_('Registration number'),
        null=False,
        blank=False
    )

    image = models.ImageField(
        upload_to='vehicles/',
        verbose_name=_('Image'),
        null=True,
        blank=True
    )

    def __str__(self):
        return f'{self.make} {self.model} owned by {self.user}'

    class Meta:
        verbose_name = _('Vehicle')
        verbose_name_plural = _('Vehicles')

```
Let's make migrations by running the following command:
```
python manage.py makemigrations
```
And migrate our changes to the database:
```
python manage.py migrate
```
We now need to register our vehicle model to our admin page:
```python
# vehicles/admin.py
from django.contrib import admin
from .models import Vehicle


@admin.register(Vehicle)
class VehicleAdmin(admin.ModelAdmin):
    list_display = ('make', 'model', 'reg_number')

```
Now run your server using this command `python manage.py runserver` and go to `http://localhost:8000/admin/`. You will see `Vehicles` section where you can view and add vehicles.

## Place
We need to add a `Place` model that will represent the origin and destination of a `Trip`. Once again let's create a new app called `places` by running this command after we've stopped our server if it was running:
```
python manage.py startapp places
```
And add the new app to the `INSTALLED_APPS` list in `kapool_project/settings.py`:
```python
# kapool_project/settings.py
INSTALLED_APPS = [
    # ...
    'places.apps.PlacesConfig', # new
]
```
Let's now create the `Place` model inside `places/models.py`:
```python
# places/models.py
from django.db import models
from django.utils.translation import ugettext_lazy as _


class Place(models.Model):
    name = models.CharField(
        max_length=100,
        unique=True,
        null=False,
        blank=False,
        verbose_name=_('Name')
    )

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = _('Place')
        verbose_name_plural = _('Places')

```
Let's migrate our changes to the database:
```
python manage.py makemigrations && python manage.py migrate
```

Register the model to the admin site:
```python
from django.contrib import admin
from .models import Place


@admin.register(Place)
class PlaceAdmin(admin.ModelAdmin):
    list_display = ('name',)

```
You can run your server and go to the admin page to see if you have a `Places` section.

## Trip
Let's now add the `Trip` model and like before we first create a new app called `trips`:
```
python manage.py startapp trips
```
and add it to the list of installed apps:
```python
# kapool_project/settings.py
INSTALLED_APPS = [
    # ...
    'trips.apps.TripsConfig', # new
]
```
We can now create our `Trip` model in `trips/models.py`:
```python
# trips/models.py
from datetime import date
from django.core.exceptions import ValidationError
from django.contrib.auth import get_user_model
from django.db import models
from django.utils.translation import ugettext_lazy as _

from places.models import Place
from vehicles.models import Vehicle


class Trip(models.Model):
    user = models.ForeignKey(
        get_user_model(),
        on_delete=models.CASCADE,
        null=False,
        blank=False,
        related_name='trips',
        verbose_name=_('Driver'),
    )

    origin = models.ForeignKey(
        Place,
        on_delete=models.CASCADE,
        null=False,
        blank=False,
        related_name='trip_origins',
        verbose_name=_('From')
    )

    destination = models.ForeignKey(
        Place,
        on_delete=models.CASCADE,
        null=False,
        blank=False,
        related_name='trip_destinations',
        verbose_name=_('To')
    )

    vehicle = models.ForeignKey(
        Vehicle,
        on_delete=models.CASCADE,
        null=False,
        blank=False,
        related_name='trip_vehicles',
        verbose_name=_('Vehicle')
    )

    trip_date = models.DateField(
        null=False,
        blank=False,
        verbose_name=_('Trip date')
    )

    def save(self, *args, **kwargs):
        if self.origin == self.destination:
            raise ValidationError(
                'Origin and destination cannot be the same'
            )
        if self.trip_date < date.today():
            raise ValidationError(
                'Trip date cannot be in the past'
            )
        super(Trip, self).save(*args, **kwargs)

    def __str__(self):
        return f'{self.origin} to {self.destination} by {self.user}'

    class Meta:
        verbose_name = _('Trip')
        verbose_name_plural = _('Trips')

```
We have added some validation to our model to ensure that `origin` and `destination` cannot be the same and also the trip date cannot be in the past. Now let's migrate our model to the database:
```
python manage.py makemigrations && python manage.py migrate
```
And finally let's register our model to the admin site:
```python
# trips/admin.py
from django.contrib import admin

from .models import Trip


@admin.register(Trip)
class TripAdmin(admin.ModelAdmin):
    list_display = ('trip_date', 'origin', 'destination', 'user',)

```

## What did we do?
In this post we created three models -- `Vehicle`, `Place` and `Trip` with some validation on the  `Trip` model. Now we have most of the models we need to build our API. In the next post we will start working on our `REST` API. You can check the source code for the project on [GitHub](https://github.com/vince-nyanga/KaPool). Once again, thanks so much for reading.