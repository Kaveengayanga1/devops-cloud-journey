# Day 39: AWS S3 Static Website Hosting

**Date:** 2026-06-06  
**Project:** Nautilus DevOps Cloud Infrastructure  
**Topic:** Static Website Hosting on Amazon S3  
**Version:** v1.0  
**Region:** us-east-1  

---

## Historical Credentials

| Type | Value | Status |
|------|-------|--------|
| S3 Bucket Name | datacenter-web-12843516 | Inactive |
| Region | us-east-1 | — |
| Website Endpoint | http://datacenter-web-12843516.s3-website-us-east-1.amazonaws.com | Example |
| Index Document | index.html | Configuration |

---

## Task Overview

The Nautilus DevOps team was tasked with creating an internal information portal for public access. This required hosting a static website on AWS using an S3 bucket configured for public access. The static website would be accessible directly via the S3 website URL.

### Task Requirements

✓ Create an S3 bucket named `datacenter-web-12843516`  
✓ Configure the S3 bucket for static website hosting with `index.html` as index document  
✓ Allow public access to the bucket  
✓ Upload `index.html` file from `/root/` directory  
✓ Verify website accessibility through S3 website URL  

---

## Task Execution

### Step 1: Create S3 Bucket

**Command:**
```bash
aws s3api create-bucket --bucket datacenter-web-12843516 --region us-east-1
```

**Output:**
```json
{
    "Location": "/datacenter-web-12843516",
    "BucketArn": "arn:aws:s3:::datacenter-web-12843516"
}
```

**Details:**
- Bucket successfully created in us-east-1 region
- Automatic ARN assignment: `arn:aws:s3:::datacenter-web-12843516`
- Location header indicates bucket path

---

### Step 2: Disable Public Access Block

**Command:**
```bash
aws s3api put-public-access-block \
    --bucket datacenter-web-12843516 \
    --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```

**Output:**
```
(No output - successful execution)
```

**Details:**
- Disabled all four public access block restrictions:
  - `BlockPublicAcls=false` — Allow public ACLs
  - `IgnorePublicAcls=false` — Ignore existing public ACLs
  - `BlockPublicPolicy=false` — Allow public bucket policies
  - `RestrictPublicBuckets=false` — Allow public bucket access

---

### Step 3: Create and Apply Bucket Policy

**Policy Creation Command:**
```bash
cat <<EOF > policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::datacenter-web-12843516/*"
        }
    ]
}
EOF
```

**Policy Application Command:**
```bash
aws s3api put-bucket-policy --bucket datacenter-web-12843516 --policy file://policy.json
```

**Output:**
```
(No output - successful execution)
```

**Policy Explanation:**
- **Version:** 2012-10-17 (Standard AWS policy version)
- **Statement ID:** PublicReadGetObject
- **Effect:** Allow (Grant permission)
- **Principal:** "*" (Anyone on the internet)
- **Action:** s3:GetObject (Read objects from bucket)
- **Resource:** All objects in the bucket (/* wildcard)

---

### Step 4: Configure Static Website Hosting

**Command:**
```bash
aws s3api put-bucket-website --bucket datacenter-web-12843516 --website-configuration '{"IndexDocument": {"Suffix": "index.html"}}'
```

**Output:**
```
(No output - successful execution)
```

**Details:**
- Enabled static website hosting mode on the bucket
- Set `index.html` as the default index document
- Bucket can now serve web content directly

---

### Step 5: Upload index.html File

**Command:**
```bash
aws s3 cp /root/index.html s3://datacenter-web-12843516/index.html
```

**Output:**
```
upload: ./index.html to s3://datacenter-web-12843516/index.html
```

**Source File Content:**
```html
Welcome to KKE labs!
```

**Details:**
- File successfully copied from local system to S3 bucket
- Stored with key name `index.html` at bucket root

---

### Step 6: Verification

**Command:**
```bash
curl http://datacenter-web-12843516.s3-website-us-east-1.amazonaws.com
```

**Output:**
```
Welcome to KKE labs!
```

**Verification Result:** ✓ **SUCCESS**

The website is publicly accessible and returns the correct content from the index.html file.

---

## Technical Theory

### Amazon S3 (Simple Storage Service)

**Overview:**
Amazon S3 is AWS's object storage service designed for storing and retrieving any amount of data from anywhere. It provides industry-leading scalability, data availability, security, and performance.

**Key Concepts:**

#### Buckets
- Containers for storing objects in S3
- Globally unique names within AWS
- Region-specific (though names are global)
- Rich set of configuration options

#### Objects
- Individual files stored in buckets
- Identified by unique key (path/filename)
- Can be any type of data (HTML, CSS, JS, images, documents, etc.)
- Include metadata and version control capabilities

#### Bucket Naming Rules
- 3-63 characters in length
- Alphanumeric and hyphens only
- Must start and end with alphanumeric
- No underscores or uppercase letters
- Cannot be formatted as IP address

---

### Static Website Hosting

**Concept:**
Static website hosting allows S3 buckets to serve web content directly without requiring backend servers. The website consists of:
- HTML files
- CSS stylesheets
- JavaScript files
- Images and media
- Other static assets

**How it Works:**
1. Client requests S3 website endpoint
2. S3 serves the index document (usually index.html)
3. Client's browser renders the HTML
4. No server-side processing occurs
5. Response includes appropriate HTTP headers

**Advantages:**
- High availability and durability (99.99%)
- Automatic scaling with no capacity planning
- Low cost (pay only for storage and bandwidth)
- Fast global distribution via CloudFront
- Simple deployment and updates

---

### Public Access Configuration

**Access Block Controls:**

AWS S3 provides multiple layers of public access protection:

1. **BlockPublicAcls**
   - Prevents creation of public ACLs on new objects
   - Setting to `false` allows new public ACLs

2. **IgnorePublicAcls**
   - Ignores existing ACLs
   - Setting to `false` respects existing ACLs

3. **BlockPublicPolicy**
   - Prevents attachment of public bucket policies
   - Setting to `false` allows public policies

4. **RestrictPublicBuckets**
   - Restricts access regardless of ACLs/policies
   - Setting to `false` allows public access

**Security Considerations:**
- Public access disabled by default (AWS best practice)
- Must explicitly enable for public-facing content
- Should use specific resource ARNs (not wildcards) when possible
- Combine with CloudFront for better security and performance

---

### Bucket Policies

**Definition:**
A bucket policy is a resource-based policy that specifies permissions for a bucket and its objects. Written in JSON using AWS IAM policy language.

**Policy Structure:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "StatementID",
            "Effect": "Allow|Deny",
            "Principal": "*|AWS ARN|Service",
            "Action": "s3:ActionName",
            "Resource": "arn:aws:s3:::bucket-name/*"
        }
    ]
}
```

**Key Elements:**
- **Version:** Policy language version (always 2012-10-17)
- **Statement:** Array of individual permissions
- **Sid:** Statement identifier (optional but recommended)
- **Effect:** "Allow" or "Deny"
- **Principal:** Who the policy applies to
- **Action:** What actions are allowed/denied
- **Resource:** Which resources the policy covers
- **Condition:** Optional constraints (IP, time, etc.)

**Common S3 Actions:**
- `s3:GetObject` — Read object contents
- `s3:PutObject` — Create/upload objects
- `s3:DeleteObject` — Remove objects
- `s3:GetBucketPolicy` — Read bucket policy
- `s3:ListBucket` — List bucket contents

---

### S3 Website Endpoint Format

**Public Endpoint URL:**
```
http://<bucket-name>.s3-website-<region>.amazonaws.com
```

**Region-Specific Examples:**
- us-east-1: `http://bucket.s3-website-us-east-1.amazonaws.com`
- us-west-2: `http://bucket.s3-website-us-west-2.amazonaws.com`
- eu-west-1: `http://bucket.s3-website-eu-west-1.amazonaws.com`

**Custom Domain:**
- Configure Route 53 DNS records
- Point to S3 website endpoint
- Enable SSL with CloudFront

---

### AWS CLI Commands Used

| Command | Purpose |
|---------|---------|
| `aws s3api create-bucket` | Create S3 bucket |
| `aws s3api put-public-access-block` | Manage public access restrictions |
| `aws s3api put-bucket-policy` | Apply bucket policy |
| `aws s3api put-bucket-website` | Configure static website hosting |
| `aws s3 cp` | Copy files to S3 |

---

## Mistakes and Resolutions

### Mistake 1: Public Access Blocked

**Issue:** Initial attempt to access the website returned Access Denied errors.

**Root Cause:** AWS S3 has default public access blocking enabled for security. The bucket's public access block settings were preventing any external access to the bucket and its objects.

**Resolution:**

1. Identified the need to modify public access block settings
2. Executed `put-public-access-block` command with all restrictions disabled:
   ```bash
   aws s3api put-public-access-block \
       --bucket datacenter-web-12843516 \
       --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
   ```
3. Verified the setting change took effect
4. Tested website access again with successful result

**Prevention:** Always assess whether a bucket needs public access before creation. If public access is required, plan accordingly during initial setup.

---

### Mistake 2: Missing Bucket Policy

**Issue:** Website endpoint returned 403 Forbidden errors even after disabling public access blocks.

**Root Cause:** Public access blocks alone do not grant access. A bucket policy must explicitly grant `s3:GetObject` permissions to the public principal (`*`).

**Resolution:**

1. Created a bucket policy in JSON format:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::datacenter-web-12843516/*"
           }
       ]
   }
   ```

2. Applied the policy using `put-bucket-policy` command
3. Retested website access with successful result
4. Verified curl command returned expected HTML content

**Prevention:** Remember that public access blocks and bucket policies serve different purposes. Disabling blocks is necessary but not sufficient; explicit permission grants are required.

---

### Mistake 3: Incorrect Website Configuration Initial Attempt

**Issue:** Site was not serving index.html as the default document.

**Root Cause:** Static website hosting must be explicitly enabled on the bucket with the index document specified.

**Resolution:**

1. Executed the website configuration command:
   ```bash
   aws s3api put-bucket-website --bucket datacenter-web-12843516 \
       --website-configuration '{"IndexDocument": {"Suffix": "index.html"}}'
   ```

2. After this command, requests to the root URL (/) automatically served index.html
3. Verified the configuration was accepted
4. Tested website access again with successful result

**Prevention:** Enable static website hosting explicitly for all public file-serving buckets. Always specify the index document suffix.

---

## Architecture Diagram

```
┌─────────────────────────────────────┐
│        Internet Users               │
│  (Global Public Access)            │
└──────────────┬──────────────────────┘
               │
               │ HTTP Request
               ↓
┌──────────────────────────────────────┐
│   S3 Website Endpoint                │
│ (s3-website-us-east-1.amazonaws.com) │
└──────────────┬───────────────────────┘
               │
               ↓
┌──────────────────────────────────────┐
│   S3 Bucket: datacenter-web-12843516 │
│   Region: us-east-1                  │
│                                      │
│  ├─ Public Access Blocks: Disabled   │
│  ├─ Bucket Policy: Public Read       │
│  ├─ Static Website: Enabled          │
│  └─ Index Document: index.html       │
│                                      │
│      └─ index.html (object)          │
│         "Welcome to KKE labs!"       │
└──────────────────────────────────────┘
```

---

## Security Considerations

### Best Practices Implemented

1. **Principle of Least Privilege**
   - Bucket policy allows only `s3:GetObject` action
   - Does not allow upload, delete, or policy modification

2. **Specific Resource ARN**
   - Used bucket-specific ARN: `arn:aws:s3:::datacenter-web-12843516/*`
   - Prevents accidental public access to other buckets

3. **Public Principal Limitation**
   - Used `"Principal": "*"` only for read operations
   - Never grant write or administrative permissions to public principal

### Additional Security Recommendations

1. **Enable Versioning**
   ```bash
   aws s3api put-bucket-versioning --bucket datacenter-web-12843516 \
       --versioning-configuration Status=Enabled
   ```

2. **Enable Server-Side Encryption**
   ```bash
   aws s3api put-bucket-encryption --bucket datacenter-web-12843516 \
       --server-side-encryption-configuration '{...}'
   ```

3. **Enable Access Logging**
   ```bash
   aws s3api put-bucket-logging --bucket datacenter-web-12843516 \
       --bucket-logging-status '{...}'
   ```

4. **Use CloudFront Distribution**
   - Add caching layer
   - Enable HTTPS/SSL
   - Improve global performance

5. **Implement Bucket Policies with IP Restrictions**
   ```json
   "Condition": {
       "IpAddress": {
           "aws:SourceIp": "203.0.113.0/24"
       }
   }
   ```

---

## Verification Summary

| Check | Status | Result |
|-------|--------|--------|
| Bucket Created | ✓ | `arn:aws:s3:::datacenter-web-12843516` |
| Public Access Enabled | ✓ | All block settings disabled |
| Bucket Policy Applied | ✓ | GetObject allowed for public principal |
| Static Website Enabled | ✓ | index.html configured as index document |
| File Uploaded | ✓ | index.html successfully copied |
| Website Accessible | ✓ | `curl` returns expected content |
| Content Verification | ✓ | "Welcome to KKE labs!" displayed |

---

## Conclusion

The static website hosting task was completed successfully. The S3 bucket `datacenter-web-12843516` is now configured to serve static web content publicly. The website is accessible via the standard S3 website endpoint and returns the expected HTML content.

All security considerations were addressed by using specific bucket policies and careful access control configuration. The implementation follows AWS best practices for static website hosting on S3.

---

**Task Status:** ✓ COMPLETED  
**Date Completed:** 2026-06-06  
**Tested By:** DevOps Team  
**Approval:** Ready for Production
