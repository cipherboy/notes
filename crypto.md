# Cryptography

## Padding

Block ciphers and most asymmetric primitives require some form of padding to
use.

### null padding

This is not useful as it isn't strictly reversible because, if the data ends
with one or more null bytes, any extra null bytes can't be correctly detected.

Not aware of any legitimate uses.

### PKCS#5 and PKCS#7 Padding

PKCS#7 padding adds up to a block length of data. The byte added is dependent
on the delta block size:

    block_size - (len(data) % block_size)

This means that in the event of a full-block sized message, a full block is
appended to the data.

This typically gets used in block cipher modes such as AES-CBC.

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

## RSA: Rivest Shamir Adleman

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

### PSS: Probabilistic Signature Scheme

This is a more secure signature scheme standardized as part of PKCS#1v2.1.
Preferred solution for modern protocols that need to rely on RSA.

### OAEP: Optimal Asymmetric Encryption Padding

This is a more secure signature scheme standardized as part of PKCS#1v2. This
is the preferred solution for modern protocols that need to rely on RSA.

## DH: Diffie-Hellman

Commonly called a key-exchange algorithm. Given a group, `G`, with generator
`g` of order `n`, and two parties (A and B):

 - A chooses a value `1 < a < n` and computes `g^a`,
 - B chooses a value `1 < b < n` and computes `g^b`,
 - A and B exchange computed values (`g^a` and `g^b`),
 - A computes `(g^b)^a`, B computes `(g^a)^b`, where
 - These two values must match due to properties of a cyclic group.

Typically the values aren't used directly but are instead e.g., hashed.
There's two forms of DH:

 1. FFC DH: Finite-Field Cryptography DH, based on modular exponentiation mod
    a prime.
 2. ECDH: Elliptic-Curve DH, based on a elliptic-curve group.

A suffix E (Ephemeral) is added to notate when private keys are chosen
per-session instead of stored and reused across multiple sessions. This is
commonly used in TLS.

## Attacks

### Brute Force

Limited utility; attacker attempts to brute-force all possible keys. This is
notably only a concern on 1-DES, where EFF's descracker is able to search the
entire key space.

### Chosen-Plaintext Attack (CPA)

Cryptanalysis attack model in which the attacker is assumed to be able to get
arbitrary ciphertexts by querying an oracle with a specified
(attacker-controlled) plaintext.

### Chosen-Ciphertext Attack (CCA)

Cryptanalysis attack model in which the attacker is assumed to be able to get
arbitrary plaintexts by querying an oracle with a specified
(attacker-controlled) ciphertext.

### Adaptive Chosen-Plaintext Attack (A-CPA)

Under traditional modeling, CPA and CCA's set of attacker-controlled inputs
are computed ahead of time. However, under A-CPA, the attacker can submit
different inputs based on the results past submissions, instead of requiring
all inputs be known ahead of time.

### Related-Key Attack

Cryptanalysis attack model in which the attacker can observe the operation
(plaintext, ciphertext pairs) for various keys with a matched pattern. The
assumption is that while the original key is not known the attacker, other
the delta (to new keys) can be controlled by the attacker. Thus the oracle
would be:

    oracle(plaintext, delta):
        return enc(key xor delta, plaintext)

### Pass the Hash

Attack in which the attacker can authenticate as another user by using the
_hash_ of the target's password rather than the password itself. In this
case, the assumption is that the hash is effectively public information
and thus cannot be used for authentication.

### Preimage Attack

Attack in which, given a hash `v`, the attacker attempts to find a message
`m` such that `hash(m) == v`.

### Collision Attack

Attack in which the attacker attempts to find any two messages `m_1` and `m_2`
whereby `hash(m_1) == hash(m_2)`.

Note that collision resistance implies second-preimage resistance but not
preimage resistance.

#### Chosen-Prefix Collision Attack

Attack in which the attacker attempts to find any two messages `m_1` and `m_2`
such that, given two prefixes `p_1` and `p_2`, the following holds:

    hash(p_1 || m_1) == hash(p_2 || m_2)

This is useful in attempting to provide collisions in X509 certificates.

### Second-Preimage Attack

Attack in which the attacker attempts to find any message `m_2`, such that
under a given `m_1` (hashing to `v`), the following holds:

    hash(m_1) == v == hash(m_2)

Note that any second-preimage attack trivially yields a collision attack.

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

 - [cryptopals](https://cryptopals.com), online challenges about cryptography.
 - [Illustrated TLS](https://tls.ulfheim.net/), step-by-step TLS.

## Canonical References

### PKCS#11

 - [PKCS#11 v2.40](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html)
 - [PKCS#11 v3.0](http://docs.oasis-open.org/pkcs11/pkcs11-base/v3.0/os/pkcs11-base-v3.0-os.html)

### TLS and Extensions

 - [RFC 2246 - TLSv1.0](https://datatracker.ietf.org/doc/html/rfc2246)
 - [RFC 4346 - TLSv1.1](https://datatracker.ietf.org/doc/html/rfc4346)
 - [RFC 5246 - TLSv1.2](https://datatracker.ietf.org/doc/html/rfc5246)
 - [RFC 8446 - TLSv1.3](https://datatracker.ietf.org/doc/html/rfc8446)
 - [draft-ietf-tls-dtls13](https://datatracker.ietf.org/doc/html/draft-ietf-tls-dtls13)
 - [RFC 7301 - TLS ALPN (Application-Layer Protocol Negotiation Extension)](https://datatracker.ietf.org/doc/html/rfc7301)
 - [RFC 8701 - GREASE (Generate Random Extensions and Sustain Extensibility)](https://datatracker.ietf.org/doc/html/rfc8701)
 - [RFC 6066 - TLS Extensions](https://datatracker.ietf.org/doc/html/rfc6066)

### Certificate Issuance and ACME

 - [RFC 1421 - Privacy Enhancement for Internet Electronic Mail](https://datatracker.ietf.org/doc/html/rfc1421)
   - Originally [RFC 1113](https://datatracker.ietf.org/doc/html/rfc1113)
 - [RFC 5652 - CMS (Cryptographic Message Syntax)](https://datatracker.ietf.org/doc/html/rfc5652)
   - Updated by [RFC 8933 - Updates to CMS for Algorithm Identifier Protection](https://datatracker.ietf.org/doc/html/rfc8933)
   - Originally [RFC 2630 - CMS](https://datatracker.ietf.org/doc/html/rfc2630)
     - Updated by [RFC 3211 - Password-based Encryption for CMS](https://datatracker.ietf.org/doc/html/rfc3211)
     - Updated by [RFC 3369 - CMS](https://datatracker.ietf.org/doc/html/rfc3369)
     - Updated by [RFC 3852 - CMS](https://datatracker.ietf.org/doc/html/rfc3852)
       - Updated by [RFC 4852 - CMS Multiple Signer Clarification](https://datatracker.ietf.org/doc/html/rfc4853)
       - Updated by [RFC 5083 - CMS Authenticated-Enveloped-Data Content Type](https://datatracker.ietf.org/doc/html/rfc5083)
 - [RFC 5272 - CMC (Certificate Management over CMS)](https://datatracker.ietf.org/doc/html/rfc5272)
   - Updated by [RFC 6402 - CMC Updates](https://datatracker.ietf.org/doc/html/rfc6402)
   - Originally [RFC 2797](https://datatracker.ietf.org/doc/html/rfc2797)
 - [RFC 8555 - ACME (Automatic Certificate Management Environment)](https://datatracker.ietf.org/doc/html/rfc8555)
 - [RFC 8737 - ACME TLS ALPN Challenge Extension](https://datatracker.ietf.org/doc/html/rfc8737)
 - [RFC 8738 - ACME IP Identifier Validation Extension](https://datatracker.ietf.org/doc/html/rfc8738)
 - [RFC 6844 - DNS CAA (Certification Authority Authorization) Resource Record](https://datatracker.ietf.org/doc/html/rfc6844)
   - Superseded by [RFC 8659 - DNS CAA Resource Record](https://datatracker.ietf.org/doc/html/rfc8659)
 - [RFC 6962 - Certificate Transparency](https://datatracker.ietf.org/doc/html/rfc6962)

### Certificate Validation

 - [RFC 6960 - OCSP (Online Certificate Status Protocol)](https://datatracker.ietf.org/doc/html/rfc6960)
   - Updated by [RFC 8954 - OCSP Nonce Extension](https://datatracker.ietf.org/doc/html/rfc8954)
   - Originally [RFC 2560](https://datatracker.ietf.org/doc/html/rfc2560)
     - Updated by [RFC 6277 - OCSP Algorithm Agility](https://datatracker.ietf.org/doc/html/rfc6277)
 - [RFC 5280 - Internet X.509 Public Key Infrastructure Certificate and CRL (Certificate Revocation List) Profile](https://datatracker.ietf.org/doc/html/rfc5280)
   - Updated by [RFC 6818 - Updates to RFC 5280](https://datatracker.ietf.org/doc/html/rfc6818)
   - Updated by [RFC 8398 - Internationalized Email Addresses in X.509 Certificates](https://datatracker.ietf.org/doc/html/rfc8398)
   - Updated by [RFC 8399 - Internationalization Updates to RFC 5280](https://datatracker.ietf.org/doc/html/rfc8399)
   - Originally [RFC 2459](https://datatracker.ietf.org/doc/html/rfc2459) and [RFC 3280](https://datatracker.ietf.org/doc/html/rfc3280)

### HTTP and Extensions

 - [RFC 1945 - HTTP/1.0](https://datatracker.ietf.org/doc/html/rfc1945)
 - [RFC 2616 - HTTP/1.1](https://datatracker.ietf.org/doc/html/rfc2616)
 - [RFC 7231 - HTTP/1.1: Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231)
 - [RFC 7540 - HTTP/2](https://datatracker.ietf.org/doc/html/rfc7540)
 - [draft-ietf-quic-http - HTTP/3](https://datatracker.ietf.org/doc/html/draft-ietf-quic-http)
 - [RFC 7469 - HPKP: Public Key Pinning for HTTP](https://datatracker.ietf.org/doc/html/rfc7469)
 - [RFC 6797 - HSTS: HTTP Strict Transport Security](https://datatracker.ietf.org/doc/html/rfc6797)

### NIST SPs

 - [NIST SP800-12 Rev. 1 - An Introduction to Information Security](https://csrc.nist.gov/publications/detail/sp/800-12/rev-1/final)
 - [NIST SP800-38A - Recommendation for Block Cipher Modes of Operation: Methods and Techniques](https://csrc.nist.gov/publications/detail/sp/800-38a/final)
 - [NIST SP800-38A Addendum - Recommendation for Block Cipher Modes of Operation: Three Variants of Ciphertext Stealing for CBC Mode](https://csrc.nist.gov/publications/detail/sp/800-38a/addendum/final)
 - [NIST SP800-38B - Recommendation for Block Cipher Modes of Operation: the CMAC Mode for Authentication](https://csrc.nist.gov/publications/detail/sp/800-38b/final)
 - [NIST SP800-38C - Recommendation for Block Cipher Modes of Operation: the CCM Mode for Authentication and Confidentiality](https://csrc.nist.gov/publications/detail/sp/800-38c/final)
 - [NIST SP800-38D - Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC](https://csrc.nist.gov/publications/detail/sp/800-38d/final)
 - [NIST SP800-38E - Recommendation for Block Cipher Modes of Operation: the XTS-AES Mode for Confidentiality on Storage Devices](https://csrc.nist.gov/publications/detail/sp/800-38e/final)
 - [NIST SP800-38F - Recommendation for Block Cipher Modes of Operation: Methods for Key Wrapping](https://csrc.nist.gov/publications/detail/sp/800-38f/final)
 - [NIST SP800-38G Rev. 1 - Recommendation for Block Cipher Modes of Operation: Methods for Format-Preserving Encryption](https://csrc.nist.gov/publications/detail/sp/800-38g/rev-1/draft)
 - [NIST SP800-52 Rev. 2 - Guidelines for the Selection, Configuration, and Use of Transport Layer Security (TLS) Implementations](https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final)
 - [NIST SP800-56A Rev. 3 - Recommendation for Pair-Wise Key-Establishment Schemes Using Discrete Logarithm Cryptography](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final)
 - [NIST SP800-56B Rev. 2 - Recommendation for Pair-Wise Key-Establishment Using Integer Factorization Cryptography](https://csrc.nist.gov/publications/detail/sp/800-56b/rev-2/final)
 - [NIST SP800-56C Rev. 2 - Recommendation for Key-Derivation Methods in Key-Establishment Schemes](https://csrc.nist.gov/publications/detail/sp/800-56c/rev-2/final)
 - [NIST SP800-57 Part 1 Rev. 5 - Recommendation for Key Management: Part 1 – General](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)
 - [NIST SP800-57 Part 2 Rev. 1 - Recommendation for Key Management: Part 2 – Best Practices for Key Management Organizations](https://csrc.nist.gov/publications/detail/sp/800-57-part-2/rev-1/final)
 - [NIST SP800-57 Part 3 Rev. 1 - Recommendation for Key Management, Part 3: Application-Specific Key Management Guidance](https://csrc.nist.gov/publications/detail/sp/800-57-part-3/rev-1/final)
 - [NIST SP800-71 - Recommendation for Key Establishment Using Symmetric Block Ciphers](https://csrc.nist.gov/publications/detail/sp/800-71/draft)
 - [NIST SP800-90A Rev. 1 - Recommendation for Random Number Generation Using Deterministic Random Bit Generators](https://csrc.nist.gov/publications/detail/sp/800-90a/rev-1/final)
 - [NIST SP800-90B - Recommendation for the Entropy Sources Used for Random Bit Generation](https://csrc.nist.gov/publications/detail/sp/800-90b/final)
 - [NIST SP800-90C - Recommendation for Random Bit Generator (RBG) Constructions](https://csrc.nist.gov/publications/detail/sp/800-90c/draft)
 - [NIST SP800-106 - Randomized Hashing for Digital Signatures](https://csrc.nist.gov/publications/detail/sp/800-106/final)
 - [NIST SP800-107 Rev. 1 - Recommendation for Applications Using Approved Hash Algorithms](https://csrc.nist.gov/publications/detail/sp/800-107/rev-1/final)
 - [NIST SP800-108 - Recommendation for Key Derivation Using Pseudorandom Functions (Revised)](https://csrc.nist.gov/publications/detail/sp/800-108/final)
 - [NIST SP800-130 - A Framework for Designing Cryptographic Key Management Systems](https://csrc.nist.gov/publications/detail/sp/800-130/final)
 - [NIST SP800-131A Rev. 2 - Transitioning the Use of Cryptographic Algorithms and Key Lengths](https://csrc.nist.gov/publications/detail/sp/800-131a/rev-2/final)
 - [NIST SP800-132 - Recommendation for Password-Based Key Derivation: Part 1: Storage Applications](https://csrc.nist.gov/publications/detail/sp/800-132/final)
 - [NIST SP800-133 Rev. 2 - Recommendation for Cryptographic Key Generation](https://csrc.nist.gov/publications/detail/sp/800-133/rev-2/final)
 - [NIST SP800-135 Rev. 1 - Recommendation for Existing Application-Specific Key Derivation Functions](https://csrc.nist.gov/publications/detail/sp/800-135/rev-1/final)
 - [NIST SP800-185 - SHA-3 Derived Functions: cSHAKE, KMAC, TupleHash, and ParallelHash](https://csrc.nist.gov/publications/detail/sp/800-185/final)
 - [NIST SP800-186 - Recommendations for Discrete Logarithm-Based Cryptography: Elliptic Curve Domain Parameters](https://csrc.nist.gov/publications/detail/sp/800-186/draft)

### NIST FIPS

 - [NIST FIPS180-4 - Secure Hash Standard (SHS)](https://csrc.nist.gov/publications/detail/fips/180/4/final)
 - [NIST FIPS186-4 - Digital Signature Standard (DSS)](https://csrc.nist.gov/publications/detail/fips/186/4/final)
 - [NIST FIPS197 - Advanced Encryption Standard (AES)](https://csrc.nist.gov/publications/detail/fips/197/final)
 - [NIST FIPS198-1 - The Keyed-Hash Message Authentication Code (HMAC)](https://csrc.nist.gov/publications/detail/fips/198/1/final)
 - [NIST FIPS202 - SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions](https://csrc.nist.gov/publications/detail/fips/202/final)
