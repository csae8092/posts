# Introduction

In this post we will implement functionalities which will display additional information to accordingly tagged entities by mouse click. This means if the editors encoded e.g. a place with something like `<rs type="place" ref="#example_1">Example Place</rs>`and there is a matching index entry with further information about this Example Place, then this information should be made available. But before we start, the usual reminder. The current code, we developed [so far](../part-5-improve-index-based-search/), can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-5/aratea-digital-0.1.xar).

## XSLT

### Inspect your data

The first thing we have to is to make the XML/TEI encoded information available in the HTML representation as well. This can be down quite easily be adding some lines of code (some template rules to be more precise) to the XSLT document which is responsible for the transformation. In the current aratea-digital project we are using (at least) to different stylesheets, one to transform editions of manuscripts and one to transform descriptions of manuscripts. For this HowTo we will focus on the first one. But before we start coding, we should inspect our data to find out, which entities are tagged, linked to index documents containing additional information and how. As far as the entities are concerned, we simply can consult the `data/index` collection to find out that there are files describing

* Person
* Places
* Organisations
* Bibliographic Items
* Titles of works of literature

But to find out how these entities are tagged and referenced in the manuscript-descriptions and -editions we actually have to inspect those documents. By doing so we find out that, e.g. Persons are tagged with the elements `<author ref='#id'>` sometimes also `<rs type='person' ref='#id'>`, places with `<origPlace ref='#id'>` or sometimes with `<rs type='place' ref='#id'>` Works with `<title ref='#id'>`, bibliographic items with `<bibl>` but the actual entity-index linking is done by the text node of an `<abbr>` child element of `<bibl>`. 

Since this HowTo strives for developing generic solutions, we are not going to write highly specif template rules for all those different mark up but try some 'catch as much as possible approach'.

### write template

A very strong indicator for an index-linked entity seems to be a `@ref` attribute. So let's write a template rule for all those elements, which adds some `<strong>` tag with an `@class='linkedEntity'` around, as well as a `@data-key` attribute which has the value of the `@ref` attribute. 

```xslt
<xsl:template match="node()[@ref]">
    <strong style='color:green' class="linkedEntity">
        <xsl:attribute name="data-key">
            <xsl:value-of select="current()/@ref"/>
        </xsl:attribute>
        <xsl:apply-templates/>
    </strong>
</xsl:template>
```

After adding this snippet of code to `resources/xslt/editions.xsl` and going to e.g. [this manuscript-description](http://localhost:8080/exist/apps/aratea-digital/pages/show.html?document=Austin_HRC_29.xml&directory=descriptions) we will see html elements like this one `<span style="color:green" class="linkedEntity" data-key="#bede">Bede</span>`. As we can see, the HTML preserved the entities' identifier but we cant distinguish any more its type.

### fetch additional info

Now we need some way to fetch the additional information to the entities from the index files. In a very basic manner this task can be accomplished quite easily with a few lines of xQuery packed into a function we might call `app:showEntityInfo` and store in `modules/app.xql`. 

```xquery
declare function app:showEntityInfo($node as node(), $model as map(*)) {
    let $ref := xs:string(request:get-parameter('entityID', ''))
    let $element := collection(concat($config:app-root,'/data/indices/'))//*[@xml:id=$ref]
    for $x in $element
    return
        $x
};
```

### write some javascript

The next thing to do is to write some JavaScript code, which will will open a modal, whenever a user clicks on a linked-entity and this modal will display the additional information fetched with function above. How a modal works is quite well described in this [tutorial](http://www.w3schools.com/bootstrap/bootstrap_modal.asp) provided by [w3schools.com](http://www.w3schools.com/). 
So let's add the following code towards the bottom of our base template `templates/page.html`.

```javascript
<script type="text/javascript">
    $(document).ready(function () {
        var trigger = $('.linkedEntity');
        $(trigger).click(function () {
            var dataKey = $(this).data('key');
            dataKey = dataKey.substring(1);
            var url = "entity-info.html?entityID=" + dataKey
            console.log([url, dataKey]);
            $('#loadModal').load(url, function () {
                $('#myModal').modal('show');
            });
        });
    });
</script>
```
This script links all entities with a click event. When this click event is triggered, the value of the data-key is used to build a url-fragment which points to webpage `entity-info.html?entityID={dataKey}`. Then the content of this website is loaded with the help of [jquerys load() function](http://api.jquery.com/load/) and placed into an element with the identifier `#loadModal'. 

### some HTML

Clicking on any entity now won't result in anything visible because there neither does yet exist any html-site called `entity-info.html` nor is there any `#loadModal` element in `templates/pages.html`. The latter is an easy fix: Just add somewhere in `<section class='main-content'>` the element `<div id='loadModal'/>`. Then create an HTML document `pages/entity-info.html` and paste the following code to it:

```html
<div class='modal' id='myModal' role='dialog'>
    <div class='modal-dialog'>
        <div class='modal-content'>
            <div class='modal-header'>
                <button type='button' class='close' data-dismiss='modal'>
                    <span class='fa fa-times'/>
                </button>
                <h4 class='modal-title'>
                   info
                </h4>
            </div>
            <div class='modal-body'>
                <div class='app:showEntityInfo'/>
            </div>
        </div>
    </div>
</div>
```

As one can see this code does contain the HTML-markup of a simple modal which calls for its content our `app:showEntityInfo` function.

### some more JavaScript

Now we have almost everything in place except a little bit more JavaScript code responsible for actually showing the modal. But this accomplished by the few lines of code added to `templates/page.html` just below our function from above:

```javascript
<script type='text/javascript'>
    $(window).load(function () {
        $('#myModal').modal('show');
    });
</script>
```

Now we should be able to click on any of the green colored entities to view additional information stored in the index files.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-6/image_0.jpg)


