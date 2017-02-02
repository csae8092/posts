# Create a custom django-app

In this project we will create a custom django-app in our rlunch-project ([see last post](../part-2-webpage)) which will serve as integration point for all django-generic-apps briefly described in the [first post](../part-1-gettings-started).

## create the app and define a model

To create the custom app, we will call it 'lunch', change into the project's root directory, and run the well known command:

`$ python manage.py startapp lunch`

This command created now a new directory 'lunch' in the project's root directory containing all basic files needed for a django-app.

As usual we will start by defining our 'lunch' models.py file:


```python
# lunch/mnodels.py

from django.db import models
from django.utils import timezone


class Talk(models.Model):
    title = models.CharField(blank=True, max_length=250)
    speaker = models.CharField(blank=True, max_length=250)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

    def __str__(self):
        return "'{}', by {}".format(self.title, self.speaker)

```

As you can see, this is a very barebone model so far and also not very sophisticated taking into account that e.g. the "speaker" is modeled as simple CharField and not as a ForeignKey (or ManyToMany in case of more speakers) to a own Speaker-Class.

After we defined the model, we have to run to register this app in the project's settings file,

```python

# rlunch/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'crispy_forms',
    'webpage',
    'lunch',
]
```

register the Talk class in lunch's admin.py file,

```python

# lunch/admin.py

from django.contrib import admin
from .models import Talk

admin.site.register(Talk)

```

and then make migrations and run migrations:

`$ python manage.py makemigrations`

`$ python manage.py migrate`

When we now start the server and browse to [http://127.0.0.1:8000/admin/lunch/talk/](http://127.0.0.1:8000/admin/lunch/talk/), we can create, edit, and delete Talk-objects.

# Conclusion and outlook

With this few lines of code we created our custom django-app, which we will be fleshed out in the next posts with the help of django-generic-apps, starting in the [next post](../part-4-vocabs-app) with implementing the vocabs-app.
For the next posts, I will add frontend CRUD-Views for the Talk-Class. But since this is basic django-stuff, I won't describe how I did it, especially as you can check out the code anyhow.
