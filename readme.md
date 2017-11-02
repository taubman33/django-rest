```bash
$ pip install
$ djangorestframework
$ pip freeze > requirements.txt
```

settings.py
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

urls.py
```python
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
```

settings.py
```python
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
}
````

serializers.py
```py
from rest_framework import serializers
from .models import Artist

class ArtistSerializer(serializers.ModelSerializer):
    class Meta:
        model = Artist
        fields = ('id', 'photo_url', 'nationality',)
```

views.py
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

tunr/urls.py
```py
from rest_framework.urlpatterns import format_suffix_patterns

urlpatterns = [
    url(r'^$', views.artist_list, name='artist_list'),
    url(r'^artists/(?P<pk>\d+)$', views.artist_detail, name='artist_detail'),
    url(r'^artists/new$', views.artist_create, name='artist_create'),
    url(r'^artists/(?P<pk>\d+)/edit$', views.artist_edit, name='artist_edit'),
    url(r'^artists/(?P<pk>\d+)/delete$', views.artist_delete, name='artist_delete'),


    url(r'^songs$', views.song_list, name='song_list'),
    url(r'^songs/(?P<pk>\d+)$', views.song_detail, name='song_detail'),
    url(r'^songs/new$', views.song_create, name='song_create'),
    url(r'^songs/(?P<pk>\d+)/edit$', views.song_edit, name='song_edit'),
    url(r'^songs/(?P<pk>\d+)/delete$', views.song_delete, name='song_delete'),

    url(r'^favorites/(?P<song_id>\d+)/create$', views.add_favorite, name='add_favorite'),
    url(r'^favorites/(?P<song_id>\d+)/remove$', views.remove_favorite, name='remove_favorite')

] + drfpatterns

```

views.py
```py
from rest_framework import generics, permissions


class ArtistList(generics.ListCreateAPIView):
    queryset = Artist.objects.all()
    serializer_class = ArtistSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,)


class ArtistDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Artist.objects.all()
    serializer_class = ArtistSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

![](product.png)