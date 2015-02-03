# HTML5 - Storage - IndexedDB

## Summary
The summary for the IndexedDB. For the key-point of basic design, and common use pattern.

The detail concept and code example could check in later chapter.

Check availability of indexedDB.
http://caniuse.com/#feat=indexeddb
* not supported in safari until iOS 8 safari, and chrome android 38(the latest version is 39)

### Key concepts
* IndexedDB databases store **key-value pairs**
* IndexedDB is built on a **transactional** database model.
* The IndexedDB API is mostly **asynchronous**.  
Use callback
* IndexedDB uses **request** all over the place.  
Requests are objects that receive the success or failure DOM events that were mentioned previously. They have `onsuccess` and `onerror` properties, and you can call `addEventListener()`` and `removeEventListener()`` on them. They also have readyState, result, and errorCode properties that tell you the status of the request. The result property is particularly magical, as it can be many different things, depending on how the request was generated (for example, an IDBCursor instance, or the key for a value that you just inserted into the database).
* IndexedDB uses **DOM events** to notify you when results are available.
* IndexedDB is **object-oriented**.
* IndexedDB does not use Structured Query Language (SQL).  
**NoSQL**
* IndexedDB adheres to a **same-origin** policy

WebSQL Database is a relational database access system, whereas IndexedDB is an indexed table system.  

### Basic pattern
* Open a database.
* Create an object store in upgrading database.
* Start a transaction and make a request to do some database operation, like adding or retrieving data.
* Wait for the operation to complete by listening to the right kind of DOM event.
* Do something with the results (which can be found on the request object).



## Constructs

### Database
A database's origin is the same as the origin of the document or worker. Each origin has an associated set of databases.

The `IDBDatabase` interface represents a connection to a database.

* Name  
This identifies the database within a specific origin and stays constant throughout its lifetime. The name can be any string value (including an empty string).
* Current version  
When a database is first created, its version is the integer 1 if not specified otherwise. Each database can have only one version at any given time.


### Object store


### Request
The operation by which reading and writing on a database is done. Every request represents one read or write operation.


### Key and value
* key generator  
A mechanism for producing new keys in an ordered sequence. If an object store does not have a key generator, then the application must provide keys for records being stored. Generators are not shared between stores. This is more a browser implementation detail, because in web development, you don't really create or access key generators.
* in-line key  
A key that is stored as part of the stored value. It is found using a key path. An in-line key can be generated using a generator. After the key has been generated, it can then be stored in the value using the key path or it can also be used as a key.
* out-of-line key  
A key that is stored separately from the value being stored.
* key path  
Defines where the browser should extract the key from in the object store or index. A valid key path can include one of the following: an empty string, a JavaScript identifier, or multiple JavaScript identifiers separated by periods. It cannot include spaces.
* value  
Each record has a value, which could include anything that can be expressed in JavaScript, including boolean, number, string, date, object, array, regexp, undefined, and null.

### Range and scope
The set of **object stores** and indexes to which a transaction applies. The scopes of read-only transactions can overlap and execute at the same time. On the other hand, the scopes of writing transactions cannot overlap. You can still start several transactions with the same scope at the same time, but they just queue up and execute one after another.


## Using indexedDB


### Basic pattern
* Open a database.
* Create an object store in upgrading database.
* Start a transaction and make a request to do some database operation, like adding or retrieving data.
* Wait for the operation to complete by listening to the right kind of DOM event.
* Do something with the results (which can be found on the request object).

### open database

```js
// In the following line, you should include the prefixes of implementations you want to test.
window.indexedDB = window.indexedDB || window.mozIndexedDB || window.webkitIndexedDB || window.msIndexedDB;
// DON'T use "var indexedDB = ..." if you're not in a function.
// Moreover, you may need references to some window.IDB* objects:
window.IDBTransaction = window.IDBTransaction || window.webkitIDBTransaction || window.msIDBTransaction;
window.IDBKeyRange = window.IDBKeyRange || window.webkitIDBKeyRange || window.msIDBKeyRange;
// (Mozilla has never prefixed these objects, so we don't need window.mozIDB*)

if (!window.indexedDB) {
    window.alert("Your browser doesn't support a stable version of IndexedDB. Such and such feature will not be available.");
}

// Let us open our database, 3 is the version of the database
var request = window.indexedDB.open("MyTestDatabase", 3);

````


```js
var db;
var request = indexedDB.open("MyTestDatabase");
request.onerror = function(event) {
  alert("Why didn't you allow my web app to use IndexedDB?!");
};
request.onsuccess = function(event) {
  db = event.target.result;
};
db.onerror = function(event) {
  // Generic error handler for all errors targeted at this database's
  // requests!
  alert("Database error: " + event.target.errorCode);
};
```

#### Creating or updating the version of the database
open existing database, `onupgradeneeded` event is triggered.
```js
// This event is only implemented in recent browsers
request.onupgradeneeded = function(event) {
  var db = event.target.result;

  // Create an objectStore for this database
  var objectStore = db.createObjectStore("name", { keyPath: "myKey" });
};
```


If the `onupgradeneeded` event exits successfully, the `onsuccess` handler of the open database request will then be triggered.


#### Structuring the database

```js
const dbName = "the_name";

var request = indexedDB.open(dbName, 2);

request.onerror = function(event) {
  // Handle errors.
};
request.onupgradeneeded = function(event) {
  var db = event.target.result;

  // Create an objectStore to hold information about our customers. We're
  // going to use "ssn" as our key path because it's guaranteed to be
  // unique - or at least that's what I was told during the kickoff meeting.
  var objectStore = db.createObjectStore("customers", { keyPath: "ssn" });

  // Create an index to search customers by name. We may have duplicates
  // so we can't use a unique index.
  objectStore.createIndex("name", "name", { unique: false });

  // Create an index to search customers by email. We want to ensure that
  // no two customers have the same email, so use a unique index.
  objectStore.createIndex("email", "email", { unique: true });

  // Use transaction oncomplete to make sure the objectStore creation is
  // finished before adding data into it.
  objectStore.transaction.oncomplete = function(event) {
    // Store values in the newly created objectStore.
    var customerObjectStore = db.transaction("customers", "readwrite").objectStore("customers");
    for (var i in customerData) {
      customerObjectStore.add(customerData[i]);
    }
  }
};
```

As indicated previously, `onupgradeneeded` is the only place where you can alter the structure of the database. In it, you can create and delete object stores and build and remove indices.


```
objectStore.createIndex(objectIndexName, objectKeypath, optionalObjectParameters);
```
https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore.createIndex

### Adding, retrieving and removing data

* When defining the scope, specify only the object stores you need. This way, you can run **multiple transactions** with **non-overlapping** scopes concurrently.
* Only specify a readwrite transaction mode when necessary. You can concurrently run multiple readonly transactions with overlapping scopes, but you can have only one readwrite transaction for an object store. To learn more, see the definition for transactions in the Basic Concepts article.

#### transaction scope
the scope, defined as **an array of object stores** that you want to access

```js
// define scope in []
var transaction = db.transaction([], IDBTransaction.READ_ONLY);

// all stores (equivalent to what use to be marked as empty array. )
var transaction = db.transaction(db.objectStoreNames, IDBTransaction.READ_ONLY);

// multiple stores:
var transaction = db.transaction(['ObjectStoreName1', 'ObjectStoreName2'],
    IDBTransaction.READ_ONLY);

// single store - these are equivalent
var transaction = db.transaction(['ObjectStoreName'], IDBTransaction.READ_ONLY);
var transaction = db.transaction('ObjectStoreName', IDBTransaction.READ_ONLY);
```

#### Adding data to the database
```js
var transaction = db.transaction(["customers"], "readwrite");
// Note: Older experimental implementations use the deprecated constant IDBTransaction.READ_WRITE instead of "readwrite".
// In case you want to support such an implementation, you can write:
// var transaction = db.transaction(["customers"], IDBTransaction.READ_WRITE);

// Do something when all the data is added to the database.
transaction.oncomplete = function(event) {
  alert("All done!");
};

transaction.onerror = function(event) {
  // Don't forget to handle errors!
};

var objectStore = transaction.objectStore("customers");
for (var i in customerData) {
  var request = objectStore.add(customerData[i]);
  request.onsuccess = function(event) {
    // event.target.result == customerData[i].ssn;
  };
}
```

#### Removing data from the database
```js
var request = db.transaction(["customers"], "readwrite")
                .objectStore("customers")
                .delete("444-44-4444");
request.onsuccess = function(event) {
  // It's gone!
};
```
#### Getting data from the database
```js
var transaction = db.transaction(["customers"]);
var objectStore = transaction.objectStore("customers");
var request = objectStore.get("444-44-4444");
request.onerror = function(event) {
  // Handle errors!
};
request.onsuccess = function(event) {
  // Do something with the request.result!
  alert("Name for SSN 444-44-4444 is " + request.result.name);
};
```

simple one:
```js
db.transaction("customers").objectStore("customers").get("444-44-4444").onsuccess = function(event) {
  alert("Name for SSN 444-44-4444 is " + event.target.result.name);
};
```
#### updating an entry in the database
```js
var objectStore = db.transaction(["customers"], "readwrite").objectStore("customers");
var request = objectStore.get("444-44-4444");
request.onerror = function(event) {
  // Handle errors!
};
request.onsuccess = function(event) {
  // Get the old value that we want to update
  var data = request.result;

  // update the value(s) in the object that you want to change
  data.age = 42;

  // Put this updated object back into the database.
  var requestUpdate = objectStore.put(data);
   requestUpdate.onerror = function(event) {
     // Do something with the error
   };
   requestUpdate.onsuccess = function(event) {
     // Success - the data is updated!
   };
};
```

#### Using a cursor
Get all items in the object store
```js
var objectStore = db.transaction("customers").objectStore("customers");

objectStore.openCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    alert("Name for SSN " + cursor.key + " is " + cursor.value.name);
    cursor.continue();
  }
  else {
    alert("No more entries!");
  }
};
```

#### Using an index

```js
var index = objectStore.index("name");
index.get("Donna").onsuccess = function(event) {
  alert("Donna's SSN is " + event.target.result.ssn);
};

```

two types of cursor
```js
// Using a normal cursor to grab whole customer record objects
index.openCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // cursor.key is a name, like "Bill", and cursor.value is the whole object.
    alert("Name: " + cursor.key + ", SSN: " + cursor.value.ssn + ", email: " + cursor.value.email);
    cursor.continue();
  }
};

// Using a key cursor to grab customer record object keys
index.openKeyCursor().onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // cursor.key is a name, like "Bill", and cursor.value is the SSN.
    // No way to directly get the rest of the stored object.
    alert("Name: " + cursor.key + ", SSN: " + cursor.value);
    cursor.continue();
  }
};
```

#### Specifying the range and direction of cursors
```js
// Only match "Donna"
var singleKeyRange = IDBKeyRange.only("Donna");

// Match anything past "Bill", including "Bill"
var lowerBoundKeyRange = IDBKeyRange.lowerBound("Bill");

// Match anything past "Bill", but don't include "Bill"
var lowerBoundOpenKeyRange = IDBKeyRange.lowerBound("Bill", true);

// Match anything up to, but not including, "Donna"
var upperBoundOpenKeyRange = IDBKeyRange.upperBound("Donna", true);

// Match anything between "Bill" and "Donna", but not including "Donna"
var boundKeyRange = IDBKeyRange.bound("Bill", "Donna", false, true);

// To use one of the key ranges, pass it in as the first argument of openCursor()/openKeyCursor()
index.openCursor(boundKeyRange).onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the matches.
    cursor.continue();
  }
};
```
IDBKeyRange - Parameters
* bound  
The value of the lower bound.
* open  
Optional. If false (default value), the range includes the lower-bound value.

*order constnts are no longer available anymore ex prev, nextunique*
in mozilla,
https://developer.mozilla.org/en-US/docs/Web/API/IDBCursor?redirectlocale=en-US&redirectslug=IndexedDB%2FIDBCursor#Constants

```js
objectStore.openCursor(boundKeyRange, "prev").onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the entries.
    cursor.continue();
  }
};
```

```js
objectStore.openCursor(null, "prev").onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the entries.
    cursor.continue();
  }
};
```

```js
index.openKeyCursor(null, "nextunique").onsuccess = function(event) {
  var cursor = event.target.result;
  if (cursor) {
    // Do something with the entries.
    cursor.continue();
  }
};
```


speed up transaction
* When defining the scope, specify only the object stores you need. This way, you can run multiple transactions with non-overlapping scopes concurrently.
* Only specify a readwrite transaction mode when necessary. You can concurrently run multiple readonly transactions with overlapping scopes, but you can have only one readwrite transaction for an object store. To learn more, see the definition for transactions in the Basic Concepts article.


### Version changes while a web app is open in another tab

在新窗口打开app时候，会触发`onblocked`事件。

When your web app changes in such a way that a version change is required for your database, you need to consider what happens if the user has the old version of your app open in one tab and then loads the new version of your app in another. When you call open() with a greater version than the actual version of the database, all other open databases must explicitly acknowledge the request before you can start making changes to the database (an `onblocked` event is fired until they are closed or reloaded). Here's how it works:

```js
var openReq = mozIndexedDB.open("MyTestDatabase", 2);

openReq.onblocked = function(event) {
  // If some other tab is loaded with the database, then it needs to be closed
  // before we can proceed.
  alert("Please close all other tabs with this site open!");
};

openReq.onupgradeneeded = function(event) {
  // All other databases have been closed. Set everything up.
  db.createObjectStore(/* ... */);
  useDatabase(db);
}  

openReq.onsuccess = function(event) {
  var db = event.target.result;
  useDatabase(db);
  return;
}

function useDatabase(db) {
  // Make sure to add a handler to be notified if another page requests a version
  // change. We must close the database. This allows the other page to upgrade the database.
  // If you don't do this then the upgrade won't happen until the user closes the tab.
  db.onversionchange = function(event) {
    db.close();
    alert("A new version of this page is ready. Please reload!");
  };

  // Do stuff with the database.
}
```

### Security
IndexedDB uses the same-origin principle, which means that it ties the store to the origin of the site that creates it (typically, this is the site domain or subdomain), so it cannot be accessed by any other origin.

indexedDB doesn't work for content loaded into a frame from another sisde
 (either `frame` or `ifram`e. This is a security and privacy measure and can be considered analogous the blocking of **third-party cookies**.


### Warning About Browser Shutdown
When browser shutdown(exit, or close button), any pending indexedDB transactions are (silently) aborted - they will not complete, and they will not trigger the error handler.

Notice
* First, you should take care to always leave your database in a **consistent state** at the end of every transaction.
* Second, you should **never** tie database transactions to **unload events**.
unload event is triggered by the browser closing.



## See also
* https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB
* [Web IDL](http://en.wikipedia.org/wiki/Web_IDL)  
Web IDL is a format for describing interfaces that are intended to be implemented in web browsers.
* [ECMAScript](http://en.wikipedia.org/wiki/ECMAScript)
* http://www.w3.org/TR/IndexedDB
* [Indexed Database API](https://docs.webplatform.org/wiki/apis/indexedDB)
* https://en.wikipedia.org/wiki/Object_database
