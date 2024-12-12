**Transport layer security (TLS)** is a cryptographic protocol which aims to ensure [[W5N2 - Cryptographic protocols for secure communication#Security requirements|confidentiality and integrity]] over a computer network.

The [[computer_security/Week 1/W1N1 - Intro#Threat modelling|threat model]] we are using is that the adversary fully controls the network, so they can:
- redirect traffic (DNS rebinding etc.)
- passively read all data (IP monitoring etc.)
- it can play an active MitM attack (TCP hijacking etc.)

The first component of securing the communication is having the server get a [[W5N1 - Public Key Infrastructure#Public key certificates|certificate]] from a CA. This means that as long as the adversary does not know the server's private key, it cannot:
- Inject forged data into the data stream
- Distinguish the data stream from random bytes

# TLS stages
|                   | Confidentiality                                                                                                                                                          | Integrity                                                   | Authentication                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Setup             | Public-key based key-exchange ([[W4N2 - Public key encryption#RSA trapdoor permutation\|RSA]] and [[W4N2 - Public key encryption#Diffie-Hellman (DH) protocol\|DH]]) | Public key [[W4N3 - Digital signatures\|digital signature]] | Public key digital signature|
| Data transmission | Symmetric encryption, e.g. [[W3N3 - Symmetric encryption systems#Advanced Encryption Standard (AES)\| AES in CBC mode]]                                           | Hash based MACs e.g. [[W3N5 - Message Authentication Codes (MACs)#HMAC\|HMAC using SHA256]] ||

## Setup
![[w6n1tls.png]]
The browser starts by sending a list of its supported cryptography algorithms, from which the server chooses the strongest algorithm it also supports. The server then sends its certificate, which the client verifies.
## Basic key exchange
The client generates a random secret value $R$, which it encrypts with the public key of the server and sends it to the server. The server can then decrypt that value with its private key.

### Forward secrecy
Compromise of the private key alone does not break confidentiality of past messages, however if the attacker has also eavesdropped and has a copy of the packet containing $R$ it can derive the encryption keys, so TLS with basic key exchange does not preserve [[W5N2 - Cryptographic protocols for secure communication#Forward secrecy|forward secrecy]]

## DH key exchange
![[w6n1TlsDhKeyExchange.png]]
Using DH to create the key preserves forward secrecy, but is still vulnerable to the DH key exchange MitM attack. This is prevented by having the server sign Y, and sometimes the client also signs X.

## LOGJAM
![[w6n1TlsLogjam.png]]
Here, the MitM intercepts the proposed crypto message and alters it to only include a known breakable algorithm. They can then calculate the value of $y$ for DH and create their own key. This allows them to create new messages and MAC/encrypt them. The MitM will also alter the transcript to state that the encryption protocol used for key exchange was a secure one. This can be fixed by having the server sign the transcript with a different key before it transmits it, preventing the MitM from being able to alter it.