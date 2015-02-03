# HTML5 Storage


# Web Storage

Before HTML5, application data had to be stored in cookies, included in every server request. Local storage is more secure, and large amounts of data can be stored locally, without affecting website performance.


Unlike cookies, the storage limit is far larger (at least 5MB) and information is never transferred to the server.

Local storage is per domain. All pages, from one domain, can store and access the same data.



* The storage limit of cookies in web browsers is limited to about 4KB.
* Cookies are sent with every HTTP request, thereby slowing down the web application performance.
* When multiple transactions
Cookies don't really handle this case well. For example, a user could be buying plane tickets in two different windows, using the same site.
If the site used cookies to keep track of which ticket the user was buying, then as the user clicked from page to page in both windows, the ticket currently being purchased would "leak" from one window to the other, potentially causing the user to buy two tickets for the same flight without really noticing.

Each site has its own separate storage area.

HTML local storage provides two objects for storing data on the client:

* window.localStorage - stores data with no expiration date
* window.sessionStorage - stores data for one session (data is lost when the tab is closed)

check support
```js
if(typeof(Storage) !== "undefined") {
    // Code for localStorage/sessionStorage.
} else {
    // Sorry! No Web Storage support..
}
```

## API
### The Storage interface
```
interface Storage {
  readonly attribute unsigned long length;
  DOMString? key(unsigned long index);
  getter DOMString? getItem(DOMString key);
  setter creator void setItem(DOMString key, DOMString value);
  deleter void removeItem(DOMString key);
  void clear();
};
```
### The sessionStorage attribute
```
[NoInterfaceObject]
interface WindowSessionStorage {
  readonly attribute Storage sessionStorage;
};
Window implements WindowSessionStorage;
```
The sessionStorage attribute represents the set of storage areas specific to the current top-level browsing context.

Each top-level browsing context has a unique set of session storage areas, one for each origin.

### The lcoalStorage attribute  
The localStorage object provides a Storage object for an origin.


### The storage event
The storage event is fired on a Document's Window object when a storage area changes, as described in the previous two sections (for session storage, for local storage).

When a user agent is to send a storage notification for a Document, the user agent must queue a task to fire a trusted event with the name storage, which does not bubble and is not cancelable, and which uses the StorageEvent interface, at the Document object's Window object.

#### The StorageEvent interface
```
[Constructor(DOMString type, optional StorageEventInit eventInitDict)]
interface StorageEvent : Event {
  readonly attribute DOMString? key;
  readonly attribute DOMString? oldValue;
  readonly attribute DOMString? newValue;
  readonly attribute DOMString url;
  readonly attribute Storage? storageArea;
};

dictionary StorageEventInit : EventInit {
  DOMString? key;
  DOMString? oldValue;
  DOMString? newValue;
  DOMString url;
  Storage? storageArea;
};
```


### Advantages and Disadvantages

The advantages of Web Storage:
* easy to use with simple name/value pairs
* session and persistent storage options are available
* an event model is available to keep other tabs and windows synchronized
* wide support on desktop and mobile browsers including IE8+
* [Web Storage polyfills](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#web-storage-localstorage-and-sessionstorage) are available for older browsers which fall-back to cookie and windows.name storage methods


The disadvantages:
* string values only — serialization may be necessary
* unstructured data with no transactions, indexing or searching facilties
* may exhibit poor performance on large datasets


If you have to support older browsers such as IE6 and IE7, use a [Web Storage polyfills](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#web-storage-localstorage-and-sessionstorage) which implements the API using cookies and window.name.

## Web SQL Database
**Been abandoned**  
The Web SQL Database was an initial attempt by vendors to bring SQL-based relational databases to the browser. It has been implemented in Chrome, Safari and Opera 15+, but was opposed by Mozilla and Microsoft in favor of IndexedDB.

Web SQL has essentially been abandoned since there is no standard for SQL / SQLite.
http://www.cnet.com/news/consensus-emerges-for-key-web-app-standard/



## Indexed Database

IndexedDB provides a structured, transactional, high-performance NoSQL-like data store with a synchronous and asynchronous API. There’s too much to document here, but the API permits you to create databases, data stores and indexes, handle revisions, populate data using transactions, run non-blocking queries, and traverse data sets using cursors.

The advantages of IndexedDB:
* designed for robust client-side data storage and access
* good support in modern desktop browsers: IE10+, Firefox 23+, Chrome 28+ and Opera 16+


The disadvantages:
*  the API is very new and subject to revision
* little support in older and mobile browsers
* a large and complex API — it would be difficult and largely impractical to create a IndexedDB polyfill
* like any NoSQL store, data is unstructured which can lead to integrity issues


## File Access
is an API for reading file content in JavaScript. Given a set of files the user has added to a "file" input element, you can read the content of the file or reference it as a URL, e.g. if the user has specified an image file, you can show the image. There are also proposals underway for file writing capabilities.

The HTML5 File API is being extended to support writing as well as reading sequential data on the local file system. Your domain is provided with a complete sand-boxed hierarchical file system to use as it chooses.


The advantages of the HTML5 File API:
* large text and binary files can be created and stored
* performance should be good

The disadvantages:
* a very early specification which is subject to revision
* an obvious security risk unless file writing is restricted
* little support in current browsers and polyfills may be impractical
* unstructured data with no transactions, indexing or searching facilties



## Reference
* [w3cschool: html5 webstorage](http://www.w3schools.com/html/html5_webstorage.asp)
* [w3: html5 webstorage](http://dev.w3.org/html5/webstorage/)
* http://www.sitepoint.com/html5-web-storage/
* http://www.sitepoint.com/html5-browser-storage-past-present-future/
