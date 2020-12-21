# The QEWD-JSdb DOM Database Model
 
Rob Tweed <rtweed@mgateway.com>  
15 December 2020, M/Gateway Developments Ltd [http://www.mgateway.com](http://www.mgateway.com)  

Twitter: @rtweed

Google Group for discussions, support, advice etc: [http://groups.google.co.uk/group/enterprise-web-developer-community](http://groups.google.co.uk/group/enterprise-web-developer-community)

# About this Document

This document provides background information on the *Persistent XML Document Object Model (DOM)*
 NoSQL database model that is included with QEWD-JSdb.

#Index

- [About the DOM Database Model](#about-the-dom-database-model)
- [Source Code for the DOM APIs](#source-code-for-the-dom-apis)
- [Enabling Use of the DOM APIs](#enabling-use-of-the-dom-apis)
- [Building a DOM Programmatically](#building-a-dom-programmatically)
  - [Getting Started](#getting-started)
  - [The documentNode](#the-documentnode)
  - [Adding an XML Tag](#adding-an-xml-tag)
    - [Low-Level APIs](#low-level-apis)
      - [createElement()](#createelement)
      - [appendChild()](#appendchild)
      - [Add Another Tag](#addanothertag)
      - [setAttribute()](#setattribute)
      - [textContent](#textcontent)
    - [High-Level Method for Tag Creation](#high-level-method-for-tag-creation)
  - [The Persistent DOM](#the-persistent-dom)
  - [Navigating the DOM](#navigating-the-dom)
    - [Navigating Between Adjacent Elements](#navigating-between-adjacent-elements)
    - [*getElementsByTagName()*](#getelementsbytagname)
    - [*getElementById()*](#getelementbyid)
  - [Interrogating and Manipulating the DOM](#interrogating-and-manipulating-the-dom)
    - [Child Elements and Child Nodes](#child-elements-and-child-nodes)
    - [Attributes](#attributes)
      - [*hasAttributes()*](#hasattributes)
      - [The *attributes* Property](#the-attributes-property)
      - [Using *setAttribute()* to Set and Change Attribute Values](#using-setattribute-to-set-and-change-attribute-values)
      - [*getAttributes()*](#getattributes)
      - [Removing Attributes](#removing-attributes)
  - [Inserting Elements](#inserting-elements)
    - [*insertBefore()*](#insertbefore)
    - [*insertBeforeChildren()*](#insertbeforechildren)
  - [Removing Elements](#removing-elements)
    - [*removeChild()*](#removechild)
    - [*removeAsParent()*](#removeasparent)
  - [Handling Text](#handling-text)
  - [Other XML Nodes](#other-xml-nodes)
    - [Processing Instruction](#processing-instruction)
    - [Document Type Definition](#document-type-definition)
    - [Comments](#comments)
    - [CDATA Section](#cdata-section)
- [Ingesting XML](#ingesting-xml)
- [Querying DOMs using XPath](#querying-doms-using-xpath)
- [Extending DOMs with *UserData*](#extending-doms-with-userdata)
- [Using JSON with the DOM](#using-json-with-the-dom)
- [Conclusions](#conclusions)


# About the DOM Database Model

The *DOM* database model is an implementation of the [W3C XML DOM API](https://www.w3.org/TR/2000/REC-DOM-Level-2-Core-20001113/introduction.html).
The implementation is currently somewhere between W3C's Levels 2 and 3 in terms of its standards adherence,
but is sufficiently complete to integrate correctly with the 3rd-party [Node.js XPath module](https://www.npmjs.com/package/xpath)
which is a dependency of the DOM API implementation. 

Unlike the usual XML DOM implementations, it is implemented in the persistent storage provided
by QEWD-JSdb, rather than in-memory.  This means that the documents can be stored as pre-parsed
XML DOMs, and can be searched at any time in-situ within the database using standard XPath queries.

In effect, the QEWD-JSdb *DOM* model provides you with a [Native XML Database](http://www.rpbourret.com/xml/XMLAndDatabases.htm).
One of the most well-known of such products is [MarkLogic](https://en.wikipedia.org/wiki/MarkLogic) which
has evolved from its original XML database roots.

A good resource for learning about the XML DOM APIs, how they work and how they can be used is the
[W3Schools XML DOM Tutorial](https://www.w3schools.com/xml/dom_intro.asp).  The only difference is that
the examples in those tutorials will be working with in-memory XML DOMs, whereas you'll be using
DOMs in database storage.


Before proceeding it is recommended that you first read and complete the [basic QEWD-JSdb tutorial](./REPL.md).  This will ensure you 
understand what's going on in the QEWD-JSdb and why!


# Source Code for the DOM APIs

Like all the QEWD-JSdb APIs, they are written in JavaScript and are built on top of the
standard QEWD-JSdb APIs.  They are all Open Source APIs, and
you are free to inspect and use the code as you wish, in accordance with the Apache 2 license under
which they are made available.

Find the [DOM source code here](https://github.com/robtweed/ewd-document-store/tree/master/lib/proto/dom).


# Enabling Use of the DOM APIs

You can create and maintain a *DOM* at any QEWD-JSdb Document Node Object.

Having instantiated the Document Object Node object, you must enable its use of the *DOM* APIs.
For example, from within a QEWD-Up back-end message handler module:

        doc = this.documentStore.use('jsdbDom', 'demo')
        doc.enable_dom()

or when using the *jsdb_shell* REPL module:

        doc = jsdb.use('jsdbDom', 'demo')
        doc.enable_dom()

The QEWD-JSdb Document Node object will be augmented with a *dom* property object, 
via which you can invoke the *DOM* APIs, eg:


        documentNode = doc.dom.createDocument();

The DOM APis allow you to:

- build and manipulate a DOM from scratch, entirely using the APIs
- ingest the text of an XML document and convert it to a DOM, after which it can be manipulated using
the DOM APIs
- output a DOM as XML text which, for example, can then be saved as a file

The DOM APIs can also be used with JSON.  For example, you can:

- ingest JSON and represent it as a DOM, after which it can be manipulated using
the DOM APIs
- output a DOM as a corresponding JSON text which can then be saved as a file.



# Building a DOM Programmatically

You can build a DOM, representing an XML Document, completly from scratch, using the DOM APIs.
In this tutorial, I'll show you how.

## Getting Started

You can do this in a back-end QEWD REST API method or interactive message handler, but
you can also do it interactively using the *jsdb_shell* Module, which is what I'll describe here.

[Start the Node.js REPL and load the *jsdb_shell* Module](./REPL.md#getting-started). Then
instantiate a QEWD-JSDB *Document Node Object* for a Document (ie YottaDB Global) named *jsdbDom*,
and a subscript of *demo*:


        var doc = jsdb.use('jsdbDom', 'demo')

Next, enable it to behave as a *DOM*:

        doc.enable_dom()

At this point, if you look in your YottaDB system, you won't see a Global named *^jsdbDom* yet.

## The documentNode

The first step is to create the (initially empty) DOM Document:

        var documentNode = doc.dom.createDocument()


If you take a look in your YottaDB system, you'll see that the following Global nodes have
been created:

        YDB> zwr ^jsdbDom
        
        ^jsdbDom("demo","documentNode")=1
        ^jsdbDom("demo","index","by_nodeName",9,"#document",1)=""
        ^jsdbDom("demo","index","by_nodeType",9,1)=""
        ^jsdbDom("demo","nextNodeNo")=1
        ^jsdbDom("demo","node",1,"nodeName")="#document"
        ^jsdbDom("demo","node",1,"nodeNo")=1


Every DOM has a notional top-level node, known as the *documentNode*, under which everything
else is attached.  It won't appear in the actual XML text: you can try this
by displaying the output from the *output()* DOM API method:

        console.log(doc.dom.output())

You won't get anything displayed.  However, we can interrogate the DOM to see what it
currently contains.

The *createDocument()* method returned a pointer to the *documentNode*.  However, 
having created a new DOM Document, we can also access its *documentNode* using:

        var dnode = doc.dom.documentNode


All DOM Nodes have several mandatory properties, in particular *nodeType* and *nodeName*.

Let's see what they are for our *documentNode*:

        console.log(dnode.nodeType)
        // 9

        console.log(dnode.nodeName)
        // #document

A DOM can, of course, only have one *documentNode*.

If you look in the YottaDB Global that represents our DOM, you'll see how and where these
properties are stored.

## Adding an XML Tag

There are two ways to add an XML Tag to our DOM:

- the low-level way, using the primitive DOM API methods
- the high-level way, using a single method that, behind the scenes, uses the appropriate primitive
DOM API methods to create a tag and attributes and attach it to the DOM

### Low-Level APIs

First let's look at the low-level way.  You add a tag by:

- creating an Element node (*createElement()*)
- optionally, adding attribute name/values to the Element (*setAttribute()*)
- optionally adding text content to the Element (*textContent()*)
- append the Element to an existing Node in the DOM (*appendChild()* or *insertBefore()*)

#### *createElement()*

So, first create an XML Element Node named *myTag*.  This will be the
equivalent of creating:

        <myTag />

Type this:

        var el = doc.dom.createElement('myTag')

If you look in YottaDB you'll see what's been created:

        YDB> zwr ^jsdbDom
        
        ^jsdbDom("demo","documentNode")=1
        ^jsdbDom("demo","index","by_nodeName",1,"myTag",2)=""
        ^jsdbDom("demo","index","by_nodeName",9,"#document",1)=""
        ^jsdbDom("demo","index","by_nodeType",1,2)=""
        ^jsdbDom("demo","index","by_nodeType",9,1)=""
        ^jsdbDom("demo","nextNodeNo")=2
        ^jsdbDom("demo","node",1,"nodeName")="#document"
        ^jsdbDom("demo","node",1,"nodeNo")=1
        ^jsdbDom("demo","node",1,"nodeType")=9
        ^jsdbDom("demo","node",2,"nodeName")="myTag"
        ^jsdbDom("demo","node",2,"nodeNo")=2
        ^jsdbDom("demo","node",2,"nodeType")=1

You can see that a second Node has been added, representing the Element.

Yet, if you output the XML:

        console.log(doc.dom.output())

you'll not see anything.  That's because, whilst the Element has been created
within the DOM, it hasn't actually been attached to one of its nodes.  This is an
important feature when working with the DOM: nodes can exist within it, yet can be
unattached to anything.

We can check its properties:

- Element nodes have a *nodeType* of 1:

        console.log(el.nodeType)
        // 1

- Its *nodeName* is the actual tag name:

        console.log(el.nodeName)
        // myTag

- Element nodes have another property: *tagName*, which, unsurprisingly, is the
same as the *nodeName* property:

        console.log(el.tagName)
        // myTag


#### *appendChild()*

You don't need to attach the Element until you've added any attributes and text content,
but the order of these steps doesn't really matter.  if we attach the Element to the DOM now,
the benefit is we'll be able to view the effect of all the subsequent steps by using
the *output()* method.

Currently the only node that exists is the *documentNode*, so that's where we attach
our Element:

      var elx = dnode.appendChild(el)

Now try this:

        console.log(doc.dom.output())

        // <myTag />

There we go! Our XML document has begun to exist!

Only one Element node can be attached to the DOM's *documentNode*, and it's known
as the *documentElement* - the top-most tag in an XML Document.  
You can access a DOM's *documentElement* using:

      var docEl = doc.dom.documentElement

All other tags in an XML document are descendents of the *documentElement*.


#### Add Another Tag

We can now add another tag to the DOM and append it as a child of the *documentElement*:

        var el2 = doc.dom.createElement('myChildTag')
        el.appendChild(el2)

        // alternatively we could have used
        //  docEl.appendChild(el2)

        console.log(doc.dom.output())

        // <myTag><myChildTag /></myTag>


We can tidy up the output - instead of streaming the XML, we can specify a number of indents, eg let's
say 2 indents:


        console.log(doc.dom.output(2))

which will now output:


        <myTag>
          <myChildTag />
        </myTag>

Building out a basic DOM is no more than these steps: creating nodes and appending them
to a parent tag, eg we could continue with:

        var el3 = doc.dom.createElement('mySecondChildTag')
        elx = el.appendChild(el3)
        var el4 = doc.dom.createElement('myGrandChildTag')
        elx = el3.appendChild(el4)
        console.log(doc.dom.output(2))

which will now output:

        <myTag>
          <myChildTag />
          <mySecondChildTag>
            <myGrandChildTag />
          </mySecondChildTag>
        </myTag>


#### *setAttribute()*

An XML tag can optionally have one or more attributes.  Let's add one to one of 
our Elements now.  We use the Element Node's *setAttribute()* method:

        var attr1 = el2.setAttribute("foo","bar")

and see what's happened:

        console.log(doc.dom.output())

        <myTag>
          <myChildTag foo="bar" />
          <mySecondChildTag>
            <myGrandChildTag />
          </mySecondChildTag>
        </myTag>

Let's add another two attributes:


        var attr2 = el2.setAttribute("hello","world")
        var attr3 = el2.setAttribute("id","firstTag")

Let's check what those two commands did:

        console.log(doc.dom.output())

        <myTag>
          <myChildTag foo="bar" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag />
          </mySecondChildTag>
        </myTag>

#### *textContent*

An XML Tag can also optionally have text content, ie between its opening
and closing Tag.  Let's add text to the *myGrandChild* tag:


        el4.textContent = "Some text content"
        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag foo="bar" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>


So these steps are how to create XML Tags along with their attributes and text content


### High-Level Method for Tag Creation

Every Element has an additional method: *appendElement()* which combines all the steps
covered above.  *appendElement()* takes a single argument which is an object defining:

- tagName (mandatory)
- attribute name/value pairs (optional)
- text content (optional)

So let's repeat the previous steps, but using *appendElement()* instead.

First, clear down your previous DOM:

        doc.delete()

Then do the following:

        
        var documentNode = doc.dom.createDocument()
        var el1 = documentNode.appendElement({tagName: 'myTag'})
        var el2 = el1.appendElement({
          tagName: 'myChildTag',
          attributes: {
            foo: 'bar',
            hello: 'world',
            id: 'firstTag'
          }
        })
        var el3 = el1.appendElement({tagName: 'mySecondChildTag'})
        var el4 = el3.appendElement({
          tagName: 'myGrandChildTag',
          text: 'Some text content'
        })

Then see the results:

        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag foo="bar" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

So we've created the very same XML Document, but much more simply!


## The Persistent DOM

As you've seen when you looked in YottaDB, the QEWD-JSdb DOM is persistent, being stored
in a Global: in our case it's the *^jsdbDom* Global.

So this take a look at what this means in practice.

Exit from the Node.js REPL that you've been using: type *CTRL & C* twice.

So you've now disconnected from YottaDB.

Now restart the REPL and re-connect:

        node

        > var jsdb = require('./jsdb_shell')


Do this again:

        var doc = jsdb.use('jsdbDom', 'demo')
        doc.enable_dom()

And now see what happens when you do this:

        console.log(doc.dom.output(2));

Yes, your DOM is still there!


## Navigating the DOM

The DOM APIs allow you to navigate within the DOM to access individual Tags, Attributes
and Text Content.  You can then edit or remove them, or add further content.

We've already seen how you can access the *documentNode* and *documentElement*.

For example:

        console.log(doc.dom.documentElement.tagName)

        myTag


### Navigating Between Adjacent Elements

Every Element Node has a number of properties to allow you to navigate to other adjacent ones:

- firstChild
- lastChild
- nextSibling
- previousSibling
- parentNode

For example:

        var el1 = doc.dom.documentElement
        console.log(el1.firstChild.tagName)

which will return:

        myChildTag

Next try:

        console.log(el1.lastChild.tagName)

which returns:

        mySecondChildTag


Alternatively we could do this:

        var el2 = el1.firstChild
        console.log(el2.nextSibling.tagName)

and again we get:

        mySecondChildTag

Or navigate in the other direction:

        var el2 = el1.lastChild
        console.log(el2.previousSibling.tagName)

which gives us:

        myChildTag

Note the way that the properties and methods can be chained, for example:

        console.log(el1.firstChild.nextSibling.firstChild.tagName)

which returns:

        myGrandChildTag


We can also navigate up the DOM's hierarchy by using the *parentNode* property.
Start here:

        var gc = el1.firstChild.nextSibling.firstChild
        console.log(gc.tagName)

Now go up to its parent:

        console.log(gc.parentNode.tagName)

and up another level:

        console.log(gc.parentNode.parentNode.tagName)

That's the top-most XML Element.  What happens if we try to go up another level?

        console.log(gc.parentNode.parentNode.parentNode.tagName)

That returns undefined.  Try this though:

        console.log(gc.parentNode.parentNode.parentNode.nodeName)

As you can see, you get:

        #document

because you've reached the DOM's *documentNode*

Try going any higher and, not surprisingly, you'll get errors.


### *getElementsByTagName()*

Clearly, the properties above are useful for moving around within adjacent Elements, but
it's a laborious way to get to a particular node that you might want to access.

So the DOM provides other methods.  For example, we could get straight to that *myGrandChildTag*
Element like this:

        var nl = doc.dom.getElementsByTagName('myGrandChildTag')

What's returned (ie *nl*) is a somewhat unusual data structure, unique to the DOM and
known as a NodeList.  It behaves like an Array, but is actually a *live* object - reflecting
the actual content of the DOM if it is subsequently changed.  For those interested in how
this is implemented in QEWD-JSdb, it uses a JavaScript Proxy Object whose properties and
methods directly access the YottaDB database when they're invoked, so they always return what's
actually in the DOM at the time they're invoked.

Try this:

        nl.length

        1

So it found 1 Element Node with a *tagName* of *myGrandChildTag*.

To access the Element Node, use nl[0] as if it's an Array, eg:

        nl[0].tagName

        myGrandChildTag


So we could have done this:

        var gc = doc.dom.getElementsByTagName('myGrandChildTag')[0]

*gc* is now the Element Node object we want


### *getElementById()*

There's another way to navigate the DOM to specific Elements, which is available
for any Tags that have an *id* Attribute.  The *id* is deemed to be a special attribute
name, and each *id* value within a DOM is unique.  The *id* Attributes are specifically
indexed, so they provide a rapid route to get to a particular Element.

If you remember, we added an *id* Attribute to one of our Tags:


          <myChildTag foo="bar" hello="world" id="firstTag" />


We can get directly to this Element as follows:

        var mc = doc.dom.getElementById('firstTag')

Because *id*s are unique, the *getElementById()* method returns the matching Element Node Object.
If the specified *id* doesn't exist, it returns a *null*.


## Interrogating and Manipulating the DOM

You'll often want to know about features or details about the part of the DOM you're currently in.

Typically that will mean you want to find out about:

- An Element's Child Elements
- An Element's Attributes

### Child Elements and Child Nodes

Typically you'll want to know whether the current Element has Child Elements,
and, if so, access them.

To illustrate how you can do this, try the following.

First, let's remind ourselves what's currently in our DOM:

        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag foo="bar" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

Next, let's get pointers to two of the Tags
in our DOM:


        var tag1 = doc.dom.getElementById('firstTag')
        var tag2 = doc.dom.getElementsByTagName('mySecondChildTag')[0]


We can see from the listing of our DOM that the first Tag does not have any Child Nodes,
while the second one does.  So how can we find this out from the DOM itself?  Try this:

        tag1.hasChildElements()

        false

        tag2.hasChildElements()

        true

Now let's try this.  First get the 'myGrandChildTag' tag, eg:

        var tag3 = tag2.firstChild

        tag3.tagName

        // myGrandChildTag

We can confirm that this tag has no Child Elements:

        tag3.hasChildElements()

        false

But if we try the *hasChildNodes()* method:


        tag3.hasChildNodes()

        true

That's because whilst the *myGrandChildTag* Tag does not have any Child Elements, 
it does have text content, which, in DOM terms, means it
has a child Text Node.  Everything in the DOM is represented as Nodes!  


Typically, we'll not just want to know if a particular Tag has Child Elements, we'll
want to access those Child Elements.

There are two ways to do this:

- using the *childNodes* property
- using the *childElements* property

The *childNodes* property will return all of the immediate Child DOM Nodes of an Element.
Those Child Nodes can be both Child Elements and Text Nodes.

The *childNodes* property (like the *getElementsByTagName()* method) returns a NodeList
of Child Nodes.  This can be treated as an Array, but is a live data structure, reflecting
the DOM content each time one of its properties or methods is invoked.

The *childElements* property will return all the immediate Child Element DOM Nodes as
an actual Array.  It is therefore not a live data structure. Its properties and methods
reflect the DOM content when the *childElements* array was constructed.

In practice the difference between a NodeList and true Array is unlikely to be noticeable,
unless the DOM is highly dynamic and subject to change while you're interrogating it.

Let's see them in use.

Our second tag has Child Elements:

        tag2.tagName

        // mySecondChildTag

        tag2.hasChildElements()

        // true

So:

        var children = tag2.childNodes
        children.length

        // 1

        children[0].tagName

        // myGrandChildTag


And let's try this grandchild tag:

        var grandChildren = children[0].childNodes
        grandChildren.length

        // 1

        grandChildren[0].nodeType

        // 3   ie text

        grandChildren[0].nodeValue

        // 'Some text content'



So let's compare this with the *childElements* property:

        var children = tag2.childElements
        children.length

        // 1

        children[0].tagName

        // myGrandChildTag


        var grandChildren = children[0].childElements
        grandChildren.length

        // 0  - because it doesn't have any Child Elements, just a child Text Node




### Attributes

#### *hasAttributes()*

Typically you'll want to know whether the current Element has any Attributes,
and, if so, access them.

To illustrate how you can do this, try the following.

First, let's remind ourselves again what's currently in our DOM:

        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag foo="bar" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

Next, let's get pointers to two of the Tags
in our DOM, one with Attributes and one without:

        var tag1 = doc.dom.getElementById('firstTag')
        var tag2 = doc.dom.getElementsByTagName('mySecondChildTag')[0]

Now interrogate the DOM about their Attributes:

        tag1.hasAttributes()

        // true

        tag2.hasAttributes()

        // false


#### The *attributes* Property

We can get all of the Attributes for a Tag using the *attributes* property:

        var attrs = tag1.attributes

What's returned is something known as a Named Node Map, again something unique to the DOM, and
just like Node Lists, Named Node Maps are active structures, and once again implemented here
as a Proxy Object which accesses the YottaDB data every time one of its properties or methods is
invoked, therefore reflecting the latest content in the DOM.

A Named Node Map has several specific properties and methods.

We can find how many Attributes were found using its *length* property:

        attrs.length

        // 3

To access a specific Attribute, you use the *item()* method, specifying the index value as
its argument.  Attribute *item*s are numbered from zero.  What is returned is an *Attr*
Node object.  The most important properties of an *Attr* Node are its *name* and *value*:

        attrs.item(0).name

        // foo

        attrs.item(0).value

        // bar

Of course you could find out its DOM nodeType:

        attrs.item(0).nodeType

        // 2  ie an Attr Node

and we can find out the Element it belongs to by using the *Attr*'s *ownerElement* property:

        attrs.item(0).ownerElement.tagName

        // myChildTag

You can also find out if the *Attr* is an *id* attribute or not:

        attrs.item(0).isId

        // false

        attrs.item(2).isId

        // true


Returning to the *attributes* Named Node Map, you can also apply its *getNamedItem()*
method to return the *Attr* Node for the Attribute with the specified name, eg:

        attrs.getNamedItem('id').value

        // firstTag

        attrs.getNamedItem('foo').value

        // bar

Of course, if all you want is an Attribute's value, you could also use the somewhat 
simpler Element Node's *getAttribute()* method, eg:

        tag1.getAttribute('id')

        // firstTag


You can also add an Attribute using the *attributes*' *setNamedItem()* method.  This
requires you to create an Attr Node, assigning it both a name and value, and the
provide this Attr Node as the *setNamedItem()*'s argument.  Try it out:

        var attr1 = doc.dom.createAttribute('class')
        attr1.value = 'bingo'
        tag1.attributes.setNamedItem(attr1)

        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag class="bingo" foo="bar" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>


#### Using *setAttribute()* to Set and Change Attribute Values


You'll probably be thinking that it's a lot easier to use the Element Node's *setAttribute()*
method, which is quite true, ie:

        tag1.setAttribute('x', 100)
        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag class="bingo" foo="bar" hello="world" id="firstTag" x="100"/>
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

*setAttribute()* is also a simple way to change the value of an existing Attribute, eg

        tag1.setAttribute('x', 200)
        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag class="bingo" foo="bar" hello="world" id="firstTag" x="200"/>
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>


#### *getAttributes()*

You may also find that the Named Node Map returned by the *attributes* property
is overkill if all you want is the names and values of all the Attributes in an
Element.  QEWD-JSdb provides a custom method: *getAttributes()* which you may
find convenient.  Try it out:

        var attrObj = tag1.getAttributes();

        // { class: 'bingo', foo: 'bar', hello: 'world', id: 'firstTag', x: 200 }

As you can see, *getAttributes()* is a *Getter* method that simply returns the attribute 
names and values as an Object.


#### Removing Attributes

Removing an Attribute can also be done either using the *attributes*' *removeNamedItem()*
method, or by using the simple Element Node's *removeAttribute()* method, eg:

        tag1.attributes.removeNamedItem('foo')
        tag1.removeAttribute('x')
        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

----

## Inserting Elements

### *insertBefore()*

So far you've seen how new Elements can be added to the DOM by appending them to a 
parent Element Node.  Of course, if the parent Element already has one or more existing
Child Elements, the new one will be appended as a new *lastChild*.

What if you want to add a new Element as a *firstChild*, or in the middle of a set of
existing Child Elements?  To do this, you can use the *insertBefore()* method.

First, create a new Element Node, eg:

        var newTag = doc.dom.createElement('myInsertedTag');

Let's insert this before the *myGrandChildTag* Tag:

        var gc = doc.dom.getElementsByTagName('myGrandChildTag')[0]
        gc.insertBefore(newTag, gc);
        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myInsertedTag />
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

The *insertBefore()* method is a bit strange in terms of its syntax.  It can actually
be applied by any Element, and it is the two arguments that actually specify the insertion
point in the DOM:

        anyNode.insertBefore(newElement, ElementToInsertBefore)

So we could have used this and the result would be the same:

        doc.dom.documentElement.insertBefore(newTag, el4);


### insertBeforeChildren

Something that neither *appendChild* nor *insertBefore* make easy is the insertion of
a new Element between an existing *parent* Element and its existing Child Elements: in other
words adding a new intermediate layer in an existing DOM hierarchy.

It can be done, but it involves removing the Child Nodes, appending the new Element to the
now childless parent Node, and then re-appending the Child Nodes to the new Element.

To save you all that hassle, you can use QEWD-JSdb's custom DOM method: *insertBeforeChildren()*.
First, create the new Element:

        var newTagx = doc.dom.createElement('intermediateTag');

Get the Element that you want to be your new Tag's parent node, eg:

        var par = doc.dom.getElementsByTagName('mySecondChildTag')[0]

and now insert your new Element:

        par.insertBeforeChildren(newTagx)
        console.log(doc.dom.output(2));

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <intermediateTag>
              <myInsertedTag />
              <myGrandChildTag>
                Some text content
              </myGrandChildTag>
            </intermediateTag>
          </mySecondChildTag>
        </myTag>

In the listing you can see the *myInsertedTag* Element has been inserted between the
*mySecondChildTag* Element and what were previously its Child Nodes.

This turns out to be a really powerful method for making what would otherwise be
really complex and difficult-to-implement changes to a hierarchical structure.

----

## Removing Elements

### *removeChild()*

You may need to remove Elements from a DOM.  To do so you use the
*removeChild()* method.  You need to be careful, however: when you remove
an Element, you are actually detatching it (**and any of its Child Nodes**)
from the DOM tree.  By default, it will still exist in the DOM, but it's just disconnected
from anything, so won't appear when you output the DOM as XML for example.
However, the QEWD-JSdb version of the *removeChild()* method allows you to optionally
specify that the removed Element (and its sub-tree of Child Nodes) is permanently
deleted from the DOM.

The *removeChild()* method is applied to the parent Element of the Element you want to remove.

For example, in your DOM that represents this XML:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <intermediateTag>
              <myInsertedTag />
              <myGrandChildTag>
                Some text content
              </myGrandChildTag>
            </intermediateTag>
          </mySecondChildTag>
        </myTag>


to remove the *intermediateTag* Element (and its sub-tree of Nodes), you would apply
*removeChild()* to its parent *mySecondChildTag* Element, eg:

        var parTag = doc.dom.getElementsByTagName('mySecondChildTag')[0]
        var imTag = doc.dom.getElementsByTagName('intermediateTag')[0]
        parTag.removeChild(imTag)
        console.log(doc.dom.output(2));


        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag />
        </myTag>


If you look in YottaDB, you'll see that the removed Nodes are still in the Global - they've just
been detached from the DOM tree, so don't appear in the XML when listed.

Just to prove this, you can get them back:

        parTag.appendChild(imTag)
        console.log(doc.dom.output(2));

and as if by magic, they re-appear:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <intermediateTag>
              <myInsertedTag />
              <myGrandChildTag>
                Some text content
              </myGrandChildTag>
            </intermediateTag>
          </mySecondChildTag>
        </myTag>

Of course, you could have re-attached them to any of the DOM Nodes if you'd wanted - something
you might want to try out.  You'll also notice that the links between the removed Nodes were
still in place, so the entire sub-tree was recovered.

Now let's repeat the *removeChild()*, but this time we'll physically delete them from the
QEWD-JSdb storage (ie from the YottaDB Global):

        parTag.removeChild(imTag, true)
        console.log(doc.dom.output(2));

If you look in YottaDB at the *^jsdbDom* Global, you'll see that the detached Nodes have all been
deleted, and now you won't be able to re-attach them.


### *removeAsParent()*

As you've seen, *removeChild()* removes not only the specified Child Node but also its
entire sub-tree on descendent Nodes.

What if you just want to remove the Child Node, but retain all its Child Node, effectively
moving them up a level to replace the removed Child and become Child Nodes of the parent?

You could do that, but it would require a laborious and potentially tricky sequence of
detaches and re-attaches.  However, to save you all that bother, QEWD-JSdb provides
a custom method called *removeAsParent()*.

So let's go back to our previous DOM:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <intermediateTag>
              <myInsertedTag />
              <myGrandChildTag>
                Some text content
              </myGrandChildTag>
            </intermediateTag>
          </mySecondChildTag>
        </myTag>


Suppose we want to remove the *intermediateTag* Element, but retain its Child Nodes, so
they now become Child Nodes of the *mySecondChildTag* Element.  We could do that as follows:

        var imTag = doc.dom.getElementsByTagName('intermediateTag')[0]
        imTag.removeAsParent()
        console.log(doc.dom.output(2));

The result is this:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myInsertedTag />
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

Note that the detached *intermediateTag* Element is still in the DOM.  To permanently
remove it, add *true* as an argument, ie:

        imTag.removeAsParent(true)


You'll have probably realised that the *removeAsParent()* method reverses the effect of
the *insertBeforeChildren()* method that was described earlier.

----

## Handling Text

You've seen near the start of this Tutorial how Text can be added to an Element,
either using the low-level *textContent* property or as a property of the argument
provided to the *appendElement()* method.

*textContent* is a read/write property.  So in our example DOM:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myInsertedTag />
            <myGrandChildTag>
              Some text content
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

we could get the *textContent* of the *myGrandChildTag* Element:


        var gcTag = doc.dom.getElementsByTagName('myGrandChildTag')[0]
        console.log(gcTag.textContent);

        // Some text content

We could replace the text:

        gcTag.textContent = 'Some different text'

Or we could add text to the *myInsertedTag*:

        var iTag = doc.dom.getElementsByTagName('myInsertedTag')[0]
        iTag.textContent = 'Hello World!'
        console.log(doc.dom.output(2));
  
The DOM will now appear as:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myInsertedTag>
              Hello World!
            </myInsertedTag>
            <myGrandChildTag>
              Some different text
            </myGrandChildTag>
          </mySecondChildTag>
        </myTag>

If you look at the XML DOM API you'll discover there are other ways to
manipulate text, using Text Nodes.  These are fully supported in QEWD-JSdb, and
allow you, for example, to create multiple Text Nodes within an Element.
Feel free to experiment with these methods.  However, in my experience,
the *textContent* read/write property is sufficient for all your likely needs.

----

## Other XML Nodes

So far we've been focusing on XML Elements and their associated Attributes and
Text.  There are, of course, a number of other,optional XML Tags (represented,
of course, as Nodes in the DOM):

- Processing Instruction
- Document Type
- Comment
- CDATA Section

### Processing Instruction

The most common so-called *Processing Instruction* you're likely to want to
add to a DOM is the standard XML Declaration which is at the start of an XML
Document, eg:

        <?xml version="1.0" encoding="utf-8"?>

QEWD-JSdb provides a custom method for adding this:

        var xd = doc.dom.createXMLDeclaration()

This creates the XML Declaration tag and inserts it before the *documentElement*
Node.

Although unlikely to be needed, you can modify the version and/or encoding by
specifying them as arguments:

        doc.dom.createXMLDeclaration(version, encoding)

By default, the version is *1.0* and the encoding is *utf-8*.


If you need to create other Processing Instructions, do so using:

        var pi = doc.dom.createProcessingInstruction(target, data)

You then need to append or insert the resulting Processing Instruction Node
into the appropriate place in your DOM.


### Document Type Definition

You can add a Document Type Definition (DTD) to your DOM by using
the *createDocumentType()* method, eg:

        var dt = doc.dom.createDocumentType(name, publicId, systemId)

The QEWD-JSdb version of this method recognises a number of common names, mainly
for HTML/XHTML documents, eg:

        var dt = doc.dom.createDocumentType('html4_strict')

This is shorthand for:

        var name = 'HTML';
        var publicId = '-//W3C//DTD HTML 4.01//EN';
        var systemId = 'http://www.w3.org/TR/html4/strict.dtd';
        var dt = doc.dom.createDocumentType(name, publicId, systemId)


Shortcuts exist for:

- html4_strict
- html4_transitional
- html4_frameset
- xhtml1_strict
- xhtml1_transitional


You must append or insert the returned DocumentType Node into your DOM.  Normally
you would use *insertBefore()* to add it before the *documentElement* Node.

You can interrogate the DOM to access its DocumentType Node (if present), by
using its *docType* property, eg:

        var dt = doc.dom.docType;

and view its name:

        console.log(dt.name)

        // HTML

Note that if you apply an HTML DTD to your DOM, the behaviour of the *output()* method
changes.  All Element (Tag) names will be converted to lower case, and certain Tag Names,
if recognised as relevant HTML tags, will be displayed as *void* tags, eg *input* and *link* tags
will be shown without an explicit closing tag, eg:

        <input type="text">

rather than

        <input type="text" />


### Comments

You can add Comment Nodes to your DOM using the *createComment()* method, eg:

        var cmnt = doc.dom.createComment(text);

You then append or insert the returned Comment Node to the appropriate Element Node, eg.

        var cmnt = doc.dom.createComment('This is my comment');
        parTag.appendChild(cmnt)

When output the Document would appear as:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myInsertedTag>
              Hello World!
            </myInsertedTag>
            <myGrandChildTag>
              Some different text
            </myGrandChildTag>
            <!--This is my comment-->
          </mySecondChildTag>
        </myTag>


### CDATA Section

You can add CDATA Section Nodes to your DOM using the *createCDATASection()* method, eg:

        var cdata = doc.dom.createCDATASection(text);

You then append or insert the returned CDATA Section Node to the appropriate Element Node, eg.

        var cdata = doc.dom.createCDATASection('Some <CDATA> data & then some');
        parTag.appendChild(cdata)

When output the Document would appear as:

        <myTag>
          <myChildTag class="bingo" hello="world" id="firstTag" />
          <mySecondChildTag>
            <myInsertedTag>
              Hello World!
            </myInsertedTag>
            <myGrandChildTag>
              Some different text
            </myGrandChildTag>
            <!--This is my comment-->
            <![CDATA[Some <CDATA> data & then some]]>
          </mySecondChildTag>
        </myTag>

----

# Ingesting XML

You aren't limited to programmatically-generating DOMs.  QEWD-JSdb's DOM 
implementation includes a *parser* module which includes method for
ingesting:

- XML from a file (*parseFile()*)
- XML from a text variable (*parseText()*)

The *parser* module makes use of the standard Node.js *sax* module for parsing the
file or text input, and its events trigger the QEWD-JSdb COM methods which store the
parsed XML as a persistent DOM in YottaDB.

*parser* is available as a property of a QEWD-JSdb DOM, eg in a back-end REST API
method or interactive message handler method:

        var doc = this.use('jsdbDom', 'demo');
        doc.enable_dom();
        doc.delete();
        var parser = doc.dom.parser;

## *parseFile()*

To ingest XML from a file and parse it into a QEWD-JSdb DOM, use the *parseFile()* method.
This takes two arguments:

- **filepath**: the file path of the XML file to be ingested
- **callback**: a function that will be invoked when the parsing has completed.  This function
has a single argument: the QEWD-JSdb DOM object that has been created.

You can try it out.  You won't be able to use the *parseFile()* method interactively in
the Node.js REPL, but you can run it via a script file.

You'll find an 
[example XML document in this repository](https://github.com/robtweed/qewd-starter-kit-yottadb/blob/master/example.xml).  Download it and put it into your QEWD system's
installation directory:

- If you're using a Native installation of QEWD and YottaDB:

  - Put the *example.xml* file into your QEWD system's installation directory
 (eg ~/qewd).

  - Then create a file in the same
directory named *loadxml.js* containing:


        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom');
        doc.enable_dom();
        doc.delete();
        var filepath = '/home/ubuntu/qewd/example.xml';  // change as appropriate
        doc.dom.parser.parseFile(filepath, function(dom) {
          console.log(dom.output(2));
        });

  - Now, in a Terminal Window (or Command Prompt) run the *loadxml.js* script:

        node loadxml
----

- If you're using the QEWD Docker Container:

  - Put the *example.xml* file into the host directory that you've mapped into the QEWD
Container (eg the host system's ~/qewd directory).

  - Then create a file in the same
directory named *loadxml.js* containing:


        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom');
        doc.enable_dom();
        doc.delete();
        var filepath = '/opt/qewd/mapped/example.xml';
        doc.dom.parser.parseFile(filepath, function(dom) {
          console.log(dom.output(2));
        });


  - Now, shell into the Container and run the *loadxml.js* script from the */opt/qewd/mapped*
directory:

        cd mapped
        node loadxml
----



After a second or two you should see the XML listing:

        <doc>
          <foo location="Drumcondra">
            <bar name="Cat and Cage">
              pub 1
            </bar>
            <bar name="Fagan's" owner="John" tied="true">
              pub 2
            </bar>
            <offlicense name="oddbins" />
            <bar name="Gravedigger's">
              pub 3
            </bar>
            <bar name="Ivy House">
              pub 4
            </bar>
          </foo>
          <foo location="Town">
            <bar name="Peter's Pub">
              pub 5
            </bar>
            <bar name="Grogan's">
              pub 6
            </bar>
            <bar name="Hogans's">
              club 1
            </bar>
            <bar name="Brogan's" owner="James" tied="false">
              pub 8
            </bar>
            <bar closed="yes">
              pub 9
            </bar>
            <offlicense name="unwins" />
          </foo>
          <aaa>
            <bbb>
              <bar name="Robin Hood">
                pub 10
                <beer name="guinness" />
                <beer name="tetleys" />
              </bar>
              <bar>
                As yet un-named pub
              </bar>
            </bbb>
            <foo>
              <ccc />
            </foo>
          </aaa>
        </doc>


Take a look in YottaDB and you'll find this DOM's data in a Global named *^exampleDOM*

You can now access it and use the QEWD-JSdb DOM API methods on it within the Node.js REPL,
in a REST API method or in an interactive message handler.  For example, in the REPL:

        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom');
        doc.enable_dom();

and you can now access it via *doc.dom*, eg:

        var fooTags = doc.dom.getElementsByTagName('foo');
        ... etc


## *parseText()*

To ingest a stream of XML from a text variable and parse it into a QEWD-JSdb DOM, use the *parseText()* method.
This takes two arguments:

- **text**: the variable containing the XML to be ingested
- **callback**: a function that will be invoked when the parsing has completed.  This function
has a single argument: the QEWD-JSdb DOM object that has been created.

You can try it out.  You won't be able to use the *parsText()* method interactively in
the Node.js REPL, but you can run it via a script file.

- If you're using a Native installation of QEWD and YottaDB:

  Create a file in your QEWD system's installation directory (eg ~/qewd)
named *loadtext.js* containing:


        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom2');
        doc.enable_dom();
        doc.delete();
        var xml = '<xml><foo hello="world">Some Text</foo></xml>';
        doc.dom.parser.parseText(xml, function(dom) {
          console.log(dom.output(2));
        });

  Now, in a Terminal Window (or Command Prompt) run the *loadxml.js* script:

        node loadtext
----

- If you're using the QEWD Docker Container:

  - Creat a file named *loadtext.js* in the host directory that you've mapped into the QEWD
Container (eg the host system's ~/qewd directory), containing:

  - Then create a file in the same
directory named *loadxml.js* containing:

        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom2');
        doc.enable_dom();
        doc.delete();
        var xml = '<xml><foo hello="world">Some Text</foo></xml>';
        doc.dom.parser.parseText(xml, function(dom) {
          console.log(dom.output(2));
        });

  - Now, shell into the Container and run the *loadtext.js* script from the */opt/qewd/mapped*
directory:

        cd mapped
        node loadtext
----


After a second or two you should see the XML listing:

        <xml>
          <foo hello="world">
            Some Text
          </foo>
        </xml>

Take a look in YottaDB and you'll find this DOM's data in a Global named *^exampleDOM2*

You can now access it and use the QEWD-JSdb DOM API methods on it within the Node.js REPL,
in a REST API method or in an interactive message handler.  For example, in the REPL:

        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom2');
        doc.enable_dom();

and you can now access it via *doc.dom*, eg:

        var fooTags = doc.dom.getElementsByTagName('foo');
        ... etc


----

# Querying DOMs using XPath

[XPath (XML Path Language)](https://www.w3.org/TR/1999/REC-xpath-19991116/) 
is a query language for selecting nodes from an XML document.
The XPath language is based on a tree representation of the XML document, and provides the ability to navigate around the tree, selecting nodes by a variety of criteria.

XPath is normally implemented to use an in-memory XML DOM.  However, because QEWD-JSdb's 
core DOM APIs conform to the XML DOM standard, the standard Node.js XPath module has been
able to be used with QEWD-JSdb's persistent DOMs: the XPath module isn't even aware that the
actual DOM data it is handling is actually stored in an YottaDB database!

QEWD-JSdb integrates the XPath module directly into the DOM's object, so it's
immediately available to you without doing anything else.

## Try out XPath

Let's use the DOM you created in the previous section above: the one created from the
*example.xml* file.  You can even try out XPath in the Node.js REPL.

First load the *jsdb_shell* module and enable DOM access to the *^exampleDom*
Global in YottaDB:


        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('exampleDom');
        doc.enable_dom();

Let's try a simple XPath query, finding all *foo* Elements (ie Tags) anywhere in
the DOM:

        var fooTags = doc.dom.xpath('//foo')

What's returned is an Array of Element Nodes, each one being a *foo* Element that it's
found in the DOM.  There should be three.  Let's check:

        fooTags.length

        // 3

and you can then inspect each member of the Array, eg:

        fooTags[0].tagName

        // foo

        fooTags[0].getAttributes()

        // { location: 'Drumcondra' }


You could then try finding all *bar* Elements that are an immediate Child of a 
*foo* Element, anywhere in the DOM:

        var barTags = doc.dom.xpath('//foo/bar')

        barTags.length

        // 9

        barTags[1].tagName

        // bar

        barTags[1].getAttributes()

        // { name: "Fagan's", owner: 'John', tied: true }


You can apply any valid XPath query to a QEWD-JSdb DOM.  The returned result will always be
an Array of Nodes that matched the query.  If you're not familiar with XPath, take 
a look at the 
[list of examples]((https://github.com/robtweed/qewd-starter-kit-yottadb/blob/master/xpath_query_examples.txt)
 that you'll find in this repository.


----

# Extending DOMs with *UserData*

The XML DOM API includes a little-known DOM feature which allows user-defined data to be
associated with a DOM Node.  The so-called *userData* consists of key/object pairs.
This feature allows the functionality of a DOM to be considerably enhanced.  For example, you are
no longer restricted to Element Nodes having just Attributes and Text properties.  They could
be adapted to be something on which you store any arbitrary objects, all within a hierarchical
structure.

QEWD-JSdb has implemented the relevant XML DOM APIs:

- **setUserData()**: add a key/object pair to a DOM Node
- **getUserData()**: retrieve the object for a specified key from a DOM Node
 
It also implements several additional Node methods to make the use of *userData* flexible,
easy and very powerful:

- **getUserDataKeys()**: returns an Array of keys (if any) in a DOM Node
- **hasUserData()**: returns true of an object exists for a specified key in a DOM Node
- **deleteUserData()**: deletes the object for a specified key in a DOM Node

The DOM Object (eg *doc.dom*) also includes a method named *walk()*.  This 
recurses through all the Element Nodes in a DOM and invokes a function that you 
specify for every Element that is found.  You could use this to access and retrieve
*userData* stored at each Element in the DOM tree.

Let's try these out in the Node.js REPL.

We'll start by creating a new DOM with a few Elements.  For simplicity, we 
won't bother adding Attributes or Text to them (though we could if we wanted to).
We could do this quickly and easily by creating a Node.js script file that we
then execute.  Create a file in your QEWD system's installation directory (eg ~/qewd or C:\qewd)
named *userdataDOM.js* containing:


        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('userDataDom');
        doc.enable_dom();
        doc.delete();
        var xml = '<xml><foo><bar /><bar /></foo></xml>';
        doc.dom.parser.parseText(xml, function(dom) {
          console.log(dom.output(2));
        });

Run it as follows:

        node userDataDOM

and you see this:

        <xml>
          <foo>
            <bar />
            <bar />
          </foo>
        </xml>


OK so now we have this simple DOM, let's add some *userData* to the *foo* Element.
We'll do these subsequent steps in the REPL, so start it up:

        node

        Welcome to Node.js v14.5.0.
        Type ".help" for more information.
        >

Then load the *jsdb_shell* module and enable access to the DOM we just created

        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('userDataDom');
        doc.enable_dom();

Get a pointed to the *foo* Element:

        var foo = doc.dom.getElementsByTagName('foo')[0]

Now we'll create an object:

        var obj = {hello: 'world', foo: 'bar'}

and add it as *userData* to the *foo* Element, using a *key* of *test*:

        foo.setUserData('test', obj)

Now take a look in YottaDB at the *^userDataDom* Global that is storing the DOM's data.
In Node 3 (representing the *foo* Element) you'll see this:

        ^userDataDom("node",3,"firstChild")=4
        ^userDataDom("node",3,"lastChild")=5
        ^userDataDom("node",3,"nodeName")="foo"
        ^userDataDom("node",3,"nodeNo")=3
        ^userDataDom("node",3,"nodeType")=1
        ^userDataDom("node",3,"parent")=2
        ^userDataDom("node",3,"ud","test","foo")="bar"
        ^userDataDom("node",3,"ud","test","hello")="world"

The *ud* subscript denotes *userData*.

So the *userData* has definitely been saved correctly against the *foo* Element!

Now try listing the DOM:

        console.log(doc.dom.output(2))

Once again, all you'll see is the XML:

        <xml>
          <foo>
            <bar />
            <bar />
          </foo>
        </xml>

*userData* isn't actually shown or retrieved when you output the DOM as XML.

Let's try some of the other *userData* methods though.

        foo.getUserDataKeys()

        // [ 'test' ]

        foo.hasUserData('test')

        // true

        foo.hasUserData('dummy')

        // false

        var data = foo.getUserData('test')

        // { foo: 'bar', hello: 'world' }


You can add as many key/Object pairs to a Node as you wish.  So we could do this:


        obj = {more: 'data'}
        foo.setUserData('more', obj)

        foo.getUserDataKeys()

        // [ 'more', 'test' ]


        data = foo.getUserData('more')

        // {more: 'data'}


And we could delete the *userData* with the *more* key:

        foo.deleteUserData('more')

        foo.getUserDataKeys()

        // [ 'test' ]


We could now write a script that uses the DOM's *walk()* method to recurse
through all the Elements in the DOM and retrieve any *userData* it finds.

The *walk()* method is invoked as follows:

        doc.dom.walk(params, fn);

- *params*: an optional object you can define for use within *fn*
- *fn*: a function that is invoked every time an Element is found in the DOM. This function has a
single argument:

  - *params*

  Its context (ie *this*) is the Element Node that has been found.

So let's write our script.  Name it *walk.js*:

        var jsdb = require('./jsdb_shell');
        var doc = jsdb.use('userDataDom');
        doc.enable_dom();

        var fn = function(params) {
          console.log('Element: ' + this.tagName);
          var keys = this.getUserDataKeys();
          var _this = this;
          keys.forEach(function(key) {
            var data = _this.getUserData(key);
            console.log('userData for key ' + key);
            console.log(JSON.stringify(data, null, 2));
          });
        };
        var params = {};
        doc.dom.walk(params, fn);


And then run it:

        node walk

and you should see:

        Element: xml
        Element: foo
        userData for key test
        {
          "foo": "bar",
          "hello": "world"
        }
        Element: bar
        Element: bar

Now try adding more *userData* to some of the other Elements in the DOM and see what it
looks like when you re-run the *walk.js* script!

The DOM is normally thought of as a means of handling XML, but QEWD-JSdb's persistent DOM
storage, coupled with the *userData* functionality means its applicability can be
re-thought: it becomes a way of managing hierarchical data, all of which can be
queried using XPath!



----

# Using JSON with the DOM

So far we've seen the DOM used as a means of representing an XML Document.  However, as
mentioned in the previous section above, the DOM isn't restricted to representing XML, even
though that's what it was originally designed for.  It can be used to represent any
hierarchical dataset, with the benefits that:

- once represented in DOM format, you can use
the DOM API methods to manipulate and modify its content in very sophisticated ways

- you can query the database using XPath.

Now, the hierarchical structure that has largely superceded XML is JSON.  So that begs
the question: can JSON be represented as a persistent DOM?  The answer is yes.
The trick is essentially to represent each property name within a JSON document as a DOM
Element, and store values of leaf properties as Text Nodes.

QEWD-JSdb provides you with a JSON parser which will ingest JSON documents and save them as
a DOM.  Like the XML *parser*, it's provided as a method of the DOM Object (eg *doc.dom*)
named *parseJSON()*:

        doc.dom.parseJSON(json)

It takes a single argument: an in-memory JavaScript/JSON Object.


You can parse and store a JSON Document in your QEWD REST API Methods, interactive message
handler functions, or in the REPL using the *jsdb_shell* module.

For example, using the REPL, try this:

        var jsdb = require('./jsdb_shell')
        var doc = jsdb.use('jsonDom')
        doc.enable_dom()
        doc.delete()
        var json = {foo: {bar: {hello: 'world'}}}
        doc.dom.parseJSON(json)

You could check to see how your JSON is being represented in the DOM:

        console.log(doc.dom.output(2))

        <json>
          <foo>
            <bar>
              <hello type="string">
                world
              </hello>
            </bar>
          </foo>
        </json>

You'll see that the JSON conversion to a corresponding XML structure is probably fairly
intuitive.  Notice the data typing that is done automatically based on the JSON data
type in the incoming JSON Object.

Let's try something a bit more complex:


        doc.delete()
        json = {foo: {bar: {hello: 'world', accept: true, x: 10, colours: ['Red', 'Blue', 'Green']}}}
        doc.dom.parseJSON(json)
        console.log(doc.dom.output(2))

and you'll see it's representing this new JSON as:

        <json>
          <foo>
            <bar>
              <hello type="string">
                world
              </hello>
              <accept type="boolean">
                true
              </accept>
              <x type="number">
                10
              </x>
              <colours type="array">
                <val type="string">
                  Red
                </val>
                <val type="string">
                  Blue
                </val>
                <val type="string">
                  Green
                </val>
              </colours>
            </bar>
          </foo>
        </json>


How do we get it back as JSON again?  For that we use the *outputAsJSON()* method:

        var obj = doc.dom.outputAsJSON()
        console.log(JSON.stringify(obj, null, 2))
        
        {
          "foo": {
            "bar": {
              "hello": "world",
              "accept": true,
              "x": 10,
              "colours": [
                "Red",
                "Blue",
                "Green"
              ]
            }
          }
        }


One word of warning: don't assume that this capability means that you can use it to output
any XML document as a corresponding JSON document.  You'll get some JSON out, but it may be
somewhat mangled!  That's because the *outputAsJSON()* method is expecting to see that stylised
XML format  that it uses to represent a JSON document.

Now, we've seen in the basic QEWD-JSdb APIs that you can use the *setDocument()* and 
*getDocument()* methods to save JSON to an YottaDB Global and retrieve it again.  Why, then,
would you use the QEWD-JSdb DOM to save and retrieve it instead?  It seems an unnecessary
overhead.

The answer is that for most general situations, the basic *setDocument()* and
*getDocument()* methods are perfectly adequate, and are also more performant and
use much less YottaDB Global storage than the DOM for representing the JSON.

However, there are three potential benefits of using the DOM for JSON:

- handling deep JSON with deep hierarchies and long property names:

  YottaDB Globals have a very strict limit on the total subscript
length for an individual Global Node.  *setDocument()* maps each property name in a JSON
hierarchy to a correspondingly-named subscript.  In a deeply nested JSON hierarchy and/or
if property names are long, it's possible to hit this YottaDB subscript length limit.

  By comparison, as you'll have seen when you inspect DOM storage in YottaDB, each Node is
represented in a flattened structure, with each Node containing pointers to its surrounding
Nodes.  It doesn't matter how deep the hierarchy of Nodes is, or how long the property
name (represented as the *nodeValue* value) is.

  So, if your JSON document is beyond the capabilities of *setDocument()*, use
the DOM to store it.

- if you need to manipulate, modify or navigate the JSON content.  The DOM API methods
you've tried out in this tutorial will allow you to perform such tasks with ease, and allow
you to perform complex transformations that would be otherwise very difficult to achieve
by any other means.

- if you need to interrogate or run queries on your JSON, as soon as it's in the DOM, you
have all the capabilities of XPath at your disposal.  The ability to apply XPath queries
to JSON is a very powerful and intriguing capability!



# Conclusions

That concludes this tutorial on the use of the QEWD-JSdb DOM Data Model.

I hope you agree, it's a very powerful technology, and a great use of YottaDB's Global
Storage.  For XML Documents, its benefits are clear.  However, the trick is to
not believe that the DOM is limited to XML.  With the ability to customise the DOM
content with *userData*, and the ability to map JSON to the DOM, the capabilities
of the QEWD-JSdb's persistent DOM are limited only by your imagination.















