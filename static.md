# CloudKida — Hands-on Labs Platform (AWS S3 Static Site)

Marketing/landing site for **CloudKida** ([cloudkida.com](https://cloudkida.com)), a
hands-on learning platform where learners practice **AWS, DevOps, Linux** and more in
real, guided labs. This static site is ready to host on Amazon S3.

## 📁 Files

| File | Purpose |
|------|---------|
| `index.html` | Landing page: hero, learning tracks, popular labs, how it works |
| `login.html` | Branded learner sign-in page (demo form, no backend) |
| `about.html` | About the platform |
| `error.html` | Custom 404 / error document |
| `style.css` | Shared stylesheet with the CloudKida theme |
| `favicon.svg` | CloudKida cloud logo used as the favicon |
| `s3-static-website.svg` | Hero illustration of the AWS services used in the labs (S3, EC2, Lambda, IAM, CloudFront, Terraform) |

## 🚀 Deploy to AWS S3

### 1. Create the bucket

For a custom domain, name the bucket exactly like the domain, e.g. `cloudkida.com`.

```bash
aws s3 mb s3://cloudkida.com --region us-east-1
```

### 2. Enable static website hosting

```bash
aws s3 website s3://cloudkida.com \
  --index-document index.html \
  --error-document error.html
```

### 3. Allow public read access

Disable "Block Public Access" for the bucket, then attach this bucket policy
(replace `cloudkida.com` with your bucket name):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudkida.com/*"
    }
  ]
}
```

```bash
aws s3api put-bucket-policy --bucket cloudkida.com --policy file://bucket-policy.json
```

### 4. Upload the site

```bash
aws s3 sync . s3://cloudkida.com \
  --exclude ".git/*" \
  --exclude "README.md"
```

### 5. Visit the site

```
http://cloudkida.com.s3-website-us-east-1.amazonaws.com
```

## 🌐 Custom domain + HTTPS (recommended)

For `https://cloudkida.com` with a valid certificate:

1. Put **Amazon CloudFront** in front of the S3 website endpoint.
2. Request a free TLS certificate in **AWS Certificate Manager** (in `us-east-1`).
3. Point your **Route 53** (or DNS provider) record at the CloudFront distribution.

## 🖥️ Preview locally

Any static file server works. For example, with Python:

```bash
python3 -m http.server 8080
```

Then open <http://localhost:8080>.

## 📝 Notes

- The login form is a front-end demo only. Static sites can't authenticate on their
  own — wire it up to Amazon Cognito, an API Gateway + Lambda, or another backend to
  enable real sign-in.

---

© CloudKida — [cloudkida.com](https://cloudkida.com)
