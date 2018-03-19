# Introduction and requirements


The fourth part of this series of tutorials starts off where the [third part](../part-3-table-of-content) ends. You can deploy the latest codebase by downloading and installing [this](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-3/thun-demo-0.1.xar) package. For details instructions on how to do this, please consult the [previous tutorial](../part-3-table-of-content).

In the last tutorial we created a very basic [table of contents](http://localhost:8080/exist/apps/thun-demo/pages/toc.html) which lists the content of the application’s data directory. For this part of the tutorial our requirements demand, that we should be able to click on any document of our interest listed in the table of contents to retrieve a (very basic) HTML representation of the underlying XML/TEI document.

To realize this requirements we are actually going to do two things. First, we have to provide some kind of links in our table of contents which will trigger an XSL transformation which will turn the XML/TEI document into an HTML document, which will be easier to read (for humans). Let’s start with the latter.

# Transform XML with XSLT

## Writing a basic XSLT Stylesheet

Since this is not a tutorial about XSLT, I won’t go into the XSL specific details of the stylesheet we are going to write. For  a basic introduction into XSLT please refer to [this tutorial](https://www.w3schools.com/xml/xsl_intro.asp) provided by [w3schools.com](http://www.w3schools.com/). (A long list of XSLT tutorials can be found at [http://www.whoishostingthis.com/resources/xslt/](http://www.whoishostingthis.com/resources/xslt/)).

To get started, we need a place to store this (and later some more) stylesheets. Therefore we create a new collection **resources/xslt/**. In this collection we create a new document called  **resources/xslt/xmlToHtml.xsl**. Our very very basic stylesheet will only contain the following few lines of code:

```xslt
<xsl:stylesheet xmlns="http://www.w3.org/1999/xhtml" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:tei="http://www.tei-c.org/ns/1.0" exclude-result-prefixes="tei" version="2.0">
    <xsl:template match="/">
        <xsl:apply-templates/>
    </xsl:template>
</xsl:stylesheet>
```

You wonder what this stylesheet does. Well, let’s find out.


## Running that transformation with XQuery function(s)

To see the effects of this XSLT document on one of our XML/TEI documents, we have to interconnect those two documents in some way. eXist-db provides a function called **transform:transform** which does this for us. What is left for us to do is to write a litter function around transform:transform which will be triggered by eXist-db templating system, whenever the page **show.html** (our first site created by our own) is loaded.

To do so, add the following lines to **modules/app.xql**:

```xquery
declare function app:XMLtoHTML ($node as node(), $model as map (*), $query as xs:string?) {
let $xml := doc("/db/apps/thun-demo/data/editions/celakovsky-an-thun_1850-10-11_A3-XXI-D75.xml")
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

As you can see below, we are transforming the document celakovsky-an-thun_1850-10-11_A3-XXI-D75.xml with the stylesheet we just created. What is left now to do is to reference the function **app:XMLtoHTML** in **show.html**. Therefore just add the following line to show.html:

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <h1>show.html</h1>
    <div data-template="app:XMLtoHTML"/>
</div>
```

Now browse to [http://localhost:8080/exist/apps/thun-demo/pages/show.html](http://localhost:8080/exist/apps/thun-demo/pages/show.html) and see the result of your first eXist-db based XSL transformation.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-4/image_0.jpg)

As already mentioned, writing XSLT stylesheets is not in the scope of this HowTo. But to have a slightly more attractive (HTML) representation of your XML/TEI (well, actually of the TEI files used for this tutorial), I will add a few more lines of code to **resources/xslt/xmlToHtml.xsl**. You can find the XSLT stylesheet I used for the screenshots [here](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-4/xmlToHtml.xsl). Just download this file and replace our basic stylesheet stored in **resources/xslt** with it.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-4/image_1.jpg)

You might have noticed it: I also removed `<h1>show.html</h1>` from **pages/show.html**.

# Link from ListView to DetailView

So far our table of contents is not linked to the detail view of our XML/TEI document (show.html) at all. Even worse, show.html generates only a HTML representation of that one particular XML file **celakovsky-an-thun_1850-10-11_A3-XXI-D75.xml.** To fix this, we could of course write as many xmlToHtml functions as there are XML/TEI documents in our `data/editions/` collections and call them by the same number of HTML pages. But this is not much fun to implement since it would involve a lot of copying and pasting, not to mention maintaining the application and adding new functionalities.

But – big surprise – there is a better solution (computers are good at stupidly repeating tasks)! This solution builds upon **dynamically genereated URL parameters.**

## Fetching (parsing) URL parameters

As we saw before, the **transform:transform** function takes as first parameter the xml document which it should transform. To turn our **app:XMLtoHTML** function into a more dynamic one, we simply have to remove the hardcoded path to one XML/TEI document and replace it with something more flexible. Let’s say we are able to pass the name of the XML/TEI document we want to transform to the function as a part (or parameter) of the URL which we type into the browser (or link to it) when we are calling **show.html**. Our URL could look like, e.g.

> http://localhost:8080/exist/apps/thun-demo/pages/show.html?document={nameOfAnyDocument.xml}

Then we just have to tell our **app:XMLtoHTML** function to fetch (parse) the name of the document from the URL, inject this name as parameter into our function and transform it. Exactly this is accomplished by the following changes in our **app:XMLtoHTML** function.

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

When you now save your changes and browse to [http://localhost:8080/exist/apps/thun-demo/pages/show.html](http://localhost:8080/exist/apps/thun-demo/pages/show.html), you will only see the result of a transformation on a non existing XML document.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-4/image_2.jpg)

But when you add an actual name of an XML/TEI document stored in the applications **data/editions/** collection as a value to the parameter **document** (e.g. [http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=celakovsky-an-thun_1850-10-11_A3-XXI-D75.xml](http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=celakovsky-an-thun_1850-10-11_A3-XXI-D75.xml)), exactly this file will be transformed by our **app:XMLtoHTML** function.

## Dynamically creating URL parameters

Because I am more of a lazy person, I don't want to type the file names into my browser to see their HTML-representations. I just want to click on the document's name listed in the application's table of contents.

To achieve this we have to modify our table of contents function **app:toc**.

Freest, we need to declare a new namespace for the helper function **functx:substring-after-last**. For more upon this (and many other very useful functions) please refer to: [http://www.xqueryfunctions.com/xq/functx_substring-after-last.html](http://www.xqueryfunctions.com/xq/functx_substring-after-last.html).

So after our TEI namespace declaration, add this line

`declare namespace functx = 'http://www.functx.com';`

And then paste the following lines into **modules/app.xql**:

```xquery
declare function functx:substring-after-last
  ( $arg as xs:string? ,
    $delim as xs:string )  as xs:string {
    replace ($arg,concat('^.*',$delim),'')
 };
```

Now we can call this function in our updated **app:toc** function.

```
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection("/db/apps/thun-demo/data/editions")/tei:TEI
        return
        <li><a href="{concat('http://localhost:8080/exist/apps/thun-demo/pages/show.html?document=',functx:substring-after-last(document-uri(root($doc)), '/'))}">{document-uri(root($doc))}</a></li>   
};
```

Yes, the code is quite clumsy and also contains hard coded links. But we are going to clean up the code in one of the following HowTos. For now, it does what we wanted. That is, when you now browse to [http://localhost:8080/exist/apps/thun-demo/pages/toc.html](http://localhost:8080/exist/apps/thun-demo/pages/toc.html), you will see that the names (and paths) of the documents are now hyperlinks.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-4/image_3.jpg)

A click on any of those links will present us the XML/TEI’s HTML representation or DetailView.

Before we conclude this fourth part of the tutorial, let’s remove the link to show.html from our application’s navigation menu since there is not much use for linking to this site without any parameters added to the URL. To remove link, just delete the following lines from the main application’s template **templates/pages.html**:

```html
<li>
    <a href="show.html">show.html</a>
</li>
```

# Conclusion and outlook

With this fourth part of our digital edition web app tutorial we are now capable of triggering XSL transformations with one mouse click from our digital edition app’s table of contents. With some XSLT as well as some CSS, HTML (and maybe some JavaScript) skills you will be able to create quite fancy (HTML) representations of your XML/TEI documents.

Looking at the requirements of our application from Part I of this tutorial, we see that the only thing which is not realized yet is a fulltext search. But before we will engage this feature, we should spend some time and energy on refactoring the code (see [https://de.wikipedia.org/wiki/Refactoring](https://de.wikipedia.org/wiki/Refactoring)) as well on removing all the hard coded links we introduced in this chapter. Therefore in the [next part](../part-5-clean-up-the-code) of this tutorial we will clean up the code and transform our application into a state which will make it very easy to pack the code and deploy it to any other eXist-db instance – without having to worry that some functions won’t work anymore.
