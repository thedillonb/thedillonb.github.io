---
layout: post
categories: Programming
tags: 
  - nodejs
  - http
published: true
title: Gracefully shutting down a Nodejs HTTP server
---

Looking to gracefully shutdown your Nodejs HTTP server? Well, it's actually a little more difficult than you think. Without handling process signals, like "ctrl-c", your application terminates immediately. Which means, there may be requests still processing that you terminated before completion. In a live environment, this is a horrible user experience. Unfortunately, even if you did catch the process' signals, Node doesn't actually have a built in mechanism for gracefully shutting down a running HTTP server.

## Taking a look at the problem

Let's take the classic HTTP server written with Node's native HTTP library:

```javascript
var http = require('http');

var server = http.createServer(function(req, res) {
	res.end('Goodbye!');
});

server.listen(3000);
```

This example, while trivial, demonstrates a working HTTP server. Now, what if I wanted to terminate it, but not disrupt any concurrent traffic? How about the `close` method?

```javascript
server.close(function() {
	console.log('We closed!');
	process.exit();
});
```

Unfortunately, not really, there's a problem with this. Reading the documentation might make it a little more clear:

> Stops the server from accepting new connections and keeps existing connections. This function is asynchronous, the server is finally closed when all connections are ended and the server emits a 'close' event. - From [Nodejs Docs](https://nodejs.org/api/net.html#net_server_close_callback)

The important part of the description to remember is "keeps existing connections". While `close` does stop the listening socket from accepting new ones, sockets that are already connected may continue to operate - which is fine if they're mid-request - but this also includes sockets that are connected with a 'keep-alive' connection type.

>  HTTP persistent connection, also called HTTP keep-alive, or HTTP connection reuse, is the idea of using a single TCP connection to send and receive multiple HTTP requests/responses, as opposed to opening a new connection for every single request/response pair - from [Wikipedia](https://en.wikipedia.org/wiki/HTTP_persistent_connection)

This means that sockets that are kept alive will remain alive and still capable of making additional HTTP requests. This is obviously not what we want. A graceful shutdown should:

1. Close the listening socket to prevent new connections
2. Close all idle keep-alive sockets to prevent new requests during shutdown
3. Wait for all in-flight requests to finish before closing their sockets.

## The solution

[http-shutdown](https://github.com/thedillonb/http-shutdown) is a [NPM package](https://www.npmjs.com/package/http-shutdown) that provides graceful shutdown functionality. It does this by leveraging several mechanisms for keeping track of active vs idle sockets. The principle is as follows:

1. Listen for socket "connection" and "close" events. When a socket connects, add it to a list of connected sockets and mark that socket as idle (it hasn't made a request yet). When it's closed, remove it from that list. This gives us a mechanism to track all sockets that are currently connected and allows us to keep track of which of those are active vs idle.
2. Listen on the HTTP server's "request" and "finish" event. When the "request" event triggers, mark the underlying socket as active. When the "finish" event occurs, mark the socket as idle. In addition, look to see if the system is "shutting down". If so, call `destroy` on that socket to close it, making sure no additional traffic flows through it.

Given the two mechanisms above, we can easily implement a `shutdown` method that will active a graceful shutdown. It only need to do the following:

1. Call the native `close` method on the HTTP server to close the listening socket and prevent all future socket connections.
2. Set a flag that indicates the server is shutting down. (Principle two above uses this flag to close sockets when they're done with their request)
3. Loop through all currently connected sockets where they're idle flag is true: call `destory` on these sockets to close them out since they're not serving traffic and we want to prevent them from serving any more.

If you're interested in the source code behind the [NPM library](https://www.npmjs.com/package/http-shutdown), check out the [GitHub repository](https://github.com/thedillonb/http-shutdown). Or, more specifically, check out the [single file](https://github.com/thedillonb/http-shutdown/blob/master/index.js) which contains all of which I discussed above.

