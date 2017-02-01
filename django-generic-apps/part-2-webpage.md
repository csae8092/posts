# Implement webpage app into a new project

# Create a new django-project

To get things started we are going to create a new django-project in which we will implement all the generic-apps mentioned in the [last post]((../part-1-gettings-started.md)). You can either follow this and the upcoming posts step-by-step or you can clone this GitHub Repo [https://github.com/acdh-oeaw/django-generic-apps](https://github.com/acdh-oeaw/django-generic-apps) and check out the matching tags. This is up to you.

start with https://github.com/acdh-oeaw/django-generic-apps/commit/337adfd12fb9358ab7977eba0a58cdf3c0468630

## Virtual environment

But before we are getting started for real, let's create a virtual environment for Python 3.4. and install django 1.9.x.

`$ conda create --name rlunch python=3.4`

`$ activate rlunch`

`$ pip install Django==1.9.5`

(As you can see, I am using [anaconda(https://www.continuum.io)])

Then I will change into the `/django-generic-apps` directory I got by cloning the repo mentioned above and create my sample project called **rlunch** (which stands for Research-Lunch).

`$ django-admin startproject rlunch`

This command created the new directory `/django-generic-apps/rlunch` which is the root directory of our django-project. Change into this directory and try if everything worked so far, by typing the following commands:

`$ cd rlunch`

`$ python manage.py runserver`

Now you should be able to browse to [http://127.0.0.1:8000/](http://127.0.0.1:8000/) and see something like depicted below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-2/image_0.jpg)

Stop the developement server (ctrl+c), and migrate the current database scheme and then create a superuser

`$ python manage.py migrate`

`$ python manage.py createsuperuser`
