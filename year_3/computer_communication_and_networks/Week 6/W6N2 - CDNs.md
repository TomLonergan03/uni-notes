# Overview
With [[W6N1 - Media Playback#DASH|DASH]] allowing for ease of playback over variable bandwidths, the challenge of streaming to hundreds of thousands of simultaneous clients remains.
While one massive server would work, it has drawbacks:
- Single point of failure;
- Network congestion at the server;
- Long path to distant clients; and
- Multiple copies of the same video being sent over the outgoing link.
Put simply, this doesn't scale at all.
# CDNs
Option two would be to store and serve multiple copies of videos at multiple geographically distributed sites. This is what a **Content Distribution Network (CDN)** achieves.
> [!info] Note Well
> CDNs can be private (owned by the content provider themselves, like Google or YouTube) or third-party (distributing content on behalf of multiple companies, such as Cloudflare).

> [!info] Note Well
> There are two server placement philosophies:
> - *Enter Deep*: Push CDN servers deep into many access networks, letting them be close to users. This requires management of many servers, which is a lot of work, not to mention expensive.
> - *Bring Home*: Operate a smaller number of larger clusters near IXPs.

CDNs store copies of content which subscribers can request from their closest CDN *node*.

> [!example] Example - CDN Addressing
> To get a video from a CDN-using site netcinema.com:
> 1. Bob requests a URL from netcinema.com/efjhfeiojfeo.
> 2. Bob's [[W3N7 - DNS#DNS requests|local DNS server]] is called to resolve the name
> 3. Bob's local DNS server sends a [[W3N7 - DNS#DNS requests|CNAME request]] to netcinema's DNS server, which returns CNAME for KingCDN's DNS server
> 4. Bob's local DNS server sends this CNAME request to KingCDN's DNS server, which returns the content's location on the KingCDN.com network
> 5. Bob's browser sends the content request to the returned KingCDN.com location, which dutifully [[W6N1 - Media Playback#DASH|streams the requested video over HTTP]].

> [!info] Note Well - CDN Challenges
> Some challenges include:
> - *Space Utilisation*: Space is at a premium in CDN clusters, so CDNs need to use the small number of machines they have well, through techniques such as load balancing.
> - *Video Preparation*: Preparing many versions of a video is computationally and spatially expensive (CPU, and disk). Methods of predicting or measuring the most popular content to prepare ahead of time are being developed.
