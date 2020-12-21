# The QEWD-JSdb KVS Database Model
 
Rob Tweed <rtweed@mgateway.com>  
11 December 2020, M/Gateway Developments Ltd [http://www.mgateway.com](http://www.mgateway.com)  

Twitter: @rtweed

Google Group for discussions, support, advice etc: [http://groups.google.co.uk/group/enterprise-web-developer-community](http://groups.google.co.uk/group/enterprise-web-developer-community)

# About this Document

This document provides background information on the *Key/Value Store (KVS)* NoSQL database model that
is included with QEWD-JSdb.

# About the KVS Database Model

The *KVS* database model is inspired by the [Sets and Hashes data types](https://redis.io/topics/data-types)
 in the Redis NoSQL database.

It's a pretty simple data model, but rather than just being a simple key/value store,
it effectively implements a key/object store with which you can save and retrieve JavaScript objects.
Retrieval of stored objects is via either a key or a field that you have opted to index.

Before proceeding it is recommended that you first read and complete the [basic QEWD-JSdb tutorial](./REPL.md).  This will ensure you 
understand what's going on in QEWD-JSdb and why!

## Source Code for the *KVS* APIs

Like all the QEWD-JSdb APIs, they are written in JavaScript and are built on top of the
standard QEWD-JSdb APIs.  They are all Open Source APIs, and
you are free to inspect and use the code as you wish, in accordance with the Apache 2 license under
which they are made available.

Find the [KVS source code here](https://github.com/robtweed/ewd-document-store/tree/master/lib/proto/kvs).

## Enabling Use of the KVS APIs

You can create and maintain a *Key/Value Store (KVS)* at any QEWD-JSdb Document Node Object.

Having instantiated the Document Object Node object, you must enable its use of the *KVS* APIs.
For example, from within a QEWD-Up back-end message handler module:

        doc = this.documentStore.use('jsdbKvs', 'demo')
        doc.enable_kvs()

or when using the *jsdb_shell* REPL module:

        doc = jsdb.use('jsdbKvs', 'demo')
        doc.enable_kvs()

The Document Node object will be augmented with a *kvs* property object, via which you can invoke
the *KVS* APIs, eg:

        var key = 123;
        var record = {a: 1, b: 2};

        doc.kvs.add(key, record);


## Adding records to the KVS

Records are added to the *KVS* using the *add()* method.  This takes two arguments:

- key: an alphanumeric value that specifies a key
- record: a JavaScript object

You can only use the key once.  Attempting to
add a record using a key that is already in use in the *KVS* will return *false* and no
action will take place.

Try the following:

        var key = 123;
        var record = {a: 1, b: 2};
        doc.kvs.add(key, record);

In YottaDB, you'll see this has created:

        YDB>zwr ^jsdbKvs

        ^jsdbKvs("demo","content",123,"a")=1
        ^jsdbKvs("demo","content",123,"b")=2


## Editing records in the KVS

The object saved against a key can be edited *in-situ* in the *KVS* by using the *edit()* method.
This takes two arguments:

- key: an alphanumeric value that specifies a key that must already exist in the *KVS*
- record: a JavaScript object that will replace the one that already exisis

Try the following:

        record = {foo: 'bar'};
        doc.kvs.edit(123, record);

In YottaDB, you'll see:

        YDB> zwr ^jsdbKvs

        ^jsdbKvs("demo","content",123,"foo")="bar"

If the key does not exist in the *KVS*, *false* is returned.


## Removing records from the KVS

You can remove a record from the KVS by using the *delete()* method.
This takes a single argument:

- key: an alphanumeric value that specifies a key that must already exists in the *KVS*

Try the following:

        doc.kvs.delete(123);

In YottaDB, you'll see that the ^jsdbKvs Global is now empty.


Specifying a key value that is not currently in use in the *KVS* will return *false*.


## Retrieving a record from the KVS

You can retrieve a record from the KVS by using the *get_by_key()* method.
This takes a single argument:

- key: an alphanumeric value that specifies a key that must already exists in the *KVS*

Try the following:

        record = {a: 1, b: 2};
        doc.kvs.add(123, record);

        var data = doc.kvs.get_by_key(123)

        // {a: 1, b: 2}

Specifying a key value that is not currently in use in the *KVS* will return an empty object.


## Adding an Index to the KVS

By default, you can only retrieve records by its key.  However, you can optionally define any or all
of the field names in the saved objects to be indexed.  There are actually two steps to setting
up an index:

- specify a property name to be an indexed field.  Note that only 1st-level property names in the
saved record objects can be specified as indexed fields.  Once a field has been specified as an index, 
whenever any new records are added, if they contain a value for that field, an index record will be
added to the *KVS*

- if you have specified a new index field, but records already exist in the database, they won't
automatically be re-indexed.  You must invoke the *reindex()* method to force a re-indexing of all existing
records. 

Try the following:

        doc.delete()
        doc.kvs.addIndex('town');

The *addIndex()* method specifies that any new records with a property of *town* will be
automatically indexed.


In YottaDB you'll see:

        YDB> zwr ^jsdbKvs

        ^jsdbKvs("demo","indices","town","index")="true"


Now do this:

        var obj = {
          name: 'Rob',
          town: 'Redhill'
        };
        var status = doc.kvs.add("key_1", obj);
        console.log('added: ' + status);
        obj = {
          name: 'Simon',
          town: 'St Albans'
        };
        status = doc.kvs.add("key_2", obj);
        console.log('added: ' + status);


In YottaDB you'll now see the following:

        YDB> zwr ^jsdbKvs

        ^jsdbKvs("demo","content","key_1","name")="Rob"
        ^jsdbKvs("demo","content","key_1","town")="Redhill"
        ^jsdbKvs("demo","content","key_2","name")="Simon"
        ^jsdbKvs("demo","content","key_2","town")="St Albans"
        ^jsdbKvs("demo","index_by","town","Redhill","key_1")=""
        ^jsdbKvs("demo","index_by","town","St Albans","key_2")=""
        ^jsdbKvs("demo","indices","town","index")="true"


You can now find records in the *KVS* using the index, by using the *key_by_index()* method.
This takes two mandatory arguments and one optional one:

- the name of the index, eg "town".  This must be an index name that has been previously defined.
- the value to be searched for indexed records, eg "Redhill"
- if set to true, the method will return the data for each matching key.  By default, only an array of matching keys is returned

Try the following:

        doc.kvs.get_by_index('town', 'Redhill');
        // [ 'key_1' ]

        doc.kvs.get_by_index('town', 'Redhill', true);
        // { key_1: { name: 'Rob', town: 'Redhill' } }


You can also optionally set a modifier when defining an index. The most common one
to use is *toLowerCase*.  You'll see in the examples above that the indexed *town*
value is mixed case.  We could instead do the following:


        doc.delete()
        doc.kvs.addIndex('town', 'toLowerCase');
        obj = {
          name: 'Rob',
          town: 'Redhill'
        };
        doc.kvs.add("key_1", obj);
        obj = {
          name: 'Simon',
          town: 'St Albans'
        };
        status = doc.kvs.add("key_2", obj);

In YottaDB you'll now see:

        YDB> zwr ^jsdbKvs

        ^jsdbKvs("demo","content","key_1","name")="Rob"
        ^jsdbKvs("demo","content","key_1","town")="Redhill"
        ^jsdbKvs("demo","content","key_2","name")="Simon"
        ^jsdbKvs("demo","content","key_2","town")="St Albans"
        ^jsdbKvs("demo","index_by","town","redhill","key_1")=""
        ^jsdbKvs("demo","index_by","town","st albans","key_2")=""
        ^jsdbKvs("demo","indices","town","index")="true"
        ^jsdbKvs("demo","indices","town","transform")="toLowerCase"

Notice how the town index records are in lower case, but the *content*
records hold the original mixed-case values.


## Seeing what fields are specified as Indexed fields

You can discover which fields are index fields by using the *getIndices()* method.
This returns an array of index names:

        doc.kvs.getIndices()
        // [ 'town' ]


## Removing an Index Field

You may want to stop indexing by a field, for example if you'd added one by mistake.

As when you added an index, this is usually a two step process:

- specify the field to be deleted as an index field
- re-index the *KVS* to remove any existing index records for that field.

        doc.kvs.deleteIndex('town')
        doc.kvs.reindex()


