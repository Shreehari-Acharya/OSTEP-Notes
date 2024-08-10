## Introduction

- Cryptography is used to protect data when it's outside the operating system's control
- Converts data from one form to another in controlled ways
- Goal is to make data unintelligible to opponents who don't have the key

## Basic Cryptography Concepts

### Encryption and Decryption

- Plaintext (P) is converted to ciphertext (C) using encryption algorithm E() and key K
- C = E(P, K)
- Decryption reverses this: P = D(C, K)

### Properties of Good Cryptography

- Deterministic - same inputs always produce same outputs
- Hard to determine P from C without knowing K
- Relies entirely on secrecy of the key

### Symmetric vs Asymmetric Cryptography

- Symmetric: Same key used for encryption and decryption
- Asymmetric (public key): Different keys for encryption and decryption
    - Public key can be widely shared
    - Private key kept secret

## Types of Cryptography

### Symmetric Cryptography

- Uses single shared secret key
- Fast but key distribution is challenging

### Public Key Cryptography

- Uses key pairs - public and private key
- Slower but solves key distribution problem
- Can be used for authentication and secure communication

### Cryptographic Hashes

- One-way function, cannot be reversed
- Used for integrity checking, password storage, etc.

## Uses of Cryptography in Operating Systems

### At-Rest Data Encryption

- Encrypts data stored on disk/storage devices
- Protects against physical theft of storage media
- Full disk encryption vs selective encryption

### Network Communication

- Protects data in transit between machines

### Cryptographic Capabilities

- Unforgeable access tokens using encryption

## Key Management

- Proper key selection and secrecy is critical
- Use truly random sources to generate keys
- Minimize storage and exposure of keys

## Limitations of Cryptography

- Does not protect against compromised OS
- Vulnerable to software flaws in implementations
- Key management remains a major challenge
