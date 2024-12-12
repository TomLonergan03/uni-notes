# Access control in the browser
Similar to [[W7N3 - Privilege separation|OS access control]], we divide access in browsers into subjects (JS/PHP scripts) and objects (DOM tree, DOM storage, HTTP cookies, JS namespace).
# Same-origin policy
The SOP restricts how a document or script loaded from one origin (e.g. `www.evil.com`) can interact with a resource from another origin (e.g. `www.bank.com`). Each origin is kept sandboxed from the rest of the web.
Windows and tabs have an origin derived from the URL of the webserver providing the content. The URL `protocol://host:port/path?args#statement` gives the origin `protocol://host:port`.
E.g.
## SOP and JavaScript
A webpage can load a cross-origin script (i.e. from a different domain), and will run it in the current page's origin. The current page cannot inspect the source of the script, but can call functions defined within it. This means that if a webpage loads a malicious script, it gives it access to the current origin.
## SOP and images
The browser can render a cross-origin image, but the SOP prevents the page from inspecting it, even pixel by pixel.
## SOP and [[W9N1 - Web basics#Frames|frames]]
An iframe can load cross-origin HTML, but the page cannot inspect the contents of the iframe, and the iframe's origin is that of its source page, and cannot inspect the contents of the parent page.
## Cross-origin communication
It is possible for 2 origins to talk to each other, but only when both windows agree to talk to each other by calling corresponding JavaScript functions.
```js
var onMessage = function(msg){
	if (msg.origin == "https://user.bank.com") {
		// Do something
	}
}
```
