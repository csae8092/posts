# Introduction and requirements

From an application’s user’s perspective, the upcoming changes to our code base are more or less invisible. But from the perspective of the people who are in charge of developing and maintaining this application, these changes will make their lives much easier, hopefully. 

In the [last chapter](../part-4-xslt-transformation) we implemented a function which takes an XML source and transforms it with a XSLT stylesheet into a more or less good looking HTML document. You can download the code [here](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-4/thun-demo-0.1.xar). The only thing the user has to do for this is to click on a link.

But when you now try to pack your application and deploy it to another server, you will run into severe troubles – as long as the server you try to deploy your package to is not named exactly as the one you developed the application on. As you can see in the following screenshot, I deployed our little application to my personal but public play and test server called [www.digital-archiv.at](http://www.digital-archiv.at) (and not to localhost).

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-5/image_0.jpg)

When you try to click on one of the links, you will be redirected to localhost thanks to our hardcoded URLs. And as long as you don't have exact the same application running on your localhost, you will receive something like this:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-5/image_1.jpg)

And even if you want to deploy your application on the same server with a new name (we will look into renaming the application in another part of the tutorial lateron), you will run into problems: Currently your table of content will stay completely blank since so far, the application will look for your XML documents in a collection called something like *db/***_thun-demo_***/*… and not in the collection *db/***_{name-of-your-app}_***/…*.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-5/image_2.jpg)

# Removing hard coded links

These malfunctions are clearly related to hard-coded URLs we introduced in the last chapter. So let’s remove them. A task which is not too difficult to manage thanks to the developers of eXist-db, as they provided us with all the necessary variables we need to replace such changeable things like the name of the server or the name of the application. Those variables are declared in `modules/config.xqm`. 

## Table of contents (app:toc)

We create the table of contents with the function *app:toc* we wrote for this, and so far this function fetches the data needed to write the table of contents from with

`for $doc in collection("/db/apps/thun-demo/data/editions")/tei:TEI`

To remove the application’s name from this function we use the variable *$config:app-root* which, according to the inline comment to this variable/function
> Determine[s] the application root collection from the current module load path. 
For our application this means that *$config:app-root* will resolve into `/db/apps/thun-demo/`. Since our XML/TEI documents are stored in `/db/apps/thun-demo/data/edition/`, we have to add this last part. We achieve this with the XQuery function *concat()* which concatenates two or more strings. Combining that results in:

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection(concat($config:app-root, '/data/editions/'))//tei:TEI
        return
        <li><a href="{concat("http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=",functx:substring-after-last(document-uri(root($doc)), "/"))}">{document-uri(root($doc))}</a></li>   
};
```

This fix brings back the table of contents even if we rename the application.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-5/image_3.jpg)

Now we also want to remove the other hard coded link from this function which is the link leading to our *show.html* page:

**modules/app.xql**

```html
<a href="{concat('http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=',functx:substring-after-last(document-uri(root($doc)), '/''))}">{document-uri(root($doc))}</a>
```

To remove this hard-coded link, we could basically do something similar as above. Unfortunately building the needed URL [http://localhost:8080/exist/apps/rita/pages/show.html](http://localhost:8080/exist/apps/rita/pages/show.html) is slightly more complicated than */db/apps/thun-demo/data/editions*. 
But since all our HTML pages are located in the same directory we can work with relative links meaning we don't have to care at all about e.g. the name of the server our application runs on. Keeping this in mind, we can modify **modules/app:toc** as follows:

```xquery
(:~
 : creates a basic table of contents derived from the documents stored in '/data/editions'
 :)
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection(concat($config:app-root, '/data/editions/'))//tei:TEI
        return
        <li>
          <a href="{concat("show.html?document=",functx:substring-after-last(document-uri(root($doc)), "/"))}">{document-uri(root($doc))}</a>
        </li>
```

To test my claim, we should browse to the table of contents and follow one of the links. Although the link seems to be correct, we simply get a not very nice error message:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-5/image_4.jpg)

BEWARE. So far I am still working with a renamed version of our thun-demo app (which is now called rita, an acronym of **R**eading **I**n **T**he **A**lps). 

## app:XMLtoHTML

As the error message points out, there seems to be something wrong in the function *transfrom:transform*, a function which we are calling in the function *app:XMLtoHTML* we wrote a few chapters back. 

Upon inspecting this function, the problem becomes quite obvious. Look at the bold lines

**modules/app.xql and app:XMLtoHTML**

```xquery
declare function app:XMLtoHTML ($node as node(), $model as map (*), $query as xs:string?) {
let $ref := xs:string(request:get-parameter("document", ""))
let $xml := doc(concat("/db/apps/thun-demo/data/editions/",$ref))
let $xsl := doc("/db/apps/thun-demo/resources/xslt/xmlToHtml.xsl")
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

The XML as well as the XSL document involved in the transformation are loaded from hard coded URLs. But we can change this easily once more with the help of **$app:config**

```xquery
declare function app:XMLtoHTML ($node as node(), $model as map (*), $query as xs:string?) {
let $ref := xs:string(request:get-parameter("document", ""))
let $xml := doc(replace(concat($config:app-root,"/data/editions/",$ref), '/exist/', '/db/'))
let $xsl := doc(concat($config:app-root, "/resources/xslt/xmlToHtml.xsl"))
...
```

Now the creation of the table of contents is showing the (right) content and sends the correct parameters to the right location of `show.html` which itself renders the XML/TEI document, without consideration of the name of the application or the server’s name. 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-5/image_5.jpg)

# Conclusion and outlook

In this 5th part of our little tutorial we removed any hardcoded links/references from our application. This allows us to pack the app and deploy it to any other eXist-db server without having to worry (too much) about breaking links.

In the [next part](../part-6-rename-the-app) of this series of tutorials, we will learn how to rename our application. This is extremely useful if you plan to use the code base written so far for similar digital edition projects.

 

