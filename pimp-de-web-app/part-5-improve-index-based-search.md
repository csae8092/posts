# Introduction

In this post we will improve the look and the feel of our index-based-search functionalities on which we already worked in the [last post](../part-4-add-and-improve-registers/). We will implement good old tablesorter as well as some KWIC preview for the matching results. The code we developed so far can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-4/aratea-digital-0.1.xar).

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

As you might have noticed, we had to implement a new variable `var fetched_searchkey` to keep our bookmarking-feature working.
Accordingly we have to change the value returned by the function `app:registerBasedSearch_hits` stored in **modules/app.xql**:

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

So far the easier part. What's left to do is to provide some content for these columns, because right now e.g. [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=germanicus) looks a bit weired:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_1.jpg)

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
Now we can e.g. start an indexed based search for all documents (editions and manuscript descriptions) referring to the book e.g. "Dell'Era (1979)" [http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=Borst,%20Schriften](http://localhost:8080/exist/apps/aratea-digital/pages/hits.html?searchkey=Borst,%20Schriften). The result will presented like this:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_2.jpg).

The result is not to bad. We see a hitcount per document, we see the context of the searched term and we see the file name of the document containing the term (or in this case the bibliographic item), we searched for.

### Broken bookmark feature

Unfortunately by introducing more columns our bookmark functionality does not work properly any more. We still can filter the rows by typing into the filter fields, and the string we are filtering for will be captured as as `&index=` parameter. Though when we try to reload this URL, the value of this parameter will be pasted by default always into tablesorters first index field. 
We could of course rewrite this bookmark functionality, creating e.g. for each column an individual parameter, but this sounds like some tedious and time consuming work. And we might spent our time better one something more fruitful. So it's much simpler to remove the all the bookmark related javascript code. 

## Displaying the number of all hits.

Something more fruitful could be an overall hit count and some better feedback about the actual search term. We already implement such functionalities for the application's full text search, so we should:

1. copy and paste the whole content of `pages/ft_search.html` to `pages/hits.html`. The only major difference is the called xQuery script. (Yes, I know this calls for refactoring).
2. make sure that `pages/hits.html` is listening to the parameter **searchkey** and not **searchexpr** as in `pages/ft_search.html`.
3. add to `app:app:registerBasedSearch_hits` in **modules/app.xql** a `class='KWIC'` attribute to the `<td>` element containing the context of the searchkey.

With these changes in places, the result page for an index based search for the bibliographic item **Borst, Schriften** will look like depicted below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_3.jpg).


## Cleaning up the nav-bar

Before we call it a day, let's clean up our application's navigation bar a bit by creating a new menu entry called somehting like **Index** and group the links for our person, places, organisations and literature indexes below this. Doing so, our base template **templages/page.html** looks like this:

```html
        ...
            <div class="navbar-collapse collapse" id="navbar-collapse-1">
                <ul class="nav navbar-nav">
                    <li class="dropdown" id="about">
                        <a href="#" class="dropdown-toggle" data-toggle="dropdown">Home</a>
                        <ul class="dropdown-menu">
                            <li>
                                <a href="index.html">Home</a>
                            </li>
                            <li>
                                <a href="toc.html">Table of Content</a>
                            </li>
                            <li>
                                <a href="../data/editions">API</a>
                            </li>
                        </ul>
                    </li>
                    <li class="dropdown">
                        <a href="#" class="dropdown-toggle" data-toggle="dropdown">Indexes</a>
                        <ul class="dropdown-menu">
                            <li>
                                <a href="persons.html">Persons</a>
                            </li>
                            <li>
                                <a href="places.html">Places</a>
                            </li>
                            <li>
                                <a href="organisations.html">Organisations</a>
                            </li>
                            <li>
                                <a href="bibl.html">Literatur</a>
                            </li>
                        </ul>
                    </li>
                </ul>
                
                <div class="pull-right">
                    <form method="get" action="ft_search.html" class="navbar-form" id="pageform">
                        <div class="form-group">
                            <div class="input-group">
                                <input type="text" class="form-control" name="searchexpr" placeholder="search in all inventories" pattern=".{3,}" required="" title="3 characters minimum"/>
                            </div>
                            <button type="submit" class="btn btn-primary">search</button>
                        </div>
                    </form>
                </div>
            </div>
        ...
```

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_4.jpg).

## Keeping things consistent (full text search)

By adapting the index based search as described above, we brought our application in a slightly inconsistent state as far as search functionalities are concerned. Because now, the **all** documents stored in any sub collection of our `data/` data directory will be searched by an index based search. But not so if we run a fulltext search, which only examines documents stored in `data/editions/`. To fix this - in case we want to - and we want to, we basically have to rewrite `app:ft_search` in **modules/app.xql** as well es adapt our full text index settings. Let's start with the latter by changing the existing configuration stored at **/db/system/config/db/apps/aratea-digital/collection.xconf** into: 

```xml
<collection xmlns="http://exist-db.org/collection-config/1.0">
    <index xmlns:tei="http://www.tei-c.org/ns/1.0" xmlns:xs="http://www.w3.org/2001/XMLSchema">
        <fulltext default="none" attributes="false"/>
        <create qname="tei:term" type="xs:string"/>
        <create qname="tei:persName" type="xs:string"/>
        <create qname="tei:placeName" type="xs:string"/>
        <create qname="tei:abbr" type="xs:string"/>
        <lucene>
            <text qname="tei:p"/>
            <text qname="tei:msDesc"/>
        </lucene>
    </index>
</collection>
```

Then we also copy and paste this code snippet into `collection.xconf` stored in our application's root directory since. By doing so, every time we install our application to an eXist-db instance, this index configuration will be copied into `/db/system/config/db/apps/aratea-digital/`. 
After this, go to the dasboard of your eXist-db instance, click on the **Collections** pane and there click on the **Reindex collection** icon. 
Now also (some parts) of the manuscript descrptions stored in `data/descriptions` should be indexed.
In the next step, we will modify the selector of our full text search function called `app:ft_search` which is defined in **modules/app.xql** so it looks like in the snippet below:

```xquery
 for $hit in collection(concat($config:app-root, '/data/'))//*[.//tei:p[ft:query(.,$searchterm)]|.//tei:cell[ft:query(.,$searchterm)] | .//tei:msDesc[ft:query(.,$searchterm)]]
```

When we now search for example for "geometrical", six matches in two documents will be returned:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-5/image_5.jpg).

But the links will lead us to nothing more than blank pages, because like in our former index based search function, we have still have hard coded stylesheet and directory links. So the only thing we have to do, is to copy&paste&adapt some lines of codes from `app:registerBasedSearch_hits` to `app:ft_search` in **modules/app.xql**:

```xquery
 declare function app:ft_search($node as node(), $model as map (*)) {
 if (request:get-parameter("searchexpr", "") !="") then
 let $searchterm as xs:string:= request:get-parameter("searchexpr", "")
 for $hit in collection(concat($config:app-root, '/data/'))//*[.//tei:p[ft:query(.,$searchterm)]|.//tei:cell[ft:query(.,$searchterm)] | .//tei:msDesc[ft:query(.,$searchterm)]]
    let $doc := document-uri(root($hit))
    let $type := tokenize($doc,'/')[(last() - 1)]
    let $params := concat("&amp;directory=", $type, "&amp;stylesheet=", $type)
    let $href := concat(app:hrefToDoc($hit), "&amp;searchexpr=", $searchterm, $params) 
    let $score as xs:float := ft:score($hit)
    order by $score descending
    return
    <tr>
        <td>{$score}</td>
        <td class="KWIC">{kwic:summarize($hit, <config width="40" link="{$href}" />)}</td>
        <td>{app:getDocName($hit)}</td>
    </tr>
 else
    <div>Nothing to search for</div>
 };
```

With this changes in places, we can search through manuscirpt descriptions as well as the edited texts of the manuscript at once.

# Conclusion and Outlook

The result page for an index based search as well as the one for the full text search look and feel now very similar. The users can filter and sort the results according to their needs, and the KWIC-feature as well as the number of hits per document allows the users easier to decided which results of the listed search results might be useful and which not. 
In the [next HowTo](../part-6-text-and-index) we will interlink the actual HTML representations of the editions and manuscript descriptions with the index-files. With the little help of some XSLT and JavaScript, users will be then able to click on a e.g. person's name mentioned (and tagged) in an edition. This click will open some pop-up window, providing the additional information to this entity fetched from the according index entry.

As usual you can download the latest code-base [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-5/aratea-digital-0.1.xar).



