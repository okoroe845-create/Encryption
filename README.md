# Encryption




# 🔐 Encryption Project — AES-256-CBC + RSA-2048 + Hybrid Encryption
### Performed by: Okoro Francis Emmanuel | New Horizon | 

<div align="center">

![Project Type](https://img.shields.io/badge/Type-Text%20Encryption%20Project-blue?style=for-the-badge&logo=lock)
![Algorithm](https://img.shields.io/badge/Algorithm-AES--256--CBC%20%7C%20RSA--2048-purple?style=for-the-badge)
![Tools](https://img.shields.io/badge/Tools-OpenSSL%20%7C%20Kali%20Linux-green?style=for-the-badge)
![PKI](https://img.shields.io/badge/Method-CA%20Certificate%20%7C%20PKI-orange?style=for-the-badge)
![Class Rank](https://img.shields.io/badge/Class%20Rank-Highest%20Score-gold?style=for-the-badge)

</div>

---

> **⚠️ Academic Disclaimer**
> This project was completed as part of the New Horizon Nigeria Cybersecurity Diploma programme. All encryption operations were performed in a controlled lab environment on Kali Linux using OpenSSL. No real sensitive data belonging to third parties was encrypted or handled at any point. All keys, certificates, and ciphertext produced in this project are for educational demonstration only.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [What Is PKI and Why It Matters](#-what-is-pki-and-why-it-matters)
- [Cryptography Concepts Covered](#-cryptography-concepts-covered)
- [Lab Environment](#-lab-environment)
- [Tools Used](#-tools-used)
- [Tool Breakdown](#-tool-breakdown)
- [Project Steps — Full Workflow](#-project-steps--full-workflow)
  - [Phase 1 — Build the Certificate Authority](#phase-1--build-the-certificate-authority-ca)
  - [Phase 2 — Generate RSA Key Pair](#phase-2--generate-rsa-2048-key-pair)
  - [Phase 3 — Create Certificate Signing Request](#phase-3--create-a-certificate-signing-request-csr)
  - [Phase 4 — CA Signs the Certificate](#phase-4--ca-signs-the-certificate)
  - [Phase 5 — Verify the Certificate Chain](#phase-5--verify-the-certificate-chain)
  - [Phase 6 — AES-256-CBC Symmetric Encryption](#phase-6--aes-256-cbc-symmetric-encryption)
  - [Phase 7 — Hybrid Encryption (RSA Wraps AES Key)](#phase-7--hybrid-encryption-rsa-wraps-the-aes-key)
  - [Phase 8 — Decryption and Verification](#phase-8--decryption-and-verification)
  - [Phase 9 — Integrity Verification with Hashing](#phase-9--integrity-verification-with-hashing)
- [Key Concepts Demonstrated](#-key-concepts-demonstrated)
- [Why Hybrid Encryption Is the Industry Standard](#-why-hybrid-encryption-is-the-industry-standard)
- [Lessons Learned](#-lessons-learned)
- [Next Steps](#-next-steps)
- [References](#-references)

---

## 🎯 Project Overview

This project is a complete, practical implementation of **hybrid encryption** using a self-built **Certificate Authority (CA)** to issue and sign RSA-2048 certificates — the same foundational method used by HTTPS, TLS, VPNs, S/MIME email, and every major PKI-based security system in the world.

The project covers the full lifecycle of Public Key Infrastructure (PKI): building a root CA, generating RSA key pairs, creating and signing certificates, encrypting data with AES-256-CBC, protecting the AES session key with RSA, performing decryption, and verifying integrity using cryptographic hashing.

Every step was executed manually using OpenSSL on Kali Linux — flag by flag — with no GUI tools or abstraction layers. This demonstrates a genuine understanding of how cryptography works at the command-line level, not just at the conceptual level.

**This project was awarded the highest score in the cohort.**

### What This Project Demonstrates

- Full understanding of symmetric, asymmetric, and hybrid encryption
- Practical PKI implementation: building a CA, issuing signed certificates, managing a trust chain
- Real-world OpenSSL command-line proficiency
- Understanding of why different algorithms are used together (not interchangeably)
- Data integrity verification using cryptographic hashing
- Professional documentation of a cryptographic workflow

---

## 🏛️ What Is PKI and Why It Matters

**PKI (Public Key Infrastructure)** is the system of cryptographic standards, policies, and technologies that enables secure digital communication across untrusted networks. Every time you see a padlock in your browser, you are trusting a PKI chain.

### The Trust Problem PKI Solves

Without PKI, asymmetric encryption has a fundamental weakness: **how do you know a public key actually belongs to the person who claims to own it?**

An attacker could generate their own RSA key pair, send you their public key while claiming to be your bank, and you would have no way to detect the deception. This is the classic Man-in-the-Middle (MitM) scenario.

PKI solves this with **certificates** — digital documents that bind a public key to an identity, signed by a trusted third party called a **Certificate Authority (CA)**. You trust the CA. The CA verified the identity. The CA signed the certificate. Therefore you can trust the certificate.

```
Without PKI:
  Alice ──── "Here is my public key" ──── Bob
  (Bob has no way to verify it is really Alice's key)

With PKI:
  Alice ──── "Here is my CA-signed certificate" ──── Bob
  (Bob verifies: CA signature is valid → key belongs to Alice → trust established)
```

### Real-World PKI Usage

| Technology | How PKI Is Used |
|------------|-----------------|
| HTTPS / TLS | Server certificates prove website identity to browsers |
| VPN | Client/server certificates authenticate VPN endpoints |
| S/MIME Email | Email certificates sign and encrypt messages |
| Code Signing | Certificates prove software was published by a known developer |
| SSH (key-based) | Public keys authenticate users without passwords |
| Smart Cards | Certificate on card authenticates the cardholder |

---

## 🧠 Cryptography Concepts Covered

| Concept | Algorithm Used | Purpose in This Project |
|---------|---------------|------------------------|
| Asymmetric encryption | RSA-2048 | Protect the AES session key |
| Symmetric encryption | AES-256-CBC | Encrypt the actual plaintext data |
| Hybrid encryption | RSA + AES | Industry-standard combination |
| Certificate Authority | X.509 v3 | Sign and vouch for public keys |
| Digital certificate | X.509 / PEM | Bind public key to identity |
| Certificate chain | Root CA → End-entity | Establish verifiable trust |
| Key derivation | PBKDF2 (via OpenSSL -pbkdf2) | Strengthen password-derived keys |
| Data integrity | SHA-256 | Verify file has not been altered |
| Certificate Signing Request | PKCS#10 | Formal request for CA signature |
| Cipher mode | CBC (Cipher Block Chaining) | Encrypt AES blocks securely |

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| **Operating System** | Kali Linux (primary machine) |
| **Username** | mr-robot |
| **Cryptography Tool** | OpenSSL (pre-installed on Kali) |
| **Shell** | bash |
| **Working Directory** | `~/encryption_project/` |
| **Key Storage** | Local filesystem (lab environment only) |
| **Certificate Type** | Self-signed Root CA + Issued End-Entity Certificate |

**Project Directory Structure:**
```
~/encryption_project/
│
├── ca/
│   ├── ca.key          ← Root CA private key (KEEP SECRET)
│   └── ca.crt          ← Root CA self-signed certificate (public trust anchor)
│
├── server/
│   ├── server.key      ← Entity's RSA-2048 private key
│   ├── server.csr      ← Certificate Signing Request (sent to CA)
│   └── server.crt      ← CA-signed certificate (contains public key)
│
├── data/
│   ├── plaintext.txt          ← Original message
│   ├── encrypted.enc          ← AES-256-CBC ciphertext
│   ├── aes_key.bin            ← Raw AES session key
│   ├── aes_key_encrypted.bin  ← AES key encrypted with RSA public key
│   └── decrypted.txt          ← Recovered plaintext after decryption
│
└── hashes/
    ├── plaintext.sha256        ← Hash of original
    └── decrypted.sha256        ← Hash of recovered file (must match)
```

---

## 🛠️ Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| **OpenSSL** | 3.x (Kali default) | All cryptographic operations: key generation, certificate management, encryption, decryption, hashing |
| **Bash Terminal** | — | Command execution, file management, workflow automation |
| **Kali Linux** | 2024.x | Operating system providing the OpenSSL environment |

---

## 🔬 Tool Breakdown

### OpenSSL — The Cryptographic Swiss Army Knife

OpenSSL is an open-source cryptographic library and command-line toolkit that implements the SSL/TLS protocols and a wide range of cryptographic functions. It is the most widely used cryptographic tool in the world — used in web servers (Apache, Nginx), cloud infrastructure, VPNs, and security tooling globally.

Understanding OpenSSL at the command-line level means understanding how real-world cryptography is implemented — not just theoretically described.

**OpenSSL Commands Used in This Project:**

| Command | Full Purpose |
|---------|-------------|
| `openssl genrsa` | Generate RSA private key of specified bit length |
| `openssl req` | Create a Certificate Signing Request (CSR) or self-sign a certificate |
| `openssl x509` | Process X.509 certificates — sign, view, verify, export |
| `openssl verify` | Verify a certificate's signature against a CA certificate |
| `openssl rand` | Generate cryptographically secure random bytes for key material |
| `openssl enc` | Symmetric encryption and decryption (AES, DES, etc.) |
| `openssl rsautl` / `openssl pkeyutl` | RSA public/private key operations (encrypt/decrypt) |
| `openssl dgst` | Compute cryptographic hash digests (SHA-256, MD5, etc.) |

**Key OpenSSL Flags Explained:**

| Flag | Context | What It Does |
|------|---------|-------------|
| `-newkey rsa:2048` | req | Generate a new 2048-bit RSA key alongside the request |
| `-keyout` | req/genrsa | Specify output file for the generated private key |
| `-out` | all | Specify output filename for the result |
| `-x509` | req | Output a self-signed certificate instead of a CSR |
| `-days 3650` | req/x509 | Set certificate validity to 3650 days (10 years) |
| `-CAcreateserial` | x509 | Create a serial number file for the CA |
| `-CA` | x509 | Specify the CA certificate for signing |
| `-CAkey` | x509 | Specify the CA private key for signing |
| `-aes-256-cbc` | enc | Use AES with 256-bit key in CBC mode |
| `-pbkdf2` | enc | Use PBKDF2 key derivation (strengthens password-derived keys) |
| `-in` / `-out` | enc | Input and output files for encryption operations |
| `-pubin` | pkeyutl | Indicates input is a public key (not a full certificate) |
| `-encrypt` / `-decrypt` | pkeyutl | RSA encryption / decryption operation |
| `-sha256` | dgst | Use SHA-256 algorithm for hashing |
| `-noout` | x509 | Do not output the certificate itself (only parsed info) |
| `-text` | x509 | Display full human-readable certificate content |
| `-subject` | x509 | Display only the certificate subject field |
| `-issuer` | x509 | Display who signed the certificate |

---

## 📐 Project Steps — Full Workflow

The project followed a nine-phase workflow covering the complete PKI-based hybrid encryption lifecycle.

```
Phase 1 — Build the Certificate Authority (Root CA)
           ↓
Phase 2 — Generate RSA-2048 Key Pair (Entity)
           ↓
Phase 3 — Create a Certificate Signing Request (CSR)
           ↓
Phase 4 — CA Signs the Certificate
           ↓
Phase 5 — Verify the Certificate Chain
           ↓
Phase 6 — AES-256-CBC Symmetric Encryption
           ↓
Phase 7 — Hybrid Encryption (RSA Wraps AES Session Key)
           ↓
Phase 8 — Decryption and Message Recovery
           ↓
Phase 9 — Integrity Verification with SHA-256 Hashing
```

---

### Phase 1 — Build the Certificate Authority (CA)

**Objective:** Create the root of trust. The CA is the organisation (or in this case, the student) that vouches for the authenticity of certificates it signs.

A real-world CA (like DigiCert or Let's Encrypt) is trusted because browser/OS vendors include their root certificates in their trust stores. In this project, we build our own root CA — functionally identical, just not pre-trusted by third-party software.

**Step 1a — Generate the CA's Private Key:**
```bash
openssl genrsa -out private.pem 2048
```
<img width="1366" height="739" alt="Screenshot From 2026-03-20 09-50-45" src="https://github.com/user-attachments/assets/0a152a20-d49c-4767-b826-249912dcaebf" />


This generates a 2048-bit RSA private key for the Certificate Authority. This key is what the CA uses to sign certificates. In a production environment, this key would be stored in an HSM (Hardware Security Module) and kept completely offline — a compromised CA key breaks the entire trust chain.

**Step 1b — Create the CA's Self-Signed Root Certificate:**
```bash
openssl req -new -x509 \
    -key ca/ca.key \
    -out ca/ca.crt \
    -days 3650 \
    -subj "/C=NG/ST=Lagos/L=Lagos/O=NewHorizonCA/OU=Security/CN=FrancisRootCA"
```
<img width="1366" height="768" alt="Screenshot From 2026-04-24 21-30-07" src="https://github.com/user-attachments/assets/d1b0ec18-575b-430b-b2f2-15689a675c7c" />

**Breaking down the `-subj` fields:**

| Field | Value | Meaning |
|-------|-------|---------|
| `/C=` | `NG` | Country — Nigeria (ISO 3166-1 alpha-2) |
| `/ST=` | `Lagos` | State or Province |
| `/L=` | `Lagos` | Locality (City) |
| `/O=` | `NewHorizonCA` | Organisation name |
| `/OU=` | `Security` | Organisational Unit |
| `/CN=` | `FrancisRootCA` | Common Name — identifies this certificate |

**Why `-x509` makes this self-signed:**
The `-x509` flag tells OpenSSL to skip the CSR process and directly produce a signed certificate. The CA signs its own certificate using its own private key — hence "self-signed root." All trust chains eventually end at a self-signed root CA.

**Result:**
- `ca/ca.key` — CA private key (secret, never shared)
- `ca/ca.crt` — CA root certificate (public, distributed to everyone who needs to verify certificates)

---

### Phase 2 — Generate RSA-2048 Key Pair

**Objective:** Create the cryptographic identity of the entity (the person/server whose data will be encrypted).

```bash
openssl genrsa -out server/server.key 2048
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/0528ce4c-d155-4341-a165-6be70f64d343" />

This generates a 2048-bit RSA private key for the end entity. RSA-2048 provides approximately 112 bits of security — considered secure against all known classical computing attacks as of 2024.

**Why 2048 bits?**
RSA security is based on the mathematical difficulty of factoring the product of two large prime numbers. 2048-bit keys cannot be factored by any known algorithm in a practical timeframe. RSA-4096 is stronger but 4x slower; RSA-1024 is broken and must not be used.

**The RSA key pair relationship:**
```
Private Key  ──── mathematically related ────  Public Key
     │                                              │
     │                                              │
 Keep secret                               Share freely
 Sign data                                 Verify signatures
 Decrypt data                              Encrypt data
     │                                              │
 Only YOU                              Anyone can use it
 can use it
```

**Extract the public key for distribution:**
```bash
openssl rsa -in server/server.key -pubout -out server/server_public.pem
```

---

### Phase 3 — Create a Certificate Signing Request (CSR)

**Objective:** Formally request that the CA sign a certificate binding our identity to our public key.

```bash
openssl req -new \
    -key server/server.key \
    -out server/server.csr \
    -subj "/C=NG/ST=Lagos/L=Lagos/O=FrancisEncryptionLab/OU=CryptoTeam/CN=francis.encryption.local"
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/b47f92e0-4272-4393-afa0-d0de2b4c76ee" />


**What a CSR contains:**
A CSR (Certificate Signing Request) is a PKCS#10 formatted message containing:
1. The applicant's public key (derived from `server.key`)
2. The requested identity information (Subject fields)
3. A digital signature using the applicant's private key (proving they control the private key)

**What a CSR does NOT contain:**
- The private key (never included — the private key stays secret)
- The CA's signature (not yet — that happens in Phase 4)
- A validity period (the CA decides that during signing)

**Inspect the CSR contents:**
```bash
openssl req -in server/server.csr -noout -text
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/75d56fdf-2847-40b4-9dd5-89f71095b5a9" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/a28a4d0d-af5b-43c4-9421-276dcf9af54c" />


This reveals the full CSR including the embedded public key and Subject fields — confirming the request was correctly formed before submitting to the CA.

---

### Phase 4 — CA Signs the Certificate

**Objective:** The CA verifies the CSR and uses its private key to sign a certificate — creating cryptographic proof that the CA vouches for this identity and public key.

```bash
openssl x509 -req \
    -in server/server.csr \
    -CA ca/ca.crt \
    -CAkey ca/ca.key \
    -CAcreateserial \
    -out server/server.crt \
    -days 365 \
    -sha256
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/38bac340-cf28-4b2f-ba52-808657fe4596" />

**What happens during signing:**
1. OpenSSL reads the CSR and verifies the embedded signature (confirming the requester controls the private key)
2. The CA applies its own digital signature to the certificate contents using `ca.key`
3. The signed certificate is written to `server/server.crt`
4. A serial number is generated and stored (via `-CAcreateserial`)

**Why `-sha256` matters:**
The CA's signature over the certificate is computed using SHA-256. This means any tampering with the certificate (changing the name, the public key, or the validity dates) would invalidate the SHA-256 hash — and therefore invalidate the CA's signature. This is how certificate integrity is enforced.

**View the signed certificate:**
```bash
openssl x509 -in server/server.crt -noout -text
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/b24222a4-fc73-4dd6-bcc7-547cdd10b7f4" />
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/2c6c26fd-2f01-45ff-b9f9-172e821f9db8" />

Key fields to inspect in the output:

```
Certificate:
  Data:
    Version: 3 (0x2)
    Serial Number: [unique number assigned by CA]
    Signature Algorithm: sha256WithRSAEncryption
    Issuer: C=NG, O=NewHorizonCA, CN=FrancisRootCA     ← Who signed it (our CA)
    Validity:
      Not Before: [issue date]
      Not After:  [expiry date — 365 days later]
    Subject: C=NG, O=FrancisEncryptionLab, CN=francis.encryption.local
    Subject Public Key Info:
      Public Key Algorithm: rsaEncryption
      RSA Public-Key: (2048 bit)                        ← Our public key embedded
    Signature Algorithm: sha256WithRSAEncryption        ← CA's signature
    [signature bytes]
```

**Result:** `server/server.crt` — a CA-signed X.509 certificate proving that the public key inside belongs to the identified entity.

---

### Phase 5 — Verify the Certificate Chain

**Objective:** Confirm that the certificate chain is valid — that `server.crt` was genuinely signed by `ca.crt`.

```bash
openssl verify -CAfile ca/ca.crt server/server.crt
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/a179ddd9-cac0-410d-9e4b-0f1aa5ca3295" />

**Expected result:**
```
server/server.crt: OK
```

This single line means:
- The CA's signature on `server.crt` is cryptographically valid
- The certificate has not been tampered with
- The certificate was signed by the key that corresponds to `ca.crt`
- The chain of trust is intact

**Check specific certificate fields:**
```bash
# Who issued the certificate?
openssl x509 -in server/server.crt -noout -issuer
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/49200d79-6bf0-411e-b87f-346bfee51d56" />

# Who is the certificate for?
openssl x509 -in server/server.crt -noout -subject
```
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/92feb2c8-4405-4c80-b4be-43c6a6d89aaf" />


# When does it expire?
openssl x509 -in server/server.crt -noout -dates
```
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/7db76eb2-80fb-4b08-ada6-afd8af066947" />


**Understanding the Trust Chain:**
```
```
Root CA Certificate (ca.crt)
  │
  │  [CA signs with ca.key → places signature in server.crt]
  ▼
Server Certificate (server.crt)
  │
  │  [Contains server's public key, CA signature, identity info]
  ▼
Anyone with ca.crt can verify server.crt is legitimate
```
```
This is functionally identical to how your browser verifies an HTTPS website certificate — except the browser already has thousands of root CA certificates pre-installed. Here we manually provided the CA certificate for verification.

---

### Phase 6 — AES-256-CBC Symmetric Encryption

**Objective:** Encrypt the actual plaintext data using AES-256 in CBC mode — the symmetric encryption layer of the hybrid system.

**Create the plaintext message:**
```bash
```
echo "This is the confidential message encrypted using hybrid PKI-based encryption.
Project by Okoro Francis Emmanuel — New Horizon Nigeria 2024.
This message is protected by AES-256-CBC encryption with an RSA-2048 wrapped key." \
> data/plaintext.txt
```
```

**Generate a cryptographically random AES session key:**
```bash
openssl rand -out data/aes_key.bin 32
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/2dbb8ec3-4b85-4a41-b6cd-fffcebe4663d" />


This produces 32 bytes (256 bits) of cryptographically secure random data from `/dev/urandom`. This is the AES-256 session key — different every time, never reused.

**Encrypt the plaintext with AES-256-CBC:**
```bash
openssl enc -aes-256-cbc \
    -in data/plaintext.txt \
    -out data/encrypted.enc \
    -pass file:data/aes_key.bin \
    -pbkdf2
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/085c3d4f-b5bc-4e8c-9521-06c885babafa" />

**Breaking down the flags:**

| Flag | What It Does |
|------|-------------|
| `enc -aes-256-cbc` | Use AES algorithm with 256-bit key in CBC (Cipher Block Chaining) mode |
| `-in data/plaintext.txt` | The file to encrypt |
| `-out data/encrypted.enc` | Output file for the ciphertext |
| `-pass file:data/aes_key.bin` | Use the random key file as the password/key source |
| `-pbkdf2` | Apply PBKDF2 key derivation — strengthens the key material |

**Why CBC (Cipher Block Chaining) mode?**

AES-CBC works by XORing each plaintext block with the previous ciphertext block before encrypting it. An Initialization Vector (IV) is used for the first block (since there is no previous ciphertext).

```
Block 1:  Plaintext₁ XOR IV        → AES Encrypt → Ciphertext₁
Block 2:  Plaintext₂ XOR Ciphertext₁ → AES Encrypt → Ciphertext₂
Block 3:  Plaintext₃ XOR Ciphertext₂ → AES Encrypt → Ciphertext₃
...
```

This means identical plaintext blocks produce different ciphertext — which is essential for security. AES in ECB (Electronic Code Book) mode does not chain blocks, meaning identical plaintext produces identical ciphertext — a serious weakness that makes patterns visible.

**Verify the ciphertext was produced:**
```bash
ls -lh data/encrypted.enc
file data/encrypted.enc
# Output: data/encrypted.enc: data (binary ciphertext — unreadable)

xxd data/encrypted.enc | head -5
# Shows hex dump confirming binary encrypted content
```

---

### Phase 7 — Hybrid Encryption (RSA Wraps the AES Key)

**Objective:** Encrypt the AES session key using the RSA public key from the CA-signed certificate. This is the hybrid encryption model — combining the speed of AES with the key exchange security of RSA.

**Extract the public key from the signed certificate:**
```bash
openssl x509 -in server/server.crt -pubkey -noout > server/server_pubkey.pem
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/52a5d0ec-688d-47dc-9b56-d072bfece180" />

**Encrypt the AES session key with the RSA public key:**
```bash
openssl pkeyutl -encrypt \
    -pubin \
    -inkey server/server_pubkey.pem \
    -in data/aes_key.bin \
    -out data/aes_key_encrypted.bin
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/fd60c2da-85d6-4cb7-aab5-1d19de66cb71" />

**Breaking down the flags:**

| Flag | What It Does |
|------|-------------|
| `pkeyutl -encrypt` | Perform RSA public key encryption |
| `-pubin` | The input key is a public key (not a private key or certificate) |
| `-inkey server_pubkey.pem` | Use this RSA public key for encryption |
| `-in data/aes_key.bin` | The data to encrypt (our 32-byte AES key) |
| `-out data/aes_key_encrypted.bin` | Output: AES key encrypted with RSA |

**Why this architecture is used everywhere:**

```
The Hybrid Encryption Model:

  PROBLEM 1: AES is fast but requires a shared secret key.
             How do we securely share the AES key?

  PROBLEM 2: RSA can securely share keys but is too slow
             to encrypt large amounts of data.

  SOLUTION: Use RSA to encrypt the AES key.
            Use AES to encrypt the data.

  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  AES Session Key (32 bytes)                     │
  │         │                                       │
  │         ▼                                       │
  │  RSA Encrypt with Public Key                    │
  │         │                                       │
  │         ▼                                       │
  │  Encrypted AES Key (256 bytes RSA output)       │
  │         │                                       │
  │  Plaintext Data                                 │
  │         │                                       │
  │         ▼                                       │
  │  AES-256-CBC Encrypt with AES Session Key       │
  │         │                                       │
  │         ▼                                       │
  │  Ciphertext                                     │
  │                                                 │
  │  → Send: [Encrypted AES Key] + [Ciphertext]     │
  │                                                 │
  └─────────────────────────────────────────────────┘

  Recipient uses their RSA private key to recover
  the AES key, then uses the AES key to decrypt
  the ciphertext.
```

**The two files that would be transmitted to a recipient:**
1. `data/aes_key_encrypted.bin` — the AES session key, RSA-encrypted with the recipient's public key
2. `data/encrypted.enc` — the actual ciphertext (AES-encrypted message)

Only the holder of `server/server.key` (the RSA private key) can decrypt `aes_key_encrypted.bin` and recover the AES key needed to decrypt the message.

---

### Phase 8 — Decryption and Message Recovery

**Objective:** Use the RSA private key to recover the AES session key, then use the AES key to decrypt the ciphertext back to plaintext.

**Step 8a — Decrypt the AES key using the RSA private key:**
```bash
openssl pkeyutl -decrypt \
    -inkey server/server.key \
    -in data/aes_key_encrypted.bin \
    -out data/aes_key_recovered.bin
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/592b73e8-cbc8-4a53-9cdb-7bef625d48a6" />

Only the holder of `server.key` can perform this operation. The RSA private key decrypts what the RSA public key encrypted. This is the foundation of asymmetric cryptography.

**Step 8b — Decrypt the ciphertext using the recovered AES key:**
```bash
openssl enc -aes-256-cbc -d \
    -in data/encrypted.enc \
    -out data/decrypted.txt \
    -pass file:data/aes_key_recovered.bin \
    -pbkdf2
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/473f266d-6efd-49fa-8ef6-6fe09fc8faee" />

The `-d` flag instructs OpenSSL to perform decryption instead of encryption. The same AES key and the same CBC IV (stored in the encrypted file header by OpenSSL) are used to reverse the encryption process block by block.

**Step 8c — Confirm the decrypted message:**
```bash
cat data/decrypted.txt
```

**Expected output:**
```
My Message
```

The original plaintext is fully recovered.

---

### Phase 9 — Integrity Verification with SHA-256 Hashing

**Objective:** Prove that the decrypted file is byte-for-byte identical to the original plaintext — no corruption, no modification.

**Generate SHA-256 hash of the original file:**
```bash
openssl dgst -sha256 -out hashes/plaintext.sha256 data/plaintext.txt
cat hashes/plaintext.sha256
# Output: SHA2-256(data/plaintext.txt)= [64-character hex hash]
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/51cdf8bc-f31e-4c68-bfc3-628054fb6003" />

**Generate SHA-256 hash of the decrypted file:**
```bash
openssl dgst -sha256 -out hashes/decrypted.sha256 data/decrypted.txt
cat hashes/decrypted.sha256
# Output: SHA2-256(data/decrypted.txt)= [64-character hex hash]
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/9fde3c1e-7c99-4320-b4fc-64372ab9bf4c" />

**Compare the two hashes:**
```bash
diff hashes/plaintext.sha256 hashes/decrypted.sha256
# No output = files are identical
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/24ca21c7-2ae9-4567-9103-fd5fd65927a4" />


If both hashes are identical, it is **cryptographically proven** that:
- The decryption process succeeded completely
- No data was corrupted at any stage
- The recovered plaintext is exactly the original

**Why SHA-256 is used for integrity verification:**
SHA-256 is a one-way cryptographic hash function that produces a fixed 256-bit (64 hex character) fingerprint of any input. It is computationally infeasible to:
- Find two different files that produce the same hash (collision resistance)
- Modify a file without changing its hash (tamper evidence)
- Reverse a hash to recover the original file (pre-image resistance)

```
plaintext.txt ──── SHA-256 ────► a3f1c9d2...b7e4 (64 hex chars)
decrypted.txt ──── SHA-256 ────► a3f1c9d2...b7e4 (64 hex chars)
                                        │
                                   Identical ✓
                         → Decryption successful
                         → Data integrity confirmed
```

---

## 🔑 Key Concepts Demonstrated

### 1. The Three Types of Encryption — Used Together

| Type | Algorithm | Key | Speed | Use in This Project |
|------|-----------|-----|-------|---------------------|
| **Symmetric** | AES-256-CBC | Single shared key | Very fast | Encrypts the actual plaintext data |
| **Asymmetric** | RSA-2048 | Public/Private key pair | Slow | Encrypts the AES key |
| **Hybrid** | AES + RSA | Both | Optimal | The complete system |

### 2. Why RSA Cannot Encrypt Large Data

RSA encryption can only encrypt data smaller than the key size minus padding overhead. With RSA-2048:
- Maximum plaintext: approximately 245 bytes (with PKCS#1 v1.5 padding)
- A typical email is 50,000+ bytes — far too large for direct RSA encryption

This is why RSA is only used to encrypt the small AES key (32 bytes), not the actual message.

### 3. The CA Signature Chain Explained

```
FrancisRootCA (ca.crt)
    │
    │  CA private key signs the certificate
    ▼
server.crt  [Contains: server's public key + CA signature]
    │
    │  Anyone with ca.crt can run: openssl verify -CAfile ca.crt server.crt
    ▼
Result: server.crt: OK  ← The CA vouches for this public key
```

### 4. What "Secure" Actually Means in This Context

| Property | How Achieved | Algorithm |
|----------|-------------|-----------|
| **Confidentiality** | Data encrypted — unreadable without the key | AES-256-CBC |
| **Key Security** | AES key protected — only private key holder can recover it | RSA-2048 |
| **Identity Trust** | Public key is certified by a trusted CA | X.509 Certificate |
| **Integrity** | Hash proves data was not altered in transit or storage | SHA-256 |
| **Non-repudiation** | CA signature proves who the public key belongs to | RSA digital signature |

---

## ⚖️ Why Hybrid Encryption Is the Industry Standard

Every major security protocol in use today — TLS 1.3, Signal Protocol, PGP, S/MIME — uses hybrid encryption for the same reason this project does.

**TLS 1.3 Handshake (What HTTPS Uses):**
```
Client                                    Server
  │                                          │
  │─── ClientHello (supported algorithms) ──►│
  │◄── ServerHello + Certificate ───────────│
  │    (Certificate contains server's       │
  │     CA-signed RSA/ECC public key)       │
  │                                          │
  │  Client verifies certificate chain      │
  │  (same as: openssl verify -CAfile ...)  │
  │                                          │
  │─── Key Exchange (ECDHE) ───────────────►│
  │    (Ephemeral DH establishes session    │
  │     key — equivalent to our AES key)   │
  │                                          │
  │◄══════ All traffic now AES encrypted ══►│
  │        (using the exchanged session key) │
```

The certificate is the CA-signed proof of the server's identity — exactly what was built in Phase 1 through Phase 5 of this project.

---

## 📚 Lessons Learned

**1. Encryption Without Trust Is Incomplete**
Generating an RSA key pair and encrypting data is only part of the security model. Without a CA to certify the public key, you have no way to guarantee the key belongs to who you think it does. The CA adds trust to the cryptographic operation.

**2. Different Algorithms Have Different Jobs**
AES and RSA are not interchangeable. AES is fast and efficient for bulk data but requires a shared key. RSA solves the key distribution problem but is impractical for large data. Hybrid encryption uses each algorithm for what it does best.

**3. The Private Key Is Everything**
The entire security of the system rests on the secrecy of the private key (`server.key`). If an attacker obtains the private key, they can decrypt the AES session key, and from that, decrypt all ciphertext encrypted for that key. In production, private keys are stored in HSMs, protected with strong passphrases, and never transmitted.

**4. Hashing and Encryption Are Different**
Hashing is one-way — you cannot recover the original data from a hash. Encryption is two-way — you can recover the original with the correct key. SHA-256 provides integrity verification (detect if data changed) not confidentiality. Both are needed in a complete security system.

**5. CBC Mode Requires an IV — And the IV Matters**
AES-CBC requires a random Initialization Vector for each encryption operation. OpenSSL handles this automatically, but understanding why it matters is important: reusing an IV with the same key can reveal information about the plaintext. Modern implementations use GCM mode which provides authenticated encryption (integrity built in).

**6. Certificate Expiry Is a Deliberate Security Feature**
The 365-day validity period on `server.crt` is not arbitrary — short certificate lifetimes limit the window of exposure if a certificate is compromised. If an attacker steals a private key, they can only impersonate the identity until the certificate expires. This is why certificates should be rotated regularly and why HTTPS certificate expiry triggers browser warnings.

**7. OpenSSL Command-Line Mastery Transfers Directly to Production Security**
Every step in this project is what real infrastructure engineers do — issuing certificates for web servers, VPNs, email gateways, and microservices. The OpenSSL commands executed here are used daily by security engineers at banks, cloud providers, and government agencies.

---

## 🚀 Next Steps

### Planned Extensions of This Project

**Phase 2 — Digital Signatures**
Use the RSA private key to sign a document and verify it with the public key from the certificate — demonstrating non-repudiation.
```bash
# Sign
openssl dgst -sha256 -sign server/server.key \
    -out data/message.sig data/plaintext.txt

# Verify
openssl dgst -sha256 -verify server/server_pubkey.pem \
    -signature data/message.sig data/plaintext.txt
```
<img width="1366" height="739" alt="image" src="https://github.com/user-attachments/assets/0f7c8699-afeb-489f-8811-4948575ee9c4" />

**Phase 3 — Certificate Chain with Intermediate CA**
Build a three-tier PKI hierarchy:
```
Root CA (offline)
    └── Intermediate CA (online)
            └── End-Entity Certificate
```
This mirrors how public CAs like DigiCert and Let's Encrypt are structured in production.

**Phase 4 — S/MIME Email Encryption**
Use the generated certificate to sign and encrypt emails using S/MIME — demonstrating PKI applied to real email communication.

**Phase 5 — TLS Server Certificate**
Use OpenSSL to configure a local HTTPS server using the CA-signed certificate, demonstrating the exact TLS deployment process used in web servers.

**Phase 6 — Migrate to ECDSA and AES-GCM**
- Replace RSA-2048 with ECDSA P-256 (smaller key, equivalent security, faster operations)
- Replace AES-CBC with AES-256-GCM (authenticated encryption — provides integrity without a separate hash step)
- This mirrors the algorithms mandated by TLS 1.3

---

### Certification Roadmap

```
Current Status                      →  Certification Path
────────────────────────────────────────────────────────────────
✅  CompTIA Network+  (Completed)    →  CompTIA Security+ SY0-701
✅  Text Encryption Project          →  CompTIA PenTest+
🔄  Security+ (Exam Pending)         →  eJPT (eLearnSecurity)
🔄  TryHackMe Jr PenTester Path      →  OSCP (OffSec)
```

---

## 📖 References

| Resource | URL |
|----------|-----|
| OpenSSL Official Documentation | https://docs.openssl.org/ |
| OpenSSL `req` Command Reference | https://www.openssl.org/docs/man3.0/man1/openssl-req.html |
| OpenSSL `enc` Command Reference | https://www.openssl.org/docs/man3.0/man1/openssl-enc.html |
| X.509 Certificate Standard (RFC 5280) | https://datatracker.ietf.org/doc/html/rfc5280 |
| NIST AES Standard (FIPS 197) | https://csrc.nist.gov/publications/detail/fips/197/final |
| RSA Cryptography Standard (PKCS #1) | https://datatracker.ietf.org/doc/html/rfc8017 |
| NIST Key Management Guidelines (SP 800-57) | https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final |
| PBKDF2 Key Derivation (RFC 2898) | https://datatracker.ietf.org/doc/html/rfc2898 |
| Certificate Signing Request Format (PKCS #10) | https://datatracker.ietf.org/doc/html/rfc2986 |
| TLS 1.3 Protocol (RFC 8446) | https://datatracker.ietf.org/doc/html/rfc8446 |
| SHA-256 Hash Standard (FIPS 180-4) | https://csrc.nist.gov/publications/detail/fips/180/4/final |
| OWASP Cryptographic Storage Cheatsheet | https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html |

---

## 👤 Project Author

| Name | Role | Institution |
|------|------|-------------|
| **Okoro Francis Emmanuel** | Project Lead — Full implementation, documentation | New Horizon Nigeria |

---

## 📁 Repository Structure

```
text-encryption-project/
│
├── README.md                          ← This file (full project documentation)
│
├── ca/
│   ├── ca.key                         ← Root CA private key (DO NOT SHARE)
│   └── ca.crt                         ← Root CA certificate (public trust anchor)
│
├── server/
│   ├── server.key                     ← Entity RSA-2048 private key
│   ├── server.csr                     ← Certificate Signing Request
│   ├── server.crt                     ← CA-signed X.509 certificate
│   └── server_pubkey.pem              ← Extracted public key
│
├── data/
│   ├── plaintext.txt                  ← Original message
│   ├── aes_key.bin                    ← Random AES session key (binary)
│   ├── encrypted.enc                  ← AES-256-CBC ciphertext
│   ├── aes_key_encrypted.bin          ← AES key encrypted with RSA public key
│   ├── aes_key_recovered.bin          ← AES key after RSA decryption
│   └── decrypted.txt                  ← Recovered plaintext
│
├── hashes/
│   ├── plaintext.sha256               ← SHA-256 of original file
│   └── decrypted.sha256               ← SHA-256 of decrypted file (must match)
│
└── screenshots/
    ├── phase1_ca_key_generation.png
    ├── phase1_ca_certificate.png
    ├── phase2_rsa_keygen.png
    ├── phase3_csr_creation.png
    ├── phase4_ca_signing.png
    ├── phase5_chain_verification.png
    ├── phase6_aes_encryption.png
    ├── phase7_rsa_key_wrapping.png
    ├── phase8_decryption.png
    └── phase9_hash_verification.png
```

---

<div align="center">

**⭐ Star this repository if you found it useful**

*Phase 2 (Digital Signatures) and Phase 3 (Intermediate CA) coming soon*

---

![OpenSSL](https://img.shields.io/badge/OpenSSL-3.x-red?style=flat-square)
![AES-256](https://img.shields.io/badge/AES-256--CBC-blue?style=flat-square)
![RSA-2048](https://img.shields.io/badge/RSA-2048%20bit-purple?style=flat-square)
![PKI](https://img.shields.io/badge/PKI-X.509%20Certificates-orange?style=flat-square)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![New Horizon Nigeria](https://img.shields.io/badge/New%20Horizon-Nigeria-green?style=flat-square)

</div>
HEREDOC

echo "Done — $(wc -l < /mnt/user-data/outputs/ENCRYPTION_README.md) lines written"
