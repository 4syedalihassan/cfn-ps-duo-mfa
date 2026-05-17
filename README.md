# Cisco Duo MFA for AWS Directory Service

Community-maintained fork of the deprecated [aws-ia/cfn-ps-duo-mfa](https://github.com/aws-ia/cfn-ps-duo-mfa) Quick Start.

## Why this fork exists

The original AWS Quick Start was deprecated in Q4 2024. AWS stopped maintaining it, the S3 assets became unavailable, and no CVE fixes were applied. This fork addresses all of that.

## What changed from original

| Component | Change |
|-----------|--------|
| Runtime base image | `amazonlinux:latest` (Docker Hub, rate-limited) → `public.ecr.aws/ubuntu/ubuntu:24.04` (ECR Public) |
| Automation base images | `public.ecr.aws/codebuild/amazonlinux2-x86_64-standard:4.0` → `public.ecr.aws/codebuild/amazonlinux2-x86_64-standard:5.0` |
| CVE-2026-27459 | pyOpenSSL upgraded to 26.0.0 inside Duo's bundled Python |
| Source control | CodeCommit (deprecated) → GitHub via AWS CodeConnections |
| buildspec.yaml | Removed `cd docker/` — pipeline now builds from `scripts/source` root |
| run_duo.sh | Windows line endings cleaned, original Duo logic preserved |
| CF template | Updated parameters, pipeline source, IAM permissions |

## Architecture

- CodePipeline + CodeBuild builds Docker image weekly (or on push)
- Inspector2 ENHANCED scanning gates image promotion to `:latest`
- ECS Fargate runs patched Duo authproxy container
- Secrets Manager stores Duo credentials
- Directory Service RADIUS updated automatically via Lambda

## Repository structure

```
├── scripts/source/docker/Dockerfile   # Ubuntu 24.04, Duo authproxy, CVE fix
├── scripts/source/docker/run_duo.sh   # Reads DuoSecret env var, writes authproxy.cfg
├── scripts/source/buildspec.yaml      # CodeBuild pipeline — build, scan, push to ECR
└── templates/
    └── duo-proxy-fargate.template.yaml   # Main CFN template
```

## Deployment

### Prerequisites

1. AWS account with Directory Service (AD Connector or Managed AD)
2. Duo account with RADIUS application configured
3. GitHub account

### Steps

**1. Fork this repo to your GitHub account**

**2. Create AWS CodeConnections connection**
- AWS Console → Developer Tools → Connections → Create connection
- Select GitHub → Authorize → Save
- Copy the Connection ARN

**3. Deploy CloudFormation stack**
- Upload `templates/duo-proxy-fargate.template.yaml`
- Fill in parameters:

| Parameter | Value |
|-----------|-------|
| `GitHubRepo` | `cfn-ps-duo-mfa` |
| `GitHubBranch` | `main` |
| `GitHubConnectionArn` | ARN from step 2 |
| `DuoIntegrationKey` | From Duo admin console |
| `DuoSecretKey` | From Duo admin console |
| `DuoApiHostName` | From Duo admin console |
| `DirectoryServiceId` | e.g. `d-xxxxxxxxxx` |
| `DirectoryServiceType` | `AD Connector` or `Managed AD` |
| `NotificationEmail` | Admin email for alerts |

**4. Verify deployment**
- CloudFormation → stack → `CREATE_COMPLETE`
- Directory Service console → your directory → Networking & security → MFA status = `Completed`
- WorkSpaces login → MFA prompt should appear

## Security

- pyOpenSSL 26.0.0 (fixes CVE-2026-27459 — DTLS buffer overflow)
- Inspector2 ENHANCED scanning on every build — blocks CRITICAL CVEs
- Secrets Manager for all Duo credentials
- KMS encryption on all resources
- Secret auto-rotation every 7 days

## Updating

Push to `main` branch → CodePipeline auto-triggers → new image built and scanned → ECS auto-deploys.

Weekly rebuild also runs every Saturday to pick up base image patches.

## Troubleshooting

**Pipeline fails — CVE found**
- Check Inspector2 findings for `duo-authproxy` repo
- If CVE is in Duo's bundled Python, open issue here — we'll update pyOpenSSL or base image

**ECS task exits**
- Check CloudWatch logs: `RadiusProxyLogs-<directory-id>/authproxy.log`
- Verify `DuoSecret` in Secrets Manager has correct key names

**RADIUS status Failed**
- Verify ECS tasks are running and have valid IPs
- Check security group allows UDP 1812 from Directory Service

## License

Apache-2.0 — original work by Duo Security and AWS Integration & Automation team.
Modifications by this fork's maintainers.
