# Upload and process adlibXML

## import data

Go to `import-documents.html` and hit the **import** button.
This will trigger a script called `totei:triggerBatchTrans`. This script will 

1. read in the identifiers of those documents from [this adlib-endpoint](http://opacbasis.w07adlib1.arz.oeaw.ac.at/wwwopac.ashx?database=archive&amp;command=getpointerfile&amp;number=15) which have been marked as ready, 
2. check if this document is already stored in Glaser-TEI at `data/imported` and if not, it will request the adlibXML, transform it with the stylesheet `resources/xslt/adlibXMLtoTEI.xsl` into a basic XML/TEI document, 
3. and finally stores it in `data/imported`.

## enhance data

`resources/xslt/adlibXMLtoTEI.xsl` does not much more then mapping (some) fields of adlibXML to the TEI (epidoc) data model but doesn't do anything with the actual transcribed texts. To parse the transcribed and partially annotaded texts of the glaser squeezes which can be located at `<div type="edition">`

1. Go to ['Workbench -> Check and Process'](http://localhost:8080/exist/apps/glaser-tei/pages/import-documents-check.html). 
2. There you find an overview of all imported texts and a check, if the annotated transcriptions are valid. 
3. If a text is not valid, report this to the annotators and delete this document (by clicking on **delete**).
4. if a text is valid (this means you can see the transcribed text), then click on **process**. 

This will trigger the script `totei:processDoc` which will

1. store a copy of the selected document and store it in `data/editions`, and
2. transform the 'plain text markup' into XML/TEI valid markup.

Be aware that documents which have already been processed can't be deleted nor processed any more. This is made on purpose to avoid duplicates as well as to keep the data processing workflow replicable.

## check status

The status/progress of the annotation process can be checked at ['Progress -> Check Progress'](http://localhost:8080/exist/apps/glaser-tei/pages/toc.html?collection=imported). This overview lists all **imported documents** and shows, if those documents have been already **processed** or marked as **done**.

## check validity

By browsing to ['Progress -> Valid?'](http://localhost:8080/exist/apps/glaser-tei/pages/validates.html), you can quickly check all documents at any stage of the workflow if they validate against either the TEI or as well as the Epidoc schema. Simply select the schema and collection you would like to check and hit **Check!**. Ideally everything is green. In case a document is