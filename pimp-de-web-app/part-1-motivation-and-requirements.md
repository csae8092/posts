# Introduction and requirements

## Introduction

In the book [How to build a digital edition web app](../books/how-to-build-a-digital-edition-web-app/) we developed a very generic digital edition web application. 'Very generic' means that the basic functionalities we implemented, like 

* a navigable table of content, 
* an on the fly transformation of XML documents to HTML views, 
* a full text search,
* a simple base template,
* some useful js-libraries

should work with any XML/TEI encoded documents stored in the application's `/data/editions` collection 'out of the box'.

We also implement some rudimentary person index allowing to search the corpus by special tagged entities. Since our implementation of such indexes are based upon it's own index documents, which also have to be stored in their according places (`/data/indices`) this feature is therefore only accessible by digital edition projects which have structured their documents in a similar way. 

## Requirements

In the upcoming posts, we will improve our already existing features. 

1. We will enhance the usability of our application by making our table of contents (for all documents as well as for our registers/indices) **filter- and sortable** by the users. 

2. We will add a **KWIC** snippets to the results of our index based view.

3. We will implement **pop ups containing further information for entities like persons and places** tagged in the edition.

4. We will add a **search-able geovisualization** of places mentioned in our editions. 

## Disclaimer
Those features mentioned above are depending on specif mark up of the XML/TEI documents of our editions as well as some extra information (like gps-coordinates for places). To implement those features into your digital edition web application you either have to adapt your data or to modify the application's code to make it fit your data. 