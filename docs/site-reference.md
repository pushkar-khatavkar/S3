# Site Reference

Details about the CloudKida demo site in this repo — the files you upload during the
[hands-on lab](../README.md), plus what to do after the booth.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Landing page: hero, learning tracks, popular labs, how it works |
| `login.html` | Branded learner sign-in page (demo form, no backend) |
| `about.html` | About the platform |
| `error.html` | Custom 404 / error document |
| `style.css` | Shared stylesheet with the CloudKida theme |
| `favicon.svg` | CloudKida cloud logo used as the favicon |
| `s3-static-website.svg` | Hero illustration of the AWS services used in the labs |
| `bucket-policy.json` | Ready-made public-read bucket policy |

## Preview locally

Any static file server works. With Python:

```bash
python3 -m http.server 8080
```

Then open <http://localhost:8080>.

## Deploy with the AWS CLI

The booth lab uses the console. If you prefer the CLI, the whole thing is four commands.
Replace `my-bucket-name` with your own globally unique bucket name.

```bash
# 1. Create the bucket
aws s3 mb s3://my-bucket-name --region us-east-1

# 2. Enable static website hosting
aws s3 website s3://my-bucket-name \
  --index-document index.html \
  --error-document error.html

# 3. Allow public read (after disabling Block Public Access on the bucket)
aws s3api put-bucket-policy \
  --bucket my-bucket-name \
  --policy file://bucket-policy.json

# 4. Upload the site
aws s3 sync . s3://my-bucket-name \
  --exclude ".git/*" \
  --exclude "docs/*" \
  --exclude "images/*" \
  --exclude "README.md"
```

Then visit:

```
http://my-bucket-name.s3-website-us-east-1.amazonaws.com
```

> `bucket-policy.json` ships with `cloudkida.com` as the bucket name. Edit the `Resource`
> ARN to match your bucket before running step 3.

## Custom domain + HTTPS

An S3 website endpoint is HTTP only. For `https://your-domain.com`:

1. Put **Amazon CloudFront** in front of the S3 website endpoint.
2. Request a free TLS certificate in **AWS Certificate Manager** (must be in `us-east-1`).
3. Point your **Route 53** (or other DNS provider) record at the CloudFront distribution.

If you want to use a custom domain with the plain S3 website endpoint, name the bucket
exactly like the domain, e.g. `cloudkida.com`.

## Notes

- The login form is a front-end demo only. Static sites can't authenticate on their own —
  wire it up to Amazon Cognito, API Gateway + Lambda, or another backend for real sign-in.

---

© CloudKida — [cloudkida.com](https://cloudkida.com)
