As many encryption algorithms are [[W3N3 - Symmetric encryption systems#Limitations of OTPs|malleable]], it is important to also be able to authenticate that a received message matches the sent message.

This is achieved using a MAC.
![[w3n5Mac.png]]
A MAC is a pair of algorithms ($S$ - sign, and $V$ - verify) defined over ($K$ - key, $M$ - message, $T$ - tag):
- $S:K\times M\rightarrow T$
- $V:K\times M\times T\rightarrow\{\top,\bot\}$
- Consistent: $V(k,m,S(k,m))=\top$
- Unforgeability: It is hard to compute a valid pair $(m,S(k,m))$ without knowing $k$

# ECBC-MAC
![[w3n5EcbcMac.png]]
ECBC-MAC uses a [[W3N3 - Symmetric encryption systems#Block ciphers|block cipher]] in a similar way to [[W3N3 - Symmetric encryption systems#The message is longer than the block|CBC]]. The final block result is then encrypted using another key as otherwise it would be possible to add extra blocks to the end of the message and easily compute a new MAC.

An example of ECBC-MAC is [WPA2](https://en.wikipedia.org/wiki/IEEE_802.11i-2004), which uses an [[W3N3 - Symmetric encryption systems#Advanced Encryption Standard (AES)|AES]] based ECBC-MAC.

# PMAC
![[w3n5Pmac.png]]
- $E:K\times\{0,1\}^n\rightarrow\{0,1\}^n$ - a block cipher
- $P:K\times\mathbb{N}\rightarrow\{0,1\}^n$ - any easy to compute function
- $PMAC:K^2\times\{0,1\}^n\rightarrow\{0,1\}^n$

PMAC uses a block cipher also, but in parallel which makes it faster to compute where multithreading is possible. Each block is combined with the result of a function on $K_2$ and the index of the block, then all but that last block are encrypted with $K_1$, combined, and encrypted again with $K_1$.

# HMAC
![[w3n5Hmac.png]]
Uses a [[W3N4 - Hash functions|hash function]] in a similar way to a [[W3N4 - Hash functions#Merkle-Damgard construction|Merkle-Damgard construction]].

Used by SSL, IPSec, and SSH, among others.