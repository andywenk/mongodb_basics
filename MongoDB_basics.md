# MongoDB Basics

Author: Andy Wenk  
Version: 0.4.0  
Date: 08.11.2012  

## Abstract

This is a summary of MongoDB basics. I decided to put them together, while following the *M101: MongoDB for Developers* course given by 10gen [1].

## Installing MongoDB

Assuming you are using Mac OS X, the easiest way is to install MongoDB is by using homebrew [2]:

    $ brew install mongodb
    
Alternatively, you can find downloads for all supported OS at [3].

## Running the MongoDB Server

Simple as that:

    $ mongod
    Sat Nov  3 20:43:41 [initandlisten] MongoDB starting : pid=37667 port=27017 dbpath=/  
      usr/local/var/mongodb 64-bit host=your-machine 
    Sat Nov  3 20:43:41 [initandlisten] db version v2.2.0, pdfile version 4.5
    Sat Nov  3 20:43:41 [initandlisten] git version: 
      f5e83eae9cfbec7fb7a071321928f00d1b0c5207
    [...]  

## Used document format

MongoDB is using document based datastore engine. The documents are stored in a binary JSON format called BSON. The specification of BSON can be found at [4]. 

## Running the MongoDB console

MongoDB is providing a good JavaScript CLI. You can simply run the console by typing:

    $ mongo
    MongoDB shell version: 2.2.0
    connecting to: test
    >

MongoDB is organizing its documents in different databases with collections. One, which is always available initially is called *test*.

## Basic MongoDB console usage

There are some simple basics for the MongoDB console usage. First of all, ther is *help*:

    > help
	    db.help()				help on db methods
	    db.mycoll.help()		help on collection methods
	    sh.help()				sharding helpers
	    rs.help()				replica set helpers
	    help admin				administrative help
	    help connect			connecting to a db help
	    help keys				key shortcuts
 	    help misc				misc things to know
	    help mr					mapreduce

	    show dbs				show database names
	    show collections		show collections in current database
	    show users				show users in current database
	    show profile			show most recent system.profile entries with 
	    						time >= 1ms
	    show logs				show the accessible logger names
	    show log [name]			prints out the last segment of 
	    						log in memory, 'global' is default
	    use <db_name>			set current database
	    db.foo.find()			list objects in collection foo
	    db.foo.find( { a : 1 } ) 	list objects in foo where a == 1
	    it						result of the last line evaluated; 
	    						use to further iterate
	    DBQuery.shellBatchSize = x   set default number of items to display 
	    						on shell
	    exit					quit the mongo shell
    
### show dbs

One of the most used commands will be *show dbs*:    

	> show dbs
	local	(empty)
	m101	0.203125GB
	students	0.203125GB
	test	0.203125GB

### use <db_name>

And secondly the *use <db_name>* command, to change to a specific db:

	> use students
	switched to db students

### show collections

To see which collections a db has, *show collections* is used:

	> show collections
	grades
	system.indexes

## The query interface

This part is showing the basic usage of the MongoDB query inteface. And **no** - it’s not like SQL. There are various methods and query-operators. Basically every method and operator is expecting a document as a parameter, which is represented as a JSON-like string or nothing at all.

### create a database

As seen before, each MongoDB instance (or cluster) can have many databases. There is no real command for creating a database. The database is created, when the first document is created. So here, a document is created in the collection (see *collections*) people which is belonging to the database earth. earth does not exist yet:

	> use earth
	switched to db earth
	> show dbs
	local	(empty)
	m101	0.203125GB
	students	0.203125GB
	test	0.203125GB

See: earth is not created yet!

	> db
	earth
	> db.people.insert({"name":"Zakk Wylde","profession":"guitarist","music-genre":
  		["metal","rock"]})

When firing this query, the database earth is created and the collection people is created with the first document.

### collections

The documents in a database are organized in collections. Each database can have *n* collections. In the following it is assumed, that there is a collection called *people* in a database called *earth*.

	> db
	earth
	> show collections
	people
	system.indexes
	> db.people.find().pretty()
	{
		"_id" : ObjectId("50957f941b310876b948aadc"),
		"name" : "Zakk Wylde",
		"profession" : "guitarist",
		"music-genre" : [
			"metal",
			"rock"
		]
	} 

One document is found in the collection people of the database earth.	
	
### Creating a document with **insert()**
URL: [http://docs.mongodb.org/manual/reference/method/db.collection.insert/](http://docs.mongodb.org/manual/reference/method/db.collection.insert/)

Creating documents is basically the task to put one or more JSON-like string(s) into a collection.

Adding one document:

	> db.people.insert(
	{
		"name" : "Jimi Hendrix", "profession" : "guitarist",
		"music-genre" : [ "rock", "blues" ] 
	})
	
Adding more than one documents:

	> db.people.insert(
	[
		{
			"name" : "Carlos Santana", "profession" : "guitarist", 
			"music-genre" : [ "rock", "latin" ] 
		},
		{
			"name" : "Stevie Ray Vaughn", "profession" : "guitarist", 
			"music-genre" : [ "rock", "blues" ] 
		}
	])

When now firing 

	> db.people.find()

there should exist four documents with great guitar players in the the people collection.

### Updating a document with **update()**
URL: [http://docs.mongodb.org/manual/reference/method/db.collection.update/](http://docs.mongodb.org/manual/reference/method/db.collection.update/)

The simplest way to update a document is rewriting the whole document by _id:

    > db.people.update(
    	{
    		"_id": ObjectId("509588e8cbba747f461b00ab")
    	},
    	{
    		"name" : "Carlos Santana", 
    		"profession" : "guitarist", 
    		"music-genre" : [ "rock", "latin" ],
    		"score":100})
		}
	)
	
But this approach is kind of lame. So the better way is to update (or add if not existing) a specific key’s value. This can be done with the operator *$set*:	
	
	> db.people.update(
		{
			"_id":ObjectId("509589a21b310876b948aade")
		},
		{
			$set: {"score": 400}
		}
	)

The first parameter does not have to be an _id but can also include some criteria like 

	{"name": "Carlos Santana"}

or also operators used in find(). 

There are two options which can be used as the third parameter:

	multi: true -> update many documents
	upsert: true -> if the document does not exist, create it.

There is also a operator called *$inc* which will increase e.g. score to the given value in *$inc*:

	> db.people.update(
		{
			"name": "Carlos Santana")
		},
		{
			$inc: {"score": 20}
		}
	)

The score is now 420.

Finally there is the operator *$unset* to remove one or more fields from one or more documents:

	> db.people.update(
		{
			'name': 'Zakk Wylde'
		}, 
		{
			$unset: {‘score': 1}
		}
	)

Adding new data to all available documents requires to set the *multi* option. An empty document {} passed as the first parameter is denoting all documents in the collection:

	> db.people.update({}, {$set: {age: 35} }, {multi: true})

### Removing a document with **remove()**
URL: [http://docs.mongodb.org/manual/reference/method/db.collection.remove/](http://docs.mongodb.org/manual/reference/method/db.collection.remove/)

The remove() method is basically the same as find() (see next). It is expecting a document with information about which document to remove from the collection. The most obvious way is to remove a documetn by its _id:

	> db.people.find()
	{ 	"_id" : ObjectId("509588e8cbba747f461b00ab"), 
		"name" : "Carlos Santana", 
		"profession" : "guitarist",
		 "music-genre" : [ "rock", "latin" ] }
	{ 	"_id" : ObjectId("509588e8cbba747f461b00ac"), 
		"name" : "Stevie Ray Vaughn", 
		"profession" : "guitarist",
		"music-genre" : [ "rock", "blues" ] }
	{ 	"_id" : ObjectId("509589a21b310876b948aade"), 
		"name" : "Jimi Hendrix", 
		"profession" :"guitarist", 
		"music-genre" : [ "rock", "blues" ] }
	{ 	"_id" : ObjectId("50958a631b310876b948aadf"), 
		"name" : "Zakk Wylde", 
		"profession" : "guitarist", 
		"music-genre" : [ "metal", "rock" ] }
	> db.people.remove({_id: ObjectId("509588e8cbba747f461b00ac")});
	> db.people.find().count()
	3

### Querying documents with findOne()
URL: [http://docs.mongodb.org/manual/reference/method/db.collection.findOne/](http://docs.mongodb.org/manual/reference/method/db.collection.findOne/)

The findOne() method is a helper method for find() and will return exactly one or zero documents. It expects a document with the query values as a first parameter. The second parameter is denoting which values should be returned:

	> db.people.findOne({"name": "Carlos Santana"}, {"profession": true, "_id": false})
	{ "profession" : "guitarist" }  

### Querying documents with **find()**
URL: [http://docs.mongodb.org/manual/reference/method/db.collection.find/](http://docs.mongodb.org/manual/reference/method/db.collection.find/)

As already shown before, documents are retrieved from a collection by firing the method find(). If no arguments given, find() will return all documents from the collection. A nice helper is the method pretty() which can be chained to the find() method invocation. This will print the resulting documents in a nicely readable format.

There are various ways to query the collection. basically the documents can be queried with each key available. So if one wants to find the document (or all documents) where the name is “Jimi Hendrix”, the query would look like this:

	> db.people.find({"name":"Jimi Hendrix"})
	{ 
		"_id" : ObjectId("509589a21b310876b948aade"), 
		"name" : "Jimi Hendrix", 
		"profession" : 	"guitarist", 
		"music-genre" : [ "rock", "blues" ] 
	}

This will only work, if the excat spelling of the name is given in the query (see $regex later for other possibilities). 

The documents key music-genre is holding a list (array) with different values. It’s also possible to query this:

	> db.people.find({"music-genre":"latin"}).pretty()
	{
		"_id" : ObjectId("509588e8cbba747f461b00ab"),
		"name" : "Carlos Santana",
		"profession" : "guitarist",
		"music-genre" : [
			"rock",
			"latin"
		]
	}

### Using $-operators in find()

There are various operators available which are very useful for advanced queries.

#### $gt and $gte (greater than and greater or equals than)

	> db.people.find({score: {$gt: 200}},{name: true, score: true, _id: false})
	{ "name" : "Jimi Hendrix", "score" : 400 }
	
	> db.people.find({score: {$gte: 200}},{name: true, score: true, _id: false}).pretty()
	{ "name" : "Carlos Santana", "score" : 200 }
	{ "name" : "Jimi Hendrix", "score" : 400 }

#### $lt and $lte (lower than and lower or equals than) 

	> db.people.find({score: {$lte: 200}},{name: true, score: true, _id: false}).pretty()
	{ "name" : "Carlos Santana", "score" : 200 }
	
	> db.people.find({score: {$lt: 200}},{name: true, score: true, _id: false}).pretty()
	// nothing
	
#### $regex, $exists
	
	> db.people.find({"name":{$regex:"Carl"},"profession":{$exists:true}},{name: true, score: 
		true, _id: false})
	{ "name" : "Carlos Santana", "score" : 200 }

#### $type	

$type is the data type specified in the BSON specification [4]. Here is a short excerpt:

	element	::=	"\x01" e_name double	Floating point
			|	"\x02" e_name string	UTF-8 string
			|	"\x03" e_name document	Embedded document
			|	"\x04" e_name document	Array
			|	"\x05" e_name binary	Binary data
			|	"\x06" e_name	Undefined — Deprecated
			|	"\x07" e_name (byte*12)	ObjectId
			|	"\x08" e_name "\x00"	Boolean "false"
			|	"\x08" e_name "\x01"	Boolean "true"
			|	"\x09" e_name int64	UTC datetime
			|	"\x0A" e_name	Null value
			|	"\x0B" e_name cstring cstring	Regular expression
			|	"\x0C" e_name string (byte*12) DBPointer (Deprecated)
			|	"\x0D" e_name string	JavaScript code
			|	"\x0E" e_name string	Symbol — Deprecated
			|	"\x0F" e_name code_w_s	JavaScript code w/ scope
			|	"\x10" e_name int32	32-bit Integer
			|	"\x11" e_name int64	Timestamp
			|	"\x12" e_name int64	64-bit integer
			|	"\xFF" e_name	Min key
			|	"\x7F" e_name	Max key

Here is an exmple for string:

	> db.people.find({name: {$type: 2}},{name: true, score: true, _id: false})
	{ "name" : "Carlos Santana", "score" : 200 }
	{ "name" : "Zakk Wylde", "score" : 600 }
	{ "name" : "Jimi Hendrix", "score" : 400 }		
	
#### $or

	> db.people.find({$or : [{score: {$lt: 400}},{score: {$gt:200}}] },
		{name: true, score: true, 	_id: false})
	{ "name" : "Carlos Santana", "score" : 200 }
	{ "name" : "Zakk Wylde", "score" : 600 }
	{ "name" : "Jimi Hendrix", "score" : 400 }

#### $and

	> db.people.find({$and : [{score: {$lte: 400}},{score: {$gt:200}}] },
		{name: true, score: true, _id: false})
	{ "name" : "Jimi Hendrix", "score" : 400 }
	
#### $in

This is kind of the IN Operator in SQL.

	> db.people.find({'music-genre': {$in : ['latin','metal']}},
		{name: true, score: true, _id: false})
	{ "name" : "Carlos Santana", "score" : 200 }
	{ "name" : "Zakk Wylde", "score" : 600 }

#### $all

	> db.people.find({'music-genre': {$all : ['rock']}},
		{name: true, score: true, _id: false})
	{ "name" : "Carlos Santana", "score" : 200 }
	{ "name" : "Zakk Wylde", "score" : 600 }
	{ "name" : "Jimi Hendrix", "score" : 400 }

But see this:

	> db.people.find({'music-genre': {$all : ['rock', 'latin']}},
		{name: true, score: true, _id: false})
	{ "name" : "Carlos Santana", "score" : 200 }

### Combining $-operators in find()

You can combine all these Operators - as already shown with $lt and $gt. 

### Querying embedded documents with dot notation

It is possible to query also embedded subdocuments. Here is such a document:

	> db.people.update({name:'Zakk Wylde'},{$set: {bands: [{name: 'BLS', year: 2000},
		{name:'Ozzy', year: 1998},{name: 'Pride and Glory'}]}}) 
	> db.people.find({name:'Zakk Wylde'})
	{
		"_id" : ObjectId("50958a631b310876b948aadf"),
		"bands" : [
		{
			"name" : "BLS",
			"year" : 2000
		},
		{
			"name" : "Ozzy",
			"year" : 1998
		},
		{
			"name" : "Pride and Glory"
		}
		],
		"music-genre" : [
			"metal",
			"rock"
		],
		"name" : "Zakk Wylde",
		"profession" : "guitarist",
		"score" : 600
	}

And here is a chained query where just the year of the band should be come back:

	> db.people.find({'bands.name': 'BLS'},{'bands.year':true})
	{
		"_id" : ObjectId("50958a631b310876b948aadf"),
		"bands" : [
			{
				"year" : 2000
			},
			{
				"year" : 1998
			},
			{
			
			}
		]
	}

### Querying documents with cursors

cursors are a convenient way to assign the result of a query to a variable - this variable is called cursor. 

	> cur = cur = db.people.find(); null;
	null

On the mongo shell you need to return something different than the cursor itself, because otherwiese the content of the cursor is returned and the cursor is “empty”.

First document from the cursor:

	> cur.next()
	{
		"_id" : ObjectId("509588e8cbba747f461b00ab"),
		"name" : "Carlos Santana",
		"profession" : "guitarist",
		"music-genre" : [
			"rock",
			"latin"
		],
		"score" : 200
	}

Check if there more docuements hold by the cursor.

	> cur.hasNext()
	true

### Manipulating the result of cursors

With sort, skip and limit the result of a cursor can be manipulated. If chained, the execution order of the three methods is always: **sort - skip - limit**

#### sort

ASC

	> cur = db.people.find(); null;
	null
	> cur.sort({'name': 1})
	// Carlos Santana, Jimi Hendrix, Zakk Wylde
	
DESC

	> cur = db.people.find(); null;
	null
	> cur.sort({'name': -1})
	// Zakk Wylde, Jimi Hendrix, Carlos Santana 

#### skip
	
	> cur = db.people.find(); null;
	null
	> cur.skip(2)
	// only Zakk Wylde returned

#### limit 

	> cur = db.people.find(); null;
	null
	> cur.limit(2); null;
	null
	> cur.count()
	3
	> cur
	{ "_id" : ObjectId("509588e8cbba747f461b00ab"), "name" : "Carlos Santana", "profession" : "guitarist", 	"music-genre" : [ "rock", "latin" ], "score" : 200 }
	{ "_id" : ObjectId("509589a21b310876b948aade"), "music-genre" : [ "rock", "blues" ], 	"name" : "Jimi Hendrix", "profession" : "guitarist", "score" : 400 }
	
### Querying Arrays inside a document with operators

	> db.people.insert({_id:'Joe',name: 'Joe Satriani'})
	{ "_id" : "Joe", "name" : "Joe Satriani" }

#### push

	> db.people.update({_id:'Joe'}, {$push: {guitars: 'Ibanez'}})
	> db.people.find({'_id':'Joe'})
	{ "_id" : "Joe", "guitars" : [ "Ibanez" ], "name" : "Joe Satriani" }

#### pop

	> db.people.update({_id:'Joe'}, {$pop: {guitars: 'Ibanez'}})
	> db.people.find({'_id':'Joe'})
	{ "_id" : "Joe", "guitars" : [ ], "name" : "Joe Satriani" }
	
#### pushAll
	> db.people.update({_id:'Joe'}, {$pushAll: {guitars: ['Ibanez','Gibson','Martin']}})
	> db.people.find({'_id':'Joe'})
	{ "_id" : "Joe", "guitars" : [ "Ibanez", "Gibson", "Martin" ], "name" : "Joe Satriani" }

#### pull

	> db.people.update({_id:'Joe'}, {$pull: {guitars: 'Gibson'}})
	> db.people.find({'_id':'Joe'})
	{ "_id" : "Joe", "guitars" : [ "Ibanez", "Martin" ], "name" : "Joe Satriani" }

#### pullAll
	
	> db.people.update({_id:'Joe'}, {$pullAll: {guitars: ['Ibanez','Martin']}})
	> db.people.find({'_id':'Joe'})
	{ "_id" : "Joe", "guitars" : [ ], "name" : "Joe Satriani" }
	
#### addToSet 
	
	> db.people.update({_id:'Joe'}, {$addToSet: {guitars: 'Gibson'}})
	> db.people.find({'_id':'Joe'})
	{ "_id" : "Joe", "guitars" : [ "Gibson" ], "name" : "Joe Satriani" }

### Indexes

Using Indexes is strongly recommended to not run into performance issues. As MongoDB can handle millions of documents, indexes are vital for a good performance.

#### create an index

	> db.people.ensureIndex({name:1})
	> db.people.getIndexes()
	[
		{
			"v" : 1,
			"key" : {
				"_id" : 1
			},
			"ns" : "earth.people",
			"name" : "_id_"
		},
		{
			"v" : 1,
			"key" : {
				"name" : 1
			},
			"ns" : "earth.people",
			"name" : "name_1"
		}
	]
	
#### drop an index

	> db.people.dropIndex({name:1})
	{ "nIndexesWas" : 2, "ok" : 1 }

#### create many indexes

	> db.people.ensureIndex({name:1,profession:1})
	> db.people.getIndexes()
    [
      {
        "v" : 1,
        "key" : {
          "_id" : 1
        },
        "ns" : "earth.people",
        "name" : "_id_"
      },
      {
        "v" : 1,
        "key" : {
          "name" : 1,
          "profession" : 1
        },
        "ns" : "earth.people",
        "name" : "name_1_profession_1"
      }
    ]

#### create unique indexes

This means, the value for the indexed key must be unique. If there are already documents conatining a key with the same value,
the index will not be created. 

    > db.people.createIndex({name:1},{unique:true})
    > db.people.getIndexes()
    [
      {
        "v" : 1,
        "key" : {
          "_id" : 1
        },
        "ns" : "earth.people",
        "name" : "_id_"
      },
      {
        "v" : 1,
        "key" : {
          "name" : 1
        },
        "ns" : "earth.people",
        "name" : "name_1",
        "uniuqe" : true
      }
    ]

If you want to force the uniqueness of the indexed keys, you can drop the dupes:

    > db.people.insert({"name" : "Carlos Santana", "profession" : "guitarist", 
    "music-genre" : [ "rock", "latin" ], "score" : 200 })
    > db.people.createIndex({name:1},{unique:true})
    E11000 duplicate key error index: earth.people.$name_1  dup key: { : "Carlos Santana" }
    > db.people.createIndex({name:1},{unique:true, dropDups:true})

But TAKE CARE! You don’t know which document is beeing dropped!

#### creating sparse indexes

You can only create an index for a key, if every document has this key. If you still want to create an index even though there are documents without that key, you need to create a sparse index

Here are some robots:

    > db.robots.insert({robot:'walle',engine:'fuel',size:'50cm'})
    > db.robots.insert({robot:'Ash',engine:'battery',size:'180cm'})
    > db.robots.insert({robot:'HAL',engine:'atomic'})
    > db.robots.insert({robot:'T-800',engine:'battery'})

If trying to create an index on the key size will fail:

    > db.robots.ensureIndex({size:1},{unique:true})
    E11000 duplicate key error index: earth.robots.$size_1  dup key: { : null }

It works when using sparse:true:

    > db.robots.ensureIndex({size:1},{unique:true,sparse:true})

But be aware, that only the documents with the key size are in the index. So if you use an aggregate function like sort(), you will only sort the documents in the index (here only two documents are found):

    > db.robots.find().sort({size:1})
    { "_id" : ObjectId("50b15e6b2bde811ff29731fe"), "robot" : "Ash", "engine" : "battery", "size" : "180cm" }
    { "_id" : ObjectId("50b15e332bde811ff29731fd"), "robot" : "walle", "engine" : "fuel", "size" : "50cm" }	

#### when is the index used - explain()

There are some things to know, when an index - especially a compound index - is used. First some data:

	> for (i = 0; i < 1000000; i++) { db.many.insert({a:i,b:i,c:i}) }
    > db.many.find().count()
    1000000

Now creating the compound index:

    > db.many.ensureIndex({a:1,b:1,c:1})
    > db.many.getIndexes()
    [
      {
        "v" : 1,
        "key" : {
          "_id" : 1
        },
        "ns" : "earth.many",
        "name" : "_id_"
      },
      {
        "v" : 1,
        "key" : {
          "a" : 1,
          "b" : 1,
          "c" : 1
        },
        "ns" : "earth.many",
        "name" : "a_1_b_1_c_1"
      }
    ]

Finding a document where the key is a. explain() shows, what is happening during the query. Here, the index is used:

    > db.many.find({a:50}).explain()
    {
      "cursor" : "BtreeCursor a_1_b_1_c_1",
      "isMultiKey" : false,
      "n" : 1,
      "nscannedObjects" : 1,
      "nscanned" : 1,
      "nscannedObjectsAllPlans" : 1,
      "nscannedAllPlans" : 1,
      "scanAndOrder" : false,
      "indexOnly" : false,
      "nYields" : 0,
      "nChunkSkips" : 0,
      "millis" : 0,
      "indexBounds" : {
        "a" : [
          [
            50,
            50
          ]
        ],
        "b" : [
          [
            {
              "$minElement" : 1
            },
            {
              "$maxElement" : 1
            }
          ]
        ],
        "c" : [
          [
            {
              "$minElement" : 1
            },
            {
              "$maxElement" : 1
            }
          ]
        ]
      },
      "server" : "MBP-andwen.local:27017"
    }

Here also - a and b are used in the index query:

    > db.many.find({a:50,b:50}).explain()
    {
      "cursor" : "BtreeCursor a_1_b_1_c_1",
      ...

And here also but only a is used in the index:

    > db.many.find({a:50,c:50}).explain()
    {
      "cursor" : "BtreeCursor a_1_b_1_c_1",
      ...

BUT - here not because querying the index requires the leftmost index key provided in the query:

    > db.many.find({b:50}).explain()
    {
      "cursor" : "BasicCursor",
      "isMultiKey" : false,
      "n" : 1,
      "nscannedObjects" : 1000000,
      "nscanned" : 1000000,
      "nscannedObjectsAllPlans" : 1000000,
      "nscannedAllPlans" : 1000000,
      "scanAndOrder" : false,
      "indexOnly" : false,
      "nYields" : 1,
      "nChunkSkips" : 0,
      "millis" : 653,
      "indexBounds" : {
        
      },
      "server" : "MBP-andwen.local:27017"
    }

So the last query takes 653 milli seconds instead of 0 milli seconds. Wow ... 

## Resources

1 [https://education.10gen.com/courses/](https://education.10gen.com/courses/)  
2 [http://mxcl.github.com/homebrew/](http://mxcl.github.com/homebrew/)  
3 [http://www.mongodb.org/downloads](http://www.mongodb.org/downloads)  
4 [http://bsonspec.org/#/specification](http://bsonspec.org/#/specification)