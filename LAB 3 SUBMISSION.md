# Lab Report: Data Protection - Encryption & Key Management

**Course:** IKB42603 Cloud Computing Security Essentials
**Student:** TENGKU EIKMAL AQIL BIN TENGKU SIFZIZUL (52215124448)
**Date:** August 24, 2026

---

## Session A (Week 5) - Encryption Fundamentals

### Task 1 - Symmetric Encryption (Data at Rest)
**Objective:** Understand how to use a single shared key to encrypt and decrypt bulk data quickly[cite: 1]. 

**Steps Taken:**
1. **Create the sensitive record:** Generated a sample file containing confidential patient data[cite: 1].
   `echo 'Patient: Ahmad, Diagnosis: confidential' > record.txt`
2. **Encrypt the file:** Used the OpenSSL toolkit to encrypt the file with the AES-256 algorithm and a custom passphrase acting as the symmetric key[cite: 1].
   `openssl enc -aes-256-cbc -pbkdf2 -salt -in record.txt -out record.enc`
3. **Decrypt and Verify:** Decrypted the file using the exact same passphrase and used the `diff` command to mathematically prove the decrypted file perfectly matches the original plaintext[cite: 1].
   `openssl enc -d -aes-256-cbc -pbkdf2 -in record.enc -out record.dec.txt`
   `diff record.txt record.dec.txt && echo 'MATCH: decryption successful'`

<img width="605" height="358" alt="LAB 3 TASK 1" src="https://github.com/user-attachments/assets/5e47a64a-99a4-425e-83ac-30de63f96673" />


---

### Task 2 - Asymmetric Encryption & Digital Signatures
**Objective:** Understand how mathematically linked key pairs solve the key-distribution problem and provide provable data integrity[cite: 1].

**Steps Taken:**
1. **Generate Keys:** Created a 2048-bit RSA private key and extracted the corresponding public key[cite: 1]. 
   `openssl genrsa -out private.pem 2048`
   `openssl rsa -in private.pem -pubout -out public.pem`
2. **Encrypt and Decrypt:** Encrypted the data using the public key (which anyone can use) and decrypted it using the private key (which must be kept secret)[cite: 1].
3. **Sign and Verify:** Reversed the roles to create a digital signature. Signed the file with the private key to prove origin, and verified the signature using the public key[cite: 1].
   `openssl dgst -sha256 -sign private.pem -out record.sig record.txt`
   `openssl dgst -sha256 -verify public.pem -signature record.sig record.txt`

<img width="769" height="313" alt="LAB 3 TASK 2" src="https://github.com/user-attachments/assets/0f436b6d-4d55-4b1e-ad97-a81a2cc64d49" />


---

### Task 3 - Encryption in Transit (TLS)
**Objective:** Protect data while it travels over a network by establishing an encrypted TLS channel to prevent eavesdropping[cite: 1].

**Steps Taken:**
1. **Generate Certificate:** Created a self-signed X.509 certificate to enable HTTPS[cite: 1].
2. **Serve the File Securely:** Spun up an Nginx Docker container configured to listen on port 443 and serve our confidential record using the generated certificates[cite: 1].
3. **Secure Retrieval:** Used `curl -k` to connect to the server securely over TLS and download the file[cite: 1].

<img width="1910" height="450" alt="LAB 3 TASK 3 P1" src="https://github.com/user-attachments/assets/91a9f2aa-63db-496c-b562-a4b4261402e3" />
<img width="381" height="82" alt="LAB 3 TASK 3 P2" src="https://github.com/user-attachments/assets/28a9b4e3-bf47-49e1-8eed-52db1279d879" />
<img width="220" height="72" alt="LAB 3 TASK 3 P3" src="https://github.com/user-attachments/assets/58675656-8813-4c39-901d-3cfec83e3b62" />


---

## Session B (Week 6) - Key Management, Envelope Encryption & Erasure

### Task 4 - Create and Use a KMS Master Key
**Objective:** Set up a centralized Customer Master Key (CMK) inside a Key Management Service (LocalStack) to handle encryption at scale[cite: 1].

**Steps Taken:**
1. **Create the Key:** Provisioned a new master key for "Tenant A" and stored its `KeyId`[cite: 1].
   `aws $EP kms create-key --description 'CCSE tenant-A master key'`
2. **Test Encryption:** Directly encrypted a small secret via the KMS API to ensure the key was active and functional[cite: 1].

<img width="965" height="535" alt="LAB 3 TASK 4" src="https://github.com/user-attachments/assets/15c856df-a323-4519-9aa8-ef143158fa16" />


---

### Task 5 - Envelope Encryption
**Objective:** Protect large amounts of data without overwhelming the KMS by using a data key to encrypt locally, and a master key to protect the data key[cite: 1].

**Steps Taken:**
1. **Generate Data Key:** Requested a new AES-256 data key from KMS, returning both a plaintext version and a wrapped (encrypted) version[cite: 1].
   `aws $EP kms generate-data-key --key-id $KEY_A --key-spec AES_256 ...`
2. **Encrypt Locally:** Used the plaintext data key to encrypt the bulk record locally via OpenSSL[cite: 1].
3. **Secure the Key:** Destroyed the plaintext data key from the local disk, leaving only the KMS-wrapped data key safely stored alongside the encrypted file[cite: 1].

<img width="1649" height="356" alt="LAB 3 TASK 5" src="https://github.com/user-attachments/assets/eb5b4c89-dc47-40fa-9493-a12253413136" />


---

### Task 6 - Per-Tenant Keys & Cryptographic Erasure
**Objective:** Demonstrate how deleting a single master key instantly and permanently renders all associated cloud data unrecoverable[cite: 1].

**Steps Taken:**
1. **Simulate Key Destruction:** Scheduled Tenant A's master key for deletion and disabled it immediately[cite: 1].
   `aws $EP kms schedule-key-deletion --key-id $KEY_A --pending-window-in-days 7`
   `aws $EP kms disable-key --key-id $KEY_A`
2. **Verify Erasure:** Attempted to use the KMS to unwrap the encrypted data key. The request failed with an `InvalidStateException` / `NotFoundException`, proving the data is mathematically destroyed[cite: 1].

<img width="1570" height="715" alt="LAB 3 TASK 6" src="https://github.com/user-attachments/assets/c3288173-d389-4dca-9f72-53853c7ddd81" />


---

### Task 7 - Integrity & Tamper-Evidence
**Objective:** Use cryptographic hashing to detect unauthorized changes to data and build a tamper-evident audit log[cite: 1].

**Steps Taken:**
1. **Detect Tampering:** Calculated the SHA-256 hash of the original file, added a single character to a copy, and compared the new hash to prove that even the smallest change completely alters the fingerprint[cite: 1].
2. **Build a Hash Chain:** Created a simulated log where each entry includes the hash of the previous entry, demonstrating how chained hashes expose unauthorized historical alterations[cite: 1].

<img width="1229" height="297" alt="LAB 3 TASK 7" src="https://github.com/user-attachments/assets/3dced3f7-e37f-46d4-bfc8-255c05046cc8" />


---

### Cleanup & Teardown
**Objective:** Maintain a secure and efficient environment by tearing down all temporary infrastructure and destroying temporary keys[cite: 1].

<img width="1295" height="95" alt="LAB 3 CLEAN UP" src="https://github.com/user-attachments/assets/649f5c4b-28d3-44a8-8def-abcbc01788e5" />


---

## Deliverables & Assessment: Short-Answer Questions

**Q1. Compare symmetric and asymmetric encryption: speed, key distribution, and typical use.**
Symmetric encryption is very fast and uses one shared key for encryption and decryption, making it the standard for encrypting bulk data at rest[cite: 1]. Its primary challenge is securely distributing that shared key. Asymmetric encryption uses a mathematically linked key pair (public to encrypt, private to decrypt), which solves the key distribution problem[cite: 1]. Because it is slower, it is typically used for digital signatures, identity verification, and securing communication channels (like TLS) to safely exchange symmetric keys[cite: 1].

**Q2. Why is key management described as the weakest link, not the algorithm?**
Modern cryptographic algorithms (like AES) are mathematically robust and practically impossible to break via brute force[cite: 1]. Attackers bypass the complex math entirely by targeting poorly protected keys. If a key is hardcoded, leaked, or left exposed on a disk (like a plaintext data key), the encryption is completely compromised regardless of the algorithm's strength[cite: 1]. 

**Q3. Explain envelope encryption and why only the master key needs hardware-grade protection.**
Envelope encryption is a tiered approach where a local "data key" is used to quickly encrypt bulk data, and a "master key" (managed by a KMS) is used to encrypt or "wrap" that data key[cite: 1]. Because the encrypted data key is useless without the master key, it can be safely stored alongside the data payload[cite: 1]. This ensures that only the single, central master key requires strict, hardware-grade protection within a Hardware Security Module (HSM)[cite: 1].

**Q4. How does cryptographic erasure achieve provable deletion where overwriting cannot (in the cloud)?**
In cloud environments, data is automatically replicated across multiple physical drives for high availability, making physical drive wiping or localized overwriting practically impossible to guarantee[cite: 1]. Cryptographic erasure solves this by encrypting the data and strictly controlling the master key. To provably delete the data, you simply destroy the master key[cite: 1]. Without the key, all replicated copies of the encrypted data instantly become permanent, unrecoverable noise[cite: 1].

**Q5. How does a hash chain make a log tamper-evident?**
A hash chain links log entries cryptographically by including the hash of the previous log entry inside the calculation of the current entry's hash[cite: 1]. If an attacker alters a historical log entry, its hash changes. This triggers a cascading failure that immediately invalidates the hash of every subsequent entry in the chain, exposing the tampering to auditors[cite: 1].
