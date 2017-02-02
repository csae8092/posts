# Implement vocabs-app
In this post we will implement the vocabs-app to our play-project set up [so far](../part-3-a-custom-app) and import some predefined controlled vocabularies from an .xlsx document.

# Copy and integrate vocabs-app

To get started we have to

1. download the zipped vocabs-app from [here]((https://github.com/csae8092/posts/raw/master/django-generic-apps/downloads/vocabs.zip))
2. Unzip it and move it into rlunch's root directory.
3. Register 'vocabs' in 'INSTALLED_APPS' in rlunch/settings.py file.
4. Include 'vocabs' urls.py file in rlunch's urls.py file `url(r'^vocabs/', include('vocabs.urls', namespace='vocabs')),`
5. Include links to the vocabs classes in your base template, maybe like this:

```html
{% if user.is_authenticated %}
  <li class="dropdown">
    <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Vocabs Curation
    <span class="caret"/>
    </a>
    <ul class="dropdown-menu procustom-dropdown-menu">
      <li>
          <a href="{% url 'vocabs:skosconceptscheme_list' %}">Concept Schemes</a>
      </li>
      <li>
          <a href="{% url 'vocabs:browse_vocabs' %}">Concepts</a>
      </li>
      <li>
          <a href="{% url 'vocabs:skoslabel_list' %}">Labels</a>
      </li>
      <li role="separator" class="divider"></li>
      <li>
          <a href="{% url 'vocabs:skos_import' %}">Import SKOS (RDF/XML)</a>
      </li>
      <li>
          <a href="{% url 'api-root' %}">Export SKOS (API)</a>
      </li>
    </ul>
  </li>
{% endif %}

```

After this steps, when you (re)start the dev-server, you will most likly be presented with the following error message(s):

> ImportError: No module named 'django_tables2'

This message indicates a missing package, so we have to install this by `$ pip install django-tables2` and include this package ('django_tables2',) in INSTALLED_APPS.

Starting the dev-server again will throw this error:

> ImportError: No module named 'dal'

This means we need to install `$ pip install django-autocomplete-light==3.2.1`, and register it (dal', 'dal_select2') before django.contrib.admin and grappelli if present ([see](https://django-autocomplete-light.readthedocs.io/en/master/install.html)).

The next error will complain about missing 'django_filters' plugin, so run `$ pip install django-filter` (without s!) and register ('django_filters').

The next error will require lxml. Install this, which on windows might be a bit tricky if you try to use the officail pip release, so I will run this command: `$ conda install -c anaconda lxml=3.6.4`

Now you should be enabled to run the dev-server without getting any erros but when you try to browse to [http://127.0.0.1:8000/](http://127.0.0.1:8000/) you will be confronted with another error complaining about missing 'api-root' URL.

# DRF - django rest framework

This error comes from not yet installed and configured DRF-packages which is used by the vocabs app to set up an API to expose its data.
So what we have to do is

1. install DRF by running: `pip install djangorestframework==3.5.3`, adding 'rest_framework' to INSTALLED_APPS as well as DRF-configurtaion:

```python

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': ('rest_framework.permissions.IsAuthenticatedOrReadOnly',),
    'PAGE_SIZE': 10
}

```

Now we only have to include the api-URLs:

```python

#rlunch/urls.py

from django.conf.urls import url, include
from django.contrib import admin
from rest_framework import routers

from vocabs import api_views

router = routers.DefaultRouter()
router.register(r'skoslabels', api_views.SkosLabelViewSet)
router.register(r'skosnamespaces', api_views.SkosNamespaceViewSet)
router.register(r'skosconceptschemes', api_views.SkosConceptSchemeViewSet)
router.register(r'skosconcepts', api_views.SkosConceptViewSet)

urlpatterns = [
    url(r'^api/', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
    url(r'^admin/', admin.site.urls),
    url(r'^', include('webpage.urls', namespace='webpage')),
    url(r'^lunch/', include('lunch.urls', namespace='lunch')),
    url(r'^vocabs/', include('vocabs.urls', namespace='vocabs')),
    url(r'^vocabs-ac/', include('vocabs.dal_urls', namespace='vocabs-ac')),
]


```

and we should be ready to go.

# Import SKOS (from RDF/XML)
