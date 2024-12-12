Where [[W4N2 - Public key encryption|asymmetric encryption]] allows for encrypted communication to occur for two parties who do not need to share a single key beforehand, a digital signature does the same, acting as an asymmetric version of [[W3N5 - Message Authentication Codes (MACs)|MACs]].

A digital signature has:
- a key generation algorithm $G:\rightarrow K\times K$
- signing algorithm $S:K\times M\rightarrow S$
- verification algorithm $V:K\times M\times S\rightarrow\{\top,\bot\}$
- such that $\forall(sk,vk)\in G$, and $\forall m\in M, V(vk,m,S(sk,m))=\top$

# Advantages of digital signatures over MACs
MACs:
- are not publicly verifiable:
	- Nobody but the key-holders can verify if a tag is correct
- do not provide non-repudiation:
	- A tag is not bound to a specific key-holding party. Either party can sign anything with that key, and all that can be said is that one of those people signed it

Digital signatures:
- are publicly verifiable
- are transferable:
	- by giving the public key to more people, they all can verify that the signer did actually sign the message
- provide non-repudiation:
	- If a document is signed with a secret key, only that key-holder could have signed it

# Textbook RSA signatures
Uses [[W4N2 - Public key encryption#RSA trapdoor permutation|raw RSA]]:
- $G_{RSA}$ remains the same
- $M$ remains the same
- Signing: $S_{RSA}(sk,x)=(x,x^d\mod N))$
- Verifying: $V_{RSA}(pk,m,x)=\cases{\top\;\;\text{if }m=x^e\mod N\\\bot\;\;\text{otherwise}}$ 
- such that $\forall(pk,sk)\in G_{RSA}(),\forall x,V_{RSA}(pk,x,S_{RSA}(sk,x))=\top$

## Flaws with textbook RSA signatures
If we have two valid signatures $\delta_1=M^d_1\mod n$ and $\delta_2=M^d_2\mod n$ , and the public key for those signatures $d$, we can then calculate the signature for $M_1\cdot M_2$ which is just $(M_1\cdot M_2)^d\mod n$.

## Solution
Before computing the RSA function, we apply a hash function $H$.
- Signing: $S_{RSA}(sk,x)=(x,H(x)^d\mod N)$
- Verifying: $V_{RSA}(pk,m,x)=\cases{\top\;\;\text{if }H(x)=x^e\mod N\\\bot\;\;\text{otherwise}}$
