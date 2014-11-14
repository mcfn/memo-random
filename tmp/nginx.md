# Nginx

nginx is event-based, so it does not follow Apache's style of spawning new processes or threads for each web page request

first version is for handle static ontent: html, css, javascript and images, to offload the concurrency and latency processing from Apached-based application servers.

Handling high concurrency with high performance and efficiency has always been the key benefit of deploying nginx. However, there are now even more interesting benefits.

nginx is very well suited for this, as it provides the key features necessary to conveniently offload concurrency, latency processing, SSL (secure sockets layer), static content, compression and caching, connections and requests throttling, and even HTTP media streaming from the application layer to a much more efficient edge web server layer. It also allows integrating directly with memcached/Redis or other "NoSQL" solutions, to boost performance when serving a large number of concurrent users.


Nginx is written in C.

Now, nginx in windows, do not full-function.

## Nginx Architecture

Traditional process- or thread-based models of handling concurrent connections involve handling each connection with a separate process or thread, and blocking on network or input/output operations.

Depending on the application, it can be very inefficient in terms of memory and CPU consumption. Spawning a separate process or thread requires preparation of a new runtime environment, including allocation of heap and stack memory, and the creation of a new execution context. Additional CPU time is also spent creating these items, which can eventually lead to poor performance due to thread thrashing on excessive context switching. All of these complications manifest themselves in older web server architectures like Apache's. This is a tradeoff between offering a rich set of generally applicable features and optimized usage of server resources.


What resulted is a modular, event-driven, asynchronous, single-threaded, non-blocking architecture which became the foundation of nginx code.

Connections are processed in a highly efficient run-loop in a limited number of single-threaded processes called workers. Within each worker nginx can handle many thousands of concurrent connections and requests per second.

在单线程的进程（worker）中，每个worker每秒可以处理上千的并发请求。


## Code structure
nginx's modular architecture generally allows developers to extend the set of web server features without modifying the nginx core.

nginx modules come in slightly different incarnations, namely core modules, event modules, phase handlers, protocols, variable handlers, filters, upstreams and load balancers.

现在还不支持动态加载。
modules are compiled along with the core at build stage.

![archetecture](nginx-architecture.png)

While handling a variety of actions associated with accepting, processing and managing network connections and content retrieval, nginx uses event notification mechanisms and a number of disk I/O performance enhancements in Linux, Solaris and BSD-based operating systems, like kqueue, epoll, and event ports.

在linux系统中, 使用这些系统I/O操作，使得在unix-based的操作系统中nginx性能得到很大的优化

* sendfile
Transfer data between file descriptors.
* AIO  
 aio - POSIX asynchronous I/O overview  
 http://man7.org/linux/man-pages/man7/aio.7.html  
 The POSIX asynchronous I/O (AIO) interface allows applications to
       initiate one or more I/O operations that are performed asynchronously
       (i.e., in the background).  The application can elect to be notified
       of completion of the I/O operation in a variety of ways: by delivery
       of a signal, by instantiation of a thread, or no notification at all.


## Worksers Model

The run-loop is the most complicated part of the nginx worker code

It includes comprehensive inner calls and relies heavily on the idea of asynchronous task handling. Asynchronous operations are implemented through modularity, event notifications, extensive use of callback functions and fine-tuned timers.

Overall, the key principle is to be as non-blocking as possible. The only situation where nginx can still block is when there's not enough disk storage performance for a worker process.

Because nginx does not fork a process or thread per connection, memory usage is very conservative and extremely efficient in the vast majority of cases.


Because nginx spawns several workers to handle connections, it scales well across multiple cores.
多核环境更好。


With some disk use and CPU load patterns, the number of nginx workers should be adjusted.

Config recommendation:
* if the load pattern is CPU intensive—for instance, handling a lot of TCP/IP, doing SSL, or compression—the number of nginx workers should match the number of CPU cores;  
* if the load is mostly disk I/O bound—for instance, serving different sets of content from storage, or heavy proxying—the number of workers might be one and a half to two times the number of cores.


## Nginx process roles

nginx runs several processes in memory; there is a single master process and several worker processes.

There are also a couple of special purpose processes, specifically a cache loader and cache manager.

All processes are single-threaded in version 1.x of nginx. All processes primarily use **shared-memory mechanisms** for inter-process communication. The **master process** is run as the root user. The cache loader, cache manager and workers run as an unprivileged user.

### master process
* reading and validating configuration
* creating, binding and closing sockets
* starting, terminating and maintaining the configured number of worker processes
* reconfiguring without service interruption
* controlling non-stop binary upgrades (starting new binary and rolling back if necessary)
* re-opening log files
* compiling embedded Perl scripts

### worker process
The worker processes accept, handle and process connections from clients, provide reverse proxying and filtering functionality and do almost everything else that nginx is capable of. In regards to monitoring the behavior of an nginx instance, a system administrator should keep an eye on workers as they are the processes reflecting the actual day-to-day operations of a web server.

### cache loader process
The cache loader process is responsible for checking the on-disk cache items and populating nginx's in-memory database with cache metadata. Essentially, the cache loader prepares nginx instances to work with files already stored on disk in a specially allocated directory structure. It traverses the directories, checks cache content metadata, updates the relevant entries in shared memory and then exits when everything is clean and ready for use.

### cache manager process
The cache manager is mostly responsible for cache expiration and invalidation. It stays in memory during normal nginx operation and it is restarted by the master process in the case of failure.


## Brief Overview of nginx Caching

Caching in nginx is implemented in the form of hierarchical data storage on a filesystem.

Cache keys are configurable, and different request-specific parameters can be used to control what gets into the cache. Cache keys and cache metadata are stored in the **shared memory segments**, which the cache loader, cache manager and workers can access.

Currently there is not any in-memory caching of files, other than optimizations implied by the operating system's virtual filesystem mechanisms.

Each cached response is placed in a different file on the filesystem. The hierarchy (levels and naming details) are controlled through nginx configuration directives. When a response is written to the cache directory structure, the path and the name of the file are derived from an MD5 hash of the proxy URL.


The process for placing content in the cache is as follows:
* When nginx reads the response from an upstream server, the content is first written to a **temporary file** outside of the cache directory structure.
* When nginx finishes processing the request it renames the temporary file and moves it to the cache directory. If the temporary files directory for proxying is on another file system, the file will be copied, thus it's recommended to keep both temporary and cache directories on the same file system. It is also quite safe to delete files from the cache directory structure when they need to be explicitly purged.

There are third-party extensions for nginx which make it possible to control cached content remotely, and more work is planned to integrate this functionality in the main distribution.

## 14.3 nginx Configuration
