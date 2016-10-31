# Introduction

In this post we will tackle the issue of how to bookmark a selection made by the user using the tablesorter we implemented in the [latest post](../part-2-a-customizable-table-of-content/). (Click [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-2/aratea-digital-0.2.xar) to download the latest code base.)
With such a feature, scholars are enabled to select and reference (bookmark) e.g. a sub corpus of editions, search results or index entities like persons. And since our person-index doesn't work anyway so far, let's start with implementing a bookmarking feature here.

## fixing the person index

Browsing now to [http://localhost:8080/exist/apps/aratea-digital/pages/persons.html](http://localhost:8080/exist/apps/aratea-digital/pages/persons.html) ends in an almost empty page. But this is an easy fix. Because as described in [this post](../part-7-index-based-search/), our person index is generated an according index document stored at `data/indices/listperson.xml`. Since the current person index file is called `data/indices/listPers.xml` there is no data for the responsible xQuery function `app:listPers` located at `modules/app.xql` to work with.
To fix this, we simply have to rename `listPers.xml` into `listperson.xml`. When we now refresh [http://localhost:8080/exist/apps/aratea-digital/pages/persons.html](http://localhost:8080/exist/apps/aratea-digital/pages/persons.html), we should see a list of all indexed persons: 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-3/image_0.jpg)

## implement table sorter

Now let's implement the tablesorter as described in the [last post](../part-2-a-customizable-table-of-content/). After this the file **pages/persons.html** should look like below: 

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.js"/>
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.widgets.js"/>
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.pager.js"/>
    <link rel="stylesheet" type="text/css" href="$app-root/resources/js/tablesorter/css/theme.bootstrap.css"/>
    <link rel="stylesheet" type="text/css" href="$app-root/resources/js/tablesorter/css/jquery.tablesorter.pager.css"/>
    <script> $(function() { $("table").tablesorter({ theme : "bootstrap", widthFixed: false,
        headerTemplate : '{content} {icon}', widgets : [ "uitheme", "filter", "zebra" ], filter_cssFilter: "form-control", }) }); 
    </script>
    <h1>Persons</h1>
    <table>
        <thead>
            <tr>
                <th>persons' name</th>
            </tr>
        </thead>
        <tbody>
            <tr data-template="app:listPers"/>
        </tbody>
    </table>
</div>
```

And of course, we also have to adapt the triggered **modules/app.xql app:listPers** xQuery function:

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection(concat($config:app-root, '/data/editions/'))//tei:TEI
        return
        <tr>
            <td>
                <a href="{app:hrefToDoc($doc)}">{app:getDocName($doc)}</a>
            </td>
        </tr>   
};
```

After these changes, our person index is now sort-, as well as filter- or search-able. 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-3/image_1.jpg)


