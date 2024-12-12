[[W3N2 - Symmetric encryption|Symmetric encryption]] requires both parties to share the same key, however in order to do this we need a secure way to share that key.

# Trusted Third Party (TTP)
A trusted third party is a single party trusted by all users who wish to communicate with each other.

If we have users $U_1,U_2,...,U_n$ then each user $U_i$ has a shared secret key $K_i$ with the TTP. If users $U_i$ and $U_j$ wish to establish a key, they contact the TTP and it generates a key and sends it to both users.

This has several disadvantages:
- The TTP must be trusted as it would be able to keep a copy of every key it generates
- The TTP must be secure, as if it was compromised it could start sharing keys with other parties
- The TTP must be online whenever two users want to establish a secure communication for the first time

It is possible to share a key without a TTP, using [[W4N2 - Public key encryption|public key encryption]].