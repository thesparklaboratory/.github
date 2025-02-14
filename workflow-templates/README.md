# GitHub Reusable Workflows

## Overview

This repository provides documentation for our GitHub reusable workflows. Reusable workflows allow you to define common sets of actions and steps that can be referenced across multiple workflows, promoting consistency and reducing duplication in CI/CD pipelines.

For more information on creating and using reusable workflows, please refer to:
- [Reusing workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Table of Contents

- [Pulumi TypeScript API Deployment](#pulumi-typescript-api-deployment) - Workflow for deploying a Pulumi TypeScript API project, handling both application build and infrastructure provisioning.

## Pulumi TypeScript Deployment

### Overview

The **Pulumi TypeScript API Deployment** workflow is designed for Pulumi-based TypeScript projects. It triggers on release publications and pushes to the `main` and `next` branches. The workflow performs the following actions:
- Checks out the repository and sets up Node.js.
- Configures deployment environment variables based on the branch or tag.
- Sets up AWS credentials.
- Installs dependencies and builds the API.
- Configures and runs Pulumi to deploy infrastructure.
- Manages Prisma commands for generating and migrating database schemas.

### Requirements and Options

This workflow requires the following secrets to be set in your repository's settings:

| Name                     | Type   | Required? | Description                                                       |
|--------------------------|--------|:---------:|-------------------------------------------------------------------|
| `AWS_ACCESS_KEY_ID`      | secret | ✅        | AWS access key ID for deploying infrastructure.                   |
| `AWS_REGION`             | input  | ✅        | AWS region for deploying infrastructure.                          |
| `AWS_SECRET_ACCESS_KEY`  | secret | ✅        | AWS secret access key for deploying infrastructure.               |
| `GH_TOKEN`               | secret | ✅        | `secrets.GITHUB_TOKEN` used for commenting on PRs.                |
| `PULUMI_PASSPHRASE`      | secret | ✅        | Passphrase for Pulumi configuration encryption.                   |
| `PULUMI_S3_BUCKET`       | input  | ✅        | S3 bucket used for storing Pulumi state.                          |

### Using the Reusable Workflow

- **Triggers:**
  - **Release:** Triggered when a release is published.
  - **Push:** Triggered on pushes to the `main` and `next` branches.
  - **Workflow Dispatch:** Triggered if you need to manually deploy a branch.

- **Environment Variables:**
  These are dynamically set during the workflow execution:
  - `REF_SHORT_SHA`: A shortened version of the commit SHA.
  - `PULUMI_STACK`: Determined based on the event:
    - `prod` for tag events.
    - `qa` for pushes to `main`.
    - `dev` for pushes to `next`.

- **Usage:**
  ```
  jobs:
    pulumiTypescriptAPIDeployment:
      uses: thesparklaboratory/.github/workflow-templates/pulumi-typescript-api-deployment.yml@main
      with:
        AWS_REGION: us-east-2
        PULUMI_S3_BUCKET: pulumi-s3-bucket
      secrets:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        PULUMI_PASSPHRASE: ${{ secrets.PULUMI_PASSPHRASE }}
  ```

### Step-by-Step Process

1. **Checkout and Node Setup:**
   The workflow begins by checking out the repository and setting up Node.js with Yarn caching and the Node version specified in `.nvmrc`.

2. **Deployment Environment Configuration:**
   Environment variables are set for the deployment context, including the short commit SHA and the Pulumi stack name based on branch or tag.

3. **AWS Credentials:**
   AWS credentials are configured to facilitate infrastructure deployment.

4. **Dependency Installation and Build:**
   The workflow installs dependencies for both the API and Pulumi infrastructure, runs Prisma generation, and builds the API.

5. **Pulumi Setup and Deployment:**
   - Logs into Pulumi using an S3 backend.
   - Cancels any running deployments if a matching stack exists.
   - Deploys the infrastructure using Pulumi with options such as refreshing the stack and commenting on pull requests.

6. **Prisma Migration:**
   After deployment, the workflow retrieves the Prisma connection string from the Pulumi stack output and applies database migrations.
