flAWS.cloud - Level 1: S3 Bucket Misconfiguration

Date: 2026-06-20

Objective: Discover and exploit a publicly accessible S3 bucket to find the secret for Level 2.


Environment Setup
| Component | Version/Details                                  |
| --------- | ------------------------------------------------ |
| OS        | Windows 11                                       |
| AWS CLI   | v2.35.9                                          |
| Shell     | PowerShell                                       |
| Challenge | [flAWS.cloud](http://flaws.cloud) by Scott Piper |


Commands Used
# Verify AWS CLI installation
aws --version
# Output: aws-cli/2.35.9 Python/3.14.5 Windows/11 exe/AMD64

# List bucket contents anonymously (no AWS credentials required)
aws s3 ls s3://flaws.cloud/ --no-sign-request

# Download the secret file
aws s3 cp s3://flaws.cloud/secret-dd02c7c.html ./ --no-sign-request

# Read the secret file contents
type secret-dd02c7c.html



Bucket Contents Discovered

2017-03-13 23:00:38       2575 hint1.html

2017-03-02 23:05:17       1707 hint2.html

2017-03-02 23:05:11       1101 hint3.html

2024-02-21 21:32:41       2861 index.html
2018-07-10 12:47:16      15979 logo.png

2017-02-26 20:59:28         46 robots.txt

2017-02-26 20:59:30       1051 secret-dd02c7c.html








Key Finding


The S3 bucket flaws.cloud was configured with public read access, allowing anyone to:

*List objects (s3:ListBucket) — see all files

*Get objects (s3:GetObject) — download any file

This was done without authentication using the --no-sign-request flag.



The Vulnerability Explained
| Misconfiguration               | Risk                              | Real-World Impact                   |
| ------------------------------ | --------------------------------- | ----------------------------------- |
| Public `ListBucket` permission | Attackers can enumerate all files | Data inventory for targeted theft   |
| Public `GetObject` permission  | Anyone can download data          | Data breaches, PII exposure         |
| No authentication required     | Zero barrier to entry             | Automated scanning and exploitation |


AWS Best Practice: S3 buckets should default to private. Public access should only be granted intentionally using:

1. Bucket policies (with specific IP restrictions when possible)
2. IAM policies (for authenticated users only)
3. S3 Block Public Access settings (enabled by default)



How to Check for This Vulnerability (Defensive)

# Check bucket ACL
aws s3api get-bucket-acl --bucket <bucket-name>

# Check bucket policy
aws s3api get-bucket-policy --bucket <bucket-name>

# Check public access block settings
aws s3api get-public-access-block --bucket <bucket-name>







Expected secure output for public access block:


{
    "PublicAccessBlockConfiguration": 
    
    {
    "BlockPublicAcls": true,
    "IgnorePublicAcls": true,
    "BlockPublicPolicy": true,
    "RestrictPublicBuckets": true }  }







AWS CLI Commands Reference

| Command                             | Purpose                          |
| ----------------------------------- | -------------------------------- |
| `aws s3 ls`                         | List all buckets in your account |
| `aws s3 ls s3://bucket/`            | List objects in a bucket         |
| `aws s3 cp s3://bucket/file ./`     | Download a file                  |
| `aws s3 sync s3://bucket/ ./local/` | Sync entire bucket locally       |
| `aws s3 mb s3://new-bucket`         | Make (create) a new bucket       |





Cloud Security Concepts Learned

1. Shared Responsibility Model — AWS secures the infrastructure; you secure your data and configurations
2. Principle of Least Privilege — Grant only necessary permissions
3. S3 Bucket Policies vs. IAM Policies — Resource-based vs. identity-based access control
4. Public Access Block — AWS's safety net to prevent accidental public exposure
5. Enumeration as Attack Vector — Just listing files can reveal sensitive information



Additional Resources
| Resource                       | Link                                                                                                    |
| ------------------------------ | ------------------------------------------------------------------------------------------------------- |
| flAWS.cloud Challenge          | <http://flaws.cloud>                                                                                    |
| AWS S3 Security Best Practices | [AWS Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html) |
| S3 Bucket Policy Examples      | [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html)                            |
| CloudGoat (AWS Vulnerable Lab) | [GitHub - RhinoSecurityLabs](https://github.com/RhinoSecurityLabs/cloudgoat)                            |
| Scott Piper's Blog             | [Summit Route](https://summitroute.com/)                                                                |




Next Steps

[ ] Complete flAWS.cloud Level 2

[ ] Set up AWS Free Tier account for hands-on practice

[ ] Explore CloudGoat for more AWS security scenarios

[ ] Study for AWS Certified Security - Specialty







Status: ✅ Completed

Next Challenge: flAWS.cloud Level 2



