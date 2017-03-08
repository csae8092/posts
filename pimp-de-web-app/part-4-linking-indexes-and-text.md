# Introduction

In this post we will have a closer look to our indexes in general, add some more indexes (or registers for places, literature and organisations) and rework the code responsible for finding documents containing an searched index. 
The code we wrote so far can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-3/aratea-digital-0.1.xar).

## adding more indexes

By looking into `data/indices` of the current project, we see that this collection does not only contain an index document for persons, but also the documents: 

* listBibl.xml - bibliography of title references in the editions and manuscript descriptions;
* listOrg.xml - a list of organisations mentioned in the manuscript descriptions, assigned with links to norm data records;
* listPlace.xml - you might have guessed it, a list of (normalized and norm data referenced) places mentioned in editions and descriptions;
* listWork.xml - A list of 'works' or historical texts. Unlike the lists mentioned before, this listWork.xml is NOT encoded in TEI but is following some custom schema (although implicitly borrowing tags from the TEI). 

The first task to fulfill in this post is now to make this indexes visible (and usable) in our web application. Since the functions of those documents as well as their structures are very similar to the already known `personlist.xml`, we can accomplish this task by a) copy and paste the existing lines of XQuery and HTML code an modify them slightly, or b) by rewriting/refactoring these lines to make them more generic and therefore usable for all index documents.
We will do something of both, meaning first we will rewrite the existing function a little bit, so we don't have to apply too many changes after copying and pasting. But before we do any of that, we will rename those index files following the naming convention already used for `personlist.xml` which can be expressed like this: `list{entity}.xml` whereas the entity should preferably match a TEI tag (e.g. `<bibl>` or `<title>`) or any used attribute value (e.g `<rs type="place"/>`, `<rs type="person">` or `<name type="org"/>`). This will makes things slightly more comfortable for us a little bit later.
So listPlace.xml turns into listplace.xml, listWork.xml becomes listtitle.xml, listBibl.xml is listbibl.xml and listOrg.xml is going to be listorg.xml.

### a place index (and some more)

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
```

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

Of course, the actual links return no matching documents. But before we take care of this, let's add similar pages for listbibl.xml and listorg.xml. Since this works more or less the same way, I won't add any further explanations. In case you run into some troubles, you can check out the final code-base of this post.

## Finding documents containing entities

As mentioned following the link for e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/persons.html?person=germa](http://localhost:8080/exist/apps/aratea-digital/pages/persons.html?person=germa) leads to [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus) which shows a very blank page.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-4/image_1.jpg)

Why?
Let's start the investigation with the code on the requested HTML page `pages/hits.html`. 

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <h1>Hits</h1>
    <div data-template="app:listPers_hits"/>
</div>
```

As we see, the function `app:listPers_hits` is called. A function which looks for matches in the application's  `data/editions/` collection. Also it looks for a handful specific combinations of TEI-elements and attributes. Both things are not really covering our current application's needs, since we also want the manuscript descriptions searched, which are stored in `data/descriptions/` as well as e.g. persons are tagged in the descriptions as e.g. `<authors>`. 
We should therefore rewrite our search function as follows:

```xquery
for $hit in collection(concat($config:app-root, '/data/'))//tei:TEI[.//*[@key=$searchkey] | .//@ref=concat("#",$searchkey)]
    let $doc := document-uri(root($hit)) 
    return
    <li>
        <a href="{app:hrefToDoc($hit)}">{app:getDocName($hit)}</a>
    </li> 
 };
```
With this changes in places, we can refresh [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus) and will receive the following matches:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-4/image_2.jpg)

But when we now try to follow these links get to the actual documents containing "Germanicus", we will only see useful results for AL_Basle.xml and AL_Paris_7886.xml. All the other links return a page like shown below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-4/image_3.jpg)

This has to be considered as feature and not as bug, because the application or, to me more precise the function `app:XMLtoHTML` in **modules/app.xql**: does exactly what we told it to do. It is looking for a document called **Leiden__VLQ__79.xml** in the collection `data/editions/` and transform this - none existing one -  with the default stylsheet `resources/xslt/xmlToHtml.xsl`.

Lucky for us the we parametrized this function already ([see Part 9 - Code refactoring](../part-9-code-refactoring/)). What is left to do now is to find out, if a document is either a description or an edition and based on this, add the according `?directory=` parameter.
Something we can achieve be rewriting `app:listPers_hits` in **modules/app.xql** into:

```xquery
declare function app:listPers_hits($node as node(), $model as map(*), $searchkey as xs:string?, $path as xs:string?)
{
for $hit in collection(concat($config:app-root, '/data/'))//tei:TEI[.//*[@key=$searchkey] | .//@ref=concat("#",$searchkey)]
    let $doc := document-uri(root($hit))
    let $directory := concat("&amp;directory=", tokenize($doc,'/')[(last() - 1)])
    return
    <li>
        <a href="{concat(app:hrefToDoc($hit),$directory)}">{app:getDocName($hit)}</a>  
    </li> 
 };
```
After saving our modifications, we can click on e.g. **Leiden_VLQ_79.xml** and should be directed from  [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus) to the [detail view of  manuscript description](http://localhost:8080/exist/apps/aratea-digital/pages/show.html?document=Leiden_VLQ_79.xml&directory=descriptions). 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-4/image_4.jpg)

This HTML representation looks a bit messy - point taken - but this is simply a matter of the used stylesheet. But since we are currently working on `app:listPers_hits` in **modules/app.xql** anyway, let's add another stylesheet slightly more specifically for manuscript descriptions. You can download this stylesheet from [here](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/downloads/part-4/descriptions.xsl).

Then we can rewrite `app:listPers_hits` in **modules/app.xql** to add an `&stylesheet=` parameter:

```xquery
declare function app:listPers_hits($node as node(), $model as map(*), $searchkey as xs:string?, $path as xs:string?)
{
for $hit in collection(concat($config:app-root, '/data/'))//tei:TEI[.//*[@key=$searchkey] | .//@ref=concat("#",$searchkey)]
    let $doc := document-uri(root($hit))
    let $type := tokenize($doc,'/')[(last() - 1)]
    let $params := concat("&amp;directory=", $type, "&amp;stylesheet=", $type)
    return
    <li>
        <a href="{concat(app:hrefToDoc($hit),$params)}">{app:getDocName($hit)}</a>  
    </li> 
 };
```

Then we rename our default stylesheet from `resources/xslt/xmlToHtml.xsl` into `resources/xslt/editions.xsl` and finally, change `app:XMLtoHTML` in **modules/app.xql** accordingly by changing the default value for `$xslPath` from "xmlToHtml" into "editions": `let $xslPath := concat(xs:string(request:get-parameter("stylesheet", "editions")), '.xsl')`. 

## and the other registers?

With these changes in place, our person index works now as expected. But what about the other indexes? Let's try e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=tegernsee_abbey](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=tegernsee_abbey), which seems to work as it returns a matching manuscript description:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-4/image_5.jpg)

Same is true for e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=strasbourg](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=strasbourg).

What does not work out of box is a search for dedicated bibliographic items. But this should not come as a surprise to us as referencing literature in editions and manuscript descriptions work sligthly different then referencing persons, places and organisations. To fix this, we have to add another condition to `app:listPers_hits` which should then look as the snippet below:

```xquery
declare function app:listPers_hits($node as node(), $model as map(*), $searchkey as xs:string?, $path as xs:string?)
{
for $hit in collection(concat($config:app-root, '/data/'))//tei:TEI[.//*[@key=$searchkey] | .//*[@ref=concat("#",$searchkey)] | .//tei:abbr[text()=$searchkey]]
    let $doc := document-uri(root($hit))
    let $type := tokenize($doc,'/')[(last() - 1)]
    let $params := concat("&amp;directory=", $type, "&amp;stylesheet=", $type)
    return
    <li>
        <a href="{concat(app:hrefToDoc($hit),$params)}">{app:getDocName($hit)}</a>  
    </li> 
 };
```

With this in change in, also documents matching the searched bibliographic item will be returned by e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=Sternbilder%20des%20Mittelalters](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=Sternbilder%20des%20Mittelalters).

What is left to do now is to rename this function since it does not only return hits for persons, but for other indexed entities as well. So I will rename it into `app:registerBasedSearch_hits`. Accordingly I have to apply this change to **pages/hits.html** as well, which looks like this:

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <h1>Hits</h1>
    <div data-template="app:registerBasedSearch_hits"/>
</div>
```

## Attention

For most queries the recently implemented functionality works as excepted. But conducting an index based search for the place [Prüfening](http://localhost:8080/exist/apps/aratea-digital/pages/places.html?place=Pr%C3%BC) leads to one match in listperson.xml: [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=pruefening](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=pruefening) because this file contains the following entry:

```xml
<person xml:id="wolfger">
    <persName xml:id="gnd_100965148" ref="http://d-nb.info/gnd/100965148">Wolfger von Prüfening</persName>
    <persName>Wolfgerus &lt;Pruveningensis&gt;</persName>
    <floruit>ca.1100-ca.1173</floruit>
    <affiliation>
        <rs type="place" ref="#pruefening">Prüfening</rs>
    </affiliation>
    <occupation>monk</occupation>
    <occupation>librarian</occupation>
</person>
```

Following then the link to this document (listperson.xml) throws an error because the script is looking for a stylesheet called `indices.xsl`, which does not exist (yet). 

# Conclusion and Outlook

With this post we implement a framework/workflow/functionality which allows us to add/customize our register- or index-documents quit easily and without too much copy&pasting. 
Now that we have set up the core functionalities we should improve the look and the usability of **pages/hits.html**. But this will be the topic of the [next post](../part-5-improve-index-based-search/). 
The recent code-base is available for download [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-4/aratea-digital-0.1.xar).