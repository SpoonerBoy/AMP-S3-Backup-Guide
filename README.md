# AMP-S3-Backup-Guide
Guide for setting up AWS S3 backups in CubeCoders AMP
# AMP S3 Backup Guide
A guide for configuring AWS S3 cloud backups in CubeCoders AMP.

## Prerequisites
- An AWS account
- A running AMP instance (tested on AMP Release "Deimos")

## Setup Steps

### 1. Create an S3 Bucket
- Log into AWS and go to **S3** → **Create bucket**
- Accept all defaults, keep **Block all public access** ON
- Note your **bucket name** and **region**

### 2. Create an IAM User
- Go to **IAM** → **Users** → **Create user**
- Name it something like `amp-backup`
- Go to **Security credentials** → **Create access key**
- Select **Application running outside AWS**
- Note your **Access key** and **Secret access key**

### 3. Set IAM Permissions
Attach the following custom policy for least-privilege access
(replace `your-bucket-name` with your actual bucket name):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ]
    }
  ]
}
```

> Alternatively you can attach the managed `AmazonS3FullAccess` policy for
> a quicker setup, but the custom policy above is recommended for security.

### 4. Configure AMP Cloud Backups
In AMP, go to your instance → **Backups** → **Cloud** tab and fill in:

| Field | Value |
|---|---|
| Use S3 Storage for Backups | ON |
| S3 Service URL | `https://s3.your-region.amazonaws.com` |
| S3 Authentication Region | Your region (e.g. `us-east-2`) |
| S3 Bucket Name | Your bucket name |
| S3 Access Key | Your IAM access key |
| S3 Secret Key | Your IAM secret key |
| Force Path Style | OFF |
| S3 Storage Class | Standard |

### 5. Test It
Trigger a manual backup from the **Backups** section and confirm
the file appears in your S3 bucket.

## Optional — S3 Lifecycle Rule (Recommended)
To avoid backups accumulating forever and growing your AWS bill:
1. Go to your S3 bucket → **Management** → **Create lifecycle rule**
2. Name it `expire-old-backups`, apply to all objects
3. Enable **Expire current versions of objects** → set to **30 days**
4. Save

## Notes
- The S3 Service URL can be left blank if AMP auto-detects your region
- AMP's "Delete Single Oldest" replacement policy manages count limits,
  but the S3 lifecycle rule handles age-based cleanup on the AWS side
