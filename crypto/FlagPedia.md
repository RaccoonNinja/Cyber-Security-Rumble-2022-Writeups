## Skills required: Knowing your block cipher [modes of operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)

Despite the apparent difficulty, this one is only marginally harder than CRYMEPLX.md.

## Solution:

Our goal is to get to the premium option, i.e. we have to forge a cookie that says `role` is `premium`.
Seeing the cookie encoded as a query string, at first I ran down some rabbit hole about [length extension attacks](https://en.wikipedia.org/wiki/Length_extension_attack).
That is not possible because:
- The signature is for the ciphertext and I cannot lengthen the ciphertext (no key)
- HMAC hashes are not prone to this attack

Nonetheless part of the idea is correct: **HTTP parameter pollution** is the way to go.
On this topic, [PwnFunction@YouTube](https://www.youtube.com/watch?v=QVZBl8yxVX0) has a really good video.

The first key insight is that the app uses only the *first* value of all query parameters:

![image](https://user-images.githubusercontent.com/114584910/195116622-35d4eff4-37f4-4400-8fde-45ab351515e5.png)

```py
    user_obj_raw = parse_qs(plaintext.decode())
    user_obj = {k: v[0] for k, v in user_obj_raw.items()}
```

For the second insight, we can refer to the schematic diagrams of CBC decryption:

![CBC decryption](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/CBC_decryption.svg/600px-CBC_decryption.svg.png)

Notice in the challenge, how the IV is **not** protected by the HMAC.

```py
    # guarantee ciphertext integrity
    mac = HMAC.new(INSTANCE_KEY, ct).digest()
    return (aes.iv + ct + mac).hex()
```

Combining the two: we can freely change the starting 16 bytes of the plaintext by tampering with the IV with simple XOR:

```py
# actual code used in the challenge aside from this comment
a = "user=stduser&role=pleb".encode('utf-8')
b = "role=premium&role=pleb".encode('utf-8')
iv = bytes.fromhex("0fb421fba7f9956fb7dbc2deaf2fd0fb")

c = [A^B^IV for A,B,IV in zip(a,b,iv)]
print("".join(hex(C)[2:].zfill(2) for C in c))
```

Using the forged identity, we can get the flag easily.
