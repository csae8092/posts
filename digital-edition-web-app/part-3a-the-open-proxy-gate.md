# Introduction and requirements

["With great power comes great responsibility."](http://quoteinvestigator.com/2015/07/23/great-power/). No matter if Uncle Ben or Voltaire is the source, they are both right as far as eXist-db's built in [REST-API](http://exist-db.org/exist/apps/doc/devguide_rest.xml) which was briefly introduced in the [last post](../part-3-table-of-content).


## What's the problem

On Friday, 10th of March Matthias Göbel posted the following mail on the [TEI-Mailing List](https://listserv.brown.edu/archives/cgi-bin/wa?A1=ind1703&L=TEI-L#44):

> Dear TEI-Community,

> thank you for offering an increasing number of documents stored in outstanding great databases like eXist-db and available via REST. Would those guys using eXist-db consider to capture&redirect the "_query" parameter (or at least a set of function names) to avoid offering an open proxy like in this example:

>  `https://tei2016app.acdh.oeaw.ac.at/data/?_query=xquery%20version%20%223.1%22;response:stream-binary(%20xs:base64Binary(%20data(httpclient:get(xs:anyURI(%22http://24.media.tumblr.com/tumblr_lt8vrdas9o1qb8xalo1_400.jpg%22),%20false(),%20())//httpclient:body))%20,%20%22image/jpg%22)`

> If you are using Apache you might want to

>        RewriteEngine on
>        RewriteCond %{QUERY_STRING} _query=
>        RewriteRule (.*) $1? [R=permanent]

> Best,
> Mathias

As Joe Wicentowski pointed out in [his response](https://listserv.brown.edu/archives/cgi-bin/wa?A2=ind1703&L=TEI-L&F=&S=&P=20827), another solution to this problem is described in the [eXist-db book](http://shop.oreilly.com/product/0636920026525.do) written by Adam Retter and Erik Siegl.

## How to fix it

At the ACDH-OeAW we followed the solution explained by Retter and Siegl on page 181f.:

> To remove the REST Server’s ability to directly receive web requests, you can modify
> the parameter hidden in $EXIST_HOME/webapp/WEB-INF/web.xml:

```xml
 <init-param>
 <param-name>hidden</param-name>
 <param-value>true</param-value>
 </init-param>
```

> Changing this parameter means that the REST Server will not directly receive
> requests; rather, it will only receive requests forwarded from an XQuery controller
> (see “URL Mapping Using URL Rewriting” on page 196). You can then choose to filter
> such requests in your own XQuery controller.

Well actually we followed only the first part and changed the `<init-param>`. Now if someone tries to access the rest endpoint of our eXist-db instance a 403 Error is thrown.

## No data endpoints?

But by applying this fix - or at least the first part of it, our data is not exposed to the public any more. Although we are big fans of open data. So to give the public access to the data, we need some other solution. But as we don't want to fiddle around as little as possible with our application's XQuery controller (I guess the `controller.xql` in our application's root directory is meant by this) we have to come up with another solution.

### RESTXQ

Shutting down the 'out of the box' REST-endpoint was the perfect excuse to start playing around with [RESTXQ](http://exquery.github.io/exquery/exquery-restxq-specification/restxq-1.0-specification.html) and it's implementation in eXist-db.
