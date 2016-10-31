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

## Code Base

For this book we will start with the code developed in [Part 10](../part-10-nicer-start-page) of the previous book [How to build a digital edition web app](../books/how-to-build-a-digital-edition-web-app/), enhanced with the `sync.xql` script added in [Part 11](../part-11-start-a-new-project/). I also changed the existing **thun-demo/collection.xconf`** document into 

```xml
<collection xmlns="http://exist-db.org/collection-config/1.0">
    <index xmlns:tei="http://www.tei-c.org/ns/1.0" xmlns:xs="http://www.w3.org/2001/XMLSchema">
        <fulltext default="none" attributes="false"/>
        <create qname="tei:term" type="xs:string"/>
        <create qname="tei:persName" type="xs:string"/>
        <create qname="tei:placeName" type="xs:string"/>
        <lucene>
            <text qname="tei:p"/>
        </lucene>
    </index>
</collection>
```

Because now, whenever you will install this application, it will automatically index it's data.

You can download the needed `xar.package` [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-1/thun-demo-0.1.xar).

## Conclusion and Outlook

Having all this done, with can start in the next [HowTo](../) with creating a **customizable Table of Content**. But since there is another project besides the Thun-Project waiting to be finished, we will start working with another data set. But I wont spoil, so if you are curious, what we are going to build, what kind of edition we are going to build, go on and [reading](../).