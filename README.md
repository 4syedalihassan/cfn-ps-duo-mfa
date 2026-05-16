# Cisco Duo MFA for AWS Directory Service

Community-maintained fork of the deprecated [aws-ia/cfn-ps-duo-mfa](https://github.com/aws-ia/cfn-ps-duo-mfa) Quick Start.

## What changed from original

- **Base image**: Switched from `amazonlinux:latest` (Docker Hub, rate-limited) to `public.ecr.aws/ubuntu/ubuntu:22.04` (ECR Public, no rate limits)
- **CVE-2026-27459 fixed**: pyOpenSSL upgraded to 26.0.0 inside Duo authproxy bundled Python
- **buildspec.yaml**: Removed `cd docker/` — files now at repo root
- **run_duo.sh**: Windows line endings cleaned, original Duo logic preserved
- **CodeCommit → GitHub**: Source migrated to GitHub via AWS CodeConnections

## Architecture

Deploys Duo Authentication Proxy on ECS Fargate as a RADIUS server for AWS Directory Service MFA.

- CodePipeline + CodeBuild builds Docker image weekly
- Inspector2 ENHANCED scanning gates image promotion
- ECS Fargate runs patched Duo authproxy container
- Secrets Manager stores Duo credentials
- Directory Service RADIUS updated automatically via Lambda

## Files

| File | Purpose |
|------|---------|
| `Dockerfile` | Builds Duo authproxy on Ubuntu 22.04 with CVE fix |
| `run_duo.sh` | Reads secrets from ECS-injected env var, writes authproxy.cfg |
| `buildspec.yaml` | CodeBuild pipeline — build, scan, push to ECR |

## Deployment

1. Fork this repo
2. Create AWS CodeConnections connection to GitHub
3. Update CloudFormation template source to point to your fork
4. Deploy stack

## Security

- pyOpenSSL 26.0.0 (fixes CVE-2026-27459)
- Inspector2 ENHANCED scanning on every build
- Secrets Manager for credential storage
- KMS encryption on all resources

## Original

Original work by Duo Security and AWS Integration & Automation team, licensed under Apache-2.0.
