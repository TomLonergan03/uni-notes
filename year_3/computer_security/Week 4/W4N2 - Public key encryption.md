A public-key encryption system consists of:
- A key generation algorithm $G:\rightarrow K\times K$
- An encryption algorithm $E:K\times M\rightarrow C$
- A decryption algorithm $D:K\times C\rightarrow M$
such that $\forall(sk,pk)\in G$, and $\forall m\in M, D(sk,E(pk,m))=m$
($sk$ is the secret key, which is used for decryption; $pk$ is the public key, which is used for encryption)

The secret key $sk_{Bob}$ is known only to Bob, the public key $pk_{Bob}$ is known to everyone. $sk_{Bob}\ne pk_{Bob}$

# Mathematical backing
$a$ and $b$ in $\mathbb{Z}$ are **relative primes** if they have no common factors

The **Euler function** $\phi(n)$ is the number of elements that are relative primes with $n$:
$$
\phi(n)=|\{m|0<m<n\text{ and }\gcd(m,n)=1\}
$$

If $p$ is prime then $\phi(p)=p-1$
If $p$ and $q$ are prime, then $\phi(p\cdot q)=(p-1)(q-1)$

Let $n\in\mathbb{N}$. We define $\mathbb{Z}_n=\{0,...,n-1\}$
$$
\forall a\in\mathbb{Z},\forall b\in\mathbb{Z}_n,a\equiv b\mod{n}\Leftrightarrow\exists k\in\mathbb{N}.a=b+k\cdot n
$$

Modular inversion: the inverse of $x\in\mathbb{Z}_n$ is $y\in\mathbb{Z}_n$ such that $x\cdot y\equiv 1\mod n$. We denote $x^{-1}$ the inverse of $x\mod n$.
E.g. $7^{-1}$ in $\mathbb{Z}_{12}$: $7$
$4^{-1}$ in $\mathbb{Z}_{12}$: $4$ has no inverse in $\mathbb{Z}_{12}$

**Theorem**
Let $n\in\mathbb{N}$. Let $x\in\mathbb{Z}_n$. $x$ has an inverse in $\mathbb{Z}_n$ iff $\gcd(x,n)=1$ 

Let $n\in\mathbb{N}$. We define $\mathbb{Z}^*_n=\{x\in\mathbb{Z}_n|\gcd(x,n)=1\}$ i.e. $\mathbb{Z}^*_n$ contains all values in $\mathbb{Z}_n$ which have an inverse.
E.g. $\mathbb{Z}^*_{12}=\{1,5,7,11\}$

Note that $|\mathbb{Z}^*_n|=\phi(n)$

**Euler theorem**
$\forall n\in\mathbb{N},\forall x\in\mathbb{Z}^*_n,\text{ if }\gcd(x,n)=1\text{ then }x^{\phi(n)}\equiv1\mod n$
and, $\forall p$ prime, $\mathbb{Z}^*_n$ is a cyclic group, i.e.
$$
\exists g\in\mathbb{Z}^*_p,\{1,g,g^2,...,g^{p-2}\}=\mathbb{Z}^*_p
$$

# Intractable problems
All of these have no known efficient (better than brute force) algorithm to find an input corresponding to a given problem.
- **Factoring**
	- Input: $n\in\mathbb{N}$
	- Output: $p_1,...,p_m$ primes such that $n=\prod^m_{i=0}p_i$
- **RSAP**
	- Input: $n$ such that $n=p\cdot q$ with $2\leq p,q$ primes
			  $e$ such that $\gcd(e,\phi(n))=1$
			  $m^e\mod n$
	- Output: $m$
- **Discrete log**
	- Input: prime $p$, generator $g$ of $\mathbb{Z}^*_p$,$y\in\mathbb{Z}^*_p$
	- Output: $x$ such that $y=g^x\mod p$
- **DHP**
	- Input: prime $p$, generator $g$ of $\mathbb{Z}^*_p,g^a\mod p,g^b\mod p$
	- Output: $g^{ab}\mod p$

# Diffie-Hellman (DH) protocol
We fix a very large prime $p$, and $g$ generator of $\mathbb{Z}^*_p$.
![[w4n2DhProtocol.png]]
## MitM attack on DH
![[w4n2DhMitmAttack.png]]
Here, the attacker is able to eavesdrop on messages between Alice and Bob, and can change or block messages as both Alice and Bob assume that the values they received from the attacker actually came from their counterpart.

# RSA trapdoor permutation
RSA consists of:
- A key generator $G_{RSA}()=(pk,sk)$
	- where $pk=(N,e)$ and $sk=(N,d)$
	  and $N=p\cdot q$ with $p,q$ random primes
	  and $e,d\in\mathbb{Z}$ such that $e\cdot d\equiv1\mod\phi(N)$
- $M=C=\mathbb{Z}_N$
- Encryption algorithm $RSA(pk,m)=m^e\mod N$
- Decryption algorithm $RSA^{-1}(pk,m)=m^e\mod N$
	- Both where $m\in M$

$(G_{RSA},RSA,RSA^{-1})$ is called raw RSA. Raw RSA is deterministic, so it is not secure against [[W3N2 - Symmetric encryption#^366bb4|chosen plaintext attacks]].

## Removing the CPA vulnerability
We can use raw RSA and make a few small changes to get a CPA resistant encryption scheme:
- Key generation stays the same $G_{RSA}()$
- $E_{RSA}(pk,m)$
	- random $x\in\mathbb{Z}^*_N$
	- $y\leftarrow RSA(pk,x)$
	- $k\leftarrow H(x)$
	- $E_{RSA}(pk,m)=y||E_S(k,m)$
	- $H$ is a secure hash function
	- $E_S(k,m)$ is a secure symmetric encryption function
- $D_{RSA}(sk,y||c)=D_S(H(RSA^{-1}(sk,y)),c)$

Essentially, we encrypt a symmetric encryption key using RSA and the destination public key, then use the hash of that symmetric key to encrypt the message. The encrypted key and the encrypted message are concatenated and transmitted.

To decrypt, the receiver uses their private key to decrypt the asymmetric key, then uses the hash of that to decrypt the message.

# ElGamal (EG)
Consists of:
- A fixed prime $p$, and generator $g\in Z^*_p$
- $M=\{0,...,p-1\}$ and $C=M\times M$
- $G_{EG}()=(pk,sk)$
	- where $pk=g^d\mod p$
	  and $sk=d$
	  and $d\xleftarrow{r}\{1,...,p-2\}$ 
- $E_{EG}(pk,x)=(g^r\mod p,m\cdot(g^d)^r\mod p)$
	- where $pk=g^d\mod p$
	  and $r\xleftarrow{r}\mathbb{Z}$
- $D_{EG}(sk,x)=e^{-d}\cdot c\mod p$

