We can use [[W4N2 - Public key encryption|asymmetric encryption]] and [[W4N3 - Digital signatures|digital signatures]] to share keys for [[W3N2 - Symmetric encryption|symmetric encryption]] which is used to secure two way conversations.

A public key encryption scheme is only secure if the authenticity of the public key is assured.

# Ways to distribute public keys
1. **Public announcements** - a participant broadcasts their public key.
	- This does not defend against forgeries, as any person could claim to be someone else when they share a key
2. **Publicly available directories** - participants publish their public keys on a public directory
	- This also does not defend against forgeries
3. **Public key authority** - participants contact a trusted authority for each public key it needs
	- This is a bottleneck in the system, as every time you want to communicate with someone you need to ask the third party for their key
4. **Public key certificates** - trusted certificate authorities (CAs) issue certificates to participants on their public keys
	- This is as reliable as a public key authority, but avoids the bottleneck

# Public key certificates
A certificate consists of:
- A public key
- A subject identifying the owner of the key
- A signature by the CA on the key and the subject binding them together

# X.509 certificates
![[w5n1x509cert.png]]X.509 is a framework for the provision of authentication services. It is used by many applications, including TLS.

![[w5n1BrowserSecure.png]]
The lock symbol means that the connection is secured using TLS. This is however, trusting the browser to be secure and that this lock symbol is only shown when the TLS connection is working, and the certificates are valid.

# Browser root certificates
The browser must also be able to certify the certificate authorities. This is done using the **browser root certificates**, which are distributed with the browser installation package. 

# Chain of trust
It would be impractical for one CA to sign all certificates, instead, a root certificate holding CA will sign a certificate for a lower level CA, which can then sign the certificate of another lower lever CA, and so on. By verifying all certificates until it reaches a root CA any certificates trustworthiness can be determined.

# Self signed certificate
Some organisations self-certify, which means that they issue a certificate without that certificate being signed by a CA that is in a chain of trust back to a root certificate. Installing one of these certificates is essentially adding a new root certificate, which leaves the user vulnerable in the case of a fraudulent self signed certificate.

# Certificate revocation
A certificate needs to be revoked if the corresponding private key or CA has been compromised.

**Certificate revocation lists (CRLs)** are used in X.509. Essentially, each CA posts a list of certificates it has issued that have been revoked, then the browser must check all certificates against that list in order to ensure that certificate is still valid.

**Online certificate status protocol (OSCP) stapling** is the modern solution, in which the server gets a time stamped proof of non-revocation from the CA which it attaches to its certificates, and must get a new proof regularly.