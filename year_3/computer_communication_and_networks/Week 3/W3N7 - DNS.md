[[W2N1 - Network core#Packet switching|Routers]] work with [[W2N4 - Protocol layering|network layer IP addresses]], and these are the identifiers for each server. End users prefer to use a more readable hostname, so we use a directory service to translate hostnames to IP addresses. This is the **domain name system (DNS)**, which consists of a distributed database implemented across a hierarchy of **DNS servers**, as well as the [[W3N4 - Application layer protocols|application layer protocol]] used to query that database.
When a user wants to access a website, their browser sends sends requests to a URL. To do this, their system first requests the IP address of that URL using their DNS client, which queries a DNS server for it. Once it receives a reply, the IP address is returned to the browser which can initiate TCP connections to the HTTP server on port 80 at that IP address.
DNS also offers some other services, mainly:
- **Host aliasing**: a server may have a complex hostname, e.g. `relay1.west-coast.enterprise.com`, but aliases could be provided for it, such as `enterprise.com` and `www.enterprise.com`, which both actually point to the same hostname
- **Mail server aliasing**: it may be desired to have the same "hostname" for both a website and an associated [[W3N6 - E-mail#Mail servers|mail server]] to have the same name, so visiting `enterprise.com` and sending an email to `bob@enterprise.com` are both possible even if the mail server is running on a completely different machine
- **DNS rotation**: a DNS can be provided with a set of IPs for one website, and it responds to each request with all of them, but in a different order each time. As browsers tend to just use the first IP, this balances the load across the whole set of web/mail servers
# DNS server hierarchy
There are 3 main tiers of DNS servers:
1. **Root DNS servers**: provide IP addresses for the TLD servers
2. **Top-level domain (TLD) servers**: each top-level domain (e.g. `.com`, `.org`, `.uk`) has a TLD DNS server, which provide IP addresses for authoritative DNS servers
3. **Authoritative DNS servers**: each organisation with publicly accessible hosts on the Internet must provide publicly accessible DNS records that map the names of those hosts to IP addresses. Small organisations tend to rent space on service providers DNS servers, while large organisations often maintain their own authoritative DNS servers
# DNS requests
When a DNS record needs resolved, the DNS client first calls a **local DNS server**, which forwards the request to the hierarchy. For example, if the host `cse.nyu.edu` wants to find the IP address of `gaia.cs.umass.edu`, the local DNS server is `dns.nyu.edu`, and the authoritative DNS server is `dns.umass.edu`, then the host first sends a DNS query to `dns.nyu.edu`, which forwards the request to the root DNS server. The root server notes the `.edu` TLD, and provides an IP for the DNS server that governs that TLD. The local DNS server then re-sends the request to that TLD server, which responds with `dns.umass.edu`, which finally responds with the IP address corresponding to `gaia.cs.umass.edu`, which is passed back to the original requester. In this example, 8 DNS messages are sent, 4 requests and 4 responses, but in reality this is generally not required due to DNS caching. It may be that there are **intermediate DNS servers** too, so continuing the example, the `.edu` TLD DNS server know about a `dns.umass.edu`, but that DNS server instead returns the IP address of `dns.cs.umass.edu` which is the true authoritative name server.
This example contains both **recursive requests** (requesting the full IP from `dns.nyu.edu` which then makes subsequent requests on the users behalf), as well as **iterative requests** (every request from `dns.nyu.edu` which gets a response and must then itself make another request)
# DNS caching
**DNS caching** is a key part of DNS resolution which significantly lowers the amount of DNS traffic moving around the network. When a DNS server receives a DNS reply, it caches that IP in its local memory for a period (often ~2 days). If it receives any requests for that hostname's IP, it can simply return the cached IP without making any further requests. Local DNS servers usually cache the TLD DNS server IPs also, meaning that in practice only a very small fraction of DNS queries make it to the root servers.
# DNS records
DNS servers store **resource records (RRs)**, and each DNS reply carries one or more resource records. A resource record is a four-tuple consisting of the following fields:
```
(Name, Value, Type, TTL)
```
- `TTL` is the time-to-live for that record, which determines how long that record can be cached for. Note that we will ignore `TTL` in the following records
- If `Type=A`, then `Name` is a hostname and `Value` is the corresponding IP address, so Type A records are the standard hostname-to-IP mapping. `(relay1.bar.foo.com, 145.37.93.126, A)` is an example Type A record.
- If `Type=NS`, then `Name` is a domain and `Value` is a DNS server that knows the next step in resolving that hostname, e.g. `(foo.com, dns.foo.com, NS)` is a Type NS record
- If `Type=CNAME` then `Value` is a canonical hostname for the alias hostname `Name`, e.g. `(foo.com, relay1.bar.foo.com, CNAME)`
- If `Type=MX` then `Value` is the canonical name of a mail server for an alias hostname `Name`, e.g. `(foo.com, mail.bar.foo.com, MX)`
# DNS messages
Both DNS request and reply messages have the same format
![[w3n7dnsMessage.png]]
- The header section has a number of fields
	- The identification section is a 16 bit number unique to the request, so that the client can match a received reply with a sent query
	- There are a number of flags in the flag section:
		- A 1 bit query/reply flag indicates whether the message is a query (0) or a reply (1)
		- A 1 bit authoritative flag is set in a reply message when the DNS server is an authoritative server for that name
		- A 1 bit recursion-desired flag is set if the client wants the DNS server to perform recursion if it doesn't have a record
		- A 1 bit recursion-available flag is set in replies if the replying server is capable of recursive resolution
	- The 4 "Number of" fields indicate the number of records in each of the 4 data sections in the data section
- The question section contains information about the query being made. It consists of a name - the name being queried - and a type - whether a type A or MX record is desired
- In a reply, the answer section contains the resource records for the name that was queried. There may be many records here, as one hostname may resolve to many IP addresses.
- The authority section contains records of other authoritative servers
- The additional section contains other helpful records, e.g. a reply to an MX query could contain an RR for the type A record providing the IP address for the canonical hostname of the mail server