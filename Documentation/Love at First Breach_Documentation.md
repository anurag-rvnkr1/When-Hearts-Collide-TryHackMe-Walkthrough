# Love at First Breach — TryHackMe Walkthrough

> Professional Web Security CTF Documentation | Cryptography • MD5 Collision • File Upload Exploitation

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme"/>
  <img src="https://img.shields.io/badge/Category-Web%20Security-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Vulnerability-MD5%20Hash%20Collision-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Status-Completed-success?style=for-the-badge"/>
</p>

---

## Executive Summary

**Love at First Breach** is a beginner-friendly **TryHackMe Web Security challenge** that demonstrates why legacy cryptographic hash functions should never be trusted for security-sensitive file verification.

The application, *Matchmaker*, compares uploaded photos against a database of dog images using **MD5 hashes**. Since MD5 is vulnerable to **collision attacks**, an attacker can create a different image that produces the same MD5 digest as a legitimate image, bypassing the application's verification logic.

This walkthrough documents the complete exploitation methodology, including reconnaissance, static resource discovery, hash analysis, collision generation using **fastcoll**, successful file upload exploitation, and recommended defensive practices.

> **Note:** The challenge flag has been intentionally **redacted** throughout this documentation to preserve TryHackMe challenge integrity and avoid plagiarism.

---

# Room Information

| Field | Details |
|-------|---------|
| **Platform** | TryHackMe |
| **Room Name** | Love at First Breach *(When Hearts Collide)* |
| **Category** | Web Security / Cryptography |
| **Difficulty** | Easy |
| **Vulnerability** | MD5 Hash Collision |
| **Attack Type** | File Upload Verification Bypass |
| **Tools Used** | Browser, Kali Linux, wget, md5sum, fastcoll |
| **Operating System** | Kali Linux |

---

# Learning Objectives

This room focuses on understanding how insecure cryptographic design decisions can introduce practical vulnerabilities in web applications.

After completing this challenge, I was able to:

- Perform basic web application reconnaissance.
- Identify publicly accessible application assets.
- Analyze file integrity using MD5 hashes.
- Understand practical MD5 collision attacks.
- Generate collision files using **fastcoll**.
- Exploit insecure hash-based file matching.
- Understand why MD5 is unsuitable for security-critical applications.
- Document a complete web exploitation workflow in a professional format.

---

# Challenge Scenario

The web application presents itself as a fun **dog compatibility matching service**.

Users upload a picture, and the application claims to find a matching dog based on the uploaded photo's "energy."

Behind the scenes, however, the application performs a much simpler operation:

1. Calculates the **MD5 hash** of the uploaded image.
2. Compares it against hashes stored in its database.
3. If the hashes match, it declares a successful match.

The application's security assumption is fundamentally flawed:

> **Identical MD5 hash = Identical image**

Because MD5 is no longer collision resistant, that assumption can be abused to impersonate another file.

---

# Vulnerability Overview

## What is an MD5 Collision?

MD5 is a cryptographic hashing algorithm that produces a **128-bit digest** for any input.

A secure hash function should make it computationally infeasible to produce two different inputs with the same output.

MD5 no longer satisfies that property.

A **collision** occurs when two different files generate exactly the same MD5 hash.

This weakness enables attackers to bypass systems that rely exclusively on MD5 for authentication, integrity verification, or file identification.

### Why is this Dangerous?

| Secure Assumption | Reality |
|-------------------|---------|
| Same hash means same file. | Different files can intentionally share an MD5 digest. |
| Hash uniquely identifies content. | MD5 collisions can be generated with specialized tools. |
| MD5 protects integrity. | MD5 is considered cryptographically broken for collision resistance. |

---

# Attack Flow

The overall exploitation process followed during this challenge is illustrated below.

```text
                    Matchmaker Web Application
                              │
                              ▼
                   Application Reconnaissance
                              │
                              ▼
                 Discover Public Image Resource
                              │
                              ▼
                  Download Reference Dog Image
                              │
                              ▼
                    Calculate MD5 Digest
                              │
                              ▼
             Generate Collision Images (fastcoll)
                              │
                              ▼
            Verify Matching MD5 Hashes Locally
                              │
                              ▼
               Upload Collision Image to Target
                              │
                              ▼
           Application Accepts Malicious Upload
                              │
                              ▼
                Challenge Successfully Completed
```

---

# Attack Lifecycle

| Phase | Objective |
|-------|-----------|
| Reconnaissance | Understand the application workflow. |
| Enumeration | Identify exposed image resources. |
| Acquisition | Download the reference image. |
| Hash Analysis | Calculate and inspect the MD5 digest. |
| Collision Generation | Produce alternate files with identical MD5 values. |
| Exploitation | Upload collision image to bypass validation. |
| Validation | Confirm successful application response. |

---

# Tools Used

<table>
<tr>
<th>Tool</th>
<th>Purpose</th>
</tr>

<tr>
<td><strong>Kali Linux</strong></td>
<td>Primary penetration testing environment.</td>
</tr>

<tr>
<td><strong>Firefox / Browser</strong></td>
<td>Reconnaissance and interaction with the web application.</td>
</tr>

<tr>
<td><strong>wget</strong></td>
<td>Download the publicly exposed reference image.</td>
</tr>

<tr>
<td><strong>md5sum</strong></td>
<td>Calculate and verify MD5 hashes.</td>
</tr>

<tr>
<td><strong>fastcoll</strong></td>
<td>Generate practical MD5 collision files.</td>
</tr>

<tr>
<td><strong>TryHackMe Lab</strong></td>
<td>Authorized vulnerable environment.</td>
</tr>

</table>

---

# Repository Structure

This repository is organized as a professional documentation project.

```text
When-Hearts-Collide-TryHackMe-Walkthrough
│
├── README.md
│
├── Documentation/
│   ├── Love at First Breach_Documentation.md
│   └── Love at First Breach_Documentation.docx
│
├── Screenshots/
│   ├── figure-1-homepage.png
│   ├── figure-2-dog-image.png
│   ├── figure-3-download.png
│   ├── figure-4-md5sum.png
│   ├── figure-5-collision.png
│   └── figure-6-flag-redacted.png
│
├── Resources/
│   ├── notes.md
│   └── references.md
│
├── docs/
│   └── index.md
│
└── _config.yml
```

The documentation is also compatible with **GitHub Pages** through the `docs/` directory.

---

# Prerequisites

Before beginning the exploitation process, the following tools were available in the Kali Linux environment:

- Linux terminal access.
- Internet connectivity to the TryHackMe VPN.
- `wget`
- `md5sum`
- `fastcoll`
- Web browser.

The attack does **not** require authentication or advanced exploitation tools.

---

# Environment Setup

The challenge was completed inside an authorized TryHackMe virtual lab using Kali Linux.

**Target Service**

| Property | Value |
|----------|-------|
| Protocol | HTTP |
| Target | TryHackMe Assigned Machine |
| Access | Browser + Terminal |
| File Upload | Enabled |

No modifications were performed outside the scope of the CTF environment.

---

# Methodology

This walkthrough follows a structured penetration testing methodology rather than a simple solution.

1. **Reconnaissance** — Understand the application and identify exposed functionality.
2. **Enumeration** — Locate publicly accessible static resources.
3. **Analysis** — Calculate and inspect the MD5 hash of the reference image.
4. **Exploitation** — Generate collision files and bypass file verification.
5. **Validation** — Confirm successful exploitation.
6. **Post-Exploitation Analysis** — Understand root cause and mitigation.

Each phase is documented with supporting screenshots and technical explanations.

---

# Evidence Collection

Every important stage of the attack is supported with screenshots captured during the lab.

| Figure | Description |
|--------|-------------|
| **Figure 1** | Matchmaker application homepage. |
| **Figure 2** | Publicly accessible dog image discovered through enumeration. |
| **Figure 3** | Downloading the target image into Kali Linux. |
| **Figure 4** | Calculating the MD5 hash using `md5sum`. |
| **Figure 5** | Generating and verifying collision files with `fastcoll`. |
| **Figure 6** | Successful match and redacted challenge flag. |

# Phase 1 — Reconnaissance & Static Asset Discovery

Reconnaissance is the first stage of every penetration test. The objective is to understand how the application behaves, identify accessible functionality, and discover resources that may contribute to an attack path.

For this challenge, reconnaissance focused on understanding the application's upload workflow and identifying any publicly exposed files that could be reused during exploitation.

---

# Step 1 — Initial Reconnaissance

The target application is accessible over HTTP through the TryHackMe machine assigned during the lab.

```text
http://TARGET_IP
```

Visiting the homepage reveals a visually designed web application called **Matchmaker**, which claims to pair users with dogs based on the uploaded photo.

### Observations During Reconnaissance

The landing page immediately provides useful information about the application's functionality.

| Observation | Security Relevance |
|------------|--------------------|
| Image upload form | Indicates user-controlled file upload functionality. |
| Featured breed match list | Suggests publicly accessible resources may exist. |
| Static design assets | Indicates predictable static file paths. |
| No authentication required | Anyone can interact with the upload feature. |

The upload functionality becomes the primary attack surface because the application processes user-supplied files.

---

## Figure 1 — Matchmaker Application Homepage

![Figure 1 — Matchmaker Homepage](../Screenshots/figure-1-homepage.png)

**Figure 1:** Homepage of the vulnerable Matchmaker application displaying the image upload interface and featured breed section.

### Interface Analysis

Several UI elements provide hints about how the application operates.

<table>
<tr>
<th>Component</th>
<th>Description</th>
</tr>

<tr>
<td>Application Title</td>
<td><strong>Matchmaker — Find your perfect dog</strong></td>
</tr>

<tr>
<td>Upload Area</td>
<td>Allows users to upload an image for processing.</td>
</tr>

<tr>
<td>Description Text</td>
<td>Suggests uploaded images are compared with dog images.</td>
</tr>

<tr>
<td>Featured Breed Link</td>
<td>Potential entry point for discovering stored images.</td>
</tr>

<tr>
<td>Navigation</td>
<td>Static pages including Home, Matchlist, and About.</td>
</tr>

</table>

### Security Observation

Although the interface appears harmless, it exposes two important attack opportunities:

1. The upload feature accepts user-controlled files.
2. The application advertises a **featured breed match list**, which may expose backend resources.

These observations justify further enumeration.

---

# Reconnaissance Notes

At this stage, no exploitation is performed.

The goal is simply to answer:

- What files does the application expose?
- Where are uploaded images stored?
- Can publicly accessible resources be downloaded?

Finding these answers helps determine whether application assets can be reused during an attack.

---

# Step 2 — Enumerating Static Resources

The featured breed section references dog images hosted by the application itself.

Instead of uploading a random image immediately, the next step is to inspect how these images are delivered.

### Enumeration Technique

A simple inspection of page elements (or the browser's developer tools) reveals that images are served from a predictable location.

Example resource path:

```text
/static/uploads/<image-id>.jpg
```

This indicates that uploaded or stored images are publicly accessible through the application's static directory.

### Why This Matters

Public access to stored files creates an opportunity to retrieve an image that the application already trusts.

That trusted image becomes the perfect candidate for local cryptographic analysis.

---

## Figure 2 — Publicly Accessible Reference Image

![Figure 2 — Public Dog Image](../Screenshots/figure-2-dog-image.png)

**Figure 2:** Dog image discovered through the application's publicly accessible upload directory.

### Analysis

The image displayed above is stored directly under the application's `/static/uploads/` path.

Important characteristics:

| Finding | Impact |
|---------|--------|
| Public URL | Image can be downloaded locally. |
| Predictable directory | Static assets are enumerable. |
| Same image used by application | Useful reference for hash comparison. |

### Security Insight

Applications frequently expose static resources for performance reasons.

However, exposing files that participate in security-sensitive logic can become dangerous when:

- hashes are predictable,
- files are publicly downloadable,
- verification relies only on file hashes.

This room intentionally demonstrates that design weakness.

---

# Why Download the Image?

The application's matching mechanism compares hashes.

To reproduce that behavior locally, the original image is required.

The attack workflow therefore becomes:

1. Obtain the trusted image.
2. Calculate its MD5 hash.
3. Generate a collision using that file.
4. Upload a different file sharing the same digest.

Without downloading the original image, generating a matching collision would not be possible.

---

# Information Gathered During Enumeration

<table>
<tr>
<th>Information Collected</th>
<th>Purpose</th>
</tr>

<tr>
<td>Application accepts image uploads.</td>
<td>Primary attack surface identified.</td>
</tr>

<tr>
<td>Dog images stored under `/static/uploads/`.</td>
<td>Reference file location discovered.</td>
</tr>

<tr>
<td>Reference image publicly accessible.</td>
<td>Can be downloaded for offline analysis.</td>
</tr>

<tr>
<td>Matching mechanism appears deterministic.</td>
<td>Likely based on image-derived values such as hashes.</td>
</tr>

</table>

---

# Phase Summary

Reconnaissance successfully identified the application's attack surface and exposed a trusted reference image that can be analyzed locally.

### Key Outcomes

- Identified the vulnerable image upload workflow.
- Located a publicly accessible reference image.
- Confirmed that application assets are downloadable.
- Established the starting point for cryptographic analysis.

# Phase 2 — Cryptographic Analysis & MD5 Collision Exploitation

After identifying a publicly accessible reference image during reconnaissance, the next objective was to obtain that file locally and understand how the application verifies uploaded images.

This phase focuses on reproducing the application's verification logic, analyzing the MD5 digest of the reference image, and generating collision files capable of bypassing the application's trust mechanism.

---

# Step 3 — Download the Reference Image

The reference dog image discovered during enumeration is publicly accessible through the application's static upload directory. Downloading this image allows local cryptographic analysis and provides the input required for collision generation.

## Downloading the Image

Using Kali Linux, the image was downloaded directly from the application's static resource endpoint.

```bash
wget http://TARGET_IP/static/uploads/<image-id>.jpg -O dog.jpg
```

### Verify the Download

After downloading, verify that the file exists locally.

```bash
ls -la dog.jpg
```

A successful download confirms that the image has been saved in the current working directory and is ready for further analysis.

---

## Figure 3 — Download Verification

![Figure 3 — Downloaded Reference Image](../Screenshots/figure-3-download.png)

**Figure 3:** The reference dog image successfully downloaded into the Kali Linux working directory using `wget`, followed by local verification with `ls -la`.

### Why This Step Matters

Downloading the trusted image is essential because the application already recognizes this file internally. Rather than attempting to reverse engineer the matching algorithm, we obtain the exact file that participates in the verification process.

### Technical Observation

| Observation | Security Relevance |
|-------------|--------------------|
| Public image download succeeds. | Static resources are publicly accessible. |
| Original file preserved locally. | Enables offline cryptographic analysis. |
| No authentication required. | Anyone can retrieve trusted application assets. |

### Evidence Collected

- Original reference image obtained.
- File stored locally as `dog.jpg`.
- File ready for hash calculation.

---

# Step 4 — Analyze the MD5 Hash

The application determines image matches by comparing MD5 hashes. To understand this behavior, the downloaded file's digest is calculated locally.

## Calculate MD5

```bash
md5sum dog.jpg
```

The command produces a 128-bit hexadecimal digest representing the file.

---

## Figure 4 — MD5 Hash Calculation

![Figure 4 — MD5 Digest of Reference Image](../Screenshots/figure-4-md5sum.png)

**Figure 4:** Calculating the MD5 digest of the downloaded reference image using the Linux `md5sum` utility.

### Understanding the Output

`md5sum` computes a deterministic fingerprint of the file contents.

Example format:

```text
<32-character hexadecimal hash>  dog.jpg
```

Every time the exact same file is hashed, the digest remains identical.

### Why the Digest Is Important

The application appears to perform the following workflow:

```text
Uploaded Image
       │
       ▼
  Calculate MD5
       │
       ▼
Compare with Stored Hash
       │
       ▼
    Match / No Match
```

The digest therefore becomes the value the application trusts when identifying images.

---

## MD5 Fundamentals

### What is MD5?

MD5 (Message Digest Algorithm 5) is a hashing algorithm that converts arbitrary input into a fixed **128-bit digest**.

### Properties of MD5

| Property | Description |
|----------|-------------|
| Output Length | 128 bits |
| Deterministic | Same input always produces the same digest. |
| Fast | Efficient to compute. |
| Collision Resistant | **No — cryptographically broken.** |

### Why MD5 Is Considered Broken

Modern cryptography requires collision resistance, meaning attackers should not be able to generate different inputs with identical hashes.

MD5 no longer provides this guarantee. Practical collision attacks have existed for years, making it unsuitable for authentication, integrity validation, or identity verification. <Cite refs={["turn0search5","turn0search6"]}/>

### Security Observation

The vulnerability is **not** that MD5 reveals information about the file.

The vulnerability is that the application **uses MD5 equality as proof that two files are identical**.

---

# Cryptographic Weakness Explained

A secure application expects this relationship:

```text
File A
   │
   ▼
 SHA-256 / MD5
   │
   ▼
Unique Digest
```

The vulnerable application instead assumes:

```text
File A
   │
   ▼
MD5 Hash
   │
   ▼
File B has Same MD5
   │
   ▼
File A == File B ❌
```

That assumption becomes exploitable through collisions.

---

# Step 5 — Install fastcoll

The next objective is to generate alternate files that produce the same MD5 digest.

## What is fastcoll?

`fastcoll` is a collision-generation utility implementing practical MD5 collision attacks. It creates two different files sharing an identical MD5 digest.

For this challenge, it is used inside an authorized TryHackMe lab to demonstrate why MD5 should never be trusted for security-sensitive comparisons. <Cite refs={["turn0search5","turn0search6"]}/>

---

## Installation

Update the package repository.

```bash
sudo apt update
```

Install the collision-generation utility.

```bash
sudo apt install fastcoll -y
```

Verify installation.

```bash
fastcoll -h
```

A successful installation displays the command usage information.

---

## Why fastcoll Works

`fastcoll` exploits weaknesses in MD5's collision resistance.

Instead of finding a preimage for an existing digest, it generates two different inputs that collide under MD5 while preserving a common prefix.

This behavior is sufficient to bypass applications relying exclusively on MD5 equality.

---

# Generate Collision Files

Using the downloaded dog image as the prefix file:

```bash
fastcoll --prefixfile dog.jpg -o collision1.jpg collision2.jpg
```

### Command Breakdown

| Argument | Purpose |
|----------|---------|
| `--prefixfile dog.jpg` | Uses the trusted reference image as the collision prefix. |
| `-o` | Specifies output files. |
| `collision1.jpg` | First generated collision file. |
| `collision2.jpg` | Second generated collision file. |

The command creates two distinct files derived from the reference image.

---

# Verify Collision Generation

Calculate the hashes of all generated files.

```bash
md5sum dog.jpg collision1.jpg collision2.jpg
```

The verification step confirms that the collision files produce the expected MD5 digest required for exploitation.

---

## Figure 5 — Collision Generation & Verification

![Figure 5 — fastcoll Collision Generation](../Screenshots/figure-5-collision.png)

**Figure 5:** Collision files generated with `fastcoll`, followed by local verification using `md5sum`.

### Technical Interpretation

This verification demonstrates the core weakness exploited in the challenge:

- Collision files are generated locally.
- The application compares only MD5 digests.
- Matching digests satisfy the application's verification logic.

The important takeaway is **the application's trust model**, not the specific hash value.

---

# Collision Verification Workflow

```text
Original Image
      │
      ▼
   MD5 Digest
      │
      ▼
Collision Generator
      │
      ▼
collision1.jpg
collision2.jpg
      │
      ▼
Identical MD5 Digest
```

The generated files become valid candidates for bypassing the application's image comparison mechanism.

---

# Security Analysis

## Why This Is an Application Vulnerability

The application performs verification using a value that attackers can reproduce.

### Weak Design

```text
if md5(upload) == stored_md5:
    accept_upload()
```

### Stronger Design

```text
Validate file type
Validate magic bytes
Validate metadata
Compare trusted identifiers
Use collision-resistant hashes when appropriate
```

The weakness exists because **hash equality is treated as identity verification**.

---

# Evidence Collected During This Phase

<table>
<tr>
<th>Evidence</th>
<th>Description</th>
</tr>

<tr>
<td>Figure 3</td>
<td>Reference image downloaded successfully into Kali Linux.</td>
</tr>

<tr>
<td>Figure 4</td>
<td>MD5 digest calculated locally using `md5sum`.</td>
</tr>

<tr>
<td>Figure 5</td>
<td>Collision files generated and verified using `fastcoll`.</td>
</tr>

</table>

---

# Phase Summary

This phase reproduced the application's verification mechanism outside the target environment and demonstrated how MD5 collisions can undermine hash-based file identification.

### Key Outcomes

- Downloaded the trusted reference image.
- Calculated the MD5 digest locally.
- Installed the `fastcoll` collision-generation utility.
- Generated collision images suitable for exploitation.
- Verified collision behavior through local hash analysis.

# Phase 3 — Exploiting the Vulnerable Upload Workflow

After generating valid MD5 collision files, the next objective is to submit one of the generated images through the application's upload interface and observe how the server validates it.

This phase demonstrates the practical exploitation of insecure hash-based verification implemented by the Matchmaker application.

---

# Step 6 — Upload the Collision Image

The application provides an image upload form on the homepage. Instead of uploading the original reference image, one of the generated collision files is submitted.

## Upload Procedure

1. Navigate back to the Matchmaker homepage.
2. Click the upload area.
3. Select **collision1.jpg** (or `collision2.jpg`).
4. Submit the image for processing.
5. Wait for the application to complete its matching process.

The upload process requires no authentication or additional verification.

---

## Upload Workflow

```text
collision1.jpg
        │
        ▼
Upload Request
        │
        ▼
Server Calculates MD5
        │
        ▼
Compare with Stored MD5
        │
        ▼
Digest Matches Reference Image
        │
        ▼
Application Returns Successful Match
```

The important point is that the server validates **only the MD5 digest**, not whether the uploaded file is actually identical to the stored image.

---

## Why Uploading the Collision Works

The generated collision image is different from the original file, but it satisfies the application's verification logic because the server performs this comparison:

```python
if md5(uploaded_file) == stored_md5:
    return "It's a Match!"
```

This logic assumes hash equality guarantees content equality.

That assumption is incorrect for MD5.

---

# Server-Side Validation Analysis

The vulnerable workflow can be simplified into the following logic.

<table>
<tr>
<th>Application Step</th>
<th>Result</th>
</tr>

<tr>
<td>User uploads image.</td>
<td>Image accepted for processing.</td>
</tr>

<tr>
<td>Server calculates MD5 digest.</td>
<td>Digest generated successfully.</td>
</tr>

<tr>
<td>Digest compared with database value.</td>
<td>Collision digest matches trusted image.</td>
</tr>

<tr>
<td>Application concludes files are identical.</td>
<td>Successful dog match returned.</td>
</tr>

</table>

### Security Observation

The server never verifies:

- File content.
- Binary equality.
- Magic bytes.
- File metadata.
- Trusted identifier.

Only a single MD5 comparison determines success.

---

# Step 7 — Successful Match

Once the collision image is processed, the application identifies it as a valid match.

The interface displays a success message indicating that a matching dog has been found.

The challenge objective is successfully completed at this stage.

---

## Figure 6 — Successful Match (Flag Redacted)

![Figure 6 — Challenge Completion](../Screenshots/figure-6-flag-redacted.png)

**Figure 6:** Successful challenge completion after uploading the collision image. The flag has been intentionally redacted in this documentation.

---

## Challenge Result

The application reveals the challenge flag after a successful match.

### Flag (Redacted)

```text
THM{********************}
```

> **Note:** The original flag has been intentionally hidden to maintain the educational value of the TryHackMe room and prevent plagiarism.

---

# Exploitation Validation

The objective of this phase was to verify that the collision image satisfies the application's verification mechanism.

### Validation Checklist

| Validation Item | Status |
|-----------------|--------|
| Upload accepted successfully. | ✅ |
| Collision image processed. | ✅ |
| Server returned successful match. | ✅ |
| Challenge completed. | ✅ |
| Flag obtained (redacted). | ✅ |

The successful response confirms that the application's trust model can be bypassed using an MD5 collision.

---

# Why the Exploit Succeeds

The attack succeeds because the application confuses **hash equality** with **file equality**.

### Actual Application Logic

```text
Uploaded File
      │
      ▼
Calculate MD5 Hash
      │
      ▼
Stored MD5 Hash
      │
      ▼
Hashes Match
      │
      ▼
Accept Upload
```

### Missing Security Controls

The application does not perform additional validation after the hash comparison.

| Missing Validation | Security Benefit |
|--------------------|------------------|
| Magic byte validation | Detects invalid or manipulated file types. |
| MIME type verification | Prevents spoofed uploads. |
| File content comparison | Ensures uploaded content matches trusted data. |
| Secure object identifiers | Avoids relying solely on hashes. |

---

# Attack Success Criteria

This challenge demonstrates a successful exploitation when the following conditions are met.

<table>
<tr>
<th>Requirement</th>
<th>Outcome</th>
</tr>

<tr>
<td>Reference image identified.</td>
<td>Completed during reconnaissance.</td>
</tr>

<tr>
<td>MD5 digest calculated.</td>
<td>Completed locally.</td>
</tr>

<tr>
<td>Collision files generated.</td>
<td>Completed using `fastcoll`.</td>
</tr>

<tr>
<td>Collision uploaded.</td>
<td>Accepted by the application.</td>
</tr>

<tr>
<td>Application trusts collision digest.</td>
<td>Successful verification bypass.</td>
</tr>

</table>

---

# Technical Breakdown of the Vulnerability

## Trust Boundary Failure

The vulnerable application crosses a security boundary by trusting attacker-controlled input after only checking its MD5 digest.

```text
User-Controlled File
         │
         ▼
Cryptographic Digest
         │
         ▼
Trusted Decision
```

This is a common design mistake when applications use weak cryptographic primitives for identity verification.

---

## Integrity Verification Failure

Hash functions can be used for integrity verification **only when collision resistance remains strong** for the intended use case.

Because MD5 collisions are practical, an attacker can intentionally satisfy the integrity check with a different file.

The application therefore loses the ability to distinguish trusted content from attacker-controlled content.

---

# Security Impact Assessment

The practical consequences of this design flaw depend on the application's purpose.

### Potential Risks

| Risk | Description |
|------|-------------|
| File Impersonation | Malicious files can appear identical to trusted files. |
| Authentication Bypass | Hash-based identity checks become unreliable. |
| Integrity Failure | File authenticity cannot be guaranteed. |
| Business Logic Abuse | Application decisions become attacker-controlled. |

### CVE-Style Summary

| Property | Assessment |
|----------|------------|
| Vulnerability Type | Cryptographic Weakness |
| CWE Category | Weak Hash / Collision Vulnerability |
| Attack Complexity | Low |
| Privileges Required | None |
| User Interaction | Upload file |
| Confidentiality Impact | Low |
| Integrity Impact | High |
| Availability Impact | Low |

---

# Post-Exploitation Notes

After validating the exploit, no further interaction with the target application is required.

This room focuses on demonstrating the weakness in MD5-based verification rather than privilege escalation or persistence.

### Evidence Collected During This Phase

| Evidence | Description |
|----------|-------------|
| Figure 6 | Successful application response after collision upload. |
| Upload Result | Collision accepted as trusted image. |
| Validation Outcome | Challenge completed successfully. |

---

# Phase Summary

The exploitation phase successfully demonstrated that an MD5 collision can bypass the application's image verification logic.

### Key Takeaways

- Collision image successfully uploaded.
- Application trusted the generated MD5 digest.
- Verification logic bypassed without modifying the original image.
- Challenge completed in an authorized TryHackMe environment.

# Phase 4 — Root Cause Analysis & Defensive Security Review

The exploitation successfully demonstrated that the application trusted a cryptographically weak hash function to identify uploaded files. This section explains **why the vulnerability exists**, its security implications, and how modern applications should defend against similar attacks.

---

# Root Cause Analysis

The vulnerability exists because the application uses **MD5 hashes as the sole source of truth** when determining whether an uploaded image matches a stored image.

### Vulnerable Verification Logic

The application's behavior can be represented as:

```python
uploaded_hash = MD5(uploaded_file)

if uploaded_hash == stored_hash:
    return "Match Found"
```

This logic assumes that identical MD5 hashes always represent identical files.

That assumption is incorrect because MD5 is vulnerable to **collision attacks**, allowing two different files to produce the same digest.

---

## Trust Boundary Failure

The application's trust boundary is crossed too early.

```text
Attacker Controlled File
          │
          ▼
    MD5 Hash Calculation
          │
          ▼
  Trusted Authentication Decision
```

The attacker controls the input before the application establishes trust, making the verification mechanism unreliable.

---

## Why This Design Is Insecure

A cryptographic hash should be collision resistant if it is used to verify identity or integrity.

### What MD5 Guarantees

| Property | Status |
|----------|--------|
| Deterministic hashing | ✅ Yes |
| Fast computation | ✅ Yes |
| Collision resistance | ❌ Broken |
| Suitable for integrity verification | ❌ Not recommended |

Modern cryptographic guidance discourages MD5 for security-sensitive use cases because practical collision attacks have existed for many years. <Cite refs={["turn0search5","turn0search6"]}/>

---

# Understanding MD5 Collision Attacks

## What Is a Collision?

A collision occurs when two different inputs generate the same hash value.

```text
File A
   │
   ▼
MD5 Digest
   ▲
   │
File B
```

Even though **File A** and **File B** contain different binary content, their MD5 digests are identical.

### Why That Matters

Applications using only hash equality cannot distinguish between:

- Legitimate content.
- Attacker-generated collision content.

This is exactly what the challenge demonstrates.

---

## Collision vs Preimage

These concepts are different.

| Concept | Description |
|--------|-------------|
| Collision | Two different files share the same hash. |
| Preimage Attack | Generate a file matching an existing hash exactly. |
| Second Preimage | Generate another file matching a chosen file's hash. |

This room demonstrates a **collision attack**, not password cracking or brute-force hashing.

---

# How `fastcoll` Fits Into the Attack

`fastcoll` is a practical MD5 collision generation utility used for cryptographic research and education.

### Purpose During This Lab

- Generate two files sharing an identical MD5 digest.
- Demonstrate weaknesses in applications trusting MD5.
- Validate the vulnerability inside an authorized CTF environment.

The challenge intentionally uses MD5 to teach why legacy hashing algorithms should not be used for identity verification.

---

# Security Impact Assessment

Although this room is intentionally simplified, similar design flaws have real-world implications.

## Potential Security Risks

<table>
<tr>
<th>Risk</th>
<th>Description</th>
</tr>

<tr>
<td>File Impersonation</td>
<td>An attacker can submit different content that appears identical to trusted content.</td>
</tr>

<tr>
<td>Integrity Verification Failure</td>
<td>The application cannot guarantee uploaded content is authentic.</td>
</tr>

<tr>
<td>Business Logic Bypass</td>
<td>Application decisions become attacker-controlled through crafted inputs.</td>
</tr>

<tr>
<td>Weak Cryptographic Trust</td>
<td>Security depends on an outdated hashing algorithm.</td>
</tr>

</table>

---

## Security Impact Summary

| Security Principle | Impact |
|--------------------|--------|
| Confidentiality | Low |
| Integrity | **High** |
| Availability | Low |
| Authentication Trust | High Risk |
| Input Validation | Weak |

The primary impact is a **loss of integrity**, because the application incorrectly authenticates attacker-controlled content.

---

# Mapping to Secure Development Concepts

<table>
<tr>
<th>Area</th>
<th>Recommendation</th>
</tr>

<tr>
<td>Cryptography</td>
<td>Use collision-resistant hashing algorithms.</td>
</tr>

<tr>
<td>File Upload Security</td>
<td>Validate MIME type, extension, and magic bytes.</td>
</tr>

<tr>
<td>Application Logic</td>
<td>Do not rely solely on hashes for identity verification.</td>
</tr>

<tr>
<td>Defense in Depth</td>
<td>Combine multiple validation mechanisms before trusting uploaded content.</td>
</tr>

</table>

---

# Mitigation Strategies

A secure implementation should use multiple validation layers instead of trusting MD5.

## 1. Replace MD5

Use a modern collision-resistant hash algorithm.

| Recommended Algorithm | Use Case |
|----------------------|----------|
| SHA-256 | General integrity verification. |
| SHA-3 | Modern cryptographic applications. |
| BLAKE2/BLAKE3 | High-performance hashing where appropriate. |

---

## 2. Validate File Content

Verify the uploaded file independently.

Recommended checks include:

- MIME type validation.
- Magic byte inspection.
- File extension verification.
- File size validation.

These checks help prevent spoofed uploads.

---

## 3. Compare Trusted Identifiers

Instead of identifying files only by hashes:

- Store internal identifiers.
- Use database references.
- Associate uploads with trusted metadata.

This prevents collision-based impersonation.

---

## 4. Use HMAC for Authenticity

If authenticity verification is required, use **HMAC** with a server-side secret instead of relying on a public hash alone.

Benefits include:

- Secret-backed verification.
- Protection against attacker-generated hashes.
- Stronger integrity guarantees.

---

## 5. Defense in Depth

A secure upload workflow should validate multiple properties.

```text
Upload File
     │
     ▼
Validate Extension
     │
     ▼
Validate MIME Type
     │
     ▼
Validate Magic Bytes
     │
     ▼
Virus / Malware Scan
     │
     ▼
Application-Level Verification
```

No single validation mechanism should determine trust.

---

# Secure Upload Workflow (Recommended)

```text
User Upload
     │
     ▼
Server Receives File
     │
     ├── Validate Extension
     ├── Validate MIME Type
     ├── Validate Magic Bytes
     ├── Verify Size Limits
     ├── Generate SHA-256
     └── Store Trusted Metadata
             │
             ▼
      Accept Valid Upload
```

This layered approach significantly reduces attack opportunities.

---

# Key Takeaways

This challenge provides an excellent introduction to insecure cryptographic design in web applications.

### Technical Lessons Learned

- MD5 is deterministic but no longer collision resistant.
- Publicly exposed static assets can aid attackers during reconnaissance.
- Weak verification logic can turn harmless functionality into an attack surface.
- Collision attacks target **trust assumptions**, not necessarily file contents.
- Security decisions should never rely exclusively on deprecated cryptographic primitives.

---

# Skills Demonstrated

This project demonstrates practical cybersecurity skills applicable to web security and secure software development.

<table>
<tr>
<th>Skill</th>
<th>Application</th>
</tr>

<tr>
<td>Web Reconnaissance</td>
<td>Identified application functionality and exposed resources.</td>
</tr>

<tr>
<td>Static Resource Enumeration</td>
<td>Located publicly accessible files used during verification.</td>
</tr>

<tr>
<td>Linux Command Line</td>
<td>Downloaded files and calculated hashes using native utilities.</td>
</tr>

<tr>
<td>Cryptographic Analysis</td>
<td>Analyzed MD5-based verification logic.</td>
</tr>

<tr>
<td>Vulnerability Validation</td>
<td>Generated collision files and validated application behavior.</td>
</tr>

<tr>
<td>Security Documentation</td>
<td>Produced a structured penetration testing report.</td>
</tr>

</table>

---

# Tools Used During the Lab

| Tool | Purpose |
|------|---------|
| **Firefox / Browser** | Application reconnaissance and upload testing. |
| **Kali Linux** | Security testing environment. |
| **wget** | Download publicly exposed resources. |
| **md5sum** | Calculate and verify MD5 digests. |
| **fastcoll** | Generate MD5 collision files. |
| **TryHackMe** | Authorized CTF environment. |

---

# Evidence Summary

Throughout this assessment, each stage was documented using screenshots collected during the lab.

| Figure | Description |
|--------|-------------|
| **Figure 1** | Matchmaker homepage reconnaissance. |
| **Figure 2** | Static resource discovery. |
| **Figure 3** | Download verification in Kali Linux. |
| **Figure 4** | MD5 digest calculation. |
| **Figure 5** | Collision generation and verification. |
| **Figure 6** | Successful challenge completion with flag redacted. |

These screenshots provide reproducible evidence for every major stage of the exploitation process.

---

# Conclusion

The **Love at First Breach** challenge demonstrates how insecure cryptographic assumptions can compromise an otherwise simple web application.

Rather than exploiting complex server-side vulnerabilities, this challenge highlights a common secure development lesson: **cryptography must be used correctly**. MD5 remains useful for non-security purposes such as checksums in some legacy contexts, but it should never be trusted as proof of identity or authenticity for attacker-controlled input.

By combining reconnaissance, static asset enumeration, cryptographic analysis, and collision generation, this assessment successfully reproduced the application's verification logic and demonstrated a practical integrity bypass inside an authorized TryHackMe environment.

The room reinforces important secure coding principles, particularly around **modern hashing algorithms, defense-in-depth validation, and secure file upload design**.

---

# References

### Official & Technical References

- **TryHackMe** — Love at First Breach (When Hearts Collide) Room. <Cite ref="turn0search0"/>
- **RFC 6151** — Updated Security Considerations for the MD5 Message-Digest Algorithm. <Cite ref="turn0search6"/>
- **HashClash / fastcoll** — Practical MD5 Collision Generation Utility. <Cite ref="turn0search5"/>
- MD5 Collision Research by Marc Stevens and collaborators. <Cite refs={["turn0search5","turn0search6"]}/>

---

# Portfolio Note

This documentation was created as part of my **Cybersecurity Learning Portfolio** to document practical hands-on labs completed on TryHackMe.

**Author:** **Anurag Ravikumar**

**Focus Areas:** Web Security • SOC • Blue Team • Active Directory • Cryptography • Security Automation

> This walkthrough is intended for cybersecurity education, research, and portfolio purposes only. All activities were performed within an authorized TryHackMe laboratory environment.
