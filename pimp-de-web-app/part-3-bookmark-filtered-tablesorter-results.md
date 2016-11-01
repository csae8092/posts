# Introduction

In this post we will tackle the issue of how to bookmark a selection made by the user using the tablesorter we implemented in the [latest post](../part-2-a-customizable-table-of-content/). (Click [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-2/aratea-digital-0.1.xar) to download the latest code base.)
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

## bookmark selection with History.js

Now let's imagine a use case where one scholars wants to refer to a subset of the persons mentioned in our Aratea texts, like those captured in the screenshot above. Of course the scholar could write down somewhere the link http://localhost:8080/exist/apps/aratea-digital/pages/persons.html and the letters typed into the filter field. But ideally those letters would be automatically captured by the url in the browser. 
Since this is seems to be quit a common use case, there are already some kind of out of the box solutions for this. In our application we will implement [History.js](https://github.com/browserstate/history.js/).
Create a new directory in `resources/js/historyjs` and add the following file [native.history.js](https://github.com/browserstate/history.js/blob/master/scripts/compressed/history.adapter.native.js). 

Now we have to load this library into our **pages/persons.html** site which is achieved by this line added somewhere towards the top of the document: `<script src="$app-root/resources/js/historyjs/jquery.history.js"/>`.
After this let's reconsider what it is we actually want this library to do. Basically whenever users are typing into the filter field, the result of this event - the filter or search expression - should be stored in the browsers URL field as a parameter like e.g. 'person'. 
All this is done with the following javascript code snippet added towards the bottom of **pages/persons.html**:

```javascript
<script>
    $( document ).ready(function() {
        $("td input").bind("keyup", function(e) {
            var searchstring = $(this).val();
            History.pushState({searchstring: "person"}, "looking for "+searchstring, "?person="+searchstring);   
        });
    });
</script>
```

When we now start typing into the filter field, we can observe that the URL in the browser (as well as the browser's title expression).

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-3/image_2.jpg)

Pretty neat, isn't it? But we are only have way there. Because if we try to copy this dynamically created URL, open a new browser window and open there this URL, we will the whole list of persons. So what we have to do now is somehow ingest the value our "?person=" param into the tablesorter's filter field, when the page loads. 
This is done with the following changes: 

```javascript
<script>
    $( document ).ready(function() {
        var fetched_param = decodeURIComponent($.urlParam("person"));
        if (fetched_param != "null"){
            $.tablesorter.setFilters( $('table'), [ fetched_param], true );
            $('table').trigger('search', true);
            }
        $("td input").bind("keyup", function(e) {
            var searchstring = $(this).val();
            History.pushState({searchstring: "person"}, "looking for "+searchstring, "?person="+searchstring);   
            });
    });
</script>
```

In case you are wondering about this `$.urlParam()` function, we used it the [first time](../part-8-full-text-search/) to create a hitcount for the application's full text search and moved it while doing some [code refactoring](../part-9-code-refactoring/) into a custom javascript file `resources/js/custom.js`. But with this function we fetch the URL-param, save it as `fetched_param`. Then we check the value of this param and if there is some we inject into the tablesorter's setFilters() function. 
With this few lines of code in place, we can now bookmark a URL like https://dse.eos.arz.oeaw.ac.at/exist/apps/thun/pages/persons.html?person=Aar and send it e.g. to our scholarly friends and they will see the same filtered results.