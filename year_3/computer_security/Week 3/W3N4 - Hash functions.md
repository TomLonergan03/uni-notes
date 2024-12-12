While encryption guards the confidentiality against eavesdroppers, hashes and signatures guard the authenticity and integrity of messages.

# One way functions (OWFs)
An OWF is a function that is easy to compute but hard to revert.
A commonly used OWF is multiplication of large primes, as factorisation has no efficient solution.

# Collision resistant functions (CRFs)
A function is a CRF if it is hard to find two messages that get mapped to the same value by the function
Similarly, multiplication of large primes is collision resistant, as every positive integer has a unique prime factorisation.

# Cryptographic hash functions
A cryptographic hash function takes messages of arbitrary length and returns a fixed size bitstring such that any change to the data will (with very high priority) change the corresponding hash value.

## Definition
A cryptographic hash function $H:M\rightarrow T$ is a function that satisfies the following 4 properties:
- $|M|>>|T|$
- It is easy to compute the hash value for any given message
- It is a one way
- It is collision resistant

## Applications
- **Commitments** - allows a participant to commit to a value $v$ by publishing the hash $H(v)$ of this value, but only reveal $v$ only later.
	- E.g. digital signatures
- **File integrity** - hashes are posted along with files in some "read-only" spaces to allow verification of integrity of the files. E.g. SHA-256 is used to authenticate Linux packages
- **Password verification** - Instead of storing passwords in cleartext, only the hash value of the password is stored. To authenticate a user, the password entered by the user is hashed, and that hash is compared with the stored hash. This means that if the password server is breached, the passwords themselves are not compromised.
- **Key derivation** - used to derive new keys or passwords from a single secure key or password
- Also used to build MACs, block ciphers, PRGs, etc.

# Birthday attack
As a hash function's message domain is much larger than its hash range there are a lot of collisions, although the hash range in an effective hash function is large enough this is still very unlikely for any pair of messages.

We can exploit this by generating $l$ messages where $l$ is the number of possible hash values. Once we've done this there is guaranteed that either a collision has already occurred, or the next hash generated will collide. Due to the [birthday paradox](https://en.wikipedia.org/wiki/Birthday_problem), we generally need to generate far fewer hashes than that, which works out to a run time of $O(2^{n/2})$. This means that cryptographic functions used should have an output size $n\geq256$.

# Merkle-Damgard construction
![[w3n4MerkleDamgardConstruction.png]]
By using a compression function $h:T\times X\rightarrow T$ which compresses an IV and a block into a hash equal in size to the IV then chaining these together we can then use the final output as the hash. This has been proven to only produce a collision if $h$ produces a collision. The initialisation vector is defined by the hash function, so e.g. all MD5 hashes use the same IV.

Many common hash functions use the MD construction such as MD5, SHA-1, and SHA-2.