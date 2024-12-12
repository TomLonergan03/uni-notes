A **symmetric cipher** consists of two algorithms:
- An encryption algorithm $E:K\times M\rightarrow C$
- A decryption algorithm $D:K\times C\rightarrow M$
$\forall k\in K,\forall m\in M, D(k,E(k,m)) = m$

This means that the encryption algorithm takes a key and a message, and produces an encrypted message, and the decryption algorithm takes the same key and the encrypted message, and returns the original message.

# Uses
- File encryption
	- The data gets en/decrypted using the same key, such as from a password. In this case it will likely be the same user encrypting and decrypting the file
- Secure communication:
	- In this case, the key for symmetric communication must be shared between the two parties (generally by starting with an asymmetric encryption scheme)

# What makes a good encryption scheme
An encryption scheme is secure against a given adversary, if this adversary cannot:
- recover the secret key $k$ from a cipher text
- recover the plaintext $m$ underlying a ciphertext $c$
- recover any bits of the plaintext $m$ underlying a ciphertext $c$

# Kerckhoff's principle
The only part of a cryptographic system that should be hidden are the secret keys. The architecture, design, and algorithm should be public, as this allows many users, white hat hackers, academics, etc. to scrutinise the system, enabling early discovery and corrections of flaws or vulnerabilities.

# Adversary capabilities
A cryptographic scheme is secure under some assumptions (against a certain type of attacker)
A cryptographic scheme may be vulnerable to certain types of attacks but not others

The attacker knows the encryption/decryption algorithm, but may also have access to:
- **ciphertext only attack** - some cyphertexts $c_1,...,c_n$
- **known plaintext attack** - some plaintext/cyphertext pairs $(m_1,c_1),...,(m_n,c_n)$ such that $c_i=E(k,m_i)$
- **chosen plaintext attack** - the attacker has access to an encryption oracle - can perhaps trick a user into encrypting messages $m_1,...,m_n$ of his choice ^366bb4
- **chosen ciphertext attack** - he has access to a decryption oracle - can perhaps trick a user into decrypting ciphertexts $c_1,...,c_n$ of his choice
- either unlimited, polynomial, or realistic ($\leq 2^{80}$) **computational power**

# Brute force attack
All encryption schemes are vulnerable to brute force attacks - i.e. by trying all possible keys we can find which key works to decrypt a message. This does still require some knowledge of what the ciphertext should decrypt to, in order to know when the correct message is found.

This is fairly simple to prevent by:
- Ensuring $K$ is sufficiently large, i.e keys should be sufficiently long
	- If we know all keys are between 0-100 then it is trivial to check all of them
	- $K$ tends to have 128+ bits
- Keys should be sampled uniformly at random from $K$
	- If we have a 128 bit key, but keys are assigned sequentially from 0, then it is likely easy to find they key used in any case, as it is unlikely to give out keys in scales greater than billions

# Perfect secrecy
A cipher $(E,D)$ over $(M,C,K)$ satisfies perfect secrecy if for all messages $m_1,m_2\in M$ of the same length ($|m_1|=|m_2|$), and for all ciphertexts $c\in C$
$$
|Pr(E(k,m_1)=c)-Pr(E(k,m_2)=c)|\leq\epsilon
$$
where $k\leftarrow K$  and $\epsilon$ is some "negligible quantity".

Essentially, no matter what ciphertext you have, you cannot say anything about what the key or message were. This does not however guarantee that you can't get information about the keys/messages if you have many ciphertexts.

