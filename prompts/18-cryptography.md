# Cryptographic Failures

**Domain:** Weak algorithms, key management, PRNG misuse, nonce and IV reuse, hashing  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit how the application applies cryptography for encryption, hashing, key generation, and certificate trust decisions.  This prompt covers OWASP A02 (Cryptographic Failures) and focuses on application-level cryptographic operations, not just transport settings.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the application's cryptographic design and implementation** across the entire codebase.

Specifically investigate:

1. **Weak or obsolete algorithms**: Check [CRYPTO_CLASSES] and all security-relevant crypto usage:
   - Look for MD5 or SHA1 used for password hashing, signatures, token integrity, or any other security purpose.
   - Look for DES, 3DES, RC4, Rijndael, or RSA key sizes that are too small for modern security expectations.
   - Find hardcoded algorithm names and determine whether a weak algorithm can be selected in production.
   - Distinguish between non-security checksum usage and security-sensitive usage.

2. **Key management**: Review how encryption and signing keys are created, loaded, stored, rotated, and destroyed:
   - Are keys hardcoded in source code, config files, scripts, or tests?
   - Are keys stored alongside encrypted data in the same file, blob, or record?
   - Is key derivation performed without a salt, with a predictable salt, or with insufficient work factor?
   - Are symmetric and asymmetric key lengths appropriate?
   - Is there any documented and implemented key rotation mechanism?

3. **Random number generation**: Check all code that generates tokens, reset links, session secrets, nonces, IVs, API keys, or security-sensitive identifiers:
   - Is `System.Random`, `Math.random`, `Random()`, `random.randint`, or equivalent used for security-sensitive values?
   - Are cryptographic PRNGs used instead, such as `RandomNumberGenerator`, `crypto.randomBytes`, or Python `secrets`?
   - Could predictable random values enable guessing, replay, or token forgery?

4. **Nonce and IV handling**: Review every encryption mode that requires an IV or nonce:
   - Are IVs or nonces static, reused, zero-filled, derived from predictable values, or shared across operations?
   - For AES-CBC, is each IV unique and random per encryption operation?
   - For AES-GCM or similar AEAD modes, is nonce reuse prevented across encryptions with the same key?
   - Are IVs stored or transmitted safely without being mistaken for secrets?

5. **Password hashing**: Check [AUTH_CLASSES] and all password storage or verification paths:
   - Are passwords hashed with plain SHA256, SHA1, MD5, or another fast general-purpose hash?
   - Is salting missing, reused, or predictable?
   - Are iteration counts or work factors too low?
   - Are bcrypt, scrypt, Argon2, or PBKDF2 used correctly for password hashing?
   - Is password verification done with a constant-time comparison where relevant?

6. **Homegrown crypto**: Look for custom encryption or encoding schemes:
   - XOR "encryption", reversible string mangling, or custom cipher implementations.
   - Base64, hex, compression, or serialization treated as if it were encryption.
   - Custom wrappers around crypto libraries that weaken security defaults.

7. **Certificate validation**: Check all certificate validation callbacks and trust overrides:
   - Does `ServerCertificateCustomValidationCallback`, `SSL_CTX_set_verify`, or equivalent logic return `true` unconditionally?
   - Are certificate errors selectively and safely handled, or broadly ignored?
   - Could a development-only bypass be reachable in production?

Search patterns:
- Files: `**/*.[EXTENSIONS]`
- Algorithms and primitives: `MD5, SHA1, SHA256, AES, DES, RSA, TripleDES, Rijndael, HMAC, PBKDF2, Rfc2898DeriveBytes, BCrypt, Argon2`
- Randomness: `RNGCryptoServiceProvider, RandomNumberGenerator, System.Random, Math.random, Random(), random.randint, GetBytes`
- Encryption APIs: `Encrypt, Decrypt, CreateEncryptor, CreateDecryptor, CipherMode, PaddingMode, KeySize, BlockSize, IV, Nonce, Salt`
- Certificate handling: `ServerCertificateCustomValidationCallback, X509Certificate, X509Certificate2, SSL_CTX_set_verify`

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info).  For each finding, explain the cryptographic weakness, the likely attack path, and the recommended remediation.  Do NOT include actual credential values, API keys, tokens, secrets, private keys, or password hashes in your output.  Use `[REDACTED]` placeholders instead.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust, and do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyWebApp, PaymentGateway, FileSyncService
`[REPO_PATH]` | Full path to the repository root
`[CRYPTO_CLASSES]` | `EncryptionService.cs`, `CryptoUtils.ts`, `crypto_helpers.py`, `TokenSigner.java`
`[AUTH_CLASSES]` | `PasswordHasher.cs`, `UserAuthService.ts`, `auth/passwords.py`, `LoginService.java`
`[EXTENSIONS]` | `cs,json,config`, `ts,js,json`, `py,yaml,json`, `java,properties,xml`

## What Good Looks Like

- AES-256 with a unique random IV per encryption operation
- Cryptographic PRNGs for all security-sensitive random values
- Passwords hashed with bcrypt, Argon2, scrypt, or PBKDF2 using appropriate work factors
- No hardcoded keys; keys stored in secure storage such as Key Vault, DPAPI, or an HSM
- No homegrown crypto; well-known and well-reviewed libraries only
- Certificate validation callbacks perform real validation and do not unconditionally allow any certificate
- Key rotation is documented and implemented
- Weak and obsolete algorithms are absent from security-sensitive code paths

## Relationship to Other Prompts

Prompt | Relationship
-------|-------------
01 - Credentials | Prompt 01 checks credential storage.  This prompt checks how cryptography is applied to protect secrets, passwords, and encrypted data.
09 - TLS Configuration | Prompt 09 checks TLS configuration and transport security behavior.  This prompt checks application-level cryptographic operations such as hashing, encryption, PRNG usage, nonce handling, and key management.
