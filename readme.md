# References

1. [Test Driven Caching Article](https://testdriven.io/blog/django-caching/)

# Django Caching Types
built-in options are:

1. *Memcached* - is a memory-based, key-value store for small chunks of data. It supports distributed caching across multiple servers. https://memcached.org/

2. *Database* - Here, the cache fragments are stored in a database. A table for that purpose can be created with one of the Django's admin commands. This isn't the most performant caching type, but it can be useful for _storing complex database queries_.

3. *Local Memory* - The cache is saved on the file system, in separate files for each cache value. This is the slowest of all the caching types, but it's the easiest to set up in a production environment.

4. *Dummy* - A "dummy" cache that doesn't actually cache anything but still implements the cache interface. It's meant to be used in development or testing when you don't want caching, but do not wish to change your code.


# Django Caching Levels

- per-site cach
- per-view cache
- template fragment cache
- low-level cache API


## a) Per-Site Cache
- easiest to implement
```python
# settings.py
MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',     # NEW
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',  # NEW
]
```
- [middleware order is important](https://docs.djangoproject.com/en/3.2/topics/cache/#order-of-middleware)
- `UpdateCacheMiddleware` comes before `FetchFromCacheMiddleware`

- add the following too
```python
# settings.py

CACHE_MIDDLEWARE_ALIAS = 'default'  # which cache alias to use
CACHE_MIDDLEWARE_SECONDS = '600'    # number of seconds to cache a page for (TTL)
CACHE_MIDDLEWARE_KEY_PREFIX = ''    # should be used if the cache is shared across multiple sites that use the same Django instance
```


## b) Per-View cache
- almost always start with this when starting to implement caching.
- use the [cache_page](https://docs.djangoproject.com/en/3.2/topics/cache/#django.views.decorators.cache.cache_page) decorate either on view function or in path

```python
# urls.py

from django.views.decorators.cache import cache_page

@cache_page(60 * 15)
def your_view(request):
    ...

# or

from django.views.decorators.cache import cache_page

urlpatterns = [
    path('object/<int:object_id>/', cache_page(60 * 15)(your_view)),
]
```

- Cache is url-based e.g `objec/1` and `object/2` will be cached separately.
- prefer implementing the cache on url. Implementing on view makes it difficult to disable the cache on certain scenarios 

```python
from django.views.decorators.cache import cache_page

urlpatterns = [
    path('object/<int:object_id>/', your_view),
    path('object/cache/<int:object_id>/', cache_page(60 * 15)(your_view)),
]
```


## c) Template Fragment Cache
- allows specifying specific areas of a template cache

```html
<!--some_template.html-->
{% load cache %}

{% cache 500 object_list %}
  <ul>
    {% for object in objects %}
      <li>{{ object.title }}</li>
    {% endfor %}
  </ul>
{% endcache %}
```
`{% cache 500 object_list %}` here cache ttl is 500 seconds. Cache fragment name is object_list


## d) Low-level cache API

- helps in managing individual objects in the cache by cache key

example
```python
from django.core.cache import cache


def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)

    objects = cache.get('objects')

    if objects is None:
        objects = Objects.all()
        cache.set('objects', objects)

    context['objects'] = objects

    return context
```
- ensure you invalidate the cache when objects are added/changed/removed from the db e.g using signals

```python
# signals.py

from django.core.cache import cache
from django.db.models.signals import post_delete, post_save
from django.dispatch import receiver


@receiver(post_delete, sender=Object)
def object_post_delete_handler(sender, **kwargs):
     cache.delete('objects')


@receiver(post_save, sender=Object)
def object_post_save_handler(sender, **kwargs):
    cache.delete('objects')
```

[more info](https://testdriven.io/blog/django-low-level-cache/)

