# Introduction

In this post we will tackle the issue of how to bookmark a selection made by the user using the tablesorter we implemented in the [latest post](../part-2-a-customizable-table-of-content/). With such a feature, scholars are enabled to select and reference (bookmark) e.g. a sub corpus of editions, search results or index entities like persons. And since our person-index doesn't work anyway so far, let's start with implementing a bookmarking feature here.

## fixing the person index

Browsing now to [http://localhost:8080/exist/apps/aratea-digital/pages/persons.html](http://localhost:8080/exist/apps/aratea-digital/pages/persons.html) ends in an almost empty page. But this is an easy fix. Because as described in [this post](../part-7-index-based-search/), our person index is generated an according index document stored at `data/indices/listperson.xml`. Since the current person index file is called `data/indices/listPers.xml` there is no data for the responsible xQuery function `app:listPers` located at `modules/app.xql` to work with.
To fix this, we simply have to rename `listPers.xml` into `listperson.xml`. When we now refresh [http://localhost:8080/exist/apps/aratea-digital/pages/persons.html](http://localhost:8080/exist/apps/aratea-digital/pages/persons.html), we should see a list of all indexed persons: 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-3/image_0.jpg)

