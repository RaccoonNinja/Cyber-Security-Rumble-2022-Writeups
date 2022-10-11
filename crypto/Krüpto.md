## Skills required: Googling

The research took some time but it was worth it.

## Solution

My solution is completely free from number theory, but for learning purpose I will provide reference readings as needed.

The challenge uses [Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) ([you can try reading this if the wiki page is hard to understand](https://kakaroto.ca/2012/01/how-the-ecdsa-algorithm-works/))
The exact algorithm is not very important, but there are 2 takeaways:
- The operations are in modulo `n`
- The signature consists of two parts: `r` and `s`

There is explicitly a blacklist condition in the source:

```java
if (decodedBytes.length != 64 || Arrays.equals(decodedBytes, new byte[64])) {
    System.out.println("REPELLED ATTACK OF RECENTLY FOUND VULNERABILITY!");
    System.exit(0);
}
```

Upon searching for `ECDSA vulnerability`, it is not hard to learn about the [Psychic Signature vulnerability](https://neilmadden.blog/2022/04/19/psychic-signatures-in-java/) in the Java language.
It is a critical vulnerability recently discovered, which allows malicious actors to use an all-zero signature `(r,s)=(0,0)`, which will pass all checks in Java.

The `Arrays.equals(decodedBytes, new byte[64])` part blacklisted this case, **but not `(r,s)=(0,n)` or `(r,s)=(n,n)`**.
In a follow-up blog post, Neil Madden [elaborated on this idea](https://neilmadden.blog/2022/04/25/a-few-clarifications-about-cve-2022-21449/) and shared [test cases compiled by Project Wycheproof](https://github.com/google/wycheproof/blob/master/testvectors/ecdsa_secp256r1_sha256_p1363_test.json#L102).

From the program output we can indeed see that [secp256r1](https://neuromancer.sk/std/secg/secp256r1) is indeed used and double check the `n` value.

We can just try the (0,n) or (n,n) ones to get the flag. I think I used `0000000000000000000000000000000000000000000000000000000000000000ffffffff00000000ffffffffffffffffbce6faada7179e84f3b9cac2fc632551`.
