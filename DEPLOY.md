# Deploying in-orbit.co

The site is hosted on **AWS S3 + CloudFront** (domain via Route 53). This repo
auto-deploys to it on every push to `main` via `.github/workflows/deploy.yml`.

The workflow stays dormant (skips cleanly) until the 5 secrets below exist.

## One-time setup (developer)

1. Create an IAM user for CI (or reuse a deploy role) with this minimal policy,
   substituting the real bucket name and CloudFront distribution ARN:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       { "Effect": "Allow",
         "Action": ["s3:PutObject","s3:DeleteObject","s3:ListBucket","s3:GetObject"],
         "Resource": ["arn:aws:s3:::YOUR_BUCKET","arn:aws:s3:::YOUR_BUCKET/*"] },
       { "Effect": "Allow",
         "Action": ["cloudfront:CreateInvalidation"],
         "Resource": "arn:aws:cloudfront::ACCOUNT_ID:distribution/YOUR_DIST_ID" }
     ]
   }
   ```

2. In GitHub: **Settings -> Secrets and variables -> Actions -> New repository
   secret**, add all five:

   | Secret name                  | Value                                  |
   |------------------------------|----------------------------------------|
   | `AWS_ACCESS_KEY_ID`          | the IAM user's access key id           |
   | `AWS_SECRET_ACCESS_KEY`      | the IAM user's secret access key        |
   | `AWS_REGION`                 | the bucket's region, e.g. `eu-west-2`  |
   | `S3_BUCKET`                  | the bucket name (no `s3://`, no slash) |
   | `CLOUDFRONT_DISTRIBUTION_ID` | the CloudFront distribution id          |

3. Done. The next push to `main` (or **Actions -> Deploy to AWS -> Run workflow**)
   syncs the repo to the bucket and invalidates the CloudFront cache.

## Notes
- The sync does **not** use `--delete`, so it won't remove bucket objects that
  aren't in the repo. Add `--delete` in the workflow if you want a strict mirror.
- Prefer OIDC over long-lived keys? Swap the credentials step for
  `role-to-assume` and drop the two AWS_* key secrets — the rest is unchanged.
- The site is fully self-contained: `index.html` + `vendor/` (self-hosted libs)
  + `ev/` (event logos). No build step.
