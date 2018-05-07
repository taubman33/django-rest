
## Framing

So far, we have written full-stack Django applications that use Django's builtin templating language to write our applications. When we are building applications in Django that use frontend frameworks or have live updating data, we have to use an API for our backend applications. Today, we are going to learn how to convert our Tunr application we have been working on to a JSON API using a package called Django REST Framework.

## Review: APIs

<details>
  <summary><strong>What is an API?</strong></summary>

  > API stands for "Application Programming Interface." While it technically applies to all of software design, the term has come to refer to web URLs that can be accessed for raw data.

</details>

<details>
  <summary><strong>How can we go about accessing an API within our programs?</strong></summary>

  > Using [jQuery's AJAX method](http://api.jquery.com/jquery.ajax/), [JavaScript's fetch method](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch), [Axios](https://github.com/axios/axios), or any other means of doing HTTP requests .

</details>

<details>
  <summary><strong>What information do we need to provide in order to be able to retrieve information from an API? What about for modifying data in an API?</strong></summary>

  > In order to "GET" or "DELETE" information, we need to provide a `url`, `type`, (HTTP method) and `dataType` (API data format).
  > In order to "POST" or "PUT", we also need to provide some `data`.

  > Example:

  ```js
  fetch('/artists', {
    method: 'POST',
    body: JSON.stringify({
      artist: {
        name: 'Limp Bizkit',
        nationality: 'USA',
        photo_url: 'http://nerdist.com/wp-content/uploads/2014/12/limp_bizkit-970x545.jpg'
      }
    })
  })
  .then((response) =>  response.json())
  .then((response) => {
    console.log(response)
  })
  ```

</details>

## JSON Responses in Django

Using Django's built-in `JsonResponse`, we can send dictionaries or lists as JSON objects in Django without installing any libraries. It will even generate an administrator interface for you to interact with your API in the browser - so no need to use Postman! It is also very customizable, so if you want to change 

For example:

```py
from django.http import JsonResponse

def artist_detail(request):
    data = {
        'name': 'Kanye',
        'photo_url': 'https://upload.wikimedia.org/wikipedia/commons/thumb/1/11/Kanye_West_at_the_2009_Tribeca_Film_Festival.jpg/1920px-Kanye_West_at_the_2009_Tribeca_Film_Festival.jpg',
        'nationality': 'USA'
    }
    return JsonResponse(data)
```

We could also convert our QuerySet of data from our database to a list and then send that as a JsonResponse. Note: you could also use a serializer to convert it to a dictionary and send the data that way.

```py
def artist_list(request):
    artists = Artist.objects.all().values('name', 'nationality', 'photo_url') # only grab some attributes from our database
    artists_list = list(artists) # convert our artists to a list instead of QuerySet
    return JsonResponse(artists_list, safe=False) # safe=False is needed if the first parameter is not a dictionary.

```
This method of sending JSON responses is very similar to what we did in Express; however, there is a simpler way of doing this using Django REST Framework.

## Django REST Framework

Using Django's built-in `JsonResponse`, we can send dictionaries or lists as JSON objects in Django without installing any libraries. It will even generate an administrator interface for you to interact with your API in the browser - so no need to use Postman! It is also very customizable, so if you want to change how your API renders, you can probably do it! 

It is also very widely used -- it is used by Mozilla, Red Hat, Heroku, and Eventbrite.

## Installation and Configuration

Change into your `tunr` directory and make sure you have the latest code from the views and templates lesson. If not, checkout the solution branch from that lesson which is called `views-solution`.

Now, install the `djangorestframework` and save it to your `requirements.txt` file so future developers know to install it as well.

```bash
$ pip install djangorestframework
$ pip freeze > requirements.txt
```

Also, add it to your `INSTALLED_APPS` list in your `settings.py` so that you can use it within your project.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

Further down in your `settings.py` file, configure Django REST Framework to require authentication to create, update, or delete items using your API. Unauthorized users will still be able to perform read actions on your data. This is all the configuration that you need to set up these permissions!

```python
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
}
```

> If you would like to use JWT in your Django REST framework app, [Django REST framework JWT](http://getblimp.github.io/django-rest-framework-jwt/) is awesome and has in-depth documentation on getting it setup. If you are using a front-end framework for your Django application, this is probably the way to go!

We also have to include some URLs for authentication for Django REST framework. These URLs will be used for sign-in and sign-out pages. The framework will handle linking to these pages, we just need to include the URLs that have already been set up.

In the `urls` list in `django/urls.py`, add the following:

```python
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
```

## Serializers

Serializers allow us to convert our data from QuerySets to data that can easily be converted to JSON (or XML) and rendered to our API. There are several types of serializers built into Django REST framework; however, we will be using the `HyperlinkedModelSerializer` today. This serializer allows us to specify model fields that we want to include in our API and it will generate our JSONs accordingly. It will also allow us to link from one model to another.

In this case, we want all of the fields from the Artist model in our serializer, so we will include all of them in our `fields` tuple.

We will create a new file, called `serializers.py` to hold our serializer class.

```py
from rest_framework import serializers
from .models import Artist


class ArtistSerializer(serializers.HyperlinkedModelSerializer):
    songs = serializers.HyperlinkedRelatedField(
        view_name='song-detail',
        many=True,
        read_only=True
    )
    class Meta:
        model = Artist
        fields = ('id', 'photo_url', 'nationality', 'name', 'songs',)
```

The  `Meta` class within our `Artist` serializer class specifies meta data about our serializer. In this class, the model it serializes and the fields we want to serialize. Also, we are creating a HyperlinkedRelatedField. This allows us to link one model to another using a hyperlink. The `view-name` specifies the name of the view given in the `urls.py` file.

### You Do: Create a Serializer for Songs

In the serializers file, add a second serializer for the Song class. Again, include all of the fields from the model in your API.

> Bonus: Try out a different [serializer](http://www.django-rest-framework.org/api-guide/serializers) to relate your models!

> [Solution](https://git.generalassemb.ly/dc-wdi-python-django/tunr/blob/django-rest-framework/tunr/serializers.py)
## Views

Django REST framework has a bunch of utility functions and classes for implementing sets of views in Django. Instead of creating each view individually, Django REST framework will create multiple views for us in a few lines of code.

For example, we can use the `ListCreateAPIView` to create both our list view for our API and our create view. We can also use `RetrieveUpdateDestroyAPIView` to create show, update, and delete routes for our API.

```py
from rest_framework import generics 
from .serializers import ArtistSerializer

class ArtistList(generics.ListCreateAPIView):
    queryset = Artist.objects.all()
    serializer_class = ArtistSerializer


class ArtistDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Artist.objects.all()
    serializer_class = ArtistSerializer
```

### You Do: Add Views for the Songs

Add in the views for the songs. 

> [Solution](https://git.generalassemb.ly/dc-wdi-python-django/tunr/blob/django-rest-framework/tunr/views.py)


## URLs

Since Django can handle multiple request types in one view and using one url, we just need to set up two routes: one for the single view and one for the list view.

```py
from django.conf.urls import url

from rest_framework.routers import DefaultRouter

from . import views

urlpatterns = [
  url(r'artists/$', views.ArtistList.as_view()),
  url(r'artists/(?P<pk>[0-9]+)/$', views.ArtistDetail.as_view())
]
```

### You Do: Add URLs for the Song Views

Add in the urls for the song views. 

> [Solution](https://git.generalassemb.ly/dc-wdi-python-django/tunr/blob/django-rest-framework/tunr/views.py)

![](product.png)
