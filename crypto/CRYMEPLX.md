## Skills required: Knowing your block cipher [modes of operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)

A simple challenge.

## Solution:

In the context of [block ciphers](https://en.wikipedia.org/wiki/Block_cipher), messages are divided into blocks and then encrypted.
If we encrypt each block in the same way independently, underlying patterns of the plaintext will be known.
This is why there are many modes of operation for block cipher besides the insecure Electronic Code Book.

CTR, the mode used in this challenge, is commonly considered more secure than CBC, as [padding oracle attacks](https://en.wikipedia.org/wiki/Padding_oracle_attack) can be possible for CBC.
However, there is one important catch. Being essentially a stream cipher, CTR generates a stream of bytes that is XORed to the plaintext/ciphertext for encryption/decryption. **the nonce and key must not be reused**.
This means that this encryption oracle allows us to calculate the flag by simple XORs even though the key and nonce are truly random:

```py
# Actual code used during CTF aside from comments
A = bytes.fromhex("1c411606df044fdfe2d80178b65e05c9d30ee86cb422a89205") # encrypted flag
B = bytes.fromhex("1c411606a104118cb79b6e29f40b6a9f9650be29ea74edcc48") # the ciphertext
C = b"CSR{" + b"0"*(len(A)-4)                                           # the plaintext to be encrypted

print(bytes(list(
    a^b^c for a,b,c in zip(A,B,C)
)))
```

P.S. It can be automated and simplified with [pwntools](https://docs.pwntools.com/en/stable/globals.html):

```py
# post event version, the servers are off already
from pwn import *
conn = process(['python3', 'encrypt.py'])

A = unhex(conn.readline().strip())
C = b'\x00'*len(A)
conn.sendlineafter(b':', C)
B = unhex(conn.readline().strip())

print(xor(A,B,C))
```
