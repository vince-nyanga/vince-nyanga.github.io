---
title: Mocking objects in Python
date: 2018-04-24
tags: [Python, Testing]
---

Over the past 18 months I have been doing a lot of Python programming in my side projects as well as at work. In all my projects I always find myself needing to mock certain objects to ensure that my tests are independent of external frameworks or APIs — something that is almost unavoidable.

### Enter the Mock library

Since I come from an Android background, mocking objects has been part of my development life with `Mockito`the go-to library. When I started serious Python development I wanted something similar to `Mockito` where I could replace certain parts of my system with mock objects during tests and I found it in the [mock](https://docs.python.org/3/library/unittest.mock.html) library. In this post I will talk about how I have used this library in my projects.

### What does it do?

The `mock` library is used to replace parts of a system under test with mock objects and make asserts on how they are used. It helps avoid creating stubs in your test suite — something that can be a pain and time consuming. With this library you can make assertions about which methods were called and the attributes that they were called with. You can also specify the return values or exceptions to be thrown by certain actions.
This library is available in the Python 3 standard library in the `unittest` module. If you’re using Python 2 you need to install it by running the following command.

```bash
pip install mock
```

For more in-depth information about this library please read the [docs](https://docs.python.org/3/library/unittest.mock.html).

### Example

In this example I will use a simple class that takes in an image and uses an external API to find matching faces in the image. Here is the class that we will be using in our example:

```python
class SearchFacesInteractor(Interactor):
    """
    Finds matching faces in an image.
    """

    def __init__(self, api, response_gateway=None):
        if not api:
            raise Exception('api is required')
        self.api = api
        self.response_gateway = response_gateway

    def execute(self, request_object):
        if not request_object:
            raise Exception('request object is required')
        if not self.response_gateway:
            return self.process_request(request_object)
        else:
            self.process_request(request_object)

    def process_request(self, request_object):
        try:
            response = self.api.find_matching_faces(request_object)
            response_object = ResponseObject.success(response)
            if not self.response_gateway:
                return response_object
            self.response_gateway.handle_response(response_object)
        except Exception, e:
            response_object = ResponseObject.error(str(e))
            if not self.response_gateway:
                return response_object
            self.response_gateway.handle_response(response_object)

```

When testing this class we will need to replace the api and response_gateway with mock objects so that we test the class independent of any external entities. Let’s see how the mock library can be used.

#### Making assertions on method calls

The process_request method makes calls to the api as well as the response_gateway. We need to ensure that these calls are made, and with the correct arguments. Here is how we can do it:

```python
from mock import Mock
# ...
def test_should_call_api_and_response_gateway(self):
    mock_api = Mock()
    mock_response_gateway = Mock()
    interactor = SearchFacesInteractor(mock_api, mock_response_gateway)
    interactor.execute('face data')
    mock_api.find_matching_faces.assert_called_once_with('face data')
    mock_response_gateway.handle_response.assert_called_once()
```

In the example above we have used the mock library to assert that the method `find_matching_faces()` in our mock api was called once with the correct arguments. We also asserted that `handle_response()` was called in the mock `response_gateway`.

#### Setting return value

Sometimes you need to test a return value for a method inside your mock object. Here is how it can be done:

```python
from mock import Mock
# ...
def test_execute_no_response_gateway(self):
    mock_api = Mock()
    # Set the return value for the method find_matching_faces()
    attrs = {'find_matching_faces.return_value': 'Success'}
    # Configure mock object
    mock_api.configure_mock(**attrs)
    interactor = SearchFacesInteractor(mock_api)
    response = interactor.execute('data')
    self.assertIsNotNone(response)
    self.assertFalse(response.has_error)
    self.assertEqual('Success', response.data)
```

In the above example we have set the return value for the method `find_matching_faces` and made assertions to check if we get the desired response.

#### Raising exceptions

Sometimes you may want to test how your system handles exceptions. With the mock library you can configure the method under test to raise an exception as shown in the example below.

```python
# ...
def test_execute_with_error(self):
    mock_api = Mock()
    # Set exception to be thrown
    attrs = {'find_matching_faces.side_effect': Exception('Error')}
    # Configure mock object
    mock_api.configure_mock(**attrs)
    interactor = SearchFacesInteractor(mock_api)
    response = interactor.execute('data')
    self.assertIsNotNone(response)
    self.assertTrue(response.has_error)
    self.assertEqual('Error', response.error_message)
```

Now the `find_matching_faces()` method in our mock object will raise an exception.

#### Mocking methods

In the examples above we have been mocking entire objects. What if we would like to use a real object but just mock a certain method inside that object? This can be useful if you want to test certain logic inside your class except for certain components. The mock library makes this possible through the `patch`decorator/context manager. This replaces the object or method specified with a mock or another object during the test and restores it when the test is done. Below is an example of how that can be done.

```python
from mock import Mock, patch
# ...
def test_execute_no_api_call(self):
    with patch.object(SearchFacesInteractor, 'process_request', return_value='Success') as mock_method:
        interactor = SearchFacesInteractor(Mock())
        response = interactor.execute('face data')
        self.assertEqual('Success', response)
    mock_method.assert_called_once_with('face data')
```

In the example above we have mocked out the `process_request()` method inside SearchFacesInteractor and set its return object. We can also make the method raise an exception by setting the `side_effect`.

### Conclusion

n this post I spoke about how the mock library can be used to mock objects and methods during tests. There is so much more one can do with this library. Check out the documentation for more. As a self-taught Python programmer, my code may not be well written. Please bear with me :).
Once again, thanks a million times for taking time to read.
