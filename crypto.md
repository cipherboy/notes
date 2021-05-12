# Cryptography

## Padding

### null padding

This is not ideal as it isn't strictly reversible because, if the data ends
with one or more null bytes, any extra null bytes can't be correctly detected.

### PKCS#5 and PKCS#7 Padding

PKCS#7 padding adds up to a block length of data. The byte added is dependent
on the delta block size:

    block_size - (len(data) % block_size)

This means that in the event of a full-block sized message, a full block is
appended to the data.

This is reversible but not ideal.

### 10 + length padding

Hash functions like MD5 through SHA-2 use a variant of 10-padding that also
includes the length. This adds a single byte `0x80` followed by one or more
null bytes (`0x00`) and then the length of the input data (without padding)
in a specified encoding form (e.g., big endian). Up to one additional block
is added.

This is reversible and used by many hash functions.

### RSA PKCS#1 v1.5 padding scheme

This is used by the insecure form of RSA. It uses the following scheme:

    0x00 || 0x02 || <random> || 0x00 || message

where `<random>` is chosen to be long enough so that the total message
fits within the modulus of the message. The standard specifies `<random>`
must be at least 8 bytes.

However [some](https://crypto.stackexchange.com/questions/66521/why-does-adding-pkcs1-v1-5-padding-make-rsa-encryption-non-deterministic)
[sources](http://www.cs.umd.edu/~jkatz/imc.html) think that `<random>` must
be at least half of the total length of the modulus space in order to be
CPA secure.

## AES Cipher Modes

### ECB: Electronic Codebook

Insecure. Each block gets encrypted individually. Trivially parallelizable.
ECB Tux. Confidentiality only.

### CBC: Cipher Block Chaining

Deprecated by NIST for key storage/transit. Confidentiality only.
Parallelizable on decryption only.

Encryption:

    enc(key, IV xor plaintext) -> previous ciphertext becomes next IV

Decryption:

    dec(key, ciphertext) xor IV -> previous ciphertext still is IV

However, it has a weakness: any error in the ciphertext (e.g., modification
after generation) causes a similar delta in the _plaintext_ of the next
block.

See also: Cryptopals Set 2, Challenge 16.

### CTR: Counter

Turns into a stream cipher. Confidentiality only. Trivially parallelizable.
Some amount of the IV is a random nonce, some amount is reserved for a
counter. Usually this is split half and half (8 bytes for nonce, 8 bytes for
counter). Requires strictly new nonce be used each time.

Encryption:

    enc(key, IV) xor plaintext

Decryption:

    enc(key, IV) xor plaintext

Note that encryption function is always used.

### GCM: Galois/Counter

Requires strictly unique nonces be used each time. AEAD.

### CCM: CBC MAC

Requires strictly unique nonces be used each time. AEAD.

### AES-GCM-SIV: Galois/Counter with Synthetic IV

Nonce-reuse resistant GCM. AEAD. Preferred solution for new work going forward
(if primitive work is required).

## RSA

 - `p`, `q`: two randomly generated primes,
 - `e`: an encryption exponent, usually 65537 (formerly 3),
 - `n`: `p*q`: public modulus; hard to reverse due to factoring problem,
 - `d`: `e^-1 mod \lambda(n) = lcm(p-1, q-1)`

Public key is `(e, n)` and private key is `(d, n)`. Optionally, `p` and `q`
are saved, along with option optimized values:

 - `d_P`: `d (mod p-1)`
 - `d_Q`: `d (mod q-1)`
 - `q_inv`: `q^-1 (mod p)`.

Encryption/Verification:

    ciphertext = plaintext^e (mod n)

Decryption/Signature:

    plaintext = ciphertext^d (mod n)

    m_1 = ciphertext^(d_P) (mod p)
    m_2 = ciphertext^(d_Q) (mod q)
    h = q_inv(m_1 - m_2) (mod p)
    plaintext = m_2 + hq (mod n)

This is more efficient due to the smaller modulus and exponents used.

### PKCS#1v1.5

Many attacks against this form by Bleichenbacher. Standard RSA implementation
with the PKCS#1v1.5 padding scheme documented above. This is largely falling
out of favor, being deprecated by NIST by 2023.

## Attacks

### Chosen-Plaintext Attack (CPA)

Cryptanalysis attack model in which the attacker is assumed to be able to get
arbitrary ciphertexts by querying an oracle with a specified
(attacker-controlled) plaintext.

## X.509

### Implementations

[NSS Certificate Validation](https://github.com/nss-dev/nss/blob/master/lib/certhigh/certvfy.c#L1372)

1. Validate that it is valid at the current time (+/- some overrides).
2. Check the certificate usage against required usages.
   1. Check the key usage against inferred usages.
   2. Validate usage against certificate type.
   3. Validate cert+usage against trust chain (to ensure usage can be
      assigned by parent keys).
3. Perform verification of the chain.
4. Perform OCSP verification of the certificate (only done once).

3 is where most of the [interesting work](https://github.com/nss-dev/nss/blob/master/lib/certhigh/certvfy.c#L594) is done.

1. Again validate key usage and key type for the expected certificate usage.
2. Validate NSS DB trust flags for the certificate usage.
3. For each possible chain:
   1. Get the constrained names for the certificate.
   2. Find its issuer.
   3. Verify the signature from issuer on the cert.
   4. Validate basic constraints from issuer.
   5. Validate namespace matches issuer.
   6. Validate the certificate issuance time is allowed.
   7. Check for a CRL entry for this certificate.
   8. Validate trust on the issuer.

Note that these are both independent certificate validation routines. The
routine we prefer for verification for TLS certificates ion JSS is
[`CERT_PKIXVerifyCert`](https://github.com/nss-dev/nss/blob/master/lib/certhigh/certvfypkix.c#L2027).

## Interesting Resources

[cryptopals](https://cryptopals.com)
