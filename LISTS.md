# The QEWD-JSdb Lists Database Model
 
Rob Tweed <rtweed@mgateway.com>  
12 December 2020, M/Gateway Developments Ltd [http://www.mgateway.com](http://www.mgateway.com)  

Twitter: @rtweed

Google Group for discussions, support, advice etc: [http://groups.google.co.uk/group/enterprise-web-developer-community](http://groups.google.co.uk/group/enterprise-web-developer-community)

# About this Document

This document provides background information on the *Lists* NoSQL database model that
is included with QEWD-JSdb.

# About the Lists Database Model

The *Lists* database model is inspired by the [Lists data type](https://redis.io/topics/data-types)
 in the Redis NoSQL database.

It's a very simple data model with which you can implement high-performance stacks or queues,
and a good introduction to the additional multi-model database capabilities provided by QEWD-JSdb.

Before proceeding it is recommended that you first read and complete the [basic QEWD-JSdb tutorial](./REPL.md).  This will ensure you 
understand what's going on in the QEWD-JSdb and why!

## Source Code for the *LISTs* APIs

Like all the QEWD-JSdb APIs, they are written in JavaScript  and are built on top of the
standard QEWD-JSdb APIs.  They are all Open Source APIs, and
you are free to inspect and use the code as you wish, in accordance with the Apache 2 license under
which they are made available.

Find the [LISTs source code here](https://github.com/robtweed/ewd-document-store/tree/master/lib/proto/list).


# Enabling Use of the Lists APIs

You can create and maintain a *List* at any QEWD-JSdb Document Node Object.

Having instantiated the Document Object Node object, you must enable its use of the *List* APIs.
For example, from within a QEWD-Up back-end message handler module:

        doc = this.documentStore.use('jsdbList', 'demo')
        doc.enable_list()

or when using the *jsdb_shell* REPL module:

        doc = jsdb.use('jsdbList', 'demo')
        doc.enable_list()

The Document Node object will now be augmented with a *list* property object, via which you can apply
the *List* APIs to it, eg:

        var record = {a: 1, b: 2};

        doc.list.lpush(record);


# Using Lists

Records are added to a *List* using:

- lpush(record_obj): adds the record object to the top of a *List*
- rpush(record_obj): adds the record object to the end of a *List*
- insert_before(list_position, record_obj): inserts the record object to a *List* before the specified list sequence position

Record can be popped off a *List* and returned as a JavaScript object using:

- lpop(): removes and returns the record object on the top of a *List*
- rpop(): removes and returns the last record object on a *List*

You can view a range of members of a *List* using:

- lrange(): This takes 2 arguments:

  - from: integer specifying the start position in the *List* from which you're interested in viewing
  - to: integer specifying the end position in the *List* at which you're interested in viewing

  Note that *lrange()* does not remove the items from a *List*.

You can trim a *List* using:

- ltrim(): removes all *List* members apart from those
initially in the position range you specify.  The arguments are the same as for *lrange()*.

You can find out how many members exist in a *List* using:

- count(): returns the number of members in a *List*


# Try it out in the Node.js REPL using *jsdb_shell*

[Start the Node.js REPL and load the *jsdb_shell* Module](./REPL.md#getting-started). Then
instantiate a QEWD-JSDB *Document Node Object* for a Document (ie YottaDB Global) named *jsdbList*,
and a subscript of *demo*:


        var doc = jsdb.use('jsdbList', 'demo')

Next, enable it to behave as a *List*:

        doc.enable_list()

At this point, if you look in your YottaDB system, you won't see a Global named *^jsdbList* yet.

Now create a record object, eg:


        var record = {a: 1, b: 2}

and push it onto your *List*.  It doesn't matter whether you use *lpush()* or *rpush()* 
for the first List member:

        doc.list.lpush(record)


Now check in YottaDB to see what's in the *^jsdbList* Global, eg:

        YDB> zwr ^jsdbList

        ^jsdbList("demo","count")=1
        ^jsdbList("demo","firstNode")=1
        ^jsdbList("demo","lastNode")=1
        ^jsdbList("demo","nextNodeNo")=1
        ^jsdbList("demo","node",1,"content","a")=1
        ^jsdbList("demo","node",1,"content","b")=2


You can view the contents of the *List* by using *lrange()*. *lrange()* retrieves an array of
member objects for the specified range. If you don't specify any
arguments, the entire *List* will be retrieved. :

        doc.list.lrange()

        // [{a: 1, b: 2}]


So let's push another record object to the end of our *List* using *rpush()*, eg:

        doc.list.rpush({foo: 'bar'})
        doc.list.lrange()

        // [{a: 1, b: 2}, {foo: 'bar'}]

and if you check in YottaDB again, you'll see the effect in terms of the
*^jsdbList* Global:

        YDB> zwr ^jsdbList

        ^jsdbList("demo","count")=2
        ^jsdbList("demo","firstNode")=1
        ^jsdbList("demo","lastNode")=2
        ^jsdbList("demo","nextNodeNo")=2
        ^jsdbList("demo","node",1,"content","a")=1
        ^jsdbList("demo","node",1,"content","b")=2
        ^jsdbList("demo","node",1,"nextNode")=2
        ^jsdbList("demo","node",2,"content","foo")="bar"
        ^jsdbList("demo","node",2,"previousNode")=1

You can probably see that a QEWD-JSdb *List* is implemented as a Linked List.

Now try adding a record object to the top of the list using *lpush()*:

        doc.list.lpush({member: 1})

and add some records to the end of the *List*, eg:


        doc.list.rpush({member: 4})
        doc.list.rpush({member: 5})
        doc.list.rpush({member: 6})

The objects you push can be as simple or as complex as you like, so try experimenting.
Check in YottaDB how they get physically stored, and compare this with what you see
when you use *doc.list.lrange()*.


Now try retrieving some records from the *List*.  To get the first member, use *lpop()*:

        var obj = doc.list.lpop();

        // {member: 1}

If you view the *List* using *lrange()*, you'll see that not only has *lpop()*
retrieved the first *List* member, but also it has removed it from the *List*.  Take a
look in YottaDB to see what's happened to the *^jsdbList* Global.

Next, try the equivalent using *rpop()*:


        obj = doc.list.rpop();

        // {member: 6}

Then view the *List* using *lrange()*.  You'll see that not only has *rpop()*
retrieved the last *List* member, but also it has removed it from the *List*.

Check to see how many *List* members remain:

        var count = doc.list.count()

        // 4


If you followed the steps shown above, your *List* should now contain:

        doc.list.lrange()

        // [ { a: 1, b: 2 }, { foo: 'bar' }, { member: 4 }, { member: 5 } ]

You aren't limited to pushing *List* members to the start or end of the *List*.  You
can also insert a member at a specified position, eg:

        doc.list.insert_before(2, {hello: 'world'})

        doc.list.lrange()

        /*
        [
          { a: 1, b: 2 },
          { foo: 'bar' },
          { hello: 'world' },
          { member: 4 },
          { member: 5 }
        ]
        */

So what happened was that the new object was inserted into the *List* before member 2
 (remember that *List* members are numbered from zero).


You can view a sub-list of *List* members by specifying arguments when using the *lrange()*
method, eg:


- Complete List:

        doc.list.lrange()
        /*
        [
          { a: 1, b: 2 },
          { foo: 'bar' },
          { hello: 'world' },
          { member: 4 },
          { member: 5 }
        ]
        */

- List members starting from member #2

        doc.list.lrange(2)

        // [ { hello: 'world' }, { member: 4 }, { member: 5 } ]

- List members from member #2 to member #3

        doc.list.lrange(2,3)

        // [ { hello: 'world' }, { member: 4 } ]

- List members from the start of the list up to and including member #3

        doc.list.lrange(0,3)

        // [ { a: 1, b: 2 }, { foo: 'bar' }, { hello: 'world' }, { member: 4 } ]

  Note that you'd get the same effect with:

        doc.list.lrange(null,3)


Finally, try out the *ltrim()* method which allows you to remove a specified range of
members from a *List*.  For example:

        doc.list.ltrim(1,3)
        doc.list.lrange()

        // [ { foo: 'bar' }, { hello: 'world' }, { member: 4 } ]

In other words, *ltrim(1,3)* removed all *List* members **except** members #1 through to #3.


# Conclusion

So, that's QEWD-JSdb *List*s.  They're very simple, but fast and highly effective if you want to
implement a queue.  And of course, because they are implemented as YottaDB Globals, the *List* or
queue can be as big as you like, and will persist for as long as you want.

