---
title: Building a carpool web API with Django (part 6)
date: 2019-11-26
tags: [Django, Python]
---

In [part 5]({{ site.baseurl }}/building-a-carpooling-api-with-django-part-5/) of our series we added more endpoints to our API. In this very short post we are going to add filtering support to our `trips` endpoint to allow users to filter trips.

## Install `django-filters`
First, let's install `django-filters`, a library that we are going to use. Ensure that your virtual environment is active by running `pipenv shell` then install the package:

```
pipenv install django-filters
```
Next, we add it to our list of installed apps inside `kapool_project/settings.py`:

```python
# kapool_project/settings.py

INSTALLED_APPS = [
    # ...
    'django_filters',
]
```
We also need to add default filter backends to the `REST_FRAMEWORK` configuration:

```python
# kapool_project/settings.py

REST_FRAMEWORK = {
    # ...
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
}
```
For more information on adding `django-filters` to your REST API check out the [documentation](https://django-filter.readthedocs.io/en/master/guide/rest_framework.html).

## Trip `FilterSet`
Now let's add a `FilterSet` for our trip model. A `FilterSet` is class that is capable of automatically generating filters for a given `model`'s fields. Inside `trips/` directory create a file `filters.py` and add the following code:
```python
# trips/filters.py

from django_filters import rest_framework as filters

from .models import Trip


class TripFilter(filters.FilterSet):
    """
    FilterSet for `Trip` model.
    """
    
    num_seats = filters.NumberFilter(
        field_name='num_seats',
        lookup_expr='gte')

    origin = filters.CharFilter(
        field_name='origin__name',
        lookup_expr='iexact')

    destination = filters.CharFilter(
        field_name='destination__name',
        lookup_expr='iexact')

    class Meta:
        model = Trip
        fields = [
            'trip_date',
            'num_seats',
            'origin',
            'destination',
        ]

```
Our `FilterSet` will create filters for `trip_date`, `num_seats`, `origin` and `destination`.

Now let's go to `TripViewSet` add add our `FilterSet`:

```python
# trips/views.py

# ...
from .filters import TripFilter

class TripViewSet(ModelViewSet):
    # ...

    filterset_class = TripFilter # new

    # ...

```
We are done! You can run your server `python manage.py runserver` and go to the `trips` endpoint and start filtering your trips. Here are the examples of how you will filter using each of the fields:
- **trip_date**: `api/v1/trips/?trip_date=2019-11-26`
- **num_seats**: `api/v1/trips/?num_seats=3`
- **origin**: `api/v1/trips/?origin=pretoria`
- **destionation**: `api/v1/trips/?destination=johannesburg`
- **All fields**: `api/v1/trips/?trip_date=2019-11-26&num_seats=3&origin=pretoria&destination=johannesburg`

## What did we do?
In this post we added filters for our `trips` endpoint to allow users of our API to filter trips on `trip_date`, `num_seats`, `origin` and `destination`. Thanks for taking your time to read. Hopefully you have learnt something new.

You may view the full source code on [GitHub](https://github.com/vince-nyanga/KaPool). 

