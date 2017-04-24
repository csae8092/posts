# Implement vocabs-app
In this post we will implement the vocabs-app to our play-project set up [so far](../part-3-a-custom-app) and import some predefined controlled vocabularies from an .xlsx document.

# Copy and Integrate vocabs-app

To get started we have to

1. download the zipped vocabs-app from [here](https://github.com/csae8092/posts/raw/master/django-generic-apps/downloads/vocabs.zip)
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

After this steps, when you (re)start the development server, you will most likely be presented with the following error message(s):

> ImportError: No module named 'django\_tables2'

This message indicates a missing package, so we have to install this by `$ pip install django-tables2` and include this package ('django\_tables2',) in INSTALLED\_APPS.

Starting the dev-server again will throw this error:

> ImportError: No module named 'dal'

This means we need to install `$ pip install django-autocomplete-light==3.2.1`, and register it ('dal', 'dal_select2') before django.contrib.admin and grappelli if present ([see](https://django-autocomplete-light.readthedocs.io/en/master/install.html)).

The next error will complain about missing 'django\_filters' plugin, so run `$ pip install django-filter` (without s!) and register ('django\_filters').

The next error will require lxml. Install this, which on windows might be a bit tricky if you try to use the officail pip release, so I will run this command: `$ conda install -c anaconda lxml=3.6.4`

Now you should be enabled to run the dev-server without getting any erros but when you try to browse to [http://127.0.0.1:8000/](http://127.0.0.1:8000/) you will be confronted with another error complaining about missing 'api-root' URL.

Notes for Linux: you may run into an import error for libgcrypt.so.11. In Debian/Ubuntu you could try to download and install it from [here](ftp://ftp.debian.org/debian/pool/main/libg/libgcrypt11/).


# DRF - django rest framework

This error comes from not yet installed and configured DRF-packages which is used by the vocabs app to set up an API to expose its data.
So what we have to do is

1. install DRF by running: `pip install djangorestframework==3.5.3`, adding 'rest\_framework' to INSTALLED\_APPS as well as DRF-configurtaion:

```python

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': ('rest_framework.permissions.IsAuthenticatedOrReadOnly',),
    'PAGE_SIZE': 10
}

```

2. and include the api-URLs:

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

Now we should be ready to go.
In case you find installing all those packages mentioned above cumbersome - like I do - be assured that there is vocabs_requirements.txt file located in vocabs.zip. So you can run `$ pip install -r vocabs_requirements.txt` to install all needed packages.
Afterwards run `$ python manage.py migrate` for migration.

# Import SKOS (from RDF/XML)

After vocabs-app is set up and running, let's import some Skos-Concepts. Therefore browse to [http://127.0.0.1:8000/vocabs/import/](http://127.0.0.1:8000/vocabs/import/) and upload a skos RDF/XML document, (you can find a sample file in vocabs/data/xml), and click **import**. The import may take some time (depending on the size of your file). But you check the status off the import by opening in another window [http://127.0.0.1:8000/vocabs/concepts/browse/](http://127.0.0.1:8000/vocabs/concepts/browse/) where you see all already imported concepts. (The import does not use any transactions!).

# Export SKOS (To RDF/XML)

You can export (parts) of your Concepts by going to [http://127.0.0.1:8000/api/skosconcepts/]. Here you can select the output format (HTML, JSON, SKOS RDF/XML) and define the paging size. So to export all concepts stored in your database, you could do something like: [http://127.0.0.1:8000/api/skosconcepts/?format=xml&page_size=100000](http://127.0.0.1:8000/api/skosconcepts/?format=xml&page_size=100000). Now you can save the output and e.g. upload it to [http://labs.sparna.fr/skos-play/](http://labs.sparna.fr/skos-play/).

# Link vocabs with custom app:

After all this tedious work, we can now use/reference all classes from the vocabs app in our custom app. Therefore let's add a new property to the Talk class:

```python

# lunch/models.py

from django.core.urlresolvers import reverse
from django.utils import timezone
from django.db import models
from vocabs.models import SkosConcept


class Talk(models.Model):
    title = models.CharField(blank=True, max_length=250)
    speaker = models.CharField(blank=True, max_length=250)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    how_was_it = models.ForeignKey(SkosConcept, blank=True, null=True)

    def __str__(self):
        return "'{}', by {}".format(self.title, self.speaker)

    def get_absolute_url(self):
        return reverse('lunch:talk_detail', kwargs={'pk': self.id})

```

Make and run migrations:

`$ manage.py makemigrations`

`$ python manage.py migrate`

This is all it needs for a very basic implementation.

For the next step urls, templates and views for lunch will be needed (work in progress) so that we can browse to [http://127.0.0.1:8000/lunch/talk/create/](http://127.0.0.1:8000/lunch/talk/create/) and select all SkosConcepts stored in the database so far.
