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

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_0.jpg)

Probably the most useful app amongst 'django-generic-apps' is called *webpage*. It basically provides:

* a base template which styles the whole project by defining those elements which appear on all pages of the project (navigation, footer) and includes all basic static content (css, js-libraries);
* a generic start page;
* a generic imprint page;
* cookie consent pop up;
* social media share button;
* and user authentication (log-in log-out).  

### used in

This application is used in the following projects (by the time of writing this post):
[DEFC](https://defc.acdh.oeaw.ac.at/) | [DAACDA](https://daacda.acdh.oeaw.ac.at/) | [PAAS](https://paas.acdh.oeaw.ac.at/) | [LaBaSi](https://labasi.acdh.oeaw.ac.at/) | [ToteTiroler](https://totetiroler.acdh.oeaw.ac.at/) | [CBAB](https://cbab.acdh.oeaw.ac.at/) | [ECCE](https://ecce.acdh.oeaw.ac.at/) | [dig-ed-cat](https://dig-ed-cat.acdh.oeaw.ac.at/)
---

## places

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_1.jpg)

As one could most likely quess by the app's name, this app can be used in projects which are dealing with 'places' where as a 'place' is an entity which can be located/described by its gps-coordinates. The app provides:

* a model of a place like entity with properties like: *name*, *longitude*, *latitude* or *geonames-id*
* an edit view which automatically tries to fetch possible geonames-id from the geonames-id-api and displays the matches on a map

### used in

[DEFC](https://defc.acdh.oeaw.ac.at/) | [DAACDA](https://daacda.acdh.oeaw.ac.at/) | [ToteTiroler](https://totetiroler.acdh.oeaw.ac.at/) | [CBAB](https://cbab.acdh.oeaw.ac.at/) | [dig-ed-cat](https://dig-ed-cat.acdh.oeaw.ac.at/)

---

## bib

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_2.jpg)

This application allows to link/snyc data from [zotero](https://www.zotero.org/) libraries with django-projects. Therefore it provides

* a simple model of a bibliographic item with the properties like *author*, *title*, *year*, ... as well as property with stores the key or ID of this item assigend by zotero;
* a script which allows to import core data from zotero items into the django-project

### used in

[DEFC](https://defc.acdh.oeaw.ac.at/) | [CBAB](https://cbab.acdh.oeaw.ac.at/)

---

## vocabs

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_3.jpg)

vocabs - nomen est omen - takes care of the controlled vocabularies in a project. It does so by providing:

* a data model very closed to SKOS,
* an API to query the vocabulary and/or retrieve in JSON or as RDF/XML
* an intreface to import/upload existing SKOS concepts from RDF/XML

### used in

 [CBAB](https://cbab.acdh.oeaw.ac.at/) | [ECCE](https://ecce.acdh.oeaw.ac.at/)

---

## charts

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/django-generic-apps/images/part-1/image_3.jpg)

charts is an app which facilitate the usage of [Highcharts: Interactive JavaScript charts for your webpage](http://www.highcharts.com/). It provides

* base templates with all the needed java-script code,
* views to render those templates,
* as well as demo data endpoints to make rebuilding custom made endpoints easier.

### used in

[ECCE](https://ecce.acdh.oeaw.ac.at/) | [DAACDA](https://daacda.acdh.oeaw.ac.at/) | [ToteTiroler](https://totetiroler.acdh.oeaw.ac.at/) | [dig-ed-cat](https://dig-ed-cat.acdh.oeaw.ac.at/)
