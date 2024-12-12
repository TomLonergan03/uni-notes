# One-Time Pad (OTP)
In a OTP, the message, key, and ciphertext are all bitstrings with the same length, i.e $M=C=K=\{0,1\}^n$. 
The encryption algorithm is to simply XOR the key with the message ($\forall k\in K. \forall m\in M.E(k,m)=k\oplus m$).
Decryption just requires XORing the key with the ciphertext ($\forall k\in K. \forall c\in C.D(k,c)=k\oplus c$).

An OTP satisfies [[W3N2 - Symmetric encryption#Perfect secrecy|perfect secrecy.]]

## Two-time pad attacks
If the same OTP key is used for both $m_1$ and $m_2$, then by XORing the ciphertexts together you get $m_1\oplus m_2$
![[w3n2TwoTimePadAttack.png]]
Generally, you can then obtain both original messages from this as there are patterns in languages and images.

## Limitations of OTPs
- Key length
	- The key must be at least as long as the plaintext
- Key sharing
	- It is difficult to share the keys for a two way conversation. Generally, both parties must share the keys in some insecure way, agree on the order in which they will be used, and then the conversation can occur. This can happen, but generally isn't done over a network. e.g. giving someone a physical disk/book of keys.
- Requires true randomness
	- If the key is not truly random then frequency analysis may be possible.
- Perfect secrecy does not guarantee security against all possible attacks
- Malleability ^b682f9
	- If we know, or strongly suspect, that a specific part of the message is a specific string, e.g. a header, then by XORing that string with the message we can recover the key for that part of the message. Therefore it is possible to modify known sections of the ciphertext without being able to decrypt the entire ciphertext.
# Stream ciphers
Stream ciphers aim to make OPT practical.

This uses a pseudorandom generator (PRG) to generate a key from a key seed:
$G:\{0,1\}^s\rightarrow\{0,1\}^n$ with $s << n$.

Using a PRG $G$:
Encryption: $E(k,m)=G(k)\oplus m$
Decryption: $D(k,c)=G(k)\oplus c$

Stream ciphers:
- Are much easier to manage key sharing, as keys are much shorter than messages. E.g you could include in each message the key seed used in the next message
- Are still vulnerable to two-time pad attacks

# Block ciphers
A block cipher encrypts a bitstring by dividing it into fixed length blocks and encrypting each block using the key.

It consists of:
- an encryption algorithm $E:\{0,1\}^k\times\{0,1\}^l\rightarrow\{0,1\}^l$
- a decryption algorithm $D:\{0,1\}^k\times\{0,1\}^l\rightarrow\{0,1\}^l$
where $k$ and $l$ are the lengths of the key and block respectively.

Current examples include:
- 3DES: $l=64,k=168$
- AES: $l=128,k=128,192,256$

# Triple DES (3DES)
3DES uses repeated application of DES encryption.
It is used in bank cards and RFID chips.

Let $DES=(E_{DES},D_{DES})$ and $3DES=(E_{3DES},D_{3DES})$.
$E_{3DES}:(\{0,1\}^k)^3\times\{0,1\}^l\rightarrow\{0,1\}^l$
$E_{3DES}((K_1,K_2,K_3),M)=E_{DES}(K_1,D_{DES}(K_2,E_{DES}(K_3,M)))$

(This encrypt then decrypt then encrypt again is for backwards compatibility. A 3DES implementation will also be able to encrypt and decrypt DES ciphers by simply setting $K_1=K_2=K_3$.)

$D_{3DES}:(\{0,1\}^k)^3\times\{0,1\}^l\rightarrow\{0,1\}^l$
$E_{3DES}((K_1,K_2,K_3),C)=D_{DES}(K_3,E_{DES}(K_2,D_{DES}(K_1,C)))$
The cost of 3DES is that it takes 3 times the runtime as DES.
3DES beats DES as it has a larger key size$=3\times56=168$ bits, which results in an exhaustive search space of $2^{168}$.

There is however a simple meet-in-the-middle attack in time $2^{118}$.

## 2DES and meet in the middle attacks
$E_{2DES}((K_1,K_2),M)=E_{DES}(K_1,E_{DES}(K_2,M))$
This has an exhaustive search time complexity of $2^{112}$, however, this is much weaker than 3DES, as we know that, for $m$ and $c$ such that $E_{2DES}((k_1,k_2),m)=c$ we have $E_{DES}(k_2,m) = D_{DES}(k_1,c)$.

Therefore, we can compute all possible $E_{DES}(k_2,M)$ and sort the results, then compute all possible $D_{DES}(k_1,C)$ and sort those results, then find the pair of keys which result in a matching value. Each of these steps is of time complexity $2^{56}\log(2^{56})$, or approximately $2^{63}$.

# Advanced Encryption Standard (AES)
AES was introduced to replace 3DES as it was too slow.
It has a block size $l=128$ bits, and key size $k=128,192,256$ bits, depending on the version.

AES encryption interprets each block being represented as a 4x4 matrix of bytes, then uses a key schedule to create 10 keys. Each of those keys is used in one round, during which various operations (the details of which are not covered in this course) are performed on the bytes, rows, and columns of the matrix.

## Attacks on AES
- **Related-key attack** on the 192 and 256-bit versions of AES exploits the AES key schedule
	- Key recovery in time $\approx2^{99}$
- There exists a key recovery attack on full AES which is 4 times faster than exhaustive search
Neither of these get to the [[W3N2 - Symmetric encryption#Adversary capabilities|realistic computation limits]] $2^{80}-2^{100}$ so AES is considered to be secure, however these attacks show that there may still be attacks that have not yet been discovered.

# Encrypting non-block size messages
## The message is shorter than the block 
- **Bit padding** - append a set bit ('1') at the end of the message, then as many reset bits ('0') as needed. After decrypting, the tail bits are stripped in reverse until the first set bit is found, meaning that if the message is exactly the length of the block, a second padding block will be needed.
- **ANSI X.923** - byte padding with zeros, then set the last byte to the number of padded bytes. Similarly it will require an extra block in the case where the message ends at the end of a block.
- **PKCS#7** - byte padding, the value of each padding byte is the total number of padding bytes. Here the padding would be `01`, or `02 02`, or `03 03 03`, and so on.

## The message is longer than the block
- **Electronic Code Book (ECB)** - to encrypt a message using ECB: ^5c41ba
	- The message is padded to be an exact multiple of the block length
	- Then it is broken into blocks of length $l$
	- Each block is then encrypted and the final ciphertext is concatenated together
	- This has a major **weakness**:
		- If two blocks have the same content, then the resulting ciphertext for those two blocks will be the same. This makes it both malleable (we can duplicate blocks within the message) and weak to frequency analysis ![[w3n3EcbWeakness.png]] 
		  Here we can see that the most of the image is preserved despite being encrypted, as every pure black or white block encrypts to the same value.
- **Cipher-block chaining (CBC)** - you use a random **initialisation vector (IV)** of length $l$ that is XORed with the first block before it is encrypted, then the ciphertext of the first block is XORed with the second message, and so on. The message also includes the initialisation vector as its first block. To decrypt, the first block of the ciphertext is decrypted, then XORed with the IV, then the second block is decrypted and XORed with the first cipher block, and so on. ^d670c6
	- This has the disadvantage that if a block is lost in transit then you cannot decrypt the message at all.
- **Counter (CTR)** - uses an IV, and encrypts each block using $E(k, IV + n)$ where $n$ is the number of the block. This means that each block can be decrypted separately, but still ensures that identical message blocks do not lead to identical cipher blocks. ^3a80c6

# Key management problem
Symmetric encryption requires a lot of keys to be distributed
![[w3n3SharedSymmetricKeys.png]]
The confidentiality problem is now reduced to a key management problem:
- Where and how are keys generated?
- How are keys shared?
- Where are keys stored?
- Where are keys actually used?
- How are keys revoked and replaced?