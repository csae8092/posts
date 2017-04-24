# Introduction

## Motivation

'django-generic-apps' resulted from the process of avoiding work. Or, to be more precise, django-generic-apps should reduce dumb and repetitive work.
But isn't [Django](https://www.djangoproject.com) dedicated to the *DRY* principle (which stands for 'don't repeat yourself'? Well on project basis it definitely is. You create your model, derive views, filters, forms, and more from it, write a view templates, configure some URLs and you are done with a neat CRUD-application as the one described in the official [Django tutorial](https://docs.djangoproject.com/en/1.10/intro/tutorial01/).
But when you are in charge of several projects, you have to do the afore mentioned steps again and again for each project. And then things start to get dumb, boring and repetitive.
This is where 'django-generic-apps' step in.

## General Remarks

Though before we go into detail a general remark:
This series of HowTos serves first and foremost as some kind of verbose documentation of the django-generic-apps and not a precise step-by-step tutorial. So if you want to implement some of those generic-apps in your own project, you will need a basic understanding of Django so you can modify, tweak an adapt those apps to your specific needs.
This will be necessary since all those apps described below were developed in the contexts of concrete projects and are therefore not (always) as generic as they could be. If you are looking for some really highly generic tool to bootstrap your project, please consider [Cookiecutter Django](https://github.com/pydanny/cookiecutter-django).

# What Kind of Apps?

In Django an app is NOT understood as full fledged web application (this would be a Django project) but more as a module, a library which provides dedicated functionalities or are responsible for certain tasks. And ideally such apps are build in a very self contained and generic way, to improve their reusability.

## Webpage

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_0.jpg)

Probably the most useful app amongst 'django-generic-apps' is called *webpage*. It basically provides:

* a base template which styles the whole project by defining those elements which appear on all pages of the project (navigation, footer) and includes all basic static content (CSS, JavaScript libraries)
* a generic start page
* a generic imprint page
* a cookie consent pop up
* a social media share button
* and user authentication (log-in log-out)

### Used in

This application is used in the following projects (by the time of writing this post):


[DEFC](https://defc.acdh.oeaw.ac.at/) | [DAACDA](https://daacda.acdh.oeaw.ac.at/) | [PAAS](https://paas.acdh.oeaw.ac.at/)/[APIS](https://apis.acdh.oeaw.ac.at/) | [LaBaSi](https://labasi.acdh.oeaw.ac.at/) | [ToteTiroler](https://totetiroler.acdh.oeaw.ac.at/) | [CBAB](https://cbab.acdh.oeaw.ac.at/) | [ECCE](https://ecce.acdh.oeaw.ac.at/) | [dig-ed-cat](https://dig-ed-cat.acdh.oeaw.ac.at/) | [HowTo](https://howto.acdh.oeaw.ac.at/) | [geodjango/world](https://geodjango.acdh.oeaw.ac.at/)

---

## Places

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_1.jpg)

As one could most likely guess by the app's name, this app can be used in projects which are dealing with 'places' where as a 'place' is an entity which can be located/described by its GPS coordinates. The app provides:

* a model of a place like entity with properties like: *name*, *longitude*, *latitude* or *geonames-id*
* an edit view which automatically tries to fetch possible geonames-id from the geonames-id-api and displays the matches on a map

### Used in

[DEFC](https://defc.acdh.oeaw.ac.at/) | [DAACDA](https://daacda.acdh.oeaw.ac.at/) | [ToteTiroler](https://totetiroler.acdh.oeaw.ac.at/) | [CBAB](https://cbab.acdh.oeaw.ac.at/) | [dig-ed-cat](https://dig-ed-cat.acdh.oeaw.ac.at/)

---

## bib

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_2.jpg)

This application allows to link/sync data from [Zotero](https://www.zotero.org/) libraries with Django projects. Therefore it provides:

* a simple model of a bibliographic item with the properties like *author*, *title*, *year*, ... as well as property with stores the key or ID of this item assigned by Zotero
* a script which allows to import core data from Zotero items into the Django project

### Used in

[DEFC](https://defc.acdh.oeaw.ac.at/) | [CBAB](https://cbab.acdh.oeaw.ac.at/)

---

## vocabs

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_3.jpg)

vocabs - nomen est omen - takes care of the controlled vocabularies in a project. It does so by providing:

* a data model very closed to SKOS
* an API to query the vocabulary and/or retrieve in JSON or as RDF/XML
* an interface to import/upload existing SKOS concepts from RDF/XML

### Used in

 [CBAB](https://cbab.acdh.oeaw.ac.at/) | [ECCE](https://ecce.acdh.oeaw.ac.at/)

---

## Charts

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_4.jpg)

charts is an app which facilitate the usage of [Highcharts: Interactive JavaScript charts for your webpage](http://www.highcharts.com/). It provides

* base templates with all the needed JavaScript code
* views to render those templates
* as well as demo data endpoints to make rebuilding custom made endpoints easier.

### Used in

[ECCE](https://ecce.acdh.oeaw.ac.at/) | [DAACDA](https://daacda.acdh.oeaw.ac.at/) | [ToteTiroler](https://totetiroler.acdh.oeaw.ac.at/) | [dig-ed-cat](https://dig-ed-cat.acdh.oeaw.ac.at/)


#  apps/projects

For the last three items in this list (**world**, **blog**, and **iso639**) the boundaries between app and project or service are not clear to draw any more because either of purpose or complexity of the app/project. "World" for example could be packed very nicely into a single application. But since this application deals with spatial data, it requires a [PostgreSQL](https://www.postgresql.org/) as database and some GIS libraries which are slightly tricky to install. Also, ACDH-OeAW intends to provide this application as a service ([see](https://geodjango.acdh.oeaw.ac.at/)) which ideally makes it superfluous to install this application into other projects. The same is true for [iso639](https://iso639.acdh.oeaw.ac.at/) which is an application/service exposing and API to query and retrieve ISO639 language codes.
Most parts of the *blog* application (which is the core this [HowTo-Blog](https://howto.acdh.oeaw.ac.at/) could be kept self contained and therefore easily reused. But the implemented full text search adds an extra layer of complexity to the projects architecture which would require some thoughtful refactoring to keep this app as generic as possible.

# Conclusion and Outlook

This was a quick tour-de-force through 'django-generic-apps'. In the [following post](../part-2-webpage) we will build a sample Django project which will implement all those apps.
