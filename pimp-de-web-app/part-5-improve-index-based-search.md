# Introduction

In this post we will improve the look and the feel of our index-based-search functionalities on which we already worked in the [last post](../part-4-add-and-improve-registers/). We will implement good old tablesorter as well as some KWIC preview for matching results. The code we developed so far can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-4/aratea-digital-0.1.xar).

## tablesorter 
As already described in the [second part](../part-2-a-customizable-table-of-content/), we will implement tablesorter also for **pages/hits.html** which then looks like this:

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
    <h1>Hits</h1>
    <table>
        <thead>
            <tr>
                <th>matching documents</th>
            </tr>
        </thead>
        <tbody>
            <tr data-template="app:registerBasedSearch_hits"/>
        </tbody>
    </table>
    <script>
        $( document ).ready(function() {
        var fetched_param = decodeURIComponent($.urlParam("index"));
        var fetched_searchkey = decodeURIComponent($.urlParam("searchkey"));
        if (fetched_param != "null"){
        $.tablesorter.setFilters( $('table'), [ fetched_param], true );
        $('table').trigger('search', true);
        }
        $("td input").bind("keyup", function(e) {
        var searchstring = $(this).val();
        History.pushState({searchstring: "index-term"}, "looking for "+searchstring, "?searchkey="+fetched_searchkey+"&amp;index="+searchstring);   
        });
        });
    </script>
</div>
```

As you might have noticed, we had to implement a new variable `var fetched_searchkey' to keep our bookmarking-feature working.
Accordingly we have to change return value of `app:registerBasedSearch_hits` stored in **modules/app.xql**:

```xquery
declare function app:registerBasedSearch_hits($node as node(), $model as map(*), $searchkey as xs:string?, $path as xs:string?)
{
for $hit in collection(concat($config:app-root, '/data/'))//tei:TEI[.//*[@key=$searchkey] | .//*[@ref=concat("#",$searchkey)] | .//tei:abbr[text()=$searchkey]]
    let $doc := document-uri(root($hit))
    let $type := tokenize($doc,'/')[(last() - 1)]
    let $params := concat("&amp;directory=", $type, "&amp;stylesheet=", $type)
    return
    <tr>
        <td>
            <a href="{concat(app:hrefToDoc($hit),$params)}">{app:getDocName($hit)}</a>
        </td>
    </tr> 
 };
```
Now a link like e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus&index=a](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus&index=a) results such a page:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_0.jpg).

## KWIC
Ideally our results table could be expanded by at least two more colums. One column would of course contain the KWIC of the searched index term whereas the other column would count the number of matches per document. For this we have to adapt the table structure in **pages/hits.html**:

```html
...
    <table>
        <thead>
            <tr>
                <th>Hits</th>
                <th>KWIC</th>
                <th>matching documents</th>
            </tr>
        </thead>
        <tbody>
            <tr data-template="app:registerBasedSearch_hits"/>
        </tbody>
    </table>
...
```
So far the easier part. What's left to do is to provide some content for these columns, because right now e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus] looks a bit weired:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_1.jpg).

As we all know, the 'content' for those rows is provided by `app:registerBasedSearch_hits` stored in **modules/app.xql** which we have to modify like this:

```xquery
declare function app:registerBasedSearch_hits($node as node(), $model as map(*), $searchkey as xs:string?, $path as xs:string?)
{
for $title in collection(concat($config:app-root, '/data/'))//tei:TEI[.//*[@key=$searchkey] | .//*[@ref=concat("#",$searchkey)] | .//tei:abbr[text()=$searchkey]]
    let $doc := document-uri(root($title))
    let $type := tokenize($doc,'/')[(last() - 1)]
    let $params := concat("&amp;directory=", $type, "&amp;stylesheet=", $type)
    let $matchingdoc := root($title)//tei:abbr[text()=$searchkey] | root($title)//*[@key=$searchkey] |  root($title)//*[@ref=concat("#",$searchkey)]
    let $hits := count($matchingdoc)
    let $snippet := 
        for $context in $matchingdoc
        let $before := $context/preceding::text()[1]
        let $after := $context/following::text()[1]
        return
            <p>... {$before} <strong><a href="{concat(app:hrefToDoc($title), $params)}"> {$context/text()}</a></strong> {$after}...<br/></p>
    order by -$hits
    return
    <tr>
        <td>{$hits}</td>
        <td>{$snippet}</td>
        <td>
            <a href="{concat(app:hrefToDoc($title),$params)}">{app:getDocName($title)}</a>
        </td>
    </tr> 
 };
```
Now we can e.g. start an indexed based search for all documents (editions and manuscript descriptions) referring to the book e.g. "Dell'Era (1979)" [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=Dell%27Era%20(1979)](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=Dell%27Era%20(1979). The result will presented like this:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_2.jpg).

## Displaying the number of all hits.




As usual you can download the latest code-base [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-5/aratea-digital-0.1.xar).



