Plain encryption is [[W3N3 - Symmetric encryption systems#^b682f9|malleable]], as the decryption algorithm will always produce a valid message from any ciphertext.

If we change one bit of the $i^{th}$ block of the ciphertext:
- [[W3N3 - Symmetric encryption systems#^d670c6|CBC decryption]] - all blocks after the $i^{th}$ block will be affected
- [[W3N3 - Symmetric encryption systems#^5c41ba|ECB decryption]] - only the $i^{th}$ block will be affected
- [[W3N3 - Symmetric encryption systems#^3a80c6|CTR decryption]] - only one bit of the $i^{th}$ block will be affected

We want to simultaneously provide data confidentiality, integrity, and authenticity in one step.

# Encrypt-then-MAC
1. Always compute the MAC on the ciphertext, never on the plaintext
2. Use two different keys, one for encryption ($K_E$) and one for the MAC ($K_M$)

## Encryption
1. $C\leftarrow E_{AES}(K_E,M)$
2. $T\leftarrow HMAC\textendash SHA(K_M,C)$
3. return $C||T$

## Decryption
1. if $T=HMAC\textendash SHA(K_M,C)$
2. then return $D_{AES}(K_E,C)$
3. else return $\bot$

Do not:
- Encrypt-and-MAC: $E_{AES}(K_E,M)||HMAC\textendash SHA(K_M,M)$
- MAC-then-encrypt: $E_{AES}(K_E,M||HMAC\textendash SHA(K_M,M))$

