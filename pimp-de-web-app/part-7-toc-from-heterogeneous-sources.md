# Introduction

This post describes one way of how to derive a table of content from documents stored in different collections. But before we start, the usual reminder. The current code, we developed [so far](../part-6-text-and-index/), can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-6/aratea-digital-0.1.xar).

# Challenges

A table of content which is derived from heterogeneous sources is confronted with up to three challenges.

1. By our understanding of heterogeneous sources they are stored in different sub collections of the applications data-collection as it is observable in the aratea-digital project, where we store editions of manuscripts in `data/editions` and descriptions of manuscripts in `data/descriptions`. The script which is responsible for the transformation of the XML/TEI document into its HTML view needs to be aware of this for being able to locate the XML/TEI document.
2. It is highly possible that the documents stored in different collections might be transformed with different stylesheet. So this information, which collection uses which stylesheet needs also be passed into the transformation script.
3. Since documents stored in different collections might be different with e.g. different meta data (e.g. letters vs. manuscript descriptions) the layout of a common table of content might be flexible enough to deal with this.

## improve and app:ToC

To deal with those aforementioned challenges we have have to invest some work into the functions mainly involved into rendering a table of content (`app:ToC`) which provides the links the HTML representations of the XML/TEI documents listed in this table of content as well as into our transformation function.

First we have to modify the function that it does not only show the documents stored in `data/editions` but also in `data/descriptions`. Since we want to omit the documents stored in `data/indices` we can't simply write something like `for $doc in (collection(concat($config:app-root, '/data'))//tei:TEI`. But we can change `app:ToC` in `modules/app.xql` into: 

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in (collection(concat($config:app-root, '/data/editions'))//tei:TEI, collection(concat($config:app-root, '/data/descriptions'))//tei:TEI)
        return
        <tr>
            <td>
                <a href='{app:hrefToDoc($doc)}'>{app:getDocName($doc)}</a>
            </td>
        </tr>
```

With this change in place, our table of content looks like on the screenshot below, listing the content of `data/editions` as well as `data/descriptions`:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-7/image_0.jpg)

But of course clicking on e.g. [Bamberg_Class_55.xml](http://localhost:8080/exist/apps/aratea-digital/pages/show.html?document=Bamberg_Class_55.xml) turns up the following, not very useful result, because app:app:XMLtoHTML looks by default for this document in `data/editions` and not in `data/descriptions`. 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-7/image_1.jpg)

To fix this, we have to inject the name of the collection of this file into the `?directory` parameter. For this we add a new variable `$collection` which stores the collection where the document is stored. The modified `app:ToC` function stored in `modules/app.xql` looks now in the snippet below:

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in (collection(concat($config:app-root, '/data/editions'))//tei:TEI, collection(concat($config:app-root, '/data/descriptions'))//tei:TEI)
    let $collection := functx:substring-after-last(util:collection-name($doc), '/')
        return
        <tr>
            <td>
                <a href="{concat(app:hrefToDoc($doc),'&amp;directory=',$collection)}">{app:getDocName($doc)}</a>
            </td>
        </tr>
};
```

When we now click on e.g. [Bamberg_Class_55.xml](http://localhost:8080/exist/apps/aratea-digital/pages/show.html?document=Bamberg_Class_55.xml&directory=descriptions), `app:XMLtoHTML` can locat the needed XML/TEI document and displays something:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-7/image_2.jpg)

What is missing now is the injection of the name of the stylesheet we want the transformation to be done with. But following the convetion that stylesheets are named like the collection in which the documents are stored for which this stylesheet was written is trivial. Just ad the following string to the link to the HTML-Representation:

```xquery 
...
<a href="{concat(app:hrefToDoc($doc),'&amp;directory=',$collection,'&amp;stylesheet=',$collection)}">{app:getDocName($doc)}</a>
...
```

With this change in place a click on [Bamberg_Class_55.xml](http://localhost:8080/exist/apps/aratea-digital/pages/show.html?document=Bamberg_Class_55.xml&directory=descriptions&stylesheet=descriptions) leads to a view like depicted below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-7/image_3.jpg)

# Conclusion and Outlook

