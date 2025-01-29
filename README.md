# school-app
## Technical Documentation

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nehiyr-tech.com)

Ce document couvre tous les aspects techniques de AFISOFT V3 notamment l'infrastructure, les procédures d'équipe...

## Team Process

To begin with, we start with Kanban. We will re-evaluate the method in the future.
We use Google Meets to plan our meetings.

Nous avons les rôles suivants : 
- un tech lead et PO
- un lead dev/sec'ops

Les équipes devs et dev/sec'ops seront constitués par les différents lead.

**Refinement**
L'équipe dev définira à sa convenance le moment où il faudra refiner des fonctionalités afin de les planifier sur le Kanban

**User Story**
A use story (US) contains these elements :
- context
- technical precision
- acceptance criterias

**Retrospective**
As needed

**Demo**
As needed

## DEV
### Git branches
Naming format `[Prefix]/[AFISOFT-#]-[Description]`

`[Prefix]` : 
- ***feat***: features
- ***fix***: bugfix
- ***doc***: documentation
- ***chore***: routine operation (bump lib versions)
- ***ops***: IaC, CI/CD...

`[AFISOFT-#]`: Issue number
`[Description]`: Short description

Example: `feat/AFISOFT-AFISOFT-152-handle-realtime-notification`

### Pull Requests
`Title`
Description

`Body`
- Link to JIRA issue in description (optional)
- Attention points for review
- Screenshot for frontend related PR

No need to create a JIRA issue for technical stuff (example: update a library version).

### GIT commit messages
Structure:

```
[PR-ID] [GH emoji] summary
<empty line>
optional additional description
```
GH emoji (from https://gitmoji.dev):
- *feat*: :sparkles:
- *fix*: :bug:
- *doc*: :memo:
- *chore*: it depends
- *ops*: it depends (:rocket:)

## CI/CD Pipeline
This documents outlines the use of CI/CD via Github Actions (GHA) with mutliple workflows triggered in different ways and utilizing various worflow files, which are explained in details below.

### Context
To streamline our development process, improve deployment efficiency, and maintain high-quality software, we have decided to leverage GitHub Actions for our CI/CD pipeline. This setup enables us to automate build processes, run tests, and deploy applications to specific environments. We use different workflows to manage deployments across various stages, and this document details key workflows in our project: the `cd-previews`workflow, the `cd-integration` workflow, the `clean`workflow, the `ci`workflow, the `release`workflow, and the `deploy`workflow.

### CD-Previews Workflow
The `cd-previews` workflow is one of the primary workflows we use in our CI/CD pipeline. It serves as a mechanism for deploying preview versions of the application, allowing developers and reviewers to validate changes before they are merged into the main codebase.

**Workflow Trigger**
 - **Trigger Method** : This workflow is triggered by a comment on a pull request.
 - **Trigger Command** : The comment must contain the phrase: `/preview`

**Workflow Configuration**
 - **Workflow File** : The workflow is defined in `github/workflows/cd-previews.yml` and calls the cd/previews.yaml Helm release file, which is responsible for deploying the application on Kubernetes (cluster dev/k8s-dev/dev).
 - **Actions Performed**: The workflow performs the following actions:
	- ***Build the Application***: The workflow compiles the project source code and ensures that no compilation issues are present.
	- ***Run Tests***: It runs the full suite of automated tests to verify the integrity of the code changes.
	- ***Deploy to Preview Environnement***: After building and testing, the workflow deploys the application to a preview environment in a Kubernetes cluster (dev/k8s-dev/dev).

**Output of the Workflow**
 - **Deployment Link** : After successful deployment, a link to the newly deployed preview application is provided in the pull request comments. This allows developers and reviewers to easily access the running version for review.
 - ***Additional Information***: The pull request comments also include links to the **Actuator** and **Grafana** dashboards, providing additional insights into the health and metrics of the deployed application.
 - ***Resource Management***: Deploying preview environments can consume additional resources in the Kubernetes cluster, particularly when multiple pull requests are open simultaneously.

### CD-Integration Workflow
The  `cd-integration`  workflow is responsible for ensuring that the latest version of the application is continuously integrated and deployed to the integration environment, allowing the team to validate changes on a stable setup.

**Workflow Trigger**
 - **Trigger Method** : This workflow is triggered by a push event to the **main branch**.

**Workflow Configuration**
 - **Workflow File** : The workflow is defined in `github/workflows/cd-integration.yml` and calls the `cd/integration.yaml` Helm release file, which is responsible for deploying the application on Kubernetes (cluster dev/k8s-dev/dev).
 - **Actions Performed**: The workflow performs the following actions:
	- ***Build the Application***: The workflow compiles the project source code and ensures that no compilation issues are present.
	- ***Run Tests***: It runs the full suite of automated tests to ensure the code changes are stable.
	- ***Deploy to Integration Environnement***: Deploys the application to the integration environment, which is a Kubernetes cluster (**dev/k8s-dev/dev**), and the pods are named `afisoft-integration`.

**Output of the Workflow**
 - **Deployment Link** : After successful deployment, a link to the newly deployed application is available, providing the team with a means to verify changes in the integration environment.
 - ***Additional Information***: The workflow also provides links to the **Actuator** and **Grafana** dashboards in the deployment comments, offering insights into the health and metrics of the running application.

### Clean Workflow
The `clean` workflow is responsible for cleaning up preview environments after they are no longer needed.

**Workflow Trigger**
 - **Trigger Method** : This workflow is triggered by closing a pull request.

**Workflow Configuration**
 - **Workflow File** : The workflow is defined in `.github/workflows/clean.yml`.
 - **Actions Performed**: The workflow performs the following actions:
	- ***Delete Configuration File***: Removes the configuration file from the `k8s-preview` repository.
	- ***Delete Kubernetes Namespace***: Deletes the corresponding namespace from the Kubernetes cluster to free up resources.

**Output of the Workflow**
 - **Environment Cleanup** : Ensures that the preview environment and associated resources are properly cleaned up, freeing up Kubernetes cluster resources.

### CI Workflow
The `ci` workflow is responsible for building the application and running the tests to ensure the stability of the codebase.

**Workflow Trigger**
 - **Trigger Method** : This workflow is triggered by a **pull request event** (either opening a pull request or modifying it) or by a **push to the main branch**.

**Workflow Configuration**
 - **Workflow File** : The workflow is defined in `.github/workflows/ci.yml`.
 - **Actions Performed**: The workflow performs the following actions:
	- ***Build the Application***: Compiles the project source code to ensure that everything is up to date and no compilation issues are present.
	- ***Run Tests***: Runs the full suite of automated tests to ensure that the code changes are stable and meet the quality standards.

**Output of the Workflow**
 - **Validation Feedback** : Provides feedback on the build and test results directly in the pull request or commit status, helping developers address any issues before the code is merged.

### Release Workflow
The `release` workflow is responsible for creating a new release of the application, ensuring that the versioning and deployment processes are followed correctly.

**Workflow Trigger**
 - **Trigger Method** : This workflow is triggered manually from the GitHub Actions UI on the **Actions** tab.

**Workflow Configuration**
 - **Workflow File** : The workflow is defined in `.github/workflows/release.yml`.
 - **Actions Performed**: The workflow performs the following actions:
	- ***Create a Release***: Releases a new version of the application.
	- ***Tag Creation***: Creates a tag in the repository for the new version.
	- ***Publish Artifacts to ECR***: Publishes the project’s Docker images to the project’s Elastic Container Registry (ECR).
	- ***Create a Pull Request***: Creates a pull request to bump the project’s version in the repository.

**Output of the Workflow**
 - **Release Tag** : A new release tag is created in the repository, representing the new version of the application.
 - ***Published Artifacts***: Artifacts are published to ECR
 - ***Version Bump PR***: A pull request is created to update the project version, ensuring version consistency.

### Deploy Workflow
The `deploy` workflow is responsible for deploying the application to UAT, prep, or production environments.

**Workflow Trigger**
 - **Trigger Method** : This workflow is triggered manually from the GitHub Actions UI on the **Actions** tab.

**Workflow Configuration**
 - **Workflow File** : The workflow is defined in `.github/workflows/deploy.yml`.
 - **Actions Performed**: The workflow performs the following actions:
	- ***Select Environment***: The user selects the target environment (UAT, prep, or prod) from a dropdown in the GitHub Actions UI.
	- ***Select Version***: 1.  The user selects the version of the application to deploy by choosing the appropriate image tag from a dropdown.
	- ***Deploy the Application***: Deploys the selected version of the application to the chosen environment.

**Output of the Workflow**
 - **Deployment Status** : Provides status updates and notifications regarding the success or failure of the deployment process.

## Project Architecture and Organization Guide

This document provides a detailed guide on the architecture and organization of our project. The primary goal is to ensure that all team members follow consistent practices, respect the established project architecture, and contribute efficiently to maintain the integrity of the overall structure. Our project uses **Java Spring** or **Kotlin Spring** for the backend and **Next JS** for the frontend, with all projects integrated into a unified **Maven or Gradle multi-module** setup.

### Backend Overview
The  **backend**  is implemented using  **Kotlin** or **Java**  and follows a  **Hexagonal Architecture**  approach. This architecture pattern allows for better decoupling, separation of concerns, and an easier testing approach. The backend is divided into multiple modules, each respecting the hexagonal principles. Here are the main components of the backend organization:
- **Module Structure**
	 **Hexagonal Architecture**: Each backend module adheres to the Hexagonal Architecture principles, and they are structured as follows:
	- Each Module contains two main submodules: ***Domain*** and ***Infrastructure*** (***Infra***)
	- The ***Domain*** submodule is responsible for business logic, containing elements such as models, ports, and use cases.
	- The ***Infrastructure*** (***Infra***) submodule is responsible for the configuration, as well as outbound and inbound adapters.

 - **Domain Module**
The **Domain Module** is the heart of the backend and includes all core business logic. It is organized as follows:
	 - **Models**: Represents the entities and core data used across the application.
	 - **Ports**: Represents the interfaces that define the contract for interactions between the domain and the external world. These interfaces are implemented in the infrastructure layer, which allows for decoupling of business logic from the underlying technology.
	 - **Use Cases**: Represents the business rules and processes that handle specific application functionalities. These use cases utilize models and ports to interact with the infrastructure.

 - **Infrastructure Module (Infra)**
The **Infrastructure Module** handles everything related to external systems, configurations, and other technical details that the domain should not be concerned with.
	 - **Configuration (Conf)**: Contains the configurations for the application, such as database connections, security settings, and other system properties.
	 - **Inbound Adapters**: Responsible for handling incoming requests and commands, such as REST APIs or message handlers. This layer is what connects the external world to the core domain.
	 - **Outbound Adapters**: Responsible for handling outgoing communication with external systems, such as databases, or other third-party services.

### Best Practices for Development
To maintain consistency across the development process, follow these best practices:

1.  **Respect Hexagonal Architecture**: Always keep business logic in the domain module and avoid adding infrastructure-related code there. This ensures proper separation of concerns.
    
2.  **Consistent Module Structure**: Make sure to organize new features into separate modules that contain both domain and infra components as needed.
    
3.  **Dependency Management**: Use Maven or Gradle to manage all dependencies centrally, ensuring compatibility and consistency across the project.
    
4.  **Code Review**: Adhere to coding standards and guidelines established by the team, and always seek feedback through code reviews.
