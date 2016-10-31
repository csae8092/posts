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

### ad the library

First thing we have to do to implement tablesorter is to add the necessary libraries to **resources/js**. You can download the needed files either from [tablesorter's website](http://tablesorter.com/docs/) or you from [here]




