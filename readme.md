# References

1. [Test Driven Caching Article](https://testdriven.io/blog/django-caching/

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
