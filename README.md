# Infrastructure Pipeline (Multi-Cloud GitOps)

## Overview

This repository provides a **centralized infrastructure pipeline** designed for **multi-cloud deployments** (AWS, Azure, GCP) using GitOps principles. 

Key features:

- **Centralized security & compliance scanning** using Cortex Cloud AppSec for Infrastructure-as-Code (IaC) scan.
- **Cloud-specific deployment workflows** triggered automatically based on the cloud provider defined in the source IaC repository.
- **Hybrid workflow architecture**:
  - A **central workflow** performs security scanning with Cortex Cloud.
  - **Cloud-specific workflows** handle deployment logic and cloud credentials.

This architecture enables DevOps and Security teams (DevSecOps) to enforce security policies **before deploying any infrastructure**.

---

## Workflow Architecture

1. **IaC Repository (Source)**
   - Contains Terraform code for cloud resources.
   - Upon merging a PR to `main`, a `repository_dispatch` event is sent to this pipeline repository.
   - The event payload includes:
     - `cloud`: `"aws"`, `"azure"`, or `"gcp"`
     - `repo`: source IaC repository
     - `ref`: branch reference (usually `main`)

2. **Central Dispatch Workflow (`dispatch.yml`)**
   - Triggered by `repository_dispatch`.
   - Performs **security & compliance scan** on the IaC code using Cortex Cloud AppSec.
   - If scan passes, triggers the **cloud-specific workflow**.

3. **Cloud-Specific Workflows**
   - Each workflow calls the **reusable `scan-and-deploy.yml` workflow**.
   - Performs Terraform init and apply using cloud-specific secrets.

---

## Secrets Required

| Secret Name        | Description |
|-------------------|-------------|
| `GH_PAT`           | GitHub PAT for cross-repo checkout & workflow dispatch |
| `CORTEX_API_KEY_ID`     | Cortex Cloud API Key ID|
| `CORTEX_API_KEY`     | Cortex Cloud API Key |
| `AWS_ACCESS_KEY_ID` | AWS credentials for deployment |
| `AWS_SECRET_ACCESS_KEY` | AWS credentials |
| `AZURE_CLIENT_ID`  | Azure service principal |
| `AZURE_TENANT_ID`  | Azure tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure subscription ID |
| `GCP_WIP`          | GCP Workload Identity Provider |
| `GCP_SA_EMAIL`     | GCP Service Account email |
| `GCP_PROJECT`      | GCP Project ID |


---

## How It Works

1. PR is merged in the IaC repository.
2. The repository dispatch sends metadata (cloud, repo, ref) to the **central pipeline**.
3. The **dispatch workflow** performs **security/compliance scanning**.
4. If scan passes, the cloud-specific workflow is triggered.
5. Terraform deploys resources in the selected cloud.

---

## Advantages

- **Security First**: Prevents insecure infrastructure deployment via IaC scan.
- **Cloud-Agnostic**: Single pipeline handles multiple clouds.
- **Centralized Management**: Easier auditing, versioning, and monitoring.
- **GitOps Friendly**: IaC repositories trigger deployment automatically.

