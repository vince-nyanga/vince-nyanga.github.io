---
title: Building a carpool web API with Django (part 2)
date: 2019-11-06
tags: [Django, Python]
---

In [part 1]({{ site.baseurl }}/building-a-carpooling-api-with-django-part-1/) of the series we created a custom `User` model that we replaced the default model that Django provides. We also added two custom parameters -- `gender` and `birth_date` to our custom `User`. In this post we are going to add some validation to our model as well as unit tests to ensure that the model works the way we expect it to.

## Validation
Before we save a user to our database we want to ensure the model passes the following validations:
- Date of birth is not in the future.
- User is 18 years or older.

Before we proceed we want to install a package that will help us easily make date comparisons. Activate your virtual environment first by running `pipenv shell` and then install `python-dateutil` by running the following command:

```bash
pipenv install python-dateutil
```
Now let's go to our `User` model inside `users/models.py` and override its `save` function.
```python
# users/models.py
from datetime import date # new
from dateutil.relativedelta import relativedelta # new
from django.core.exceptions import ValidationError # new
# ...

class User(AbstractUser):
    # ...
    def save(self, *args, **kwargs):
        if self.birth_date:
            # check if date is in future
            if self.birth_date > date.today():
                raise ValidationError(
                    'Birth date cannot be in the future'
                )
            # check if user is 18 or older
            age = relativedelta(date.today(), self.birth_date).years
            if age < 18:
                raise ValidationError(
                    'User should be 18 years or older'
                )
        super(User, self).save(*args, **kwargs)

```
Now whenever we save a `User` model we first validate the `birth_date` to ensure it passes the validation we described above.

## Unit tests
Let's now write unit tests to ensure that our `User` model works in the way we expect it to. Go to `users/tests.py` and add the following:
```python
# users/tests.py
from datetime import date
from dateutil.relativedelta import relativedelta
from django.contrib.auth import get_user_model
from django.core.exceptions import ValidationError
from django.test import TestCase


class UserTests(TestCase):
    def test_create_user(self):
        User = get_user_model()
        user = User.objects.create_user(
            username='vince',
            email='vince@test.com',
            password='testpass123'
        )
        self.assertEqual(user.username, 'vince')
        self.assertEqual(user.email, 'vince@test.com')
        self.assertTrue(user.is_active)
        self.assertFalse(user.is_staff)
        self.assertFalse(user.is_superuser)

    def test_create_superuser(self):
        User = get_user_model()
        user = User.objects.create_superuser(
            username='vince',
            email='vince@test.com',
            password='testpass123'
        )
        self.assertEqual(user.username, 'vince')
        self.assertEqual(user.email, 'vince@test.com')
        self.assertTrue(user.is_active)
        self.assertTrue(user.is_staff)
        self.assertTrue(user.is_superuser)

    def test_create_user_younger_than_18_should_raise(self):
        User = get_user_model()
        with self.assertRaises(ValidationError) as cm:
            user = User.objects.create_superuser(
                username='vince',
                email='vince@test.com',
                password='testpass123',
                birth_date=date.today()
            )
        error = cm.exception
        self.assertEqual(error.message, 'User should be 18 years or older')

    def test_create_user_future_birth_date_should_raise(self):
        User = get_user_model()
        next_year = date.today() + relativedelta(years=1)

        with self.assertRaises(ValidationError) as cm:
            user = User.objects.create_superuser(
                username='vince',
                email='vince@test.com',
                password='testpass123',
                birth_date=next_year
            )
        error = cm.exception
        self.assertEqual(error.message, 'Birth date cannot be in the future')

```
Now let's run our tests using the following command:
```
python manage.py test
```
If everything went well you should see something like this:
```
Ran 4 tests in 0.393s

OK
Destroying test database for alias 'default'...
```

## What did we do?
In this post we added some validation to our `User` model to ensure that the date of birth cannot be in the future and a user cannot be younger than 18 years old. We also wrote some unit tests for our custom user model to make sure that it be works as expected. You can check the source code for the project on [GitHub](https://github.com/vince-nyanga/KaPool). Once again, thanks for reading.
