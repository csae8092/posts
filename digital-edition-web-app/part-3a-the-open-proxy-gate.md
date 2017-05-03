# Introduction and requirements

> ["With great power comes great responsibility."](http://quoteinvestigator.com/2015/07/23/great-power/)

No matter if Uncle Ben or Voltaire, they are both right as far as eXist-db's built in [REST-API](http://exist-db.org/exist/apps/doc/devguide_rest.xml), which was briefly introduced in the [last post](../part-3-table-of-content), is concerned.


## What's the problem

On Friday, 10th of March 2017, [Mathias Göbel](https://www.sub.uni-goettingen.de/kontakt/personen-a-z/personendetails/person/mathias-goebel/) posted the following mail on the [TEI-Mailing List](https://listserv.brown.edu/archives/cgi-bin/wa?A1=ind1703&L=TEI-L#44):

> Dear TEI-Community,

> thank you for offering an increasing number of documents stored in outstanding great databases like eXist-db and available via REST. Would those guys using eXist-db consider to capture&redirect the "_query" parameter (or at least a set of function names) to avoid offering an open proxy like in this example:

>  `https://tei2016app.acdh.oeaw.ac.at/data/?_query=xquery%20version%20%223.1%22;response:stream-binary(%20xs:base64Binary(%20data(httpclient:get(xs:anyURI(%22http://24.media.tumblr.com/tumblr_lt8vrdas9o1qb8xalo1_400.jpg%22),%20false(),%20())//httpclient:body))%20,%20%22image/jpg%22)`

> If you are using Apache you might want to

>        RewriteEngine on
>        RewriteCond %{QUERY_STRING} _query=
>        RewriteRule (.*) $1? [R=permanent]

> Best,
> Mathias

As Joe Wicentowski pointed out in [his response](https://listserv.brown.edu/archives/cgi-bin/wa?A2=ind1703&L=TEI-L&F=&S=&P=20827), another solution to this problem is described in the [eXist-db book](http://shop.oreilly.com/product/0636920026525.do) written by Adam Retter and Erik Siegel.

## How to fix it

At the [ACDH-OeAW](http://www.oeaw.ac.at/acdh/) we followed the solution explained by Retter and Siegel on page 181f.:

> To remove the REST Server’s ability to directly receive web requests, you can modify
> the parameter hidden in $EXIST_HOME/webapp/WEB-INF/web.xml:

> ```xml
 <init-param>
 <param-name>hidden</param-name>
 <param-value>true</param-value>
 </init-param>
```

> Changing this parameter means that the REST Server will not directly receive
> requests; rather, it will only receive requests forwarded from an XQuery controller
> (see “URL Mapping Using URL Rewriting” on page 196). You can then choose to filter
> such requests in your own XQuery controller.

Well actually we followed only the first part and changed the `<init-param>`. Now if someone tries to access the rest endpoint of our eXist-db instance a `403 Error` is thrown.

## No data endpoints?

By applying only the first part of this fix, our data is not exposed to the public any more. Even though we are big fans of open data. So to give the public access to the data, we need some other solution. But as we don't want to fiddle around as little as possible with our application's XQuery controller (I guess the `controller.xql` in our application's root directory is meant by this) we have to come up with another solution.

### RESTXQ

Shutting down the 'out of the box' REST-endpoint was the perfect excuse to start playing around with [RESTXQ](http://exquery.github.io/exquery/exquery-restxq-specification/restxq-1.0-specification.html) and it's implementation in eXist-db. Following this specification, RESTXQ

>[...] defines a standard set of vendor agnostic XQuery Annotations and functions for XQuery. When implemented by vendors, these annotations provide a standard W3C XQuery compliant approach to delivering RESTful Web Services from XQuery, whilst the functions provide user convenience for interacting with implementations of RESTXQ.

So if the RESTXQ holds what its specification promises, we simply have to right a few very basic XQUery functions which will fetch the

1. the content, the documents of a directory/collection and expose basic metadata of those documents to create some kind of list view
2. the content of one document and display it to the user to create some kind of detail view.

Then we have to annotate this functions according to RESTXQ and that's it.
I won't post the actual code listing here because it is (1) quite long, (2) might use function and variables which are not yet part of our tutorial app, (3) and since I am still experimenting with RESTXQ such a listing will be outdated pretty sure pretty fast. Alternatively you can follow [this link](https://github.com/acdh-oeaw/tei-abstracts-app/blob/master/modules/api.xqm) to see how I implemented a RESTXQ data retrieval endpoint in [tei-abstracts-app](https://tei2016app.acdh.oeaw.ac.at/)

# Where to go from here

eXist-db's out of the box REST-API has the great advantage that you don't have to do anything but using it. But if you plan to use it in production you have to be aware of some risks and responsibilities.
One alternative is a custom written API, maybe based on RESTXQ. You'll have to invest some time and energy into it, but you'll be rewarded with much better control over your endpoint e.g. by customizing your HTTP header, defining which information you want to expose how and where, or by modelling your endpoint along some API-standards. Especially the last point is something which is not implemented yet but which is definitely something I'd like to implement one day. 
