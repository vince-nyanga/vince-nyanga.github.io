---
title: Building a carpool web API with Django (part 7)
date: 2020-01-02
tags: [Django, Python]
---

Happy New Year! I hope you had a wonderful time during the December holidays. Myself I had so much fun at my rural home in Zimbabwe where the we were celebrating my grandfather's 80th birthday. I also experienced the effects of climate change first hand. There was a severe shortage of drinking water both for humans and livestock which forces everyday life to revolve around looking for water, digging wells and rescuing animals that fall into muddy pools. Now that the holidays are over we can continue working on our REST API. 

In [part 6]({{ site.baseurl }}/building-a-carpooling-api-with-django-part-6/) of our series we added filtering support to our `trips` endpoint to allow users to filter trips by origin, destination, number of seats and date. In this post we are going to talk about writing unit tests for our REST API. This is going to be a very short read as most of the things are repetitive. We are going to work with the `trips` endpoint. For more in-depth information on writing tests with the Django REST Framework visit this [link](https://www.django-rest-framework.org/api-guide/testing/).

## `TripApiTest`
Inside `trips/tests.py` add the following:
```python
# trips/tests.py
# ...

from rest_framework import status
from rest_framework.test import APITestCase

# ...
def create_data():
    user = User.objects.create_user(
        username='vince',
        email='vince@test.com',
        password='testpass123'
    )

    vehicle = Vehicle(
        make='Make',
        model='Model',
        reg_number='1234',
        user=user
    )
    vehicle.save()

    origin = Place(name='Origin')
    origin.save()

    destination = Place(name='Destination')
    destination.save()
```
### Test Get Trips
Let's create our `TripApiTest` class that inherits from `APITestCase` and add a test for getting trips:

```python
# trips/tests.py
class TripApiTest(APITestCase):

    def test_get_trips(self):
        create_data()
        user = User.objects.get(pk=1)
        vehicle = Vehicle.objects.get(pk=1)
        origin = Place.objects.get(pk=1)
        destination = Place.objects.get(pk=2)

        trip = Trip(
            user=user,
            origin=origin,
            destination=destination,
            vehicle=vehicle,
            trip_date=date.today()
        )
        trip.save()

        response = self.client.get('/api/v1/trips/')
        self.assertEqual(json.loads(response.content), [
            {
                "id": 1,
                "url": "http://testserver/api/v1/trips/1/",
                "trip_date": str(date.today()),
                "num_seats": 1,
                "origin": {
                    "id": 1,
                    "name": "Origin"
                },
                "destination": {
                    "id": 2,
                    "name": "Destination"
                },
                "vehicle": {
                    "id": 1,
                    "url": "http://testserver/api/v1/vehicles/1/",
                    "make": "Make",
                    "model": "Model",
                    "reg_number": "1234",
                    "image": None,
                    "owner_url": "http://testserver/api/v1/users/1/"
                },
                "driver": {
                    "username": "vince",
                    "email": "vince@test.com",
                    "first_name": "",
                    "last_name": "",
                    "gender": "wont-say",
                    "birth_date": None,
                    "url": "http://testserver/api/v1/users/1/",
                    "profile_pic": None
                }
            }

        ])
```
The `APITestCase` class provide a `self.client` attribute which is an instance of `APIClient`. It gives us the ability to easily call REST methods -- `get`, `post` etc. 

### Adding authentication to tests
The `APIClient` class provides many ways of authenticating your requests. Below are the different ways of authentication and when they are useful.

#### .login
The `APIClient` class provides a `login` method. It allows you to authenticate requests against any views which include `SessionAuthentication`. This method is appropriate for testing APIs that use session authentication, for instance websites which include AJAX interaction with the API. Here's an example of how you can use the `login` method:
```python
client = APIClient()
client.login(username='vince', password='testpass1234')

# log out
client.logout()
```

#### .credentials
The `credentials` method is used to set headers that will be included in all requests by the test client. This method is appropriate for testing APIs that require authentication headers, such as basic authentication, OAuth and simple token authentication schemes. Here is an example of how you can use the `credentials` method:
```python
from rest_framework.authtoken.models import Token
from rest_framework.test import APIClient

token = Token.objects.get(user__username='vince')
client = APIClient()
client.credentials(HTTP_AUTHORIZATION='Token ' + token.key)
```

#### .force_authenticate
This method is used when you want to bypass authentication entirely and force all requests to be automatically treated as authenticated. This is a very useful shortcut if you don't want to construct valid authentication credentials. We are going to use this method when we test adding a trip.

Inside the `TripApiTest` class add a test for adding a new trip:

```python
def test_create_trip(self):
        create_data()
        user = User.objects.get(pk=1)

        self.client.force_authenticate(user=user) # force authentication

        response = self.client.post('/api/v1/trips/', {
            'trip_date': str(date.today()),
            'origin_id': 1,
            'destination_id': 2,
            'vehicle_id': 1
        })
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        trip = Trip.objects.get(pk=1)
        self.assertEqual(trip.origin.name, 'Origin')
        self.assertEqual(trip.destination.name, 'Destination')
        self.assertEqual(trip.trip_date, date.today())
        self.assertEqual(trip.user, user)
```
I am not going to add all the tests here. I just wanted to show you how you can write unit tests for your REST API. If you want to see all the tests check them out on [GitHub](https://github.com/vince-nyanga/KaPool). Many thanks for taking your time to read and once again, Happy New Year! I wish you a happy and prosperous 2020.


