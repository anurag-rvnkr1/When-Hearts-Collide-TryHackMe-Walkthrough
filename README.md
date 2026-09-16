# 🐶 Love at First Breach — TryHackMe Web Security Case Study

<p align="center">
  <img src="https://img.shields.io/badge/TryHackMe-Web%20Security-red?style=for-the-badge&logo=tryhackme"/>
  <img src="https://img.shields.io/badge/Category-Cryptography-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Vulnerability-MD5%20Collision-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Portfolio-CTF%20Walkthrough-success?style=for-the-badge"/>
</p>

<p align="center">
  <strong>A professional cybersecurity case study documenting the complete exploitation of an MD5 hash collision vulnerability in a vulnerable web application hosted on TryHackMe.</strong>
</p>

---

## 📖 Project Overview

**Love at First Breach** is a web security Capture The Flag (CTF) challenge from **TryHackMe** that demonstrates how insecure cryptographic design can compromise an application's trust model.

The vulnerable application, **Matchmaker**, compares uploaded images using **MD5 hashes** to determine whether a user's photo matches a stored dog image. Because MD5 is vulnerable to practical collision attacks, an attacker can upload a different image that produces the same digest and bypass the application's verification logic.

This repository documents the challenge as a **professional penetration testing case study** rather than a simple solution.

> **Flag Redaction Policy:** The original TryHackMe flag has been intentionally hidden throughout this repository to preserve the educational value of the room.

---

# 🎯 What This Repository Contains

Instead of only providing a solution, this repository documents the entire assessment lifecycle.

| Section | Description |
|---------|-------------|
| 📄 README | Portfolio-friendly overview of the challenge. |
| 📚 Documentation | Complete technical penetration testing report. |
| 🌐 GitHub Pages | Interactive documentation site. |
| 📸 Screenshots | Evidence collected during the assessment. |
| 📝 Resources | Technical notes and references. |

---

# 🧩 Challenge Snapshot

<table>
<tr>
<td><strong>Platform</strong></td>
<td>TryHackMe</td>
</tr>

<tr>
<td><strong>Room</strong></td>
<td>Love at First Breach</td>
</tr>

<tr>
<td><strong>Category</strong></td>
<td>Web Security • Cryptography</td>
</tr>

<tr>
<td><strong>Difficulty</strong></td>
<td>Easy</td>
</tr>

<tr>
<td><strong>Focus Area</strong></td>
<td>MD5 Collision Attack</td>
</tr>

<tr>
<td><strong>Environment</strong></td>
<td>Kali Linux</td>
</tr>

<tr>
<td><strong>Primary Tool</strong></td>
<td>fastcoll</td>
</tr>

</table>

---

# 🧠 Skills Demonstrated

This project highlights practical cybersecurity skills used during the challenge.

<table>
<tr>
<th>Domain</th>
<th>Skills</th>
</tr>

<tr>
<td><strong>Web Security</strong></td>
<td>Reconnaissance, Static Asset Discovery, File Upload Analysis</td>
</tr>

<tr>
<td><strong>Cryptography</strong></td>
<td>MD5 Analysis, Collision Resistance, Collision Generation</td>
</tr>

<tr>
<td><strong>Linux</strong></td>
<td>wget, md5sum, Package Management, Terminal Workflow</td>
</tr>

<tr>
<td><strong>Security Testing</strong></td>
<td>Application Logic Analysis, Vulnerability Validation</td>
</tr>

<tr>
<td><strong>Documentation</strong></td>
<td>Professional Penetration Testing Reporting</td>
</tr>

</table>

---

# 🗺️ Attack Story (High-Level)

The attack did **not** rely on brute force or guessing.

Instead, it abused an insecure design decision.

```text
User Upload
     │
     ▼
MD5 Hash Generated
     │
     ▼
Application Trusts Hash
     │
     ▼
Collision Image Uploaded
     │
     ▼
Verification Bypass
     │
     ▼
Challenge Completed
```

The complete technical methodology is documented separately inside the report.

---

# 📸 Walkthrough Preview

## Matchmaker Web Application

<img src="docs/assets/figure-1-homepage.png" width="100%">

*Initial reconnaissance of the vulnerable web application.*

---

## Static Resource Discovery

<img src="docs/assets/figure-2-dog-image.png" width="80%">

*A publicly accessible dog image became the trusted reference file for cryptographic analysis.*

---

## Collision Generation in Kali Linux

<img src="docs/assets/figure-5-collision.png" width="100%">

*Generating collision files using `fastcoll` and verifying identical MD5 hashes.*

---

# ⚙️ Tools & Environment

| Tool | Purpose |
|------|---------|
| **Kali Linux** | Security testing environment |
| **Firefox** | Web application interaction |
| **wget** | Download application assets |
| **md5sum** | Hash calculation |
| **fastcoll** | MD5 collision generation |
| **TryHackMe VPN** | Authorized lab connectivity |

---

# 📂 Repository Architecture

```text
When-Hearts-Collide-TryHackMe-Walkthrough
│
├── README.md                         
│
├── Documentation/
│   ├── Love at First Breach_Documentation.md
│   └── Love at First Breach_Documentation.pdf
│
├── Resources/
│   ├── notes.md
│
├── Screenshots/
│
├── docs/
│   ├── index.md                     
│   └── assets/
│
└── _config.yml
```

---

# 📚 Documentation Hub

This repository separates project overview from technical reporting.

### 📄 Technical Report

The complete penetration testing report includes:

- Reconnaissance
- Enumeration
- Hash Analysis
- Collision Generation
- Exploitation
- Root Cause Analysis
- Security Impact
- Mitigation Strategy
- Lessons Learned

➡️ **Open:** `Documentation/Love at First Breach_Documentation.md`

---

### 🌐 GitHub Pages Portfolio

A polished documentation website is available through GitHub Pages.

It includes:

- Interactive walkthrough
- Embedded screenshots
- Attack lifecycle
- Security analysis
- Professional formatting

➡️ **Open:** `docs/index.md`

---

# 💥 Security Insight

### Vulnerability

> **MD5 Hash Collision**

The application assumes:

```text
Same MD5 Digest
      │
      ▼
Same Image
```

This assumption is incorrect because MD5 no longer guarantees collision resistance.

### Security Principle

Never use MD5 as proof of authenticity or identity for attacker-controlled input.

---

# 🛡️ Defensive Perspective

The challenge also reinforces secure software development practices.

### Recommended Defenses

- Replace MD5 with SHA-256 or SHA-3.
- Validate MIME types.
- Validate magic bytes.
- Verify uploaded file metadata.
- Use defense-in-depth validation.
- Use HMAC where authenticity is required.

---

# 📈 Learning Outcomes

This CTF strengthened practical understanding of:

- Cryptographic weaknesses in legacy hash functions.
- Secure file upload validation.
- Static asset enumeration.
- Application trust boundaries.
- Practical vulnerability documentation.

---

# 🎓 Portfolio Value

This repository is part of my **Cybersecurity Portfolio**, where I document hands-on labs, CTFs, and security projects using a structured penetration testing methodology.

### Portfolio Focus Areas

- 🌐 Web Security
- 🔐 Cryptography
- 🛡️ SOC & Blue Team
- 🖥️ Active Directory
- ⚙️ Security Automation
- 📚 Technical Documentation

---

# ⚠️ Disclaimer

This walkthrough documents activities performed exclusively inside an **authorized TryHackMe laboratory environment** for cybersecurity education and research.

The techniques discussed should only be used against systems where explicit authorization has been granted.

---

<div align="center">

## 👨‍💻 Author

### **Anurag Ravikumar**

Cybersecurity Enthusiast • Web Security • SOC • Blue Team • TryHackMe Practitioner

*Building a practical cybersecurity portfolio through hands-on labs, CTF walkthroughs, security tools, and technical documentation.*

⭐ *If you found this repository useful, consider starring it.*

</div>
