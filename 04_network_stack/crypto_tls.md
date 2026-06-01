# Cryptography and Native TLS 1.2

A modern operating system needs secure communications. Rather than relying on massive external libraries like OpenSSL or mbedTLS, LumiaOS includes a **native, from-scratch TLS 1.2 implementation**.

## The Cryptographic Primitives

The `src/crypto/` directory contains all the underlying mathematical algorithms required to establish a secure connection. These are written purely in C/C++ without external dependencies.

1. **AES-128 / AES-GCM:** 
   - `aes128.cpp`: The core Advanced Encryption Standard block cipher.
   - `ghash.cpp`: Galois Hash function used for message authentication.
   - `aes_gcm.cpp`: The Galois/Counter Mode authenticated encryption mechanism, which is the standard cipher suite for modern TLS.

2. **X25519 (Elliptic Curve Diffie-Hellman):**
   - `x25519.cpp`: Implements the Curve25519 key exchange algorithm. This allows the client (LumiaOS) and the server to securely agree on a shared secret over an insecure channel.

3. **Hashing and HMAC:**
   - `sha256.cpp`: The SHA-256 cryptographic hash function.
   - `hmac_sha256.cpp`: Hash-based Message Authentication Code used for verifying data integrity.
   - `tls_prf.cpp`: The TLS Pseudo-Random Function, used to derive the final encryption keys from the shared secret.

4. **X.509 Certificates:**
   - `asn1.cpp` & `x509.cpp`: Basic parsers for ASN.1 DER-encoded structures. Currently used to extract public keys from server certificates during the handshake.

## The TLS 1.2 Handshake State Machine

The logic for establishing a secure connection is housed in `src/net/tls.cpp`. It acts as a wrapper over the raw TCP socket.

When the Lumia Browser encounters an `https://` URL, it enters the TLS handshake state machine:
1. **ClientHello:** The OS sends its supported cipher suites (specifically `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`) and a random sequence.
2. **ServerHello & Certificate:** The server responds with its chosen cipher, its certificate chain, and its X25519 public key.
3. **Key Exchange:** The OS parses the server's public key, generates its own ephemeral X25519 keypair, and calculates the shared secret.
4. **ClientKeyExchange & Finished:** The OS sends its public key to the server, uses `tls_prf` to derive the Master Secret and AES keys, and switches the socket into encrypted mode.
5. **Application Data:** All subsequent HTTP GET requests and responses are encrypted and decrypted using `aes_gcm`.

*Note: The native TLS stack is currently highly experimental and strictly tied to AES-128-GCM and X25519. It is designed as an educational proof-of-concept rather than a production-ready security boundary.*
