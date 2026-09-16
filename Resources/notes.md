# 📒 Love at First Breach — Technical Notes

> Quick reference notes for the **Love at First Breach (When Hearts Collide)** TryHackMe Web Security CTF.

These notes summarize the exploitation workflow, commands, cryptographic concepts, security findings, and remediation techniques used during the challenge. They are intended as a lightweight companion to the full technical documentation.

---

## 📌 Challenge Summary

| Property | Value |
|----------|-------|
| **Platform** | TryHackMe |
| **Room** | Love at First Breach *(When Hearts Collide)* |
| **Category** | Web Security / Cryptography |
| **Difficulty** | Easy |
| **Primary Vulnerability** | MD5 Hash Collision |
| **Attack Type** | File Upload Verification Bypass |
| **Operating System** | Kali Linux |
| **Main Tool** | `fastcoll` |

---

# 🎯 Objective

Exploit an image upload application that identifies files using **MD5 hashes** instead of secure content verification.

The goal is to upload a **different image** that produces the **same MD5 digest** as a trusted dog image stored by the application.

> **Flag intentionally omitted** in this notes file.

---

# ⚙️ Attack Workflow

```text
Reconnaissance
      │
      ▼
Discover Static Image
      │
      ▼
Download Reference Image
      │
      ▼
Calculate MD5 Digest
      │
      ▼
Generate Collision Files
      │
      ▼
Verify Matching MD5 Hashes
      │
      ▼
Upload Collision Image
      │
      ▼
Application Accepts Upload
      │
      ▼
Challenge Completed
```

---

# 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Firefox / Browser** | Interact with Matchmaker application. |
| **wget** | Download exposed reference image. |
| **md5sum** | Calculate MD5 hashes. |
| **fastcoll** | Generate MD5 collision files. |
| **Kali Linux** | Penetration testing environment. |

---

# 📂 Important Commands

## Download Reference Image

```bash
wget http://TARGET_IP/static/uploads/<image-id>.jpg -O dog.jpg
```

Verify the download.

```bash
ls -la dog.jpg
```

---

## Calculate MD5 Digest

```bash
md5sum dog.jpg
```

Purpose:

- Compute the MD5 hash used by the application.
- Identify the digest that will be targeted during collision generation.

---

## Install fastcoll

Update packages.

```bash
sudo apt update
```

Install collision generation utility.

```bash
sudo apt install fastcoll -y
```

Verify installation.

```bash
fastcoll -h
```

---

## Generate Collision Files

```bash
fastcoll --prefixfile dog.jpg \
-o collision1.jpg collision2.jpg
```

Purpose:

- Create alternate files sharing the same MD5 digest.

---

## Verify Collision

```bash
md5sum dog.jpg collision1.jpg collision2.jpg
```

Expected result:

- Collision files produce identical MD5 hashes required for exploitation.

---

# 🔍 Key Reconnaissance Findings

| Finding | Security Observation |
|---------|----------------------|
| Public image upload form | Primary attack surface. |
| Static upload directory | Publicly accessible application assets. |
| Dog reference image | Trusted application resource. |
| No authentication | Anyone can interact with upload feature. |

---

# 🔐 MD5 Notes

## MD5 Overview

- Produces a **128-bit hexadecimal digest**.
- Deterministic hashing algorithm.
- Extremely fast.
- Historically used for integrity verification.
- No longer collision resistant. <Cite refs={["turn0search0","turn0search6"]}/>

---

## Collision Concept

A collision occurs when:

```text
File A
   │
   ▼
 MD5 Hash
   ▲
   │
File B
```

Two different files generate the same digest.

This challenge exploits that behavior.

---

## Why MD5 Is Unsafe

Applications should **not** rely on MD5 where collision resistance is required, including file identity verification and digital signatures. <Cite refs={["turn0search0","turn0search6"]}/>

---

# 🧪 fastcoll Notes

**fastcoll** is an MD5 collision generation utility used for cryptographic research and educational demonstrations.

### Usage

```bash
fastcoll --prefixfile dog.jpg \
-o collision1.jpg collision2.jpg
```

### Output

- `collision1.jpg`
- `collision2.jpg`

Both satisfy the collision condition expected by the vulnerable application.

---

# 📤 Exploitation Notes

### Upload Process

1. Return to Matchmaker homepage.
2. Upload `collision1.jpg`.
3. Wait for server-side processing.
4. Application returns successful match.

### Why It Works

Server logic effectively performs:

```python
if md5(uploaded_file) == stored_md5:
    accept_match()
```

The server trusts hash equality instead of verifying file identity.

---

# 💥 Root Cause

**Insecure Design Decision**

The application assumes:

```text
Same MD5 Hash
       │
       ▼
Same File
       │
       ▼
Trusted Upload
```

The first assumption is invalid because MD5 collisions are practical.

---

# 🛡️ Recommended Mitigations

## Replace MD5

Use modern cryptographic algorithms:

- SHA-256
- SHA-3
- BLAKE2 / BLAKE3 (where appropriate)

---

## Validate Uploaded Files

Perform multiple validation checks:

- MIME type
- File extension
- Magic bytes
- File size
- Trusted metadata

---

## Defense in Depth

A secure upload pipeline should validate:

```text
Extension
   │
   ▼
MIME Type
   │
   ▼
Magic Bytes
   │
   ▼
Hash (SHA-256)
   │
   ▼
Application Validation
```

---

# 📚 Security Concepts Learned

| Concept | Summary |
|---------|---------|
| Reconnaissance | Identify exposed application functionality. |
| Enumeration | Discover publicly accessible resources. |
| Hash Analysis | Calculate MD5 digest locally. |
| Collision Attack | Generate files with identical MD5 values. |
| Verification Bypass | Abuse insecure hash comparison logic. |
| Secure Development | Replace MD5 with collision-resistant alternatives. |

---

# 🎓 Skills Demonstrated

- Web Application Reconnaissance
- Static Resource Enumeration
- Linux Command-Line Operations
- MD5 Hash Analysis
- Collision Generation with `fastcoll`
- File Upload Testing
- Cryptographic Weakness Identification
- Security Documentation

---

# 📝 Evidence Collected

| Figure | Description |
|--------|-------------|
| Figure 1 | Matchmaker homepage reconnaissance. |
| Figure 2 | Public reference dog image discovered. |
| Figure 3 | Download verification in Kali Linux. |
| Figure 4 | MD5 digest calculation. |
| Figure 5 | Collision generation and verification. |
| Figure 6 | Successful upload (flag redacted). |

---

# 🚩 Key Takeaways

- Never use **MD5** for security-sensitive file verification.
- Collision attacks exploit **trust assumptions**, not necessarily file contents.
- Publicly exposed application assets can significantly aid reconnaissance.
- Secure upload validation requires **multiple independent verification layers**.
- Modern applications should prefer collision-resistant cryptographic algorithms and defense-in-depth validation.

---

# 📖 References

- **RFC 6151 — Updated Security Considerations for the MD5 Message-Digest Algorithm** (IETF). MD5 should not be used where collision resistance is required. <Cite refs={["turn0search0","turn0search6"]}/>
- **HashClash / fastcoll** — Practical MD5 collision generation utility used for research and educational demonstrations. <Cite ref="turn0search0"/>
- **TryHackMe — Love at First Breach** (authorized CTF environment).

---

## 👨‍💻 Portfolio Note

These notes accompany the complete technical report located in:

```text
Documentation/Love at First Breach_Documentation.md
```

This repository is maintained as part of my cybersecurity portfolio to document practical hands-on labs, CTF walkthroughs, and secure development learning.

**Author:** **Anurag Ravikumar**
