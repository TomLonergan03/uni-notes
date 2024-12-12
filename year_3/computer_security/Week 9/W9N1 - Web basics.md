The [[W2N3 - HTTP and DNS#Uniform Resource Locators (URLs)|URL naming scheme]] allows for referring to content on remote computers in a simple and consistent manner:
`protocol://host:port/path?arg1=val1&arg2=val2#statement`
- `protocol` is the protocol for accessing the resource, e.g. `http`, `https`, `ftp`, etc.
- `host` is the domain/IP address of the remote server
- `path` is the file path to the resource on the host
- Resources can be static (`file.html`) or dynamic (`script.js`, among others)
- URLs for dynamic content usually include arguments to pass to the process (`arg1`, `arg2`)
# HTTP requests
Once the [[W2N2 - IP, UDP, and TCP#Transmission Control Protocol (TCP)|TCP]] connection is established, the browser generally sends HTTP requests to that server. A HTTP request begins with a request line (usually `GET` or `POST`), then comes the header section, then the message body.
![[w9n1HttpRequest.png]]
The server then replies with an HTTP response:
![[w9n1HttpResponse.png]]
which includes a status code (e.g. `200` for successful, `404` for content not found, etc.), then some headers, then a body, in this case containing a HTML document for a webpage.
# Hypertext Markup Language (HTML)
The main body of a web page is encoded using HTML, which provides a structural description of a document using tags, e.g. `<header></header>,<body></body>`, etc.
HTML includes forms, which allow users to provide input to a website in the form of variables represented by name-value pairs. Forms submit this data either using `GET`, with name-value pairs encoded in the URL, or the `POST` method, with name-value pairs encoded in the message body.
```html
<form method="POST" action="login.php">
	FirstUsername: <input type="text" name="username">
	Password: <input type="password" name="password">
	<input type="submit" value="submit">
</form>
```
# Dynamic content
HTML is only capable of static webpages, while pages with dynamic content can change after delivery to the client browser, eg. in response to user interaction. To provide dynamic content, we can use a scripting language such as JavaScript.
```html
<html>
	<body>
		<p id="p1">Hello World!</p>
			
		<script>
			document.getElementById("p1").innerHTML = "New text!";
		</script>
	</body>
</html>
```
# Web page rendering
The **document object model (DOM)** is a means to represent and access the content of the page. Scripts can alter the content of a page by modifying the DOM of the page.
# HTTP session statefulness
HTTP is stateless, so each request a client sends gets a response from the server, but the server does not retain any information on the previous requests. In many cases, we do want to maintain state between requests, e.g. if we have a login, we want the user to stay logged in for a period of time and not sign in again for every HTTP request. The approach for this is generally to insert a token into the page when it is requested, then the client responds with that token as part of each subsequent request. There are two main approaches:
## Hidden fields
- We can create a form with a hidden field with a unique session ID:
  ```html
  <input type="hidden" name="sessionid" value="12345">
  ```
  When the form is submitted, the specified name and value are automatically included in the `GET`/`POST` data
- This has some disadvantages:
		- It takes a lot of effort, as each page has to be dynamically generated to include this hidden field
		- The session ends as soon as the browser is closed
- It does have the advantage that HTML forms are part of the HTML specification, so all browsers support them
## Cookies
- A cookie is a small piece of information that a server sends to a browser and is then stored inside the browser. A cookie has a name, value, and various other attributes such as domain and path, expiration date, version number, and comments
- The browser automatically sends the cookie in all subsequent requests to the originating host of the cookie
- Cookies are only send back to their originating host and not any other, with the domain and path specifying where to return the cookie to
- The server can set a cookie with a unique session/user ID, or to hold personalised information, eg. shopping cart contents
- Cookies are set when the server uses the `Set-Cookie` field in the HTTP header of its response `Set-Cookie: name=value[; expires=date][; domain=dom][; path=path][; Secure][; HttpOnly]` where
	- `expires` is when to delete the cookie
	- `domain` and `path` are where to send the cookie
	- `secure` says to only send over SSL
	- `HttpOnly` says to only send over HTTP
- The main limitation of cookies is that users are able to disable them on their browser
# Frames
Frames embed one webpage within another one:
```html
<iframe src="URL"></iframe>
```
