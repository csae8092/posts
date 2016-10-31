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

4. Replace the content of the data directory (and its sub directories) with the directories bundled in [this zip-file] which contains the data of our new project. In case you want to know more about this project, please refer to http://www.digital-archiv.at/aratea-online/.

5. Again in eXides menu bar go to **Application --> Download app** or, alternatively evaluate the **sync.xql** script to serialize/save the application's code to your local machine. 

In the end you should have a .xar package like the one you can download [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-1/aratea-digital-0.1.xar).

## Tablesorter-JS

