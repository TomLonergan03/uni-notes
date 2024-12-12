Web applications should provide the same security guarantees as those required for standalone applications. Specifically:
1. visiting `evil.com` should not infect my computer with malware, or read or write files
	- Defences: JavaScript is sandboxed, browsers are regularly updated to fix bugs and vulnerabilities, privilege separation, etc.
2. visiting `evil.com` should not compromise my sessions with `gmail.com`
	- Defences: the [[W9N3 - Same origin policy#Same-origin policy|same-origin policy]] means that [[W9N1 - Web basics#Cookies|cookies]] are only accessible to the issuing website, and the [[W9N4 - Cookie policies|cookie policy]] also controls the scope and lifetime of cookies
3. sensitive data stored about `gmail.com` should be protected

# Threat model
## Web attacker
A web attacker controls a website (eg. `evil.com`), has valid SSL/[[W6N1 - TLS|TLS]] certificates for `evil.com`, and the victim visits (either deliberately or accidentally) `evil.com`
## Network attacker
A network attacker controls the whole network, so can intercept, forge, and send messages. A network attacker is significantly stronger than a web attacker.