# Introduction and requirements

In this part we will 1. rename our thun-demo project, 2. add new data and 3. add a customizable Table of Content. Although basically steps one and two are merely repetitions of [this post](../part-11-start-a-new-project/), so I keep the instructions very limited.

## Start the new project

1. Modify the application's descriptors. Therefore open eXide, then open any document stored in thun-demo's root directory, then in eXides menu bar go to **Aplication --> Edit descriptors** and change 

* Target Collection to e.g. **aratea-digital**
* Name to e.g. **http://www.digital-archiv.at/ns/aratea-digital**
* Abbreviation to e.g. **aratea-digital**
* and Title to e.g. **Aratea Digital**

2. Replace the used namespaces **from http://www.digital-archiv.at/ns/thun-demo** to e.g. **http://www.digital-archiv.at/ns/aratea-digital**

3. Remove "thun-demo" traces in the **build.xml** and replace it with something like **aratea-digital**

4. Replace the content of the data directory (and its sub directories) with the directories bundled in [this zip-file] which contains the data of our new project. In case you want to know more about this project, please refer to [http://www.digital-archiv.at/aratea-online/](http://www.digital-archiv.at/aratea-online/).

5. Again in eXides menu bar go to **Application --> Download app** or, alternatively evaluate the **sync.xql** script to serialize/save the application's code to your local machine. 

In the end you should have a .xar package like the one you can download [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-1/aratea-digital-0.1.xar).

## Tablesorter-JS

Basically we could enable some user interaction on the client side (in the front end) or on the server side on the back end. For the latter we would have to allow the users to send choose/modify some parameters which then would be send to the server, processed by the server and the according result would be finally sent back to the client. The first approach instead would send all information at once to the client who would then be responsible for any further manipulation through the user. 
We will opt for a client side solution because then we can use a JavaScript library called [tablesorter](http://tablesorter.com/docs/) which fulfills our requirements almost completely out of the box. With this, we don't have to do much coding by ourself and we don't have to communicate too much with the server.
But we have to be aware that this solutions does not scale very well. In case we have a lot of documents stored in `data/editions` all this information would be sent to and processed by the client at once which could lead waiting times each time our page with the table of content will be loaded. 

### add the library

First thing to do for implementing **tablesorter** is to add the necessary libraries to **resources/js**. You can download the needed files either from [tablesorter's website](http://tablesorter.com/docs/) or in zipped from from [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/libraries/tablesorter.zip). In case you chose the latter option, unzip the files and add them **resources/js/tablesorter**. This directory should now look like depicted below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-2/image_0.jpg)

### include/load the library

In a next step we have to import/load those files into our html-pages. We could do this one time in our application's base template (**templates/base.html**) or only in those pages, where we actually going to use this library. Let's go with the second option so we don't produce too much traffic.
So open **pages/toc.html** and add the following lines of code:

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.js"/>
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.widgets.js"/>
    <script src="$app-root/resources/js/tablesorter/js/jquery.tablesorter.pager.js"/>
    <link rel="stylesheet" type="text/css" href="$app-root/resources/js/tablesorter/css/theme.bootstrap.css"/>
    <link rel="stylesheet" type="text/css" href="$app-root/resources/js/tablesorter/css/jquery.tablesorter.pager.css"/>
    <script> $(function() { $("table").tablesorter({ theme : "bootstrap", widthFixed: false,
        headerTemplate : '{content} {icon}', widgets : [ "uitheme", "filter", "zebra" ], filter_cssFilter: "form-control", }) }); 
    </script>
    <h1>Table of Content</h1>
    <div data-template="app:toc"/>
    
    <table class="table table-striped table-condensed table-hover">
        <thead>
            <tr align="center" font-size="1.0em">
                <th class="header">
                    file
                </th>
            </tr>
        </thead>
        <tr data-template="app:toc"/>
    </table>
</div>
```

When we browse no to [http://localhost:8080/exist/apps/aratea-digital/pages/toc.html](http://localhost:8080/exist/apps/aratea-digital/pages/toc.html) we won't see any breathtaking results. The only visible change is the little word "file".

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-2/image_1.jpg)

This is of course related to the xQuery function **app:toc** which we have to modify now according to the code snippet below: **modules/app.xql**

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection(concat($config:app-root, '/data/editions/'))//tei:TEI
        return
        <tr>
            <td>
                <a href="{app:hrefToDoc($doc)}">{app:getDocName($doc)}</a>
            </td>
        </tr>   
};
```

Now changes are slightly more visible but only still not very useful: 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-2/image_2.jpg)

The reason for this are obviously some css-issues which we can solve in (at least) two ways. We could 1) override tablesorters own css (**resources/js/tablesorter/css/theme.bootstrap.css**) or simiply exchange our application's bootstrap theme. Since the later option is related to less work, I chose this one and will use the [Yeti-Theme](http://bootswatch.com/yeti/). See [here](../part-2-getting-started) on how to apply a new theme.

Finally, our users can sort and filter the table of content according to their needs. 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-2/image_3.jpg)


### tablesorter everywhere

The table of content is not the only place in our application where we could use the tablesorter. Such a search-/filter-, and sortable table would also be quite handy for the result page of our fulltext search. Searching e.g. for [perseus](http://localhost:8080/exist/apps/aratea-digital/pages/ft_search.html?searchexpr=perseus)** results in table like the one depcited below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-2/image_4.jpg)

To implement tablesorter, we simply have to import the same libraries as before to **pages/ft_search.html** and the result will look like on the picture below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-2/image_5.jpg). 


# Conclusion and Outlook

Our Table(s of content) can now be searched/filtered and sorted by the users. In the [next part](../part-3-bookmark-filtered-tablesorter-results) we will tackle the not so well working person-index and by addressing this issue we will also investigate a solution on how to bookmark results, filtered with tablesorter. As usual, the code created so far can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-2/aratea-digital-0.2.xar)). 