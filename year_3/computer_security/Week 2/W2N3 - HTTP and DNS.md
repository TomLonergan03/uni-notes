# Uniform Resource Locators (URLs)
URLs are a standardised format for describing the location and access methods of resources on the internet. A URL is in the form:
`<scheme>://<user>:<password>@<host>:<port>/<url-path>?<query-string>`

`<host>` is often in the form `<subdomain>.<domain>.<top-level-domain>` e.g. maps.google.com
The domain and top level domain will be bought from a registrar, while the subdomain can be configured to whatever the site owner prefers, including nesting subdomains of subdomains.

People tend to be bad at identifying where a URL will take them, and will assume that if a company name is anywhere in the URL, that is where it will take them:
![[w2n3SubdomainQuestions.png]]

# Domain Name System (DNS)
The function of the DNS is to map domains to [[W2N2 - IP, UDP, and TCP|IP addresses]]. This mapping is many to many, i.e. one domain can map to many different IPs, and many domains can map to 1 IP.

More generally, DNS is a distributed database that stores resource records. There are 3 types of resource records:
- **Address (A)** record: An IP address associated with a host name
- **Mail exchange (MX)** record: The mail server of a domain
- **Name server (NS)** record: An IP address of a server which is the name server for a specific domain

# Domains
A domain name consists of 2 or more labels, separated by dots (e.g. inf.ed.ac.uk)

A top level domain (TLD) is either:
- Generic (gTLD): e.g. .com, .org, .net
	- These are managed by ICANN
- Country code (ccTLD): e.g. .ca, .it#
	- These are managed by their respective country government
- A new one e.g. .scot, .tirol

## ICANN
ICANN is the (non-profit) Internet Corporation for Assigned Names and Numbers. It ensures domain registrars play fair, and keeps a database of registered gTLDs ([InterNIC](https://www.internic.net/)).

# DNS Trees
![[w2n3DnsTree.png]]
Essentially, there is a top level name server which has a list of all the name servers for each top level domain, then each of those name servers has a list of all their domains' name servers, which would each have a list of their subdomains, and so on.

# Name servers
A name server keeps a local database of DNS records with which it answers DNS queries. It can ask other name servers if a record is not in its local database.

An **authoritative name server** stores a reference version of DNS records for a zone (partial tree).

The **root name servers** are authoritative for the root zone (i.e. they are the top level name servers). These are \[a-m].root-servers.net, and are supervised by ICANN.

# Name resolution
Name resolution is the process by which a **resolver** converts a domain to a host machine.

## Resolver
A resolver is a program which retrieves DNS records. It connects to a name server, either a preset default, the root NS, or a specified NS. Examples of resolvers include `dig` on Linux of `nslookup` on Windows. Resolvers generally cache the records they receive.

## Resolution approaches
### Iterative resolution
The name server refers the client to an authoritative server via an NS record. The client then queries that NS and repeats until the domain is resolved.
![[w2n3IterativeResolution.png]]
### Recursive resolution
The name server queries another server and forwards the final answer to the client. Each name server also caches previous requests.
![[w2n3RecursiveResolution.png]]
# DNS caching
DNS servers cache records that are the result of queries for an amount of time specified in the time-to-live field. This is necessary as otherwise there would be extreme amounts of traffic going to the root and TLD servers.
## Local DNS cache
The OS maintains a DNS cache which is shared between all running applications and all users.

This has some privacy issues as a user can monitor other users' recent activity based off of DNS caches. Private/incognito browsing also does not clear the DNS cache.

## DNS cache poisoning
### The attack
A DNS name server can be given a false address record which it will then cache. This is as a DNS query is issued over UDP (port 53) with no authentication, although it does have a 16-bit request identifier in its payload to match a response to a query. If the attacker answers before the authoritative name server, they can get an incorrect/malicious entry into the resolver cache. The server will also ignore the legitimate response as only the first response with the correct request identifier will be accepted.
### Mitigations
- Query randomisation:
	- By randomising the request identifier and return port the probability of responding with the correct request ID and to the correct port is less than 1 in 4 billion

## Subdomain DNS cache poisoning
### The attack
An attacker can cause a victim to send millions of DNS requests (to a real name server) for nonexistent subdomains of a target domain. This then gives the attacker the chance to reply with millions of spoofed DNS responses in the hopes of guessing the request ID and port. If they are successful they can get their malicious DNS address into the resolver's cache.
![[w2n3SubdomainDnsCachePoisoning.png]]
### Mitigations
- DNSSEC:
	- Authenticate DNS responses by signing packets coming from the DNS name servers using public key cryptography
	- This reduces the speed of a NS response
	- DNSSEC has had a slow deployment, the root servers have supported it since 2010 but some name servers still don't