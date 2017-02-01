# Introduction

## Motivation

'django-generic-apps' resulted from the process of avoiding work. Or, to be more precise, django-generic-apps should reduce dumb and repetitive work.
But isn't [django](https://www.djangoproject.com) dedicated to the *DRY* principle (which stands for 'don't repeat yourself'? Well on project basis it definitely is. You create your model, derive views, filters, forms, and more from it, write a view templates, configure some URLs and you are done with a neat CRUD-application as the one described in the official [django-tutorial](https://docs.djangoproject.com/en/1.10/intro/tutorial01/).
But when you are in charge of several projects, you have to do the afore mentioned steps again and again for each project. And then things start to get dumb, boring and repetitive.
This is where 'django-generic-apps' step in.

## general remarks

Though before we go into detail a general remark:
This series of HowTos serves first and foremost as some kind of verbose documentation of the django generic apps and not a precise step-by-step tutorial. So if you want to implement some of those generic-apps in your own project, you will need a basic understanding of django so you can modifiy, tweak an adapt those apps to your specific needs.
This will be necessary since all those apps described below were developed in the contexts of concrete projects and are therfore not (always) as generic as they could be. If you are looking for some really highly generic tool to bootstrap your project, please consider [Cookiecutter Django](https://github.com/pydanny/cookiecutter-django).

# what kind of apps?

In django-speak an app is NOT understood as full fledged web application (this would be a django-project) but more as a module, a library which provides dedicated functionalities or are responsible for certain tasks. And ideally such apps are build in a very self contained and generic way, to improve their reusability.

## webpage

Probably the most useful app amongst 'django-generic-apps' is the app *webpage*.  
