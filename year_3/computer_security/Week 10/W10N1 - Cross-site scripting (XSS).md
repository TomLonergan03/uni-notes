ZWhen you have a [[W9N1 - Web basics#Cookies|session token]], it is possible for an adversary to hijack the session in a variety of ways:
- Token theft:
	- If a token is predictable, then the token could be guessed, so tokens should be randomly generated
	- If a token is sent over HTTP, then it can be sniffed off the network. This can also happen by accident if a site has a mix of HTTP and HTTPS pages, and the token is accidentally sent over HTTP. Cookies should always be sent with the [[W9N4 - Cookie policies#^a0cfda|Secure]] flag set, and a new cookie should always be reissued when elevating a user from unauthenticated to logged-in
	- Cross-site scripting vulnerabilities
# Cross-site scripting (XSS)
An XSS attack is where malicious scripts are injected into otherwise benign and trusted web sites. This allows an attacker to run a script in a different website's [[W9N3 - Same origin policy|origin]], giving the script access to that origin's cookies.
There are two types of XSS attacks: **stored** and **reflected**
## Stored XSS attacks
In a stored XSS attack, the injected script is permanently stored on the target server, e.g. in a database, visitor log, comment field etc.

An example attack would be if a website has a comment section, and does not sanitise the comments, an attacker can leave a comment that is `<script>/*malicious code here*/</script>`. and as the comment is not sanitised it is directly inserted into the HTML page by the browser, which treats it as a valid HTML section and executes the contents.
## Reflected XSS attacks
In a reflected XSS attack, the injected script is reflected off the web server, e.g. in an error message or search result.

An example attack would be if the browser visits a malicious site, and follows a link from there to a legitimate site, where the URL has been crafted to include a piece of code, e.g. being given a link that is something like `https://www.bank.com/search?term=<script>/*malicious code here*/</script>`. If `bank.com` doesn't sanitise this input, it may return something like `<p> you searched for: <script>/*malicious code here*/</script></p>` in which case the script is executed by the browser.

# XSS defences
1. **Escape/filter outputs:** escape dynamic data before inserting it into HTML, e.g. `<` becomes `&lt`, `"` becomes `&quot` etc. and remove any `<script>`, `<javascript>` etc. tags. This is error prone though, as there are a lot of ways to introduce JavaScript, e.g. through CSS tags.
2. **Input validation:** check that inputs (headers, cookies, query strings, form fields, hidden fields) are of the expected form (whitelisting)
3. **Content security policy (CSP):** the server supplies a whitelist of scripts that are allowed to appear on the page
4. **[[W9N4 - Cookie policies#^191115|HTTP-Only attribute]]:** if enabled, scripting languages cannot access or manipulate cookies, but this does not prevent all exploits