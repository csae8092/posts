# Introduction

In this post we will have a closer look to our indexes in general, add some more indexes (or registers for places, literature and works) and think about how to connect the information stored in this indexes with the actual editions and manuscript descriptions.
The code we wrote so far can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-3/aratea-digital-0.1.xar).

## adding more indexes

By looking into `data/indices` of the current project, we see that this collection does not only contain an index document for persons, but also the documents: 

* listBibl.xml - bibliography of title references in the editions and manuscript descriptions
* listOrg.xml - a list of organisations mentioned in the manuscript descriptions, assigned with links to norm data records.
* listPlace.xml - you might have guessed it, a list of (normalized and norm data referenced) Places mentioned in editions and descriptions
* listWork.xml - A list of 'works', historical texts. Unlike the lists mentioned before, this listWork.xml is NOT encoded in TEI but is following some custom schema (although implicitly borrowing tags from the TEI). 

The first task to fulfill in this post is now to make this indexes visible (and usable) in our web application. Since the functions of those documents as well as their structures are very similar to the already known `personlist.xml`, we can accomplish this task by a) copy and paste the existing lines of xQuery and HTML code an modify them slightly, or b) by rewriting/refactoring these lines to make them more generic and therefore usable for all index documents.
We will do something of both, meaning first we will rewrite the existing function a little bit, so we don't have to apply too many changes after copying and pasting. But before we do any of that, we will rename those index files following the naming convention already used for `personlist.xml` which can be expressed like this: `list{entity}.xml` whereas the entity should preferably match a TEI tag (e.g. `<bibl>` or `<title>`) or any used attribute value (e.g `<rs type="place"/>`, `<rs type="person">` or `<name type="org"/>`). This will makes things slightly more comfortable for us a little bit later.
So listPlace.xml turns into listplace.xml, listWork.xml becomes listtitle.xml, listBibl.xml is listbibl.xml and listOrg.xml is going to be listorg.xml.

### a place index

To add a place index we first copy paste and modifiy (replace all traces of person with place) the existing `app:listPers` function located at **modules/app.xql** to:

```xquery
declare function app:listPlace($node as node(), $model as map(*)) {
    let $hitHtml := "hits.html?searchkey="
    for $place in doc(concat($config:app-root, '/data/indices/listplace.xml'))//tei:listPlace/tei:place
        return
        <tr>
            <td>
                <a href="{concat($hitHtml,data($place/@xml:id))}">{$place/tei:placeName}</a>
            </td>
        </tr>
};

Then create a new document `pages/places.html` and, copy&paste the code from `pages/persons.html` and again, replace any person related passages with the according place-strings.

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.js"/>
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.widgets.js"/>
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.pager.js"/>
    <link rel="stylesheet" type="text/css" href="$app-root/resources/js/tablesorter/css/theme.bootstrap.css"/>
    <link rel="stylesheet" type="text/css" href="$app-root/resources/js/tablesorter/css/jquery.tablesorter.pager.css"/>
    <script src="$app-root/resources/js/historyjs/native.history.js"/>
    <script> $(function() { $("table").tablesorter({ theme : "bootstrap", widthFixed: false,
        headerTemplate : '{content} {icon}', widgets : [ "uitheme", "filter", "zebra" ], filter_cssFilter: "form-control", }) }); 
    </script>
    <h1>Places</h1>
    <table>
        <thead>
            <tr>
                <th>places' name</th>
            </tr>
        </thead>
        <tbody>
            <tr data-template="app:listPlace"/>
        </tbody>
    </table>
    <script>
        $( document ).ready(function() {
        var fetched_param = decodeURIComponent($.urlParam("place"));
        if (fetched_param != "null"){
        $.tablesorter.setFilters( $('table'), [ fetched_param], true );
        $('table').trigger('search', true);
        }
        $("td input").bind("keyup", function(e) {
        var searchstring = $(this).val();
        History.pushState({searchstring: "place"}, "looking for "+searchstring, "?place="+searchstring);   
        });
        });
    </script>
</div>
```

Finally add a link to this new page into `templates/page.html`: 

```html
...
    <li>
        <a href="places.html">Places</a>
    </li>
...
```

Now we can browse to e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/places.html?place=A](http://localhost:8080/exist/apps/aratea-digital/pages/places.html?place=A) and see all Places containing the letter A.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-4/image_0.jpg)

Of course, the actual links return no matching documents. But before we take care of this, let's add similar pages for listbibl.xml and listorg.xml. Since this works more or less the same way, I won't add any further explanations. In case you run into some troubles, you can check out final code-base of this post.