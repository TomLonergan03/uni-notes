When a browser visits a webpage, it will generally also make a number of requests to other websites also (e.g. to get adverts and trackers). A CSRF attack occurs when a victim is logged in to a vulnerable website, then visiting a malicious website which includes a request to the vulnerable website that *changes the vulnerable website's state* in some form (i.e. no data is leaked to the attacker, as the request is sent directly from the client's browser to the vulnerable site). This means that [[W9N1 - Web basics#Cookies|cookies]] are insufficient authentication for actions with side effects.
# Example
1. Alice logs in to `bank.com` and gets a session token
2. `bank.com` has a form for requesting bank transfers
   ```html
   <form action="https://bank.com/transfer.php" method="GET">
	   <input type="text" name="acct"/>
	   <input type="text" name="amount"/>
	   <input type="submit"/>
   </form>
	```
3. Alice visits `evil.com` which includes
   ```html
   <form action="https://bank.com/transfer.php" method="GET">
	   <input type="text" name="acct" value="Eve"/>
	   <input type="text" name="amount" value="100000"/>
	   <input type="submit"/>
   </form>
	```
4.  Alice's browser sends the session cookie along with the HTTP GET request
# Defences
1. **`Referer` header:** An HTTP request includes the `referer` field in its header, which states what site that request is originating from (e.g. clicking a link, sending an HTTP request etc.). In the example's case, `bank.com` could restrict that form to only be sent from the `bank.com` page, which would prevent this attack.
2. **CSRF tokens:** The server stores a fresh CSRF token in every form as a hidden field. On each request, the server checks that the CSRF token is valid.
3. **`SameSite` cookie attribute:** this is a cookie attribute which says to only send that cookie with a request originating from the same site the cookie is set on.
