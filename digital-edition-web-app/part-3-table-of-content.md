# Introduction and requirements

In this third part of our series of HowTos we will upload the XML/TEI files to our database and write our first XQuery function which will generate a very basic table of contents from the uploaded XML/TEI documents. To present this table of content to the users of our web app, we will also learn how to integrate XQuery functions within HTML code. (Yes, these lines were copy-pasted from Part II).

Part III of this tutorial builds upon the work done in [Part II](../part-2-getting-started). In case you lost your laptop in the train, deleted your eXist-db instance or just didn’t follow Part II, you will need [the code created in Part II](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-2/thun-demo-0.1.xar).

## Download…

The probably most simple way to distribute your application is to download it with the help of eXide. So browse to [http://localhost:8080/exist/apps/eXide/index.html](http://localhost:8080/exist/apps/eXide/index.html), and open any files stored in your application root directory like *thun-demo/repo.xml*. Then click on **Application** in eXide’s main menu and click on **Download app**.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_0.jpg)

This triggers a download of a file called *thun-demo.xar*. This is actually just a .zip file containing your application’s code. So you can rename it to *thun-demo.zip*, unzip it and inspect its content, which should look strikingly similar to the directories and documents of your application stored in eXist-db.

Such a *thun-demo.xar* containing the code from the end of Part II can be downloaded [here](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-2/thun-demo-0.1.xar).

## … and (re)install your application.

To (re)install your application, browse to the eXist-db dashboard [http://localhost:8080/exist/apps/dashboard/index.html](http://localhost:8080/exist/apps/dashboard/index.html) and open the **Package Manager**. This gives you a list of all your currently available as well as installed packages (some packages might be applications). Check ‘installed’ to only see your installed apps. In case you did follow along coding while reading Part II, you should see an entry: "Thun Demo App".

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_1.jpg)

Let's uninstall it - but please make sure you download your web app before as described above. To download an app, hover over the entry and a quite self-explanatory icon appears. Click on it and confirm your command. When you now return to the dashboard, the tile named "Thun-Demo" should be gone (the latest on reloading the page).

Ok, now try and install "Thun-Demo". Open the **Package Manager** again and click on the top right symbol:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_2.jpg)

Inside the pop up dialog, click in the **Upload** field, browse to the directory where you saved your previously downloaded thun-demo.xar file and upload it. Since the application is very small, the upload and the following unzipping and installing should go very fast. After a successfull upload/install you should see (again) a tile on your dashboard named "Thun Demo App". Start it and browse through all pages to check if everything is working.

Congrats! You just downloaded and (re)deployed your first application. If you are very bold and brave (and have access to another eXist-db instance) you could try installing your application on another eXist-db instance.

I tried this and by doing so I noticed that the eXist-db logo is not displayed. It seems like the eXist-db instance I deployed the application to does not have such a logo stored in its general *shared/resources* directory. Luckily for us, this logo is the only thing left that still depends on eXist-db’s s*hared/resources* directory… 

# Upload XML/TEI documents

Ok, but now let's really start uploading some XML/TEI documents to turn our humble web app into a digital edition web app. The only question left to answer is where to upload/store the XML/TEI documents to.

First you have to decide if you want to store your data next to your application’s code or if you prefer a clear separation between data and code. The good thing is that we strive to build a highly self contained, small and not too complex application. And the even better thing is that our application is not responsible for any kind of long term preservation of the data contained within. Therefore I see no really good arguments against storing the application's code and its data in the same place.

Secondly, we have to figure out a smart directory layout for storing our data. This is not too complicated when you know in advance what kind of data you will have to deal with and what kind of functionalities you would like to provide. In our case of building a new digital edition for the **Thun correspondence**, I can tell that we will be confronted with two different kind of datasets. On the one hand have we have an XML/TEI document for each letter. On the other hand we have a couple of index files containing information about places and persons.

Mapping these two types of documents in directories/collections leads to a **data/** collection which is located in our application’s root collection, containing two subcollections called **data/editions/** and **data/indices/**. Looking at the following screenshots might makes things even easier to follow.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_3.jpg)![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_4.jpg)

Now we can upload our XML/TEI documents from the Thun correspondence into the **data/editions/** collection. You can upload data either using eXist-db's **Collection Browser** tile or with the help of oXygen. In case you are going to use the **Collection Browser** browse to `/db/apps/thun-demo/data/editions`, click on the **Upload resources** icon.  
Just in case you don't have any (play)data at hand, you can use the XML documents stored in [this zip-archive](/downloads/editions.zip).

# Publish XML/TEI via exist-db’s (REST) API

If our only goal was to publish our XML/TEI documents on the Internet then we would have accomplished our goal and could go all for a drink. This is because eXist-db ships with an API that exposes the content of our database to the public by default. The nice thing about this application programming interface is that humans as well as computers can interact with it. So as long as we don’t restrict access to our data – and there is absolutely no reason why we should do this –, our uploaded XML/TEI can be read and downloaded.
You don’t believe me? Well, then type the following url [http://localhost:8080/exist/rest/db/apps/thun-demo/data/editions](http://localhost:8080/exist/rest/db/apps/thun-demo/data/editions) into your browser and you see an XML representation of the content of our *data/editions* collection.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_5.jpg)

Getting from this table of contents or list view to the representation (or detail view) of one of the documents is accomplished by adding a slash ("/") and the document's name, the value of the **name** attribute of one of the `<exist:resource>` elements, to the current URL. So for example the following URL leads to the XML/TEI file (or a representation of this file) in your browser:

[http://localhost:8080/exist/rest/db/apps/thun-demo/data/editions/diepenbrock-an-thun_1849-08-30_A3-XXI-D5.xml](http://localhost:8080/exist/rest/db/apps/thun-demo/data/editions/diepenbrock-an-thun_1849-08-30_A3-XXI-D5.xml)

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_6.jpg)

## The open proxy gate.

The usage of eXist-db's 'out of the box' REST-API has some security issues in production environments as described in the post [The open proxy gate](../part-3a-the-open-proxy-gate)

# Table of Contents

Although these views automatically by eXist-db generated are very useful for publishing data in a standardized and machine readable manner,  for humans they are neither very user friendly (a lot of typing is involved to navigate) nor do they look very inviting. So let’s create our first and very basic table of contents, which will list the contents of the *data/editions/* collection and will provide links to the documents stored in this collection.

## HTML

To achieve this, we create a new HTML document called **toc.html** and save it in the *pages/* collection. For the time being, this file will only contain a headline "Table of Contents" and the markup needed to include the code from our base template *templates/page.html*:

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <h1>Table of Contents</h1>
</div>
```

We also want a link to our table of contents page in our applications nav bar. So we open our base template/page.html document and add the following (bold) lines:

```html
<ul class="dropdown-menu">
    <li>
        <a href="index.html">Home</a>
    </li>
    <li>
        <a href="show.html">show.html</a>
    </li>
    <li>
        <a href="toc.html">Table of Contents</a>
    </li>
</ul>
```

Save your changes and check if everything works. Browse to [http://localhost:8080/exist/apps/thun-demo/](http://localhost:8080/exist/apps/thun-demo/), click on **Home** and follow the "Table of Contents" link which should lead you to this page:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_7.jpg)

## XQuery

Impressive! Well not really yet. We should add some content. And for this, we are going to write our first XQuery function. This function will browse through our *data/editions/* collection, fetch the names of all documents stored in this collection and present them in form of a simple list. 

To make sure that our function is going to the expected things, it is a good idea to play around with it a little bit while writing/developing it. For this we open eXist-db’s XML editor eXide:

[http://localhost:8080/exist/apps/eXide/index.html](http://localhost:8080/exist/apps/eXide/index.html) and add the following few lines of code in a new document.

```xquery
xquery version "3.0";
declare namespace tei="http://www.tei-c.org/ns/1.0";

for $doc in collection("/db/apps/thun-demo/data/editions")/tei:TEI
return
    <li>{document-uri(root($doc))}</li>
```

To execute or run this function, just click on **Eval**. This should create a list of the document names at the bottom of eXide. Be aware that this is only a preview showing the first ten hits.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_8.jpg)

 But a little temporary pop up note in the bottom right corner informs you about the actual number of hits:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_9.jpg)

This number should match the number of documents stored in *data/editions/*. Or, to be more precise, this number should match all documents stored in *data/editions/* whose root element or node is named `<TEI>`.

## HTML and XQuery

We have now a (very basic) script that fetches the names and paths of all TEI documents stored in *data/editions/*. Now we need to somehow combine this script with our *toc.html*. To accomplish this, we open *modules/app.xql*, which is a collection of XQuery functions used by our application. 

By default this document just contains some kind of demo function **app:test**. You can inspect the results of this test function on our application's start page (*index.html*).

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_10.jpg)

Let’s add our own table of contents function to this file. The easiest way to start with is probably to copy and paste the **app:test** function, delete everything between the curly brackets {} and exchange the **:test** against a more declarative name. Since the function we are going to write will be responsible for creating a table of contents, why don't name it **toc** for example. The barebone of our toc-function looks like this:


```xquery
declare function app:toc($node as node(), $model as map(*)) {};
```

Now let’s add the code snippet from our little play script between the curly brackets:

```xquery
declare function app:toc($node as node(), $model as map(*)) {
    for $doc in collection("/db/apps/thun-demo/data/editions")/tei:TEI
        return
        <li>{document-uri(root($doc))}</li>   
};
```

Now eXide (or Oxygen) will complain abouta  missing namespace definition for TEI. So let’s add a TEI namespace declaration to the **app.xql** document right after the imported module namespaces.

```
declare namespace tei="http://www.tei-c.org/ns/1.0";
```

Now the only thing we have to do is to link the **toc.html** with our **app:toc** function. To achieve this, let’s open **toc.html** and add the following (bold) line to your already existing mark up.

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <h1>Table of Contents</h1>
    <div data-template="app:toc"/>
</div>
```

Save the changes and browse to [http://localhost:8080/exist/apps/thun-demo/pages/toc.html](http://localhost:8080/exist/apps/thun-demo/pages/toc.html)

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-3/image_11.jpg)

# Conclusion and outlook

Congrats, you wrote your first XQuery script in this session of our tutorial and you played with it in a quite interactive way using eXide. You also wrote your first XQuery function and called it with the help of eXist-db’s templating system. Oh, and by the way, you have also just created a very basic digital edition application. This application publishes XML/TEI in a machine readable and yes – also human readable manner. Even reading the XML/TEI model of the encoded text is usually not big fun for most of your users (from the humanities domain). 

In the [4th part](../part-4-xslt-transformation) of this little series of tutorials we will engage with the detail view of our application. This means that we will write another XQuery function to fetch an XML/TEI document and transform it into a very basic HTML document with the help of an XSL-T script we are also going to write in the next session. 

