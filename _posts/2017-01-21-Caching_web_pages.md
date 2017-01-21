---
layout: post
title: A simple web page cache
---

Our project this week involves web scraping. The first thing I did was
write a little code to cache web pages locally. That way, I'm a better
web citizen for not hitting the host with repeat requests for the same
page, and I can work faster when refining my design and algorithms.

The cache is very basic. The function ```get_page``` takes a URL and
returns the page as a string. The first time you get a URL, it
downloads it (using ```requests.get```), saves a copy in a
```.\cache``` subdirectory, and returns the text. The next time you
request the same URL, it returns the cached copy.

The cache uses ```uuid.uuid4``` to generate unique names for the local
files, and a simple dictionary as the index that maps URLs to file
names. The index is saved in the ```./cache/index.pkl```.

To discard the cache and start over, just delete the
```./cache``` directory.

Here's the code:

```python
import os
import pickle
import requests
import uuid

_cache = None
_cache_dir = "./cache"
_cache_index = os.path.join(_cache_dir, 'index.pkl')

def cache_init():
    global _cache
    if _cache == None:
        if os.path.exists(_cache_index):
            with open(_cache_index, 'rb') as fd:
                _cache = pickle.load(fd)
        else:
            _cache = {}
    return _cache

def cache_get(key):
    return cache_init().get(key, '')

def cache_add(key, value):
    cache = cache_init()
    cache[key] = value
    with open(_cache_index, 'wb') as fd:
       pickle.dump(cache, fd)

def get_page(url):
    """Get a web page."""

    # Check if we have this page
    
    filename = cache_get(url)
    if filename and os.path.exists(filename):
        with open(filename, 'rb') as fd:
            return fd.read()

    # Otherwise, download the page ...
    
    r = requests.get(url, timeout=10)
    r.raise_for_status()
    
    # ... and cache it

    global _cache_dir
    if not os.path.isdir(_cache_dir):
        os.mkdir(_cache_dir)
        
    if not filename:
        filename = os.path.join(_cache_dir, uuid.uuid4().hex)

    with open(filename, 'wb') as fd:
        for chunk in r.iter_content(chunk_size=4096):
            fd.write(chunk)

    cache_add(url, filename)
    
    return r.text
```

Nothing fancy here. Just enough to do the job for this project. Enjoy!
