# Understanding QEWD-JSdb with the REPL Explorer
 
Rob Tweed <rtweed@mgateway.com>  
11 December 2020, M/Gateway Developments Ltd [http://www.mgateway.com](http://www.mgateway.com)  

Twitter: @rtweed

Google Group for discussions, support, advice etc: [http://groups.google.co.uk/group/enterprise-web-developer-community](http://groups.google.co.uk/group/enterprise-web-developer-community)

# About this Document

This document provides a detailed Tutorial on the fundamentals of QEWD-JSdb.

It explains how to start the Node.js REPL Explorer environment for QEWD-JSdb, within
which you'll be able to try out its abstraction of your YottaDB database for yourself.

Once you've completed this tutorial, you'll be able to understand and appreciate
how the other database models supported by QEWD-JSdb work and have been constructed,
and, if you wish, you'll be armed with the information you'll need to develop your
own custom database models.

# Getting Started

One of the things that has been included in your QEWD Installation directory is a file
named *jsdb_shell.js*.  This is a Node.js module that allows you to access and use
the QEWD-JSdb abstraction of your connected YottaDB database, interactively from within the
Node.js REPL (ie the Node.js interactive shell).

Using QEWD-JSdb in this way is a great way to familiarise yourself with how it works, and
provides a *playground* where you can try things out during REST API development or interactive
message handler development.

## Start the Node.js REPL

### Native Installations of QEWD

- Start a Terminal or Command Prompt session

- Make sure you are in your QEWD Installation Directory, eg:


  - Linux/Raspberry Pi:

        cd ~/qewd


Then start the Node.js REPL:

        node

You should see something like this:

        Welcome to Node.js v14.5.1.
        Type ".help" for more information.
        >


### Dockerised Versions of QEWD

On the host system on which you're running the QEWD Docker Container, 
in a Terminal process, shell into the Container:

        docker exec -it qewd bash

You'll find yourself in the */opt/qewd* directory.  Change to the
*opt/qewd/mapped* directory:

        cd mapped

Then start the Node.js REPL:

        node

You should see something like this:

        Welcome to Node.js v14.5.1.
        Type ".help" for more information.
        >



## Load the *QEWD-JSdb Shell* Module

Within the Node.js REPL, type:


        var jsdb = require('./jsdb_shell')

You now have a connection set up with YottaDB, using the *mg-dbx* interface.
This is exactly the same connection that is established automatically in QEWD's
Worker Processes.  However, you now have access to it within the REPL process, so you can
start interactively accessing your YottaDB database via the *QEWD-JSdb* APIs.

The *jsdb* object that you just created provides you with access to those QEWD-JSdb APIs.

The tutorial that follows will teach you how to use the QEWD-JSdb APIs!


## Exiting the Node.js REPL

You can exit the Node.js REPL in the usual way, by pressing the CTRL & C keys twice.


----


# Getting Started with QEWD-JSdb

You're now ready to try something out.  In the REPL, type the following:

        var doc = jsdb.use('demo', 'x')
        doc.value = 123

## Check What's Happened in YottaDB

Let's take a look what's happened in YottaDB:

### Natively Installed YottaDB

If you're using a natively-installed
version of YottaDB with QEWD, in another Linux process, type the *ydb* command to 
open the YottaDB interactive shell.  You should see the YottaDB prompt:

        YDB>

Now take a look at the *^demo* Global:

        YDB> zwr ^demo
         
        
        ^demo("x")=123

Note: to exit the YottaDB shell, type *h* followed by the *Return* key.

### QEWD Docker Container

If you're using the QEWD Docker Container, in another Linux process, shell into the
Container.  For example, if the Container was started with a *name* of *qewd*:

        docker exec -it qewd bash


You'll find yourself in the Container's */opt/qewd* directory.  Type the command:

        ./ydb

You should see the YottaDB prompt:

        YDB>

Now take a look at the *^demo* Global:

        YDB> zwr ^demo
         
        
        ^demo("x")=123

Note: to exit the YottaDB shell, type *h* followed by the *Return* key.


## Back to the REPL:


So now try typing this in the REPL:

        doc.delete()

and if you check for the *^demo* Global in YottaDB, you'll see that it has disappeared.

Clearly, we've been able to set an YottaDB Global Node and then delete it again, all via
a set of JavaScript commands.  So let's look in more detail at what QEWD-JSdb is all about.


## The Document Node Object

The first thing we typed was this line:

        var doc = jsdb.use('demo', 'x')

So let's explain what that is all about.

The *use()* method instantiates what, in QEWD-JSdb, we refer to as a *Document Node Object*.  It
provides an object that represents a physical Global node.  It's your fundamental starting point
for QEWD-JSdb.

The *use()* method's arguments are:

- 1st argument: the physical Global name (without any prefixed ^)
- optionally the 2nd and subsequent arguments specify the subscripts for the Global Node

Note that the *use()* method does NOT create a corresponding physical Global node.  However, the
Document Node object that it returns provides you with a range of properties and methods that
allow you to manipulate that phsyical Global Node and also to navigate from it to any other related
Global Nodes.

### Document Node Object Properties

The properties of a Document Node Object are:

- **exists**: Does the corresponding Global Node physically exists in the database? (*true* | *false*)
- **hasValue**: Is the corresponding Global Node a leaf node with a value? (*true* | *false*)
- **hasChildren**: Is the corresponding Global Node an intermediate node within the
hierarchy? (*true* | *false*)

- **value**: a read/write property that either sets a value for the Document Node, or returns its value.  If
the Node does not exist, an empty string value is returned.  Note that it is quite valid for a Global
Node to also have an empty string as a value.

- **parent**: returns a Document Node Object representing the hierarchical parent of the current Document Node.
- **firstChild**: returns a Document Node Object representing the first child Node (if it exists) of the current Document Node.
- **lastChild**: returns a Document Node Object representing the last child Node (if it exists) of the current Document Node.
- **nextSibling**: returns a Document Node Object representing the current Document Node's next sibling node (if it exists).
- **previousSibling**: returns a Document Node Object representing the current Document Node's previous sibling node (if it exists).

- **path**: returns an array of the subscripts for this Document Node
- **documentName**: returns the documentName, aka the corresponding Global name
- **name**: the Node's property name, ie the last subscript for this Node


### Document Node Object Methods

The methods of a Document Node Object are:

- **delete()**: deletes the corresponding Global Node from the physical database.  The Document Node Object will, however, continue to exist
- **increment(*by*)**: increments the value of the Document Node by the amount specified in the 
argument (default = 1).  Useful for counters or automatic assignment of identifiers
- **countChildren()**: returns the number of Child Document Nodes that physically exist in the database below the current Document Node
- **$(*subscript*)** or **$(*arrayOfSubscripts*)**: returns a new Document Node object that represents the additional level of 
Global Node subscripting below the current Node
- **getDocument(*useArrays*)**: retrieves the data represented in the sub-tree starting with the 
current Document Node as a JSON document.  If *useArrays* is *true*, it will attempt to retrieve 
arrays where consecutive numeric subsript values exist in the physical Global storage.  Unless you're 
sure that the stored data did not include arrays, it is best to use *true*.  However, *getDocument(false)* 
is faster to execute, but will return numeric subscript as properties in the returned JSON.
- **setDocument(*object*)**: Stores the specified JavaScript object/JSON as a sub-tree of Global Nodes beneath the
current Document Node.
- **forEachChild()**: allows you to iterate through the child subscripts of the current Document Node.
- **forEachleafNode**: allows you to iterate through any leaf nodes that are descendents of the current
Document Node.
- **lock(*timeout*)**: attempts to set a lock on the Document Node.  If unable to within the specified timeout (seconds),
it returns *false*, otherwise *true* if successful.  Once locked, any other process attempting to apply a 
lock() to the same Document Node will be unsuccessful.
- **unlock()**: removes a previously set lock on the Document Node.



# Time to Continue Exploring!

Now that we've run through the underlying concepts and documented the various method and properties you
can use in QEWD-JSdb, we can begin exploring it in earnest.

Return, if necessary, to the section above that summarises the properties and methods of
the Document Node Object.  They provide the basic building blocks for *everything* else in
QEWD-JSdb.  You'll see how most of the work and are used throughout the rest of this tutorial.

So back to the REPL.

So let's start with that first node again:

        doc.value = 123

When you looked in YottaDB, you saw that this had caused a Global Node to be stored in the database:

        ^demo("x")=123

That's because *doc* is a Document Node Object we defined using:

 
        var doc = jsdb.use('demo', 'x')

which meant a Document Object Node representing a physical Global named *demo* with a single
subscript: *x*

We then applied its *value* *setter* property to assign the value *123*

In the REPL, we can now retrieve its value from the database using *value* as a *getter* property:

        console.log(doc.value)

and you'll see *123* displayed

You can change its value to something else:

        doc.value="abcdef"

and you'll see that new value appearing in the *^demo* Global in YottaDB.


Now let's create another Document Node Object within the *^demo* Global.  We can do that in several ways:

- using an absolute reference with use();
- using a reference relative to the current Document Node using the $() function.


## Absolute References for instantiating Document Node Objects

Try the first way:

        var doc2 = jsdb.use('demo', 'y', 'z')
        doc2.value = "xyz"

In YottaDB, you'll now see the following Global Nodes:

        ^demo("x")="abcdef"
        ^demo("y","z")="xyz"

Now let's create a 
Document Node object that represents the top of the ^demo Global Node hierarchy - ie just using the Global
name and no subscripts:

        var topDoc = jsdb.use('demo')

Now try this:

        var obj = topDoc.getDocument()

and display the object *obj* this created by using, for example:

        console.log(JSON.stringify(obj, null, 2))

You should see:

        {
          "x": "abcdef",
          "y": {
            "z": "xyz"
          }
        }

So what was returned was a JavaScript object corresponding to the data we saved into those Global Storage
nodes, so, for example, we could now refer to:

        console.log(obj.y.z)

and get the value *xyz*.



## Relative References for instantiating Document Node Objects

Very often, having instantiated one Document Node Object, we'll want to access or manipulate
other nodes that are descendents of it.

For example, we used this above:

        var doc2 = jsdb.use('demo', 'y', 'z')

Let's first delete its value:

        doc2.delete()

and now we'll instantiate a child Document Node:

        var child1 = doc2.$('child1')

Now if you set a value for this, see what happens in the *viewer*:

        child1.value = 'hello world'

Now you'll see this was created in YottaDB:

        ^demo("y","z","child1")="hello world"

Of course, we could have achieved exactly the same thing using the absolute reference approach:

        var child1 = jsdb.use('demo', 'y', 'z', 'child1')

Which you use is up to you, but the relative technique using $() is very convenient, particularly
if you're repetitively creating child Nodes of a parent Node. So, for example, we can now create another child:

        var child2 = doc2.$('child2')
        child2.value = "another world"

and let's create one more:

        var child3 = doc2.$('child3')
        child3.value = "last world"

Check in YottaDB and you should now see the following in the *^demo* Global, eg:

        YDB> zwr ^demo

        ^demo("x")="abcdef"
        ^demo("y","z","child1")="hello world"
        ^demo("y","z","child2")="another world"
        ^demo("y","z","child3")="last world"


and in the REPL, QEWD-JSdb can provide the same information in terms of JSON:

        console.log(JSON.stringify(topDoc.getDocument(), null, 2))

        {
          "x": "abcdef",
          "y": {
            "z": {
              "child1": "hello world",
              "child2": "another world",
              "child3": "last world"
            }
          }
        }



# Exploring the Properties of Document Nodes

So now let's go back to *doc2* which represented the Global Node *^demo("y","z")*
 and look at some of its properties:

        console.log(doc2.exists)

That's true because, although no data physically exists at that Node - ie it's an intermediate, rather
than a leaf node, a value does physically exist in the database for at least one of its descendents.

How about:

        console.log(doc2.hasValue)

That's *false* because it's an intermediate node with no actual data of its own.

Then try:

        console.log(doc2.hasChildren)

That is *true* because it's an intermediate Node, and at least one of its Child Nodes has a
value physically stored in the database.

Compare those results with what you see for a leaf node such as child3:

        console.log(child3.exists)
        console.log(child3.hasValue)
        console.log(child3.hasChildren)


## Parent, Child and Sibling Properties

If we take the *child3* Node, which represents ^demo("y","z","child3"), then see what happens
if we do this:

        console.log(JSON.stringify(child3.parent.getDocument(), null, 2))


You should see:

        {
          "child1": "hello world",
          "child2": "another world",
          "child3": "last world"
        }


Because child3.parent represents ^demo("y","z"), the previous level up the hieararchy.

Of course we could further chain up the parents:

        console.log(JSON.stringify(child3.parent.parent.getDocument(), null, 2))

and now we see this:

        {
          "z": {
            "child1": "hello world",
            "child2": "another world",
            "child3": "last world"
          }
        }


Now let's start with *doc2* which, if you remember, represented ^demo("y","z")

        console.log(doc2.firstChild.value)


This returns "hello world" as the first child node of doc2 represents ^demo("y","z","child1")


        console.log(doc2.lastChild.value)


This returns "last world" as the last child node of doc2 represents ^demo("y","z","child3")


        console.log(doc2.firstChild.nextSibling.value)


This returns "another world" because:

- doc2.firstChild represents ^demo("y","z","child1")
- and its nextSibling Node represents ^demo("y","z","child2")


Hopefully you can understand why we get the same result if we do this:

        console.log(doc2.lastChild.previousSibling.value)


By the way, if, rather than getting the value of a node, you want to know the property name (aka
Global Subscript name), you can use the Node's *name* property, eg:


        console.log(doc2.lastChild.name)

will return *child3*, because this is the node's property name


# Saving JSON in a Document Node

So far we've seen how you can assign a value to a single Document Node.  However, you can
save entire Javacript objects / JSON documents into a Document Node with the single
command *setDocument()*.  It begins to show the real power of this database, so let's take
a look at it in operation:

We'll create a new Document Node:

        var jsonNode = jsdb.use('demo', 'json')

Now let's create a JavaScript Object, eg:

        var obj = {hello: {world: 123, there: 'xyz', you: 'abc123'}}

So that's an in-memory JavaScript object.  Now we'll persist it into that
Document Node we instantiated:

        jsonNode.setDocument(obj)


In YottaDB you'll see this has been created:

        ^demo("json","hello","there")="xyz"
        ^demo("json","hello","world")=123
        ^demo("json","hello","you")="abc123"

and these Global Nodes have been added to the data that was already in the *^demo* Global.

So that previously in-memory JSON object is now also physically in the database.

Of course you can retrieve it again as an in-memory object using getDocument():


        var objCopy = jsonNode.getDocument()

So now you can get to the same values either via in-memory:

        console.log(objCopy.hello.world)

or directly from disk (ie from the database):

        console.log(jsonNode.$('hello').$('world').value)

Note that instead of using two chained $() methods, we could alternatively have done this:

        console.log(jsonNode.$(['hello', 'world']).value)
 
which is less verbose and also more efficient as it has instantiated just one resulting
Document Node object rather than two (one for each $() invocation).


Also one further little trick to be aware of: when you invoke the $() method for a specific
node value, a shortcut property is created in the parent Document Node.  This 
shortcut property name is created from $ followed by the property name.

So having done this:

        console.log(jsonNode.$('hello').$('world').value)

I could now do the same thing using this notation:

        console.log(jsonNode.$hello.$world.value)

demonstrating how much the distinction between in-memory and on-disk JavaScript Objects is now
blurred in QEWD-JSdb!

QEWD-JSdb also demonstrates something else.  Consider a Document Database such as MongoDB which is
designed for the storage of JSON documents.  In MongoDB, the "unit of storage" is an entire
document.  To manipulate or access its contents, you must first fetch the document from the database into
an in-memory JavaScript object before working on it.  If you modify any of its contents, you save back
the modified object to the database.

Now compare with what you've just been doing with QEWD-JSdb.  Sure, just like MongoDB you can save an
in-memory object to disk and retrieve a JSON object from the database as an in-memory instance.  But
your "unit of storage" is now right down to an individual name/value pair, anywhere in the JSON document!
You can view that name/value pair's value *in situ* directly from the database using its Document Node's *value* property.
You can also modify that name/value pair's value *in situ*, directly into the database.

Furthermore, you can work with sub-sets of the JSON, retrieving sub-sections of the hierarchy.  You can
also insert new sub-sections of JSON into the saved document.  And of course you can delete whole
sections of the stored JSON using the *delete()* method.

We've previously seen the *delete()* method applied to a leaf node.  However, if applied to 
an intermediate node, the entire sub-tree of Global Nodes that are descendents of the Node are
deleted too.

Indeed, apply the *delete()* method to the top-level Document Node (ie representing the Global Node alone, 
without any subscripts), and the entire document disappears from the YottaDB database.

So, in summary, QEWD-JSdb provides you with the basis of a uniquely powerful and flexible persistent 
JavaScript Object / JSON document database, quite different from anything else you've probably used before!


# Traversing a QEWD-JSdb Document

A key part of the power of Global Storage is the ability to highly-efficiently traverse your way around
the hierarchical structure.  This ability is provided to you in QEWD-JSdb through two methods available
to a Document Node Object:

- forEachChild(): iterates through the Document Node's child Nodes
- forEachLeafNode(): iterates through the Document Node's descendent leaf nodes.

Let's take a look at both in action.

## forEachChild()

This is the most frequently used method of the two, and once again its apparent simplicitly is highly deceptive.

 Copy and paste the following into your REPL and hit the Enter key:

        jsonNode.$('hello').forEachChild(function(name, node) {
          console.log(name)
        });

You should see the result:

        there
        world
        you

which are the property names (ie the Global subscript values) of each of the child nodes of the jsonNode.$('hello') node

Of course, you can probably guess that we could have written the same logic like this instead:

        jsonNode.$hello.forEachChild(function(name, node) {
          console.log(name)
        });

since we've already previously accessed the *hello* subscript via the *$()* method.

So how and why did we get this result?

Remember, *jsonDemo* represented this physical Global Node:

        ^demo("json")

and *jsonDemo.$('hello') therefore represents:

        ^demo("json","hello")

and so what happened was that the *forEachChild()* function iterated through its child subscripts:

        ^demo("json","hello","there")  <== name = 'there'
        ^demo("json","hello","world")  <== name = 'world'
        ^demo("json","hello","you")    <== name = 'you'

So, that's how to iterate through a Document Node's child node property names, but what if we want the values of those child nodes, or if
we want to perform some manipulation on some or all of the child Document Nodes?

That's where the second argument of the *forEachChild()* method comes in.  It provides you with the Document Node
Object for the child node at each iteration.  So try cuttiing and pasting this variation on the code:

        jsonNode.$hello.forEachChild(function(name, node) {
          console.log(name, ': ' + node.value)
        });

and this time it returns:

        there : xyz
        world : 123
        you : abc123


The *forEachChild()* method can be nested.  We could iterate through the first 3 levels of our document's hiearchy.
If you remember we had earlier instantiated *topDoc* as a Document Node object that represented the top-leve of our
persistent document named *demo*.  So try cutting and pasting the following:


        topDoc.forEachChild(function(name, lvl1Node) {
          if (lvl1Node.hasValue) {
            console.log('level 1: ' + name + ': ' + lvl1Node.value)
          }
          else {
            lvl1Node.forEachChild(function(name, lvl2Node) {   
              if (lvl2Node.hasValue) {
                console.log('  level 2: ' + name + ': ' + lvl2Node.value)
              }
              else {
                lvl2Node.forEachChild(function(name, lvl3Node) {
                  if (lvl3Node.hasValue) {
                    console.log('    level 3: ' + name + ': ' + lvl3Node.value)
                  }
                });
              }
            });
          }
        });


So this time you should see:

            level 3: there: xyz
            level 3: world: 123
            level 3: you: abc123
        level 1: x: abcdef
            level 3: child1: hello world
            level 3: child2: another world
            level 3: child3: last world

and if you look at the physical *^demo* Global in YottaDB, you can see that we've iterated through the entire
Global:

        ^demo("json","hello","there")="xyz"
        ^demo("json","hello","world")=123
        ^demo("json","hello","you")="abc123"
        ^demo("x")="abcdef"
        ^demo("y","z","child1")="hello world"
        ^demo("y","z","child2")="another world"
        ^demo("y","z","child3")="last world"


### forEachChild() Modifiers

You can control and modify the behaviour of the *forEachChild()* method via an optional first argument.

#### Traversal Direction

You can control the direction of the traversal.  By default the direction is *forwards*, but you can change it to 
"reverse", eg:


        jsonNode.$hello.forEachChild({direction: 'reverse'}, function(name, node) {
          console.log(name)
        });

and this time you'll see the child property names returned in reverse order:


        you
        world
        there


#### Ranges

You can specify a range of child property names within which to limit the traversal.  To demonstrate this properly, let's
first add the following data to the *demo* document:

        var data = {
          "james": "",
          "frederick": "",
          "william": "",
          "alan": "",
          "andrew": "",
          "anthony": "",
          "brian": "",
          "brendan": "",
          "billy": "",
          "charles": "",
          "colin": ""
        }
        topDoc.$('names').setDocument(data)

Let's add a few more names using the *value* method:

        topDoc.$names.$('david').value = ''

        topDoc.$names.$('richard').value = ''

        topDoc.$names.$('Graham').value = ''

Of course, if you look in YottaDB, the names, because they're defined as Global subscripts,
 will have been automatically collated alphabetically in the Global:

        YDB> zwr ^demo("names")

        ^demo("names","Graham")=""
        ^demo("names","alan")=""
        ^demo("names","andrew")=""
        ^demo("names","anthony")=""
        ^demo("names","billy")=""
        ^demo("names","brendan")=""
        ^demo("names","brian")=""
        ^demo("names","charles")=""
        ^demo("names","colin")=""
        ^demo("names","david")=""
        ^demo("names","frederick")=""
        ^demo("names","james")=""
        ^demo("names","richard")=""
        ^demo("names","william")=""


We now have some good data with which to demonstrate *forEachChild()* ranges.

We can specify a starting point for the range in which to iterate:

        topDoc.$names.forEachChild({range: {from: 'd'}}, function(name, node) {
          console.log(name)
        });

and you should get back:

        david
        frederick
        james
        richard
        william


We can alternatively specify an endpoint to the range:


        topDoc.$names.forEachChild({range: {to: 'd'}}, function(name, node) {
          console.log(name)
        });

and now you should see:

        Graham
        alan
        andrew
        anthony
        billy
        brendan
        brian
        charles
        colin
        david

Naturally they can be combined.  Try this:


        topDoc.$names.forEachChild({range: {from: 'b', to: 'd'}}, function(name, node) {
          console.log(name)
        });

You should now just get these names back:

        billy
        brendan
        brian
        charles
        colin
        david

You're not restricted to a single letter for the *from* and *to* values.  Try this:

        topDoc.$names.forEachChild({range: {from: 'br', to: 'col'}}, function(name, node) {
          console.log(name)
        });

Now we just see:


      brendan
      brian
      charles
      colin

If a *from* or *to* value isn't present in the actual subscripts, that doesn't matter - they are just seed/terminating
values, so this:

        topDoc.$names.forEachChild({range: {from: 'elvis', to: 'kelly'}}, function(name, node) {
          console.log(name)
        });

will return

        frederick
        james


#### Prefixes


The *from* and *to* values can be the same, in which case all subscript names starting with the specified values are
returned, eg:

        topDoc.$names.forEachChild({range: {from: 'br', to: 'br'}}, function(name, node) {
          console.log(name)
        });


will return

        brendan
        brian


Note that ranges can be specified together with direction:

        topDoc.$names.forEachChild({direction: 'reverse', range: {from: 'br', to: 'br'}}, function(name, node) {
          console.log(name)
        });

which now returns

        brian
        brendan

However, there's a simpler modifier, *prefix* available to achieve the same effect much more succinctly.  Try this:

        topDoc.$names.forEachChild({prefix: 'br'}, function(name, node) {
          console.log(name)
        });

and of course it can be combined with the *direction* modifier:

        topDoc.$names.forEachChild({direction: 'reverse', prefix: 'br'}, function(name, node) {
          console.log(name)
        });


So that's the *forEachChild()* method, which is available for all Document Nodes. Hopefully you can begin to appreciate 
and understand how and why it is a core part of developing with QEWD-JSdb.


## forEachLeafNode()

The *forEachLeafNode()* method is a somewhat less used method, but in certain circumstances it is extremely useful and
can be a lot more efficient than *forEachChild*.

A good use-case is exemplified by reviewing something we did earlier:

        topDoc.forEachChild(function(name, lvl1Node) {
          if (lvl1Node.hasValue) {
            console.log('level 1: ' + name + ': ' + lvl1Node.value)
          }
          else {
            lvl1Node.forEachChild(function(name, lvl2Node) {   
              if (lvl2Node.hasValue) {
                console.log('  level 2: ' + name + ': ' + lvl2Node.value)
              }
              else {
                lvl2Node.forEachChild(function(name, lvl3Node) {
                  if (lvl3Node.hasValue) {
                    console.log('    level 3: ' + name + ': ' + lvl3Node.value)
                  }
                });
              }
            });
          }
        });

If, what we were intending to achieve with this logic was to locate all the leaf node values in the document, then it
had a couple of key disadvantages:

- we had to know, in advance, that the hierarchy was no more than 3 levels deep.  So this logic isn't generic enough
to keep track of changes within the document that might introduce an unknown number of additional levels of hierarchy.

- we had to "walk" the entire list of subscripts at each level to get to the leaf nodes.  With the data that existed
at the time we originally ran it, ie:

        ^demo("json","hello","there")="xyz"
        ^demo("json","hello","world")=123
        ^demo("json","hello","you")="abc123"
        ^demo("x")="abcdef"
        ^demo("y","z","child1")="hello world"
        ^demo("y","z","child2")="another world"
        ^demo("y","z","child3")="last world"

That involved: 

- 3 iterations through level 1
- 2 iterations through level 2
- 6 iterations through level 3

So a total of 11 iteration steps to locate 7 leaf nodes.

In a big hierarchy with lots of intermediate levels and leaf nodes, this approach could be very wasteful, if all we
want to get to is the leaf nodes.

So here's how we could rewrite that same logic to address those two deficiencies of nested *forEachChild()* methods:

        topDoc.forEachLeafNode(function(value, node) {
          console.log(node.path + ': ' + value);
        });

This now returns:

        json,hello,there: xyz
        json,hello,world: 123
        json,hello,you: abc123
        names,Graham:
        names,alan:
        names,andrew:
        names,anthony:
        names,billy:
        names,brendan:
        names,brian:
        names,charles:
        names,colin:
        names,david:
        names,frederick:
        names,james:
        names,richard:
        names,william:
        x: 123
        y,z,child1: hello world
        y,z,child2: another world
        y,z,child3: last world


As you can see, the argument for the *forEachLeafNode()* method is a callback function which provides two arguments at
each iteration:

- value: the value of the leaf node
- node: a Document Node object representing the leaf node

The *forEachLeafNode()* method returns all the leaf nodes that are descendents of the Document Node Object to which it
is being applied, so we can limit the iterations to just part of the hierarchy, eg try this:


        topDoc.$('y').forEachLeafNode(function(value, node) {
          console.log(node.path + ': ' + value);
        });


Now, you just see this, the leaf nodes below the *y* Document Node:

        y,z,child1: hello world
        y,z,child2: another world
        y,z,child3: last world


Now remember, in all of the examples you've seen so far, everything you're seeing is being applied to data on
disk, in the database, not to in-memory JavaScript objects!



# Handling JavaScript Arrays in Global Storage

The very simple rules of how Global Storage works that we described at the beginning of this document do not
explicitly accommodate JavaScript arrays.  Instead we, ourselves, need to create and apply a convention through which they can be
accommodated.  The convention applied by QEWD-JSdb is to represent the elements of an array using integer values for
subscripts: the values match the JavaScript array element index value.

It's pretty simple once you see in action.  Try this:

        var arr = ["element 1", "element 2", "element 3", "element 4", "element 5"]
        topDoc.$('array').setDocument(arr)

Take a look at the *^demo* Global in YottaDB and you'll see how it's stored the array:

        zwr ^demo("array")

        ^demo("array",0)="element 1"
        ^demo("array",1)="element 2"
        ^demo("array",2)="element 3"
        ^demo("array",3)="element 4"
        ^demo("array",4)="element 5"


Now try this in the REPL:

        var arrCopy = topDoc.$array.getDocument()
        console.log(JSON.stringify(arrCopy, null, 2))

This is a surprise!  It's returned it as an object:

        {
          "0": "element 1",
          "1": "element 2",
          "2": "element 3",
          "3": "element 4",
          "4": "element 5"
        }


Actually if you think about it, it's not so much of a surprise, because we're applying an implicit abstraction for arrays,
and it results in a potential ambiguity: how can QEWD-JSdb distinguish between an array and a numerically-keyed object?

As you can see from the *getDocument()* method, by default it can't cater for the ambiguity and falls back on the 
assumption that everything in Global storage is an object.

However, now try this:

        var arrCopy2 = topDoc.$array.getDocument(true)
        console.log(JSON.stringify(arrCopy2, null, 2))

This time it successfully returns the array!

        [
          "element 1",
          "element 2",
          "element 3",
          "element 4",
          "element 5"
        ]

The optional first argument of the *getDocument()* method, if set to *true*, tells the method to scan each level of
subscripting.  If it sees only a consecutive, unbroken sequence of integers, starting with 0, then it assumes the
data represents an array.  If not, it must be an object at this level in the hierarchy.

The rule is applied at all levels in its traversal through a document.  Try applying this to the document:

        var obj = {abc: 'xyz', arr: [1,2,3, {x: "zxc", z: [5,6,7]}, 8], q: "ggg"}
        topDoc.$array.$(5).setDocument(obj)

You'll see that this mixture of arrays and objects has been added as a new last element of our original array,
and is saved in Global Storage thus:

        ^demo("array",5,"abc")="xyz"
        ^demo("array",5,"arr",0)=1
        ^demo("array",5,"arr",1)=2
        ^demo("array",5,"arr",2)=3
        ^demo("array",5,"arr",3,"x")="zxc"
        ^demo("array",5,"arr",3,"z",0)=5
        ^demo("array",5,"arr",3,"z",1)=6
        ^demo("array",5,"arr",3,"z",2)=7
        ^demo("array",5,"arr",4)=8
        ^demo("array",5,"q")="ggg"


Try retrieving the new expanded array:

        var arrCopy2 = topDoc.$array.getDocument(true)
        console.log(JSON.stringify(arrCopy2, null, 2))


and back it comes as expected:

        [
          "element 1",
          "element 2",
          "element 3",
          "element 4",
          "element 5",
          {
            "abc": "xyz",
            "arr": [
              1,
              2,
              3,
              {
                "x": "zxc",
                "z": [
                  5,
                  6,
                  7
                ]
              },
              8
            ],
            "q": "ggg"
          }
        ]


There is a cautionary note that is worth bringing to your attention.  Applying this implicit abstraction for
arrays comes at a cost.  The cost won't be significant for most small to medium-sized saved QEWD-JSdb objects.
However it will increase in significance as your saved documents get larger.  There are actually two
implications of using the *true* argument to ensure that you get arrays properly returned:

- to do so, the *getDocument()* logic has to recursively call *forEachChild()* methods to exhaustively walk the entire tree
via its subscripts;
- at every iteration of each *forEachChild()* method, at every level of the hierarchy, the logic has to spend time
inspecting the subscript values to see whether they start at zero and continue as an unbroken sequence of 
consecutive integers.  This gets costly if there are lots of huge arrays containing lots of members.

By comparison, if you don't add the *true* argument, *getDocument()* uses the more efficient *forEachLeafNode()* method 
to walk the hierarchy, and applies no subscript inspection.

You can see the [*getDocument()* implementation logic here](https://github.com/robtweed/ewd-document-store/blob/master/lib/proto/getDocument.js) 
, if you're interested to see how it actually works.


Just one further way to demonstrate how this array abstraction is applied.  Let's deliberately break the consecutive
sequence of integer subscript values in our original array by doing this:


        topDoc.$array.$('no_longer_an_array').value = 'This should change things!'

In YottaDB you'll see:

        ^demo("array",0)="element 1"
        ^demo("array",1)="element 2"
        ^demo("array",2)="element 3"
        ^demo("array",3)="element 4"
        ^demo("array",4)="element 5"
        ^demo("array",5,"abc")="xyz"
        ^demo("array",5,"arr",0)=1
        ^demo("array",5,"arr",1)=2
        ^demo("array",5,"arr",2)=3
        ^demo("array",5,"arr",3,"x")="zxc"
        ^demo("array",5,"arr",3,"z",0)=5
        ^demo("array",5,"arr",3,"z",1)=6
        ^demo("array",5,"arr",3,"z",2)=7
        ^demo("array",5,"arr",4)=8
        ^demo("array",5,"q")="ggg"
        ^demo("array","no_longer_an_array")="This should change things!"

That second subscript is no longer an unbroken sequence of integers, so even if we use the *true* argument,
*setDocument()* will now return it as an object.  Try it for yourself and see:


        var arrCopy3 = topDoc.$array.getDocument(true)
        console.log(JSON.stringify(arrCopy3, null, 2))



# Creating and using Indices in QEWD-JSdb

## What's an Index?

So far, we've focused on the process of getting data in and out of QEWD-JSdb, and navigating within the data structures.

Databases need to be designed so that they can be queried efficiently, and that requires what are known as indices to be
maintained.  Those indices provide a fast means of interogating the database and extracting the specific data records 
requested by the user.

It is searching via indices that sets databases apart from simple files for data storage.

Let's consider a simple example:

Suppose we had a set of records that provide a simple telephone directory.  Each record defines a telephone number,
a first name, last name and town, eg:

        07123 456 789, Rob, Tweed, Redhill
        07712 345 678, Simon, Tweed, St Albans
        07321 987 765, Chris, Munt, Banstead
        07713 473 812, John, Smith, London
        07654 321 123, Susanne, Salling, Redhill
        ... etc

These could be saved into a text file.

If you wanted to find the telephone numbers for everyone with a last name of *Tweed*, you would have to exhaustively
inspect every line of text in the file before you could return the definitive result.  This wouldn't be a big deal if
there were only a few records, even a few hundred records.  But if it contained tens of thousands, or millions of records,
it would become a very time-consuming exercise, that would have to be repeated every time you made a similar query.

By comparison, in a database, we would store two things:

- the core data records, according to some particular model, eg in our case, a set of persistent JSON objects
- a derived set of index records, which, for each last name value, would provide a pointer to the corresponding data record.

So, in order to find all the telephone numbers for everyone with a last name of *Tweed*, the database would start with the
*last name* index for its records containing the value *Tweed*.  Each of these records would provide a pointer directly to
the corresponding data record, and from that record, the telephone number would be returned.

With this approach, there is no need to exhaustively search the entire database.  We just search the specific records
within the index.


## What are the downsides, if any, of Indexing?

Indices come at a cost:

- they need to be scrupulously maintained to correspond to any adds, changes or deletions in the main data records
- they add to the physical disk drive storage consumed by the database

However, in these days of high-performance, low cost CPUs and Hard Disk Drives and Solid State Drives, neither of these are
particularly significant costs, and in return you get near instantaneous retrieval of the records that match your query.
Generally speaking, the performance of databases is determined by the number of indices you maintain in parallel with the
main data records.  The more likely a database record field is to be searched on, the more important it will be to have
an index for it.  And, of course, if queries are constructed across multiple criteria, eg *find me all the phone numbers 
of people named Tweed who live in Redhill*, then the queries can benefit from an index that identifies records by the 
cobination of their *lastName* and *town* values.

## Let's try using a simple Index

So, armed with what we now know about the QEWD-JSdb properties, methods and techniques, let's implement that example telephone directory, create an index for it,
and then demonstrate how a last name query could be carried out.  To begin with we'll do it manually, so you can see
how it can all b made to work.

Let's start with the main data records.  We could do that using a simple JSON structure for each record:

        {
          firstName: 'Rob',
          lastName: 'Tweed',
          town: 'Redhill',
          tel: '07123 456 789'
        }

We could assign each record an otherwise meaningless identifier - a simple incrementing integer would suffice,
and then use this as what, in database terminology, is known as the primary key for the record.  So let's create that
main data structure and save it:

        var data = {
          "1": {
            "firstName": "Rob",
            "lastName": "Tweed",
            "town": "Redhill",
            "tel": "07123 456 789"
          },
          "2": {
            "firstName": "Simon",
            "lastName": "Tweed",
            "town": "St Albans",
            "tel": "07712 345 678"
          },
          "3": {
            "firstName": "Chris",
            "lastName": "Munt",
            "town": "Banstead",
            "tel": "07321 987 765"
          },
          "4": {
            "firstName": "John",
            "lastName": "Smith",
            "town": "London",
            "tel": "07713 473 812"
          },
          "5": {
            "firstName": "Susanne",
            "lastName": "Salling",
            "town": "Redhill",
            "tel": "07654 321 123"
          }
        }

       var telDoc = jsdb.use('telDirectory', 'data')
       telDoc.setDocument(data);


You'll see this in YottaDB, saved as:

        ^telDirectory("data",1,"firstName")="Rob"
        ^telDirectory("data",1,"lastName")="Tweed"
        ^telDirectory("data",1,"tel")="07123 456 789"
        ^telDirectory("data",1,"town")="Redhill"
        ^telDirectory("data",2,"firstName")="Simon"
        ^telDirectory("data",2,"lastName")="Tweed"
        ^telDirectory("data",2,"tel")="07712 345 678"
        ^telDirectory("data",2,"town")="St Albans"
        ^telDirectory("data",3,"firstName")="Chris"
        ^telDirectory("data",3,"lastName")="Munt"
        ^telDirectory("data",3,"tel")="07321 987 765"
        ^telDirectory("data",3,"town")="Banstead"
        ^telDirectory("data",4,"firstName")="John"
        ^telDirectory("data",4,"lastName")="Smith"
        ^telDirectory("data",4,"tel")="07713 473 812"
        ^telDirectory("data",4,"town")="London"
        ^telDirectory("data",5,"firstName")="Susanne"
        ^telDirectory("data",5,"lastName")="Salling"
        ^telDirectory("data",5,"tel")="07654 321 123"
        ^telDirectory("data",5,"town")="Redhill"


Now let's manually add a corresponding set of *lastName* index records.  To make searching easier, we'll lower-case
each last name:

        var index = {
          "tweed": {
            "1": "",
            "2": ""
          },
          "munt": {
            "3": ""
          },
          "smith": {
            "4": ""
          },
          "salling": {
            "5": ""
          }
        }

We'll instantiate a different part of the *telDirectory* document for these index records, eg:

        var indexDoc = jsdb.use('telDirectory', 'index', 'by_lastName')
        indexDoc.setDocument(index);

Look in YottaDB and you'll see that this has resulted in these records being added to the database:

        ^telDirectory("index","by_lastName","munt",3)=""
        ^telDirectory("index","by_lastName","salling",5)=""
        ^telDirectory("index","by_lastName","smith",4)=""
        ^telDirectory("index","by_lastName","tweed",1)=""
        ^telDirectory("index","by_lastName","tweed",2)=""

Notice the first cool thing: that built-in automated subscript collation of the Global Storage database 
has sorted the index in alphabetic order for us!  This is extremely useful for indices, as you'll see.

So how and why these have we defined these index records in this way?  The idea is
that against each lower-cased instance of *lastName*, we define the record id numbers
for the corresponding data records containing those last names.  Notice that in an index record, 
as you'll soon see, it's the property value (eg *tweed*) that is important, and not the index node's data value. So
we'll just assign an empty string value to each index record.


## Using the Index to locate records in the Database

We can now manually perform our search for all the telephone numbers of people with a last name of *tweed*.
Try this out:

        indexDoc.$('tweed').forEachChild(function(id) {
          console.log(telDoc.$([id, 'tel']).value)
        });

And hey presto! Back should come the telephone numbers for the two *tweed* records.

You can hopefully see how simple the task has been made with QEWD-JSdb: simply use 
the *forEachChild()* method to get all the matching record ids,
and, at each iteration, use this id to point to the data record (telDoc.$('id')), 
and finally get the value for its *tel* property.

Remember that we can use the *range* or *prefix* modifiers to make the query more generic, eg:


        indexDoc.forEachChild({prefix: 's'}, function(lastName, node) {
          node.forEachChild(function(id) {
            console.log(lastName + ': ' + telDoc.$([id, 'tel']).value)
          });
        });

will now match on any last name starting with the specified string, in this case *s*:

        salling: 07654 321 123
        smith: 07713 473 812


If we now add a new data record and corresponding index record:

        var data = {
          "6": {
            "firstName": "David",
            "lastName": "Smythe",
            "town": "Redhill",
            "tel": "07613 173 475"
          }
        }
        var index = {
          "smythe": {
            "6": ""
          }
        }
        telDoc.setDocument(data)
        indexDoc.setDocument(index)


Now try the query again:

        indexDoc.forEachChild({prefix: 's'}, function(lastName, node) {
          node.forEachChild(function(id) {
            console.log(lastName + ': ' + telDoc.$([id, 'tel']).value)
          });
        });

It's picked up the new record, because the index record was automatically collated into the correct alphabetic sequence!

        salling: 07654 321 123
        smith: 07713 473 812
        smythe: 07613 173 475


And of course if we now search for last names starting with *sm*:

        indexDoc.forEachChild({prefix: 'sm'}, function(lastName, node) {
          node.forEachChild(function(id) {
            console.log(lastName + ': ' + telDoc.$([id, 'tel']).value)
          });
        });

We now just get these records:

        smith: 07713 473 812
        smythe: 07613 173 475


## Implementing APIs to Maintain Data and Index Records

So far, we've done everything manually.  In the real world, we'd actually want to automate this process:

- create a set of CRUD APIs that allow new data records to be added and existing ones to be changed or deleted
- for each of those APIs to create, change or delete the corresponding index record
- create a generic lastName search API

Let's implement those APIs as functions that we can use in the REPL Explorer.  We'll just
create them as local functions rather than save them as modules.  You can try doing that
yourself later as an exercise.

Copy and paste the following functions into the REPL:

        function saveAndIndex(id, recordObj, jsdb) {
          if (id && id !== '' && recordObj && recordObj.lastName && recordObj.lastName !== '') {
            // set up the Document Node objects we'll need to use
            var telDoc = jsdb.use('telDirectory');
            var dataDoc = telDoc.$('data');
            var indexDoc = telDoc.$(['index', 'by_lastName']);
            recordObj.id = id;
            // save the data record
            dataDoc.$(id).setDocument(recordObj);
            // save the index record
            indexDoc.$([recordObj.lastName.toLowerCase(), id]).value = '';
          }
        }


        function addRecord(recordObj, jsdb) {
          // first check the record object is valid
          if (recordObj && recordObj.lastName && recordObj.lastName !== '') {
            // set up the Document Node objects we'll need to use
            var telDoc = jsdb.use('telDirectory');
            // get the next id (1 will be returned the first time this is invoked)
            var id = telDoc.$('next_id').increment();
            // save and index the new record
            saveAndIndex(id, recordObj, jsdb);
            return id;
          }
        }

The second one above is the API we'll use to add a new record into our telephone directory.  It uses a Document Node
method that we've not seen before: *increment()*.  It's an ideal method for creating identifiers.
We're using a specific Document Node - *(telDoc.$('next_id'))* - as the counter for
getting the next new id each time we add a record.  Watch for this in the *viewer* later when we
try these API functions out.

Now copy and paste the delete API:

        function deleteRecord(id, recordObj, jsdb) {
          // check the incoming record object for validity
          if (recordObj && recordObj.lastName && recordObj.lastName !== '') {
            // set up the Document Node objects we'll need to use
            var telDoc = jsdb.use('telDirectory');
            var dataDoc = telDoc.$('data');
            var indexDoc = telDoc.$(['index', 'by_lastName']);
            //check the incoming id is valid and is already in use
            if (id && id !== '' && dataDoc.$(id).exists) {
              // get the current lastName
              var selectedDoc = dataDoc.$(id);
              var lastName = selectedDoc.$('lastName').value.toLowerCase();
              // delete the corresponding index;
              indexDoc.$([lastName, id]).delete();
              // Now delete the data
              selectedDoc.delete();
              dataDoc.$(id).setDocument(recordObj);
              return true;
            }
          }
        }


And next copy and paste the edit API function:

        function editRecord(id, recordObj, jsdb) {
          var ok = deleteRecord(id, recordObj, jsdb);
          if (ok) {
            // save and index the new data for this record
            saveAndIndex(id, recordObj, jsdb);
          }
        }

Finally we'll add the search API:

        function getByLastName(lastName, jsdb) {
          var nodes = [];
          if (lastName !== '') {
            // set up the Document Node objects we'll need to use
            var telDoc = jsdb.use('telDirectory');
            var dataDoc = telDoc.$('data');
            var indexDoc = telDoc.$(['index', 'by_lastName']);
            indexDoc.forEachChild({prefix: lastName}, function(ix, node) {
              node.forEachChild(function(id) {
                nodes.push(dataDoc.$(id).getDocument());
              });
            });
          }
          return nodes;
        }


OK let's clear down the data we'd previously saved in our telephone directory:

        var telDoc = jsdb.use('telDirectory');
        telDoc.delete();


OK let's add a first record using these APIs:

        var record = {
          firstName: 'Rob',
          lastName: 'Tweed',
          town: 'Redhill',
          tel: '07123 456 789'
        }
        addRecord(record, jsdb)

Check in YottaDB.  You should see that it's saved it as:

        ^telDirectory("data",1,"firstName")="Rob"
        ^telDirectory("data",1,"id")=1
        ^telDirectory("data",1,"lastName")="Tweed"
        ^telDirectory("data",1,"tel")="07123 456 789"
        ^telDirectory("data",1,"town")="Redhill"
        ^telDirectory("index","by_lastName","tweed",1)=""
        ^telDirectory("next_id")=1


Let's add a second record:

        record = {
          firstName: 'Simon',
          lastName: 'Tweed',
          town: 'St Albans',
          tel: '07712 345 678'
        }
        addRecord(record, jsdb)

Look in YottaDB and you'll see that an *id* of 2 was assigned to this, and the *next_id*
node was incremented by 1:

        ^telDirectory("next_id")=2

Let's try a search:

        var nodes = getByLastName('tweed', jsdb)

What's been returned is an array of matching data records.  Type:

        nodes

and you'll see the records.  Do these look correct?  Hopefully they should!

Next, try editing a record, for example:

        record = {
          firstName: 'Simon',
          lastName: 'Smith',
          town: 'St Albans',
          tel: '07712 345 678'
        }
        editRecord(2, record, jsdb)

If you look in YottaDB, you should see that the data record has correctly changed:

        ^telDirectory("data",2,"lastName")="Smith"

and the index records have been appropriately changed too:

        ^telDirectory("index","by_lastName","smith",2)=""
        ^telDirectory("index","by_lastName","tweed",1)=""


In order to edit that record, we had to know that its *id* was 2.  This isn't something
we'd usually have known, so how would we have found that out?

A simple solution is provided by the *saveAndIndex* function adding the *id* to the data
record, eg:

        ^telDirectory("data",2,"id")=2

This might appear to be a redundant record, but remember when we ran the query, it returned the data values:


        > nodes
        [
          {
            firstName: 'Rob',
            id: 1,
            lastName: 'Tweed',
            tel: '07123 456 789',
            town: 'Redhill'
          },
          {
            firstName: 'Simon',
            id: 2,
            lastName: 'Tweed',
            tel: '07712 345 678',
            town: 'St Albans'
          }
        ]

So we could have retrieved the *id* as nodes[1].id, having first found the record using a search.


Anyway, having edited that telephone directory record, let's try our name search again:


        nodes = getByLastName('tweed', jsdb)

Now it should return just one matching record in the array.  And then try:

        nodes = getByLastName('smith', jsdb)


# Building Database Models with QEWD-JSdb

The APIs we've created above have effectively implemented a simple database model.  In
this case, quite deliberately, it's a very restricted use-case with quite limited
capabilities, but it hopefully demonstrates the basic techniques you can use to
build higher-level database functionality using the basic buidling-block properties and
methods provided by QEWD-JSdb.

QEWD-JSdb includes a number of pre-implemented data models, all of which are just really
variations on the themes we've explored in this tutorial.  Of course, the source logic for each
of them is available for you to inspect and learn from.

So, in addition to the basic QEWD-JSdb persistent JSON/JavaScript Obect and Document Database
capabilities, QEWD-JSdb includes the following pre-implemented database models

- [lists](./LISTS.md): a Redis-like List database
- [kvs](./KVS.md): a Redis-like Key Value store
- [dom](./DOM.md): a persistent XML DOM implementation, complete with XPath support for querying it

These are examined in more detail in separate documents (use the links above).

You can see that QEWD-JSdb is therefore not just a "one trick pony" database, limited to just
one database model.  It's a fully-functional multi-model database abstraction of YottaDB.

Enjoy your further exploration of QEWD-JSdb!
