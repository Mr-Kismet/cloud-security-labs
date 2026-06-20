flAWS.cloud - Level 2: S3 Bucket with Overly Permissive IAM
Date: 2026-06-20
Objective: Use your own AWS account to access a misconfigured S3 bucket that allows authenticated access from any AWS account.

| Level       | Bucket                                                | Access Required            | Key Lesson                                                                               |
| ----------- | ----------------------------------------------------- | -------------------------- | ---------------------------------------------------------------------------------------- |
| **Level 1** | `flaws.cloud`                                         | None (`--no-sign-request`) | Public buckets expose data to everyone                                                   |
| **Level 2** | `level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud` | Any valid AWS account      | Misconfigured IAM/resource policies grant access to authenticated users indiscriminately |


Commands Used
# Configure your own AWS credentials
aws configure --profile KISMETWARE

# List bucket contents (requires authenticated AWS account)
aws s3 --profile KISMETWARE ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud

# Download the secret file
aws s3 --profile KISMETWARE cp s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/secret-e4443fc.html ./

# Read the secret
type secret-e4443fc.html


Bucket Contents Discovered:

" 2017-02-26 21:02:15         80751 everyone.png

2017-03-02 22:47:17           1433 hint1.html

2017-02-26 21:04:39           1035 hint2.html

2017-02-26 21:02:14           2786 index.html

2017-02-26 21:02:14           26 robots.txt

2017-02-26 21:02:15           1051 secret-e4443fc.html "


The Vulnerability Explained
This bucket wasn't fully public (anonymous users couldn't access it), but its resource-based policy or ACL was misconfigured to allow access from any authenticated AWS principal.
Possible misconfigurations that cause this:

| Misconfiguration                                                                 | What It Means                                |
| -------------------------------------------------------------------------------- | -------------------------------------------- |
| `Principal: "*"` with `Condition: Bool: {"aws:MultiFactorAuthPresent": "false"}` | Requires authentication but not MFA          |
| `Principal: "*"` with no conditions in bucket policy                             | Any AWS account can access                   |
| Overly permissive ACL                                                            | Grants access to "Authenticated Users" group |

AWS Best Practice: Bucket policies should specify exact AWS account IDs, IAM roles, or use conditions like aws:SourceIp and aws:PrincipalOrgID to restrict access.


Troubleshooting: Region Issues
Error encountered:
Could not connect to the endpoint URL: "https://s3.ohio.amazonaws.com/..."
Root cause: Default region (us-east-2 / Ohio) didn't match the bucket's actual region.
Fix: Reconfigured profile to us-east-1:

aws configure --profile KISMETWARE
# Default region name [ohio]: us-east-1

Lesson: S3 is global in namespace but regional in physical storage. The CLI needs to know the correct region, or it may fail to resolve the endpoint.



Cloud Security Concepts Learned
*Authenticated vs. Anonymous Access — Public (*) is worse, but "any authenticated user" is still dangerous

*Resource-Based Policies — S3 buckets can have their own policies that bypass or supplement IAM

*Cross-Account Access — AWS allows sharing resources between accounts; misconfiguration leaks data to all accounts

*Region Awareness — S3 buckets exist in specific regions; CLI commands may fail with wrong region

*Principle of Least Privilege — Never use "Principal": "*" without strict conditions




Defensive Checks

# Check bucket policy for overly permissive principals
aws s3api get-bucket-policy --bucket <bucket-name>

# Check bucket ACL
aws s3api get-bucket-acl --bucket <bucket-name>

# Check who has access (AWS IAM Access Analyzer)
aws accessanalyzer analyze-resource --resource-arn arn:aws:s3:::<bucket-name>



What to look for in a bucket policy:

// DANGEROUS - allows any AWS account

1. "Principal": "*"

2. "Action": "s3:GetObject"

3. "Resource": "arn:aws:s3:::bucket/*"



// SAFER - restricts to specific account

1. "Principal": {"AWS": "arn:aws:iam::123456789012:root"}


| Component   | Version/Details |
| ----------- | --------------- |
| OS          | Windows 11      |
| Shell       | PowerShell      |
| AWS CLI     | v2.35.9         |
| AWS Region  | us-east-1       |
| AWS Account | Free Tier       |


| Level   | Status     | Key Takeaway                                                   |
| ------- | ---------- | -------------------------------------------------------------- |
| Level 1 | ✅ Complete | Public S3 buckets expose data to everyone                      |
| Level 2 | ✅ Complete | Authenticated-but-unrestricted access is still a vulnerability |
| Level 3 | ⬜ Next     | TBD                                                            |


Status: ✅ Completed
Next Challenge: Level 3

