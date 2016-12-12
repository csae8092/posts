# Introduction

In this post we will add a so called **post-install.xql** to our application. This script will be triggered after our application (the .xar package) has been installed and will then create a new user-group in the application and add write and execution rights to users of this group of some selected documents and collections. Such a user-rights management is needed when e.g. editors want to curate/add data to the database.
But before we start, the usual reminder: The current code, that we developed [so far](../part-7-toc-from-heterogeneous-sources/), can be downloaded [here](https://github.com/csae8092/posts/raw/master/pimp-de-web-app/downloads/part-7/aratea-digital-0.1.xar). 

## post-install.xql

For the beginning, let's create a new document in our applications root directory called `post-intall.xql`. Then we add the following code snippet into it: 

```xquery
xquery version "3.0";
import module namespace config="http://www.digital-archiv.at/ns/aratea-digital/config" at "modules/config.xqm";

declare variable $app-user := "aratea-digital";

if (not(sm:group-exists($app-user)))
then sm:create-group($app-user)
else (),
```

As you can see we declared a variable named `$app-user` to store some arbitrary string which will be the name of a freshly created user-group, in case there does not yet exist such a group.
In a next step we iterate over all collections and documents which we want to grant write-access to the members of our $app-usergroup. This is easily achieved with the follwoing few lines: 

```xquery
for $collection in xmldb:get-child-collections($config:app-root||"/data")
    let $changed := sm:add-group-ace(xs:anyURI($config:app-root||"/data/"||$collection), $app-user, true(), "rwx")
    for $resource in xmldb:get-child-resources(xs:anyURI($config:app-root||"/data/"||$collection))
        return
            sm:add-group-ace(xs:anyURI($config:app-root||"/data/"||$collection||"/"||$resource), $app-user, true(), "rwx")
```

Now any user of $app-user group can now write, modify and delete any resources stored in 'data/' and its sub collections. 

## register post-install.xql

Before we can try out this script we will have to register it first the application's `repo.xml` document which is also located in the application's root directory. In this document, there is an empty element called `<finish/>` just below the element `<prepare>pre-install.xql</prepare>`, which we have to modify like this: `<finish>post-install.xql</finish>`.

## Test post-install.xql

In case we want to test if our post-install.xql works - and we do want to test it -, we

1. have to run the sync.xql script,
2. then pack the application with **ant**,
3. uninstall the current application from using eXist-db's package manager, 
4. and install the freshly packed .xar.

After the installation process is completed, we can open eXist-dbs user manager, click on the 'goups' tab and check if there is an **aratea-digial** user.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/pimp-de-web-app/images/part-8/image_0.jpg)

## Restrict access / enforce log-in