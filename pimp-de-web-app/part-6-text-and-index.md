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

### write templates

A very strong indicator for an index-linked entity seems to be a `@ref` attribute. So let's write a template rule for all those elements, which adds some span tag with an `@class='linkedEntity'` around, as well as a `@data-key` attribute which has the value of the `@ref` attribute. 

```xslt
<xsl:template match="node()[@ref]">
    <span style='color:green' class="linkedEntity">
        <xsl:attribute name="data-key">
            <xsl:value-of select="current()/@ref"/>
        </xsl:attribute>
        <xsl:apply-templates/>
    </span>
</xsl:template>
```

After adding this snippet of code to `resources/xslt/editions.xsl` and going to e.g. [this manuscript-description](http://localhost:8080/exist/apps/aratea-digital/pages/show.html?document=Austin_HRC_29.xml&directory=descriptions) we will see html elements like this one `<span style="color:green" class="linkedEntity" data-key="#bede">Bede</span>`. As we can see, the HTML preserved the entities' identifier but we cant distinguish any more its type.
