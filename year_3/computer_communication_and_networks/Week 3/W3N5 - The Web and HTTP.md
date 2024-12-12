The Web is what we think of when we hear the word "Internet". It is a collection of protocols and applications which include web browsers and webservers that enable the access and modification of websites.
# The Web
The Web consists of **webpages**, which is build up from **objects** such as HTML files, video, audio, and programs. Most webpages consist of a base HTML file and a number of referenced object. All objects on the web have a unique **uniform resource locator (URL)** which is used to identify that object.
# HyperText transfer protocol (HTTP)
The **HyperText transfer protocol (HTTP)** is the application layer protocol for the Web, and is implemented in two programs, a client and a server. It defines how a browser and webserver communicate using a series of requests and responses. A browser sends requests for objects when it visits a page, and the associated server responds with the requested objects. HTTP uses [[W3N3 - Transportation services#Transmission control protocol (TCP)|TCP]] as its transport protocol, which means that every HTTP request is guaranteed to eventually arrive at the server if it's sent to a valid URL. HTTP is a **stateless protocol**, meaning that the server doesn't store any state information about the client, so a client may request the same object several times in succession and there is nothing within HTTP that tracks what session is making what request.
## Non-persistent and persistent connections
In many Internet applications, a client and server communicate for an extended amount of time, with many requests and responses. There are two approaches to this: either each request establishes its own TCP session which is terminated once the response is received (a **non-persistent connection**), or one TCP session is maintained across many requests (a **persistent connection**). HTTP can use either, but defaults to persistent connections.
### HTTP with non-persistent connections
If a page consists of a base HTML file and 10 JPEG images, and the URL of that base file is
```
http://www.someSchool.edu/someDepartment/home.index
```
then these are the steps to access the page:
1. The HTTP client initiates a TCP connection to the server `www.someSchool.edu` on [[W3N1 - Network applications#Ports|port 80]], resulting in a [[W3N1 - Network applications#Sockets|socket]] being created on both the client and server
2. The client sends an HTTP request to the server via its socket. The request message includes the path name `/someDepartment/home.index`
3. The server receives the request message from its socket, retrieves the object from storage/memory, encapsulates the object in a HTTP response message, and sends it to the client via its socket
4. The HTTP server tells TCP to close the TCP connection (which doesn't actually terminate until the message arrives at the client intact)
5. The HTTP client receives the message and the connection terminates. The HTTP message indicates that the contents are an HTML file, and the client examines it and finds references to the JPEG objects.
6. Steps 1-4 are repeated for each of the JPEGs

Note that different browsers may display webpages somewhat differently, as HTTP only defines the communication protocol.
Also, each TCP connection may be serial, or in parallel. Most modern browsers open a number of parallel connections (usually 5-10 but this is often user configurable) as it reduces response times.

![[w3n5httpNonpersistentRequestTime.png]]
A single HTTP request takes approximately $2\cdot\text{round trip time}+\text{file transmission time}$ with a non-persistent connection, as establishing a TCP connection is a 3 message handshake but the final message can be combined with the HTTP request.
### HTTP with persistent connections
With a HTTP 1.1 persistent connection, we no longer need to perform the full handshake on every request. This means that every request after the first takes only $\text{round trip time}+\text{file transmission time}$. With HTTP 2 persistent connections, it is possible to send many requests on the same connection without waiting for the response, and the HTTP implementation handles matching responses to requests. These are called **multiplexed requests**.
## HTTP message format
A HTTP message is either a request or a response
### HTTP request
A HTTP request looks something like this:
```HTTP
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```
Messages are written in ordinary ASCII text, and consists of at least 1 line (but generally several) separated by a carriage return and line feed (rendered as a newline). The first line is the **request line**, and all following lines are **header lines**. The request line consists of a method, URL, and HTTP version. The **method** can take several values, such as `GET`, `POST`, `PUT`, or `DELETE`, though most requests are `GET`s, which request and object at the specified URL. The `host` line specifies which server the request is to - unnecessary as the TCP connection is already established, but is used by web caches, the `Connection: close` tells the server that it is a non-persistent connection, `User-agent` says what browser is making the request (this can allow the server to respond with different files depending on the browser), and `Accept-language: fr` is an example of a content negotiation header, which asks for the French language copy of that file (if it exists, otherwise the server will respond with a default language).  A HTTP request also has an **entity body**, but it may be left blank depending on the method.
#### HTTP request methods
These are the common request methods, but there are more.
- `GET` - requests an object from the server
- `POST` - requests an object from the server based off of data in the entity body
- `HEAD` - requests an object from the server, but instructs the server not to send the file but only the header, can be used for debugging
- `PUT` - uploads an object to the server
- `DELETE` - deletes an object on the server

Note that `GET` can also send information to the server, by extending the URL with queries, e.g. `somesite.com/search?search=a+search+string`
### HTTP response
A HTTP response looks like this:
```HTTP
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
data goes here
```
A response consists of a **status line**, 6 **header lines**, and an **entity body**. The status line consists of the protocol version, and a **status code** and its corresponding status message. The `Connection: close` line tells the client that the server will close the connection that the response was sent over, `Date` indicates the time and date the response was created and sent, `Server` indicates what webserver software generated the response, `Last-Modified` states the time and date at which the object was last modified, `Content-Length` is the number of bytes in the sent object, `Content-Type` says how to interpret the sent object. The entity body contains the requested object.
#### HTTP status codes
There are many response codes, but common ones include:
- `200 OK` - the request succeeded and the content is included in the body
- `301 Moved Permanently` - the requested object has been moved, the client will automatically request the new URL instead
- `400 Bad Request` - the server didn't understand the request
- `404 Not Found` - the requested object doesn't exist on the server
- `505 HTTP Version Not Supported` - the requested HTTP version is not supported by the server
# Cookies
Although HTTP is stateless, we often want to track sessions anyway, for example if a we want a user to stay logged in between visits to a webpage or to store data on shopping cart contents. We can do this using **cookies** - files stored by the browser on the user's device.
Cookie technology consists of 4 components:
1. The `Set-cookie` header in a HTTP response
2. The `Cookie` header in a HTTP request
3. A cookie file kept on the user's system and managed by their browser
4. A cookie database on the webserver
When a website wants to set a cookie, it uses the `Set-cookie` header with the identification number it wants to set. This is stored by the browser, and included in subsequent requests to that server (as governed by the [[W9N4 - Cookie policies|cookie policy]]). When the website receives a request with a cookie, it looks it up in its database to find the corresponding information.
# Web caching
A **web cache/proxy server** is a server which satisfies HTTP requests on the behalf of an origin server. A browser can be configured to request from a web cache, often run by an ISP, and when a user attempts to visit a website the request goes to the cache, which either directly satisfies it if it has a copy of the file, or if not it requests it from the origin server, caches it for a period, and sends it back to the client. A cache is both a server and a client at the same time, acting as a server to the end user and a client to the origin server. Web caches are also often run by [[W2N2 - Internet structure|content delivery networks (CDNs)]].
## Conditional `GET`s
While a cache can reduce perceived response times, it runs the risk of sending users stale objects. To solve this, HTTP has a mechanism to allow a cache to verify its objects are up to date, called a **conditional GET**. This is a `GET` message that includes the `If-Modified-Since` header line. If the file has been modified since that time, the server responds with the new object. If it hasn't, the server responds with a `304 Not Modified` message, and sends no entity body.