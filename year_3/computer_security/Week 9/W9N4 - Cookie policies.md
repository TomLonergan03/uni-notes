When setting a cookie with [[W9N1 - Web basics#Cookies|Set-Cookie]], the scope set by the server must be a suffix of the webserver's hostname (other than the top-level domain), e.g. `sub.example.com` can set a cookie domain to `example.com`, while the path can be set to anything.

E.g. a server host at `http://www.bar.example.com/` can set cookie domains on `bar.example.com` or `example.com`, but not on `foo.bar.example.com`.

Cookies are sent back to the server by the browser if the HTTP request is in the URL's scope, i.e. a cookie with scope `domain` and `path` will be sent to all URLs of the form `http://*.domain/path/*`.

E.g. if there are 2 cookies set in a browser:
`cookie1` is set for `(foo.example.com, /)`
`cookie2` is set for `(example.com, /)`

Then the following occurs:

| If we visit                | we send                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| `http://bar.example.com/`  | `cookie2`                                                                                  |
| `http://foo.example.com/`  | `cookie1` and `cookie2`                                                                    |
| `https://foo.example.com/` | `cookie1` and `cookie2` (the [[W9N3 - Same origin policy\|SOP does not apply to cookies]]) |
| `http://example.com/`      | `cookie2`                                                                                  |
| `http://sample.com/`       | neither                                                                                    |

>[!note] 
>JS scripts with origin O will have access to all cookies in the scope of O

According to the SOP `foo.example.com` and `bar.example.com` should be viewed as different origins and isolated, while according to the cookie policy they are trusted to share cookies set with domain `example.com`.

# Cookie flags
- `HttpOnly` - if enabled then scripting languages cannot access or manipulate the cookie, e.g. to prevent an analytics service from accessing authentication cookies ^191115
- `Secure` - prevents sharing cookies in the case of visiting a site using an unsecured protocol, e.g. if an attacker tricks a victim into visiting `http://bank.com` instead of `https://bank.com`, in which case they could listen in on the unsecured communication. ^a0cfda
- `SameSite` - prevents sending a cookie with a domain to any other domain, even via a JS script.
