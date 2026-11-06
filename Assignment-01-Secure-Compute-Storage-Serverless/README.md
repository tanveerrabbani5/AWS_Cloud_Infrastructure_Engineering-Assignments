
<div align="center">

# AWS Cloud Infrastructure Engineering

## Assignment 01 — Secure Compute, Storage & Serverless

**Assignment:** `Assignment-01-Secure-Compute-Storage-Serverless`

[![AWS](https://img.shields.io/badge/Cloud-AWS-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![Region](https://img.shields.io/badge/Region-ap--south--1-blue)](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)
[![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)](https://www.python.org/)

**Secure Compute | Storage Tiering | Serverless Integration**

**Prepared by:** Tanveer Rabbani A

</div>

---

## 1. Project Overview

A media streaming enterprise is modernizing its ingestion pipeline.

This assignment demonstrates three major AWS implementations:

1. **Secure Compute:** Provision a hardened Amazon EC2 instance using a custom AMI, restricted Security Groups, and CloudWatch automatic recovery.
2. **Storage Tiering:** Configure Amazon S3 with encryption, versioning, and lifecycle transitions.
3. **Serverless Integration:** Automatically generate image thumbnails using AWS Lambda when JPG files are uploaded to S3.

**AWS Region:** Asia Pacific (Mumbai) — `ap-south-1`

---


## 2. Architecture Diagram

The following architecture illustrates the three major AWS implementations: Secure Compute, S3 Storage Tiering, and Serverless Thumbnail Generation.

<div align="center">

![AWS Assignment 01 Architecture](docs/architecture-diagram.png)

**Figure 1: AWS Secure Compute, Storage & Serverless Architecture**

</div>

**Architecture Note:** EC2 and S3-to-Lambda are independent workflows. EC2 is not involved in thumbnail generation.
---
The complete assignment documentation is available below.

<div align="center">

### 📄 Assignment 01 — Documentation

[**View Assignment PDF**](docs/Assignment-01-Documentation.pdf)

[**Download Assignment PDF**](docs/Assignment-01-Documentation.pdf)

</div>

---

## 3. AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon EC2 | Secure compute environment |
| Amazon AMI | Reusable EC2 image |
| Amazon S3 | Media object storage |
| AWS Lambda | Image thumbnail processing |
| AWS IAM | Access control and permissions |
| Amazon CloudWatch | Monitoring and logging |
| AWS CloudShell | Pillow dependency packaging |
| Lambda Layers | Pillow library integration |

---

## 4. Task 1 - EC2 Security Hardening

### 4.1 Custom AMI

| Configuration | Value |
|---|---|
| AMI Name | `assignment-01-secure-batch-processing-ami` |
| AMI ID | `ami-07c987294f2bc292b` |
| Operating System | Amazon Linux 2023 |
| Instance Type | `t3.micro` |
| EC2 Name | `assignment-01-secure-batch-processor` |

### 4.2 Security Group Inbound Rules

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | `223.236.154.126/32` |
| HTTPS | TCP | 443 | `10.0.0.0/16` |
| Custom TCP | TCP | 8080 | `10.0.0.0/16` |

**Security Note:** Verify that `10.0.0.0/16` matches the actual VPC CIDR before applying these rules.

### 4.3 CloudWatch Automatic Recovery

| Configuration | Value |
|---|---|
| Alarm | `assignment-01-ec2-auto-recovery` |
| Metric | `StatusCheckFailed_System` |
| Evaluation | Two one-minute periods |
| Action | EC2 Recover |

The alarm triggers EC2 recovery for qualifying underlying infrastructure failures.

---

## 5. Task 2 - Amazon S3 Storage Tiering

### 5.1 S3 Bucket Configuration

| Configuration | Value |
|---|---|
| Bucket Name | `tanveer-assignment-01-media-datalake` |
| Region | `ap-south-1` |
| Block Public Access | Enabled |
| Versioning | Enabled |
| Encryption | SSE-S3 |
| Lifecycle Rule | `assignment-01-video-lifecycle` |

### 5.2 Lifecycle Policy

| Storage Class | Transition |
|---|---|
| S3 Standard | Initial storage |
| S3 Standard-IA | After 30 days |
| S3 Glacier Instant Retrieval | After 90 days |

**Note:** The lifecycle rule currently applies to the entire bucket. A production environment could use a dedicated `videos/` prefix.

### 5.3 Encryption

- SSE-S3: Implemented.
- SSE-KMS: Not implemented.
- Versioning: Enabled.

---

## 6. Task 3 - Serverless Thumbnail Generation

### 6.1 Lambda Configuration

| Setting | Value |
|---|---|
| Function Name | `assignment-01-thumbnail-generator` |
| Runtime | Python 3.12 |
| Architecture | x86_64 |
| Memory | 512 MB |
| Timeout | 30 seconds |
| Ephemeral Storage | 512 MB |

### 6.2 Pillow Lambda Layer

**Layer Name:** `assignment-01-pillow`

**Version:** `1`

**Layer ARN:**

```yaml
arn:aws:lambda:ap-south-1:742851379229:layer:assignment-01-pillow:1
```

### 6.3 S3 Event Notification

| Configuration | Value |
|---|---|
| Event | ObjectCreated |
| Prefix | `uploads/` |
| Suffix | `.jpg` |
| Output Prefix | `thumbnails/` |

### 6.4 Image Processing Workflow

1. Upload a JPG image to S3.
2. S3 detects the ObjectCreated event.
3. S3 asynchronously invokes Lambda.
4. Lambda downloads the uploaded image.
5. Pillow corrects EXIF orientation.
6. Image is resized to fit within 200 × 200 pixels.
7. Image is converted to RGB when required.
8. Thumbnail is saved to the S3 `thumbnails/` prefix.

**Example:**

```text
Input:
uploads/Butterfly.jpg

Output:
thumbnails/Butterfly.jpg
```

### 6.5 IAM Permissions

**Inline Policy:**

`assignment-01-lambda-s3-thumbnail-access`

| Permission | Resource |
|---|---|
| `s3:GetObject` | `uploads/*` |
| `s3:PutObject` | `thumbnails/*` |

Basic CloudWatch Logs permissions are also configured.

---

## 7. Implementation and Testing

### 7.1 Implementation Status

| Component | Status |
|---|---|
| Custom AMI Creation | Completed |
| EC2 Provisioning | Completed |
| Security Group Configuration | Configured |
| CloudWatch Recovery Alarm | Configured |
| S3 Bucket Configuration | Completed |
| S3 Versioning | Enabled |
| SSE-S3 Encryption | Enabled |
| Lifecycle Policy | Configured |
| Lambda Deployment | Completed |
| Pillow Layer | Attached |
| S3 Event Trigger | Configured |
| Thumbnail Generation | Successful |

### 7.2 End-to-End Test

A JPG image was uploaded to:

```text
uploads/Butterfly.jpg
```

The S3 event triggered Lambda, which generated the thumbnail:

```text
thumbnails/Butterfly.jpg
```

**Result:** Successful.

---

## 8. Implementation Limitations

- The legacy batch-processing application was not deployed on EC2.
- SSE-KMS was not implemented.
- SQS was not implemented.
- The lifecycle policy applies to the entire bucket rather than a video-only prefix.
- EC2 and Lambda thumbnail generation operate independently.

---

## 9. Skills Demonstrated

- AWS EC2 and AMI Management
- Security Groups and Network Security
- CloudWatch Monitoring and Recovery
- Amazon S3 Storage Management
- S3 Lifecycle Policies
- SSE-S3 Encryption and Versioning
- AWS Lambda and Python 3.12
- Lambda Layers
- S3 Event Notifications
- IAM Least-Privilege Access
- CloudWatch Logs
- Event-Driven Architecture

---

## 10. Repository Information

**Author:** Tanveer Rabbani A

**GitHub Repository:**

[AWS_Cloud_Infrastructure_Engineering-Assignments](https://github.com/tanveerrabbani5/AWS_Cloud_Infrastructure_Engineering-Assignments)

**Assignment Folder:**

[Assignment-01-Secure-Compute-Storage-Serverless](https://github.com/tanveerrabbani5/AWS_Cloud_Infrastructure_Engineering-Assignments/tree/main/Assignment-01-Secure-Compute-Storage-Serverless)

---

<div align="center">

**AWS Cloud Infrastructure Engineering**

*Assignment 01 — Secure Compute, Storage & Serverless*

**Designed and Implemented by Tanveer Rabbani A**

</div>
