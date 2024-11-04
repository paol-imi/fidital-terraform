# Design Document: Cloud Infrastructure & DevOps Practices

## Introduction

We are a small development team working on the initial iterations of a new product. Due to the early stage of the product, there is a high degree of change and a need for rapid iteration. Additionally, the team is building a DevOps culture from scratch, with no previous strong DevOps practices in place.

## Service Overview

The application is designed as a standard microservices-based app deployed on AWS. Each customer has an exclusive workspace, managed within the same infrastructure but isolated appropriately to provide dedicated services.

---

## Cloud Infrastructure

**TODO**: Further define infrastructure details and architecture.

---

## Repository Organization and DevOps Practices

We maintain three types of repositories, each serving a different purpose within our cloud infrastructure and DevOps workflows.

### Repository Types

1. **Infrastructure Repository**:
   - Contains Terraform configurations, which are the single source of truth for all cloud infrastructure.
   - Any changes to infrastructure must be made through this repository, ensuring all infrastructure is versioned and traceable.

2. **Microservices Repositories**:
   - Separate repositories for each microservice within the application.
   - Each microservice repository is designed to be customer-agnostic, meaning all customers use the same codebase, configured differently per customer’s specific requirements.

3. **Library Repository**:
   - Contains shared libraries, both frontend and backend, that are used across multiple microservices.
   - This repository allows microservices to reuse common functionality, improving consistency and reducing duplication.

### Repository Strategies

To align with our infrastructure needs and deployment workflows, we use different branching and release strategies for different types of repositories:

1. **Git Flow (Application & Infrastructure Repositories)**:
   - For the microservices and infrastructure repositories, we use Git Flow to establish strict versioning and deployment conventions.
   - This approach allows for controlled releases, hotfixes, and snapshot versions within `develop` and `main` branches, which are suitable for our infrastructure and microservice needs.

2. **Trunk-Based Development (Library Repository)**:
   - For the library repository, we use a trunk-based workflow to enable faster delivery and patching of shared libraries.
   - This approach allows for continuous integration and ensures that updates to shared libraries can be quickly propagated across microservices.

### Mapping Repositories to Cloud Infrastructure

Each repository has a designated purpose and behavior when integrated into the cloud infrastructure, summarized as follows:

#### Infrastructure Repository (Terraform)

- **Branches**: `dev` and `main` represent two separate private infrastructure environments, each with its own VPC.
- **Pull Requests**: Any PR to `main` or `dev` generates a Terraform plan, which is displayed in the pull request body for review and approval.
- **Environments**:
  - The infrastructure is based on a standard VPC design.
  - Microservices are managed via ECS (Elastic Container Service) clusters within these VPCs.
  - Terraform is the source of truth for ECS services and the VPC setup, ensuring all configurations are centralized despite the distribution of microservices across multiple repositories.

#### Microservices Repositories

- **Versioning and Docker Image Generation**:
  - The version in each microservice repository is specified in `pom.xml` (for Maven projects) or `package.json` (for NPM projects).
  - **Version Conventions**:
    - On the `main` branch, versions follow the **semantic versioning convention** (`major.minor.patch`, e.g., `1.0.0`), indicating a stable release.
    - On the `develop` branch, versions use a **snapshot convention** (`major.minor.patch-SNAPSHOT`, e.g., `1.1.0-SNAPSHOT`), representing an in-progress development version.

- **Stable Releases (Main Branch)**:
  - For `main`, the pipeline checks for version changes in `pom.xml` or `package.json`.
  - If a version change is detected, the pipeline:
    - Automatically tags the commit with the new version (e.g., `v1.0.0`).
    - Generates an immutable Docker image with the stable version tag, which is pushed to Amazon ECR (Elastic Container Registry).
    - Publishes the application artifact to AWS CodeArtifact with the same stable version.

- **Development Snapshots (Develop Branch)**:
  - For `develop`, each commit generates a new Docker image tagged as `develop` or `latest`, but it inherits the **snapshot version** specified in `pom.xml` or `package.json` (e.g., `1.1.0-SNAPSHOT`).
  - This new image replaces the previous snapshot in ECR, allowing the latest state of the application to be tested.
  - The `-SNAPSHOT` suffix is mandatory in the version for `develop` to clearly indicate it is a development build and not stable.

#### Library Repository

- **Main Branch (Stable)**:
  - For stable versions in `main`, the library repository pipeline checks for version changes in `pom.xml` or `package.json`.
  - When a new stable version is detected, the pipeline:
    - Automatically tags the commit with the new version (e.g., `v1.0.0`).
    - Pushes the updated library as an immutable artifact to CodeArtifact, making it available to microservices with a stable version identifier.

- **Feature Branches (Snapshot Builds)**:
  - In feature branches, each commit pushes a snapshot version of the library to CodeArtifact, replacing the previous snapshot artifact.
  - This ensures that teams working on feature branches always have the latest updates without accumulating multiple snapshot artifacts.

---

## Summary Table

| Repository       | Branching Strategy | Versioning                          | CI/CD Triggers                   | Artifact/Image Behavior |
|------------------|--------------------|-------------------------------------|----------------------------------|-------------------------|
| **Infrastructure** | Git Flow         | N/A                                 | PRs to `main` or `dev` generate Terraform plan | Maintains separate VPCs for `main` and `dev` with ECS-managed microservices. |
| **Microservices** | Git Flow          | - `main`: semantic versioning (`v1.0.0`)<br> - `develop`: snapshot versioning (`1.1.0-SNAPSHOT`) | Version change in `main` auto-tags and generates Docker image | `main` images are immutable in ECR; `develop` snapshots override previous images. |
| **Library**       | Trunk-Based       | - `main`: semantic versioning (`v1.0.0`)<br> - feature branches: snapshot versions (`1.1.0-SNAPSHOT`) | Version change in `main` auto-tags and pushes to CodeArtifact | `main` artifacts are immutable in CodeArtifact; feature branch snapshots override previous artifacts. |

---

## Key Definitions and Concepts

To clarify terms and processes mentioned in this document:

- **Stable Version**: A version released from the `main` branch, following the format `major.minor.patch` (e.g., `1.0.0`). Stable versions are tagged and create immutable artifacts in the registry.

- **Snapshot Version**: A version released from the `develop` branch, following the format `major.minor.patch-SNAPSHOT` (e.g., `1.1.0-SNAPSHOT`). Snapshot versions are in-progress builds used for development and testing, and can override previous snapshots in the registry.

- **Immutable Artifacts**: Artifacts or images tagged with stable versions from `main` (e.g., `v1.0.0`) that cannot be overridden. This ensures that stable releases are always traceable and reproducible.

- **Overrideable Artifacts**: Artifacts or images tagged with a snapshot version from `develop` (e.g., `1.1.0-SNAPSHOT`) or `develop`/`latest` that replace previous builds. These are used for testing the latest state of the application without creating new, permanent versions.

- **Git Tagging**: On `main`, tagging is automated based on the stable version in `pom.xml` or `package.json`. For `develop`, tags are used primarily for documentation and tracking changes.

---

This document is a living reference that will continue to evolve as we iterate on our DevOps practices and refine our infrastructure setup.
