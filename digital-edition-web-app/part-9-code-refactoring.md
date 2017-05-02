# Introduction and requirements

In the [last part](../part-8-full-text-search) we implemented a KWIC full text search which actually works. And as always, you can download the current code of the application [here](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-8/thun-demo-0.1.xar). Now the only missing thing is a functional link from the KWIC view to the HTML representation of those documents which contain the search string. By the time it should be quite obvious how to generate such a link. All we have to do is to fetch the document’s name and add it as a parameter to [http://localhost:8080/exist/apps/thun-demo/pages/show.html?document={nameOfTheDocument}](http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=czernin-an-thun_o.D._A3-XXI-D80.xml) as we are doing in our [table of content](http://localhost:8080/exist/apps/thun-demo/pages/toc.html) or [index based search](http://localhost:8080/exist/apps/thun-demo/pages/persons.html). 

In both cases we first extracted the documents name with some string juggling and the XQuery functions **document-root()** and **root()** and used this to crate a `<a href="">` HTML element.

# New functions

## app:getDocName

Let’s start with writing a function which will extract the name (or URI, or the location where this document is stored in the database) of the document of an XML node passed to this function. As mentioned above we did this already several times, so it shouldn’t be too hard to encapsulate this procedure in a small function like shown below:

```xquery
declare function app:getDocName($node as node()){
    let $name := functx:substring-after-last(document-uri(root($node)), '/')
    return $name
};
```

Now we can pass an XML node of any XML document stored in our database to this function it and it will return the documents name like e.g. 

*aufsatz-von-langenau-ueber-einfluss-von-windischgraetz-1848_1849-12_A3-XXI-D23.xml*.

## app:hrefToDoc

To create a link which actually works and shows the HTML representation of of a chosen XML/TEI document, we can use app:getDocName() in another custom made function we will call app:hrefToDoc:

```xquery
declare function app:hrefToDoc($node as node()){
    let $name := functx:substring-after-last($node, '/')
    let $href := concat('show.html','?document=', app:getDocName($node))
        return $href
};
```

This function now returns a string like e.g.  *show.html?document=aufsatz-von-langenau-ueber-einfluss-von-windischgraetz-1848_1849-12_A3-XXI-D23.xml* if we pass in any xml node from this document.

Now we have everything we need to create dynamically the correct links leading from our full text search result view to the HTML representations of the matching XML/TEI documents. The only thing left to do is to adapt our **app:ft_search()** function written in the tutorial like demonstrated below:

```
 declare function app:ft_search($node as node(), $model as map (*)) {
    if (request:get-parameter("searchexpr", "") !="") then
    let $searchterm as xs:string:= request:get-parameter("searchexpr", "")
    for $hit in collection(concat($config:app-root, '/data/editions/'))//tei:p[ft:query(.,$searchterm)]
       let $href := app:hrefToDoc($hit)
       let $score as xs:float := ft:score($hit)
       order by $score descending
       return
       <tr>
           <td>{$score}</td>
           <td>{kwic:summarize($hit, <config width="40" link="{$href}" />)}</td>
           <td>{app:getDocName($hit)}</td>
       </tr>
    else
       <div>Nothing to search for</div>
 };
```

Finally we can browse to [http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html](http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html) search for e.g. "leben" (living), 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-9/image_0.jpg)

and clicking on "Leben" to get to the actual document. 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-9/image_1.jpg)

## Highlight search results in actual document

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-9/image_2.jpg)

Ideally the occurrences of 'leben' would be highlighted, so users can find the locations of the expression they searched for quickly. Of course they could always use the in build browser search by pressing `ctrl + f` but not everyone is aware of this functionality. 
Such a highlighting could be handled - once more - either by the server or by the client (browser). And once more we chose the second option.
Basically the browser has to accomplish the following tasks:
 
* fetch the search string;
* search in the HTML (or to be more precise in the DOM, the [Document Object Model](linkToDOM)) for all occurrences of this search string;
* and wrap e.g. a `<span style="background-color: orange;"/>` element around.

All this can be done with the help of a little bit of javaScript/jQuery scripting in **pages/show.html**. But before we can get started working on this page, we have to somehow inform **pages/show.html** about the expression we are actually searching for. This can be accomplished quit easily by passing this information as an URL parameter to **pages/show.html**. Therefore we have to modify the value of `$href` variable in the function **modules/app.xql app:app:ft_search** from `let $href := app:hrefToDoc($hit)` to `let $href := concat(app:hrefToDoc($hit), "&amp;searchexpr=", $searchterm)`. 
When we are now looking for 'leben' and follow now the links listed in the result page, we see that the links to the detail views are now 'storing' the search string as value of the URL param 'searchexpr': http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=aufsatz-von-langenau-ueber-einfluss-von-windischgraetz-1848_1849-12_A3-XXI-D23.xml&searchexpr=leben.

The next step should be familiar from the [last](../part-8-full-text-search) tutorial. With a little function we are parsing the URL parameters and store the results in a variable (`fetched_url_param`). Then we go through all child nodes of an element with the id 'transcribed_text' (which according to the stylesheet we are using to transform the XML/TEI into HTML holds the transcribed text), search for the value stored in `fetched_url_param`) and wrap a `<span/>` element around. All this is accomplished by the following code snippet added to **pages/show.html**

```javascript
<script>
  $( document ).ready(function() {
    /*http://stackoverflow.com/questions/16090487/find-a-string-of-text-in-an-element-and-wrap-some-span-tags-round-it*/
    $.urlParam = function(name){
      var results = new RegExp('[\?&amp;]' + name + '=([^&amp;]*)').exec(window.location.href);
      if (results == null ){
        return null;
      }
      else{
        return results[1] || 0;
      }
    };
    var fetched_url_param = decodeURIComponent($.urlParam('searchexpr'));
      console.log(fetched_url_param);
      $('#transcribed_text').html(function(_, html) {
        var re = new RegExp(fetched_url_param, "g");
        return  html.replace(re, '<span style="background-color: orange;">'+fetched_url_param+'</span>')
      });
  });
</script>
```
Now, if users are looking for 'leben' again all occurrences of this search expression should be highlighted with orange.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-9/image_3.jpg)

### not perfect yet

The solution for highlighting search results presented above is not perfect yet. 

* First it will search and replace all strings found in every child node of an element with `id='transcribed_text'. This includes e.g. attributes and their values, so be aware of possible unwanted side effects.
* Unlike the actual full text search, the highlighting function is case sensitive. This means you can search for 'leben' or 'Leben' and it will list the same results. But if the URL-param passed to the show.html is 'leben' any string in the HTML containing 'Leben' will NOT be highlighted.
* Finally you also have be aware that we are defining exactly the same function for parsing URL params two times in our code base.

At least the last issue should be resolved immediately, especially when we remind ourself that this chapter is called "Code refactoring". Luckily this is an easy fix. We simply have to create a new document **resources/js/custom.js** and add the following lines of code:

```javascript
$.urlParam = function(name){
    var results = new RegExp('[\?&amp;]' + name + '=([^&amp;]*)').exec(window.location.href);
    if (results == null ){
        return null;
    }
    else{
        return results[1] || 0;
    }
};
```

Then we import this `custom.js` in our base template **templates/pages.html** just before the closing tag of the `<head>`element:

```html
...
  <script type="text/javascript" src="$app-root/resources/js/custom.js"/>
</head>
<body id="body">
...
```

After this, we can go to **pages/show.html** and **ft_search.html** and remove the `$.urlParam` function definition.

# All or nothing

Since we now wrote functions to avoid too much copy pasting, we should now go through our code and replace lines of code which are doing the same as our functions against those functions. Yes, this sounds like some boring task and yes, our application will continue working without doing this. But keeping the code in a consistent state, meaning that same things are always build the same way, will ease the burden of maintaining the application, no matter if you or someone else will be in charge. 

## app:listPers_hits

```xquery
declare function app:listPers_hits($node as node(), $model as map(*), $searchkey as xs:string?, $path as xs:string?)
{
    for $hit in collection(concat($config:app-root, '/data/editions/'))//tei:TEI[.//tei:persName[@key=$searchkey] |.//tei:rs[@ref=concat("#",$searchkey)] |.//tei:rs[@key=contains(./@key,$searchkey)]]
    let $doc := document-uri(root($hit)) 
    return
    <li>
        <a href="{app:hrefToDoc($hit)}">{app:getDocName($hit)}</a>
    </li> 
 };
```

## app:toc

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection(concat($config:app-root, '/data/editions/'))//tei:TEI
        return
        <li>
            <a href="{app:hrefToDoc($doc)}">{app:getDocName($doc)}</a>
        </li>   
};
```

# make app:XMLtoHTML fit for the future

As we are working on **moduels/app.xql** anyway, we can also attack another issue. Currently the function app:XMLtoHTML is rather static, meaning that the not the stylesheet we want to apply on our XML document is hard coded but also the directory in where our XML document is supposed to be. Albeit this function suffices for our needs so far, it could be handy if we were able to dynamically tell the function the name of the directory where to look for the XML documents as well as which stylesheet should be used. Luckily these features are easy to implemented. We simply have to declare additional variables for the directory and the name of the stylesheet. Those variables retrieve their values from URL parameters. And since we don't want to change any other of our already written code, we add some default values to those variables. The rewritten **app:XMLtoHTML** function expresses this in valid XQuery:

```xquery
declare function app:XMLtoHTML ($node as node(), $model as map (*), $query as xs:string?) {
let $ref := xs:string(request:get-parameter("document", ""))
let $xmlPath := concat(xs:string(request:get-parameter("directory", "editions")), '/')
let $xml := doc(replace(concat($config:app-root,'/data/', $xmlPath, $ref), '/exist/', '/db/'))
let $xslPath := concat(xs:string(request:get-parameter("stylesheet", "xmlToHtml")), '.xsl')
let $xsl := doc(replace(concat($config:app-root,'/resources/xslt/', $xslPath), '/exist/', '/db/'))
let $params := 
<parameters>
   {for $p in request:get-parameter-names()
    let $val := request:get-parameter($p,())
    where  not($p = ("document","directory","stylesheet"))
    return
       <param name="{$p}"  value="{$val}"/>
   }
</parameters>
return 
    transform:transform($xml, $xsl, $params)
};
```

# Conclusion and outlook

Congrats! You created a lightweight web application to publish digital editions. And especially due to the recent chapter, the code base of this application does not look too bad any more. Unfortunately we can't say the same about the [start page](http://localhost:8080/exist/apps/thun-demo/pages/index.html) of our application. But this will be the topic of the [next chapter](../part-10-nicer-start-page). 
