# XMLSTARLET4DH

`xmlstarlet` is a very powerful tool for XML analysis and manipulation in the Shell.

As it's got a rather steep learning curve (and I've had a hard time working through the [official tutorial](http://xmlstar.sourceforge.net/doc/UG/xmlstarlet-ug.html) and the `man` pages), I collect some hopefully useful examples esp. for DH applications (working with TEI-XML, e.g.).


## Get an overview of TEI structure

If you want to explore the tree structure of your XML file, the `el`/`elements` command is your friend:

```
xmlstarlet el yourfile.xml
```

Adding the `-u` option creates a sorted list of unique paths. If you've got a file with 1000 `<l>` elements, the `-u`option just shows the path once.
If you don't want to drill down to the lowest level, you can use `-d<n>` where `n` indicates the depth.
So `xmlstarlet el -d2 yourfile.xml` outputs just the root element (level 1) and its children (level 2).
The `-a` option shows includes the attribute names as well, an `-v` gives you the attribute values.

Applied to a TEI document with `<pb>` elements and `@facs` attributes, this means that

- `-u` gives you the `<pb>` path just once (multiples are collapsed)
- `-a` gives you the `<pb>` path once and another path with the `@facs` attribute
- `-v` gives you the `<pb>` path of every page including the value of the `@facs` attribute.


Note that the four options a mutually exclusive, you can only use one of them.


## Output entire file

<!--
Does not work!

```
xmlstarlet -c 
```
-->

## Select particular elements

You might wish to select elements of interest, e.g. the `<title>` element of an TEI file.
Generally speaking, namespaces are a mess at the beginning.
Fortunately, `xmlstarlet` has a match-all namespace variable which makes it a lot easier to work with standard TEI files where it shouldn't be too bad if you don't distinguish namespaces carefully.

So the following command outputs the value of all `title` elements:

```
xmlstarlet sel -t -v "//_:title" yourfile.xml
```

The `-c` (copy) option instead of the `-v` option gives you the entire xml node (with `<title>...</title>` tags).

If you really want to (or have to) keep things clean and define the namespace(s) properly, you can use the `-N` option (after `sel` and before `-t`).

So in the case of a standard TEI document this command should produce the same output as above:

```
xmlstarlet sel -N xmlns="http://www.tei-c.org/ns/1.0" -t -v "//xmlns:title" yourfile.xml
```

Note that `xmlns` is often used as the standard namespace variable in TEI, but you can choose whichever string you'd like.

If you want to define more than one namespaces you can use `-N string=value` repeatedly.

Other examples: Provided all people involved are mentioned in the `<respStmt>` in a `<name>` tag, you can list them so:

```
xmlstarlet sel -N xmlns="http://www.tei-c.org/ns/1.0" -t -v "//xmlns:respStmt/xmlns:name" yourfile.xml
```

### Select element that contains specific text

```bash
xmlstarlet sel -t -c '//_:title[contains(text(), "sample string")]' <XML_FILE>.xml
```
In order to select nodes that do **not** contain `sample string` use `not()`In order to select nodes that do **not** contain `sample string` use `not()`:

```bash
xmlstarlet sel -t -c '//_:title[not(contains(text(), "sample string")]' <XML_FILE>.xml
```

Selects any `<title>` node whose text contains the string `sample string`. 

Collection of online ressources:
- [Wikipedia with a number of examples](https://en.wikipedia.org/wiki/XMLStarlet)
- [short introduction](https://opensource.com/article/21/7/parse-xml-linux)
- [xmlstarlet on Stackoverflow](https://stackoverflow.com/questions/tagged/xmlstarlet)


## Replace the content of a specific node in a number of xml-files

Now, I would like to update the <date> node in a bunch of xml files.

First, you can check if you got the path right by using the `sel` tool:

```
xmlstarlet sel -t -v "path/to/not" *.xml
```

If you're confident that you've addressed the node you wanted to you can do:

```
xmlstarlet edit --update "//_:publicationStmt/_:date" --value "new content of node" *.xml
```

This outputs the modified xml-files to STDOUT.
You can change the actual files directly by adding the `-L` option:

```
xmlstarlet edit -L --update "//_:publicationStmt/_:date" --value "new content of node" *.xml
```

(After <https://stackoverflow.com/questions/46390426/xmlstarlet-replace-xml-node-value>)

Now let's do something more complex:
I would like to add a new nested node to my xml.

[This Stackoverflow answer](https://stackoverflow.com/questions/71871258/xmlstarlet-add-element-with-namespace-and-attributes) was very helpful as it hinted at the `$prev' variable which came in handy for this use case.

```
xmlstarlet ed -s "//_:titleStmt[last()]" -t elem -n "respStmt" -s '$prev' -t elem -n "name" -v "Jane Doe" -a '$prev' -t elem -n "resp" -v "done important work" *.xml
```

This command adds a subnode (`-s`) of type 'element' to the last `<titleStmt>` element.
Then it takes the element that has just been created (`$prev`) and adds a `<name>` element with value "Jane Doe".
Finally, it appends to the `<name>` element a `<resp>`element with value "doen important work"

## Deal with namespaces

The 'quick'n'dirty' way of dealing with namespaces is to just use the placeholder `_:/<tag>`.

The cleaner way is to explicitly register the the namespace using the `-N` option:

```
xmlstarlet sel -N tei="http://www.tei-c.org/ns/1.0" -t -v "//tei:title" XMLFILE.xml
```

## Select multiple values

Often you will wish to print multiple children of a node.


```
xmlstarlet sel -t -m "//_:editor -v "_:forename"
```

The following command selects the `editor/persName` node, prints the `surname` child an the `ref` attribute of `persName`.

```
xmlstarlet sel -t -m "//_:editor/_:persName" --nl -v "_:surname" -v "@ref" XMLFILE.xml
```

Putting `--nl` in front of a `-v' (print value) option insert a newline after the value.


Create list of `name` elements with `@type` attribute:

```
curl "http://mateo.uni-mannheim.de/camena/bald1/books/baldepoemata1_1.xml" | xmlstarlet sel -t -m '//name' -v '.' -o "," -v '@type' --nl
```


