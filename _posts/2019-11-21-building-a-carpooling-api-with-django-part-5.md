---
title: Building a carpool web API with Django (part 5)
date: 2019-11-21
tags: [Django, Python]
---

In [part 4]({{ site.baseurl }}/building-a-carpooling-api-with-django-part-4/) of our series we installed Django Rest Framework to our project and exposed our first API endpoint for the `Users` model. In this post we are going to create endpoints for our other models and have a fairly functional API.

## Update To Code

I made some updates to the source code which I think is pretty straight forward and don't need to be added in the post. Here's the list of the update made and links to the source code where you can check to ensure your code the same as mine:

- **`User` Model** ([link](https://github.com/vince-nyanga/KaPool/blob/41ef739fe9fe541cfdc7a62685586059ddd9392a/users/models.py)): Added `profile_pic` field. After adding it you need to run migrations by running the following command:
  ```
  python manage.py makemigrations && python manage.py migrate
  ```
- **`CustomUserAdmin`** ([link](https://github.com/vince-nyanga/KaPool/blob/41ef739fe9fe541cfdc7a62685586059ddd9392a/users/admin.py)): Updated `add_fieldsets` and `fieldsets`.
- **`UserSerializer`** ([link](https://github.com/vince-nyanga/KaPool/blob/41ef739fe9fe541cfdc7a62685586059ddd9392a/users/serializers.py)): Added `profile_pic` to `fields` list.
- **`Trip` Model** ([link](https://github.com/vince-nyanga/KaPool/blob/41ef739fe9fe541cfdc7a62685586059ddd9392a/trips/models.py)): Added `num_seats` field. Again, migrate your changes to the database using the command above.

## Add More Endpoints

Now that you have made the changes to the models let's add more endpoints to our API to give it more flesh.

### Places

The `places` endpoint is going to expose the list of places and is going to be read-only. New places will only be added and updated in the admin interface. First, let's add a serializer class for our model. Go to the `places/` directory and create a new file `serializers.py` and add the following code:

```python
# places/serializers.py
from rest_framework import serializers

from .models import Place


class PlaceSerializer(serializers.ModelSerializer):
    class Meta:
        model = Place
        fields = [
            'id',
            'name',
        ]

```

We now need a `ViewSet` for the model so let's go to `places/views.py` and add the following:

```python
# places/views.py
from rest_framework import viewsets

from .models import Place
from .serializers import PlaceSerializer


class PlaceViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Place.objects.all()
    serializer_class = PlaceSerializer

```

All that's left is to register the `ViewSet` to our router inside `kapool_project/api.py`:

```python
# kapool_project/api.py

#...
from places.views import PlaceViewSet

#...
router.register('places', PlaceViewSet)

```

Our `places` endpoint is now up and running. You can run your server (`python manage.py runserver`), add places in the admin interface and go to `http://localhost:8000/api/v1/places/` to view the list of your places.

### New Custom Permission

Before we proceed we need to add another custom permission that we will use in our `ViewSet`s. We want this permission to be shared by more than one package so let's create another package called `shared` and add `permissions.py` to which we will add our global permissions:

```python
# shared/permissions.py

from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Gives write access only to the owner of the object.
    """
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.user == request.user

```

This permission will give write access only to the owner of the object.

### Vehicle

Let's now add an endpoint for our `Vehicle` model. Like we did with the `Place` model we will create a serializer and a `ViewSet` then register the `ViewSet` to our router.

```python
# vehicles/serializers.py
from rest_framework import serializers

from .models import Vehicle

class VehicleSerializer(serializers.HyperlinkedModelSerializer):
    owner_url = serializers.HyperlinkedRelatedField(
        source='user',
        view_name='user-detail',
        read_only=True
    )
    class Meta:
        model = Vehicle
        fields = [
            'id',
            'url',
            'make',
            'model',
            'reg_number',
            'image',
            'owner_url',
        ]
```

```python
# vehicles/views.py

from rest_framework import permissions
from shared.permissions import IsOwnerOrReadOnly
from .models import Vehicle
from .serializers import VehicleSerializer


class VehicleViewSet(ModelViewSet):
    queryset = Vehicle.objects.all()
    serializer_class = VehicleSerializer

    permission_classes = [
        IsOwnerOrReadOnly,
        permissions.IsAuthenticatedOrReadOnly,
    ]

    def perform_create(self, serializer):
        # set user to the logged in user
        serializer.save(user=self.request.user)
```

Now register it to the router:

```python
# kapool_project/api.py

# ...
from vehicles.views import VehicleViewSet
# ...
router.register('vehicles', VehicleViewSet)
```

### Trip

Lastly, let's add an endpoint for our `Trip` model. Again, we are going to add a serializer and a `ViewSet` then register the `ViewSet` to the router:

```python
# trips/serializers.py

from datetime import date
from rest_framework import serializers

from vehicles.models import Vehicle
from vehicles.serializers import VehicleSerializer
from places.models import Place
from places.serializers import PlaceSerializer
from users.serializers import UserSerializer
from .models import Trip


class UserFilteredPrimaryKeyRelatedField(serializers.PrimaryKeyRelatedField):
    """
    Filters `PrimaryKeyRelatedField` based on the logged in user.

    See  https://stackoverflow.com/questions/27947143
    """

    def get_queryset(self):
        request = self.context.get('request', None)
        queryset = super(
            UserFilteredPrimaryKeyRelatedField,
            self).get_queryset()
        if not request or not queryset:
            return None
        return queryset.filter(user=request.user)


class TripSerializer(serializers.HyperlinkedModelSerializer):
    origin_id = serializers.PrimaryKeyRelatedField(
        queryset=Place.objects.all(),
        source='origin',
        write_only=True
    )

    origin = PlaceSerializer(read_only=True)

    destination_id = serializers.PrimaryKeyRelatedField(
        queryset=Place.objects.all(),
        source='destination',
        write_only=True

    )
    destination = PlaceSerializer(read_only=True)

    vehicle_id = UserFilteredPrimaryKeyRelatedField(
        queryset=Vehicle.objects,
        source='vehicle',
        write_only=True
    )

    vehicle = VehicleSerializer(read_only=True)

    driver = UserSerializer(source='user', read_only=True)

    def validate_trip_date(self, value):
        if value < date.today():
            raise serializers.ValidationError(
                'Trip date cannot be in the past')
        return value

    def validate(self, data):
        if data['origin'] == data['destination']:
            raise serializers.ValidationError(
                'Origin and destination cannot be the same')

        return data

    class Meta:
        model = Trip
        fields = [
            'id',
            'url',
            'trip_date',
            'num_seats',
            'origin_id',
            'origin',
            'destination_id',
            'destination',
            'vehicle_id',
            'vehicle',
            'driver',
        ]

```

```python
# trips/views.py

from rest_framework.viewsets import ModelViewSet
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from shared.permissions import IsOwnerOrReadOnly

from .models import Trip
from .serializers import TripSerializer


class TripViewSet(ModelViewSet):
    queryset = Trip.objects.all()
    serializer_class = TripSerializer
    permission_classes = [
        IsOwnerOrReadOnly,
        IsAuthenticatedOrReadOnly,
    ]

    def perform_create(self, serializer):
        serializer.save(user=self.request.user)

```

Register it:

```python
# kapool_project/api.py

# ...
from trips.views import TripViewSet
#...
router.register('trips', TripViewSet)
```

You can now go to your API root -- `http://localhost:8000/api/v1/` and see all the endpoints we have created. You can now add new vehicles and trips using the API! Our API is now faily functional but there is still a lot we still need to add to it. Thanks so much for taking your time to read.

You can check the source code for the project on [GitHub](https://github.com/vince-nyanga/KaPool).
