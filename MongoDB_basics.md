# MongoDB Basics

Author: Andy Wenk  
Version: 0.0.2  
Date: 03.11.2012  

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

This part is showing the basic usage of the MongoDB query inteface. And **no** - it’s not like SQL. There are various methods and query-operators. Basically every method and operator is expecting a document as a parameter, which is represented as a JSON string or nothing at all.

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
	
### Creating a document with **insert**

Creating documents is basically the task to put one or more JSON string(s) into a collection.

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

### Removing a document with **remove**

The remove() method is basically the same as find() (see next). It is expecting a JSON string with information about which document to remove from the collection. The most obvious way is to remove a documetn by its _id:

	> db.people.find()
	{ "_id" : ObjectId("509588e8cbba747f461b00ab"), "name" : "Carlos Santana", "profession" : 		"guitarist", "music-genre" : [ "rock", "latin" ] }
	{ "_id" : ObjectId("509588e8cbba747f461b00ac"), "name" : "Stevie Ray Vaughn", "profession" : "guitarist", 		"music-genre" : [ "rock", "blues" ] }
	{ "_id" : ObjectId("509589a21b310876b948aade"), "name" : "Jimi Hendrix", "profession" : 		"guitarist", "music-genre" : [ "rock", "blues" ] }
	{ "_id" : ObjectId("50958a631b310876b948aadf"), "name" : "Zakk Wylde", "profession" : 		"guitarist", "music-genre" : [ "metal", "rock" ] }
	> db.people.remove({_id: ObjectId("509588e8cbba747f461b00ac")});
	> db.people.find().count()
	3

### Querying documents with **find**

As already shown before, documents ar retrieved from a collection by firing the method find(). If no arguments given, find() will return all documents from the collection. A nice helper is the method pretty() which can be chained to find() or any other method invocation. This will print the resulting JSON strings in a nicely readable format.

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


	
#### Resources

1 [https://education.10gen.com/courses/](https://education.10gen.com/courses/)  
2 [http://mxcl.github.com/homebrew/](http://mxcl.github.com/homebrew/)  
3 [http://www.mongodb.org/downloads](http://www.mongodb.org/downloads)  
4 [http://bsonspec.org/#/specification](http://bsonspec.org/#/specification)