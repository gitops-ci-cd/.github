# GitOps CI/CD Pipeline Overview

This organization provides reference implementations for applications that are Continuously Integrated and Deployed using a GitOps-based approach via [Argo CD](https://argo-cd.readthedocs.io/en/stable/) and [GitHub Actions](https://docs.github.com/en/actions), leveraging separate repositories for application code, infrastructure configuration, and deployment logic.

## Repositories Structure

1. [[app]](https://github.com/gitops-ci-cd/acme-node) Repository

  	Contains the source code for the application, including any PR-triggered development workflows. Use this template [TODO]().

1. [[app]-infra](https://github.com/gitops-ci-cd/acme-node-infra) Repository

  	Manages infrastructure resources specific to the application, such as databases or storage, keeping infrastructure configuration separate from application code. Use this template [TODO]().

1. [[app]-deployment](https://github.com/gitops-ci-cd/acme-node-deployment) Repository

    Holds deployment manifests and configuration files for the application. It serves as the single source of truth for deployment configurations. Use this template [TODO]().

1. [.github](https://github.com/gitops-ci-cd/.github) Repository

    Special repository within GitHub that provides functionality across the organization

    - [Customizing Organization Profile](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile)
    - [Creating Default Community Health Files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
    - [GitHub Action Workflow Templates](https://docs.github.com/en/actions/sharing-automations/creating-workflow-templates-for-your-organization)
    - [Reusing GitHub Action Workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)

1. [.github-private](https://github.com/gitops-ci-cd/.github-private) Repository

    Special repository within GitHub that extends profile customization

    - [Customizing Organization Profile](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile#adding-a-member-only-organization-profile-readme)

## Development Workflow

1. Feature Development and Pull Requests

    - Developers work in feature branches within the app repository.
    - When a pull request (PR) is opened, a GitHub Action initiates linting and unit testing.

1. Automatic Preview Deployments

    - The [Argo CD PR Generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Pull-Request/) detects PRs labeled with "ephemeral" and generates a preview environment for the specific PR branch.
    - The PR-specific environment allows for isolated testing and review.
    - Upon successful deployment, [Argo's Notification tooling](https://argo-cd.readthedocs.io/en/stable/operator-manual/notifications/) sends [deployment status updates to GitHub](https://docs.github.com/en/rest/deployments/deployments), notifying the PR's deployment with statuses `in_progress`, `success`, `failure`, or `error` for real-time feedback on the preview environment.
    - The preview deployment is torn down upon removing the aformentioned label or closing the PR.

1. Releasing to Production

    - Once the PR is approved and merged to main, a GitHub Action builds a production-ready image, tags/releases the repo, and triggers a [deployment](https://docs.github.com/en/rest/deployments/deployments) workflow targeting production.
    - The release serves as an artifact of the build, versioned and traceable for production use.
  
```mermaid
flowchart TD

    registry[GitHub Container Registry]

    subgraph env[Ephemeral Environment]
        argo[Argo CD]
        argo-app[Argo ApplicationSet]
    end

    subgraph dev[Development Workflow]
        app -- pull_request --> pr[Pull Request]
        pr -- synchronize --> lint[Lint] & test[Test]
        pr -- labeled --> deployment[GitHub Deployment] -- deployment --> deployment-status[GitHub Deployment Status]
        deployment-status -- deployment_status --> build[Docker Image]
        build --> registry

        env
        argo-app -- 1 --o registry
        argo-app ---o app-deployment
        argo -- 2 --> deployment-status

        pr -- closed --> app
    end
```

## Production Deployment Workflow

1. New Image Detection

    - The new image tagged in the release is automatically detected by [Argo CD Image Updater](https://argocd-image-updater.readthedocs.io/en/stable/).
    - Argo CD Image Updater submits a PR to the [app]-deployment repository with the updated image tag. This PR can be set for manual approval or configured for auto-merge, depending on team practices.

1. Deployment to Production

    - Upon merging to main, Argo CD picks up the updated deployment configuration and rolls out the new image to the production environment.
    - [Argo's Notification tooling](https://argo-cd.readthedocs.io/en/stable/operator-manual/notifications/) then sends [deployment status updates to GitHub](https://docs.github.com/en/rest/deployments/deployments), notifying the deployment with statuses `in_progress`, `success`, `failure`, or `error`.

```mermaid
flowchart TD

    registry[GitHub Container Registry]

    subgraph env[Production Environment]
        argo[Argo CD]
        argo-image-updater[Argo CD Image Updater]
        argo-app[Argo Application]
    end

    subgraph prod[Production Workflow]
        app -- push --> release[Tag/Release]
        release -- publish --> deployment[GitHub Deployment] -- deployment --> deployment-status[GitHub Deployment Status]
        deployment-status -- deployment_status --> build[Docker Image]
        build --> registry

        env
        argo-image-updater -- 1 --o registry
        argo-image-updater -- 2 --> app-deployment

        app-deployment -- pull_request --> pr[Pull Request]
        pr -- synchronize --> validate[Validate]
        pr -- closed --> app-deployment

        argo-app -- 3 --o app-deployment
        argo -- 4 --> deployment-status
    end
```

## Summary

This pipeline leverages GitOps principles and automated deployments to ensure a streamlined, consistent deployment process:

- Preview environments for each PR with isolated, PR-specific deployments.
- Automated versioning and tagging upon merging to main, ensuring each production release is traceable.
- Argo CD Image Updater PRs to enforce review controls for production changes or enable auto-rollouts.
- GitHub Deployment Statuses reflect the deployment’s lifecycle, from feature development through production deployment, providing continuous feedback and visibility.

This structure combines GitOps with automated CI/CD workflows, providing both flexibility and control over each stage of deployment, while ensuring the system remains fully traceable, auditable, and Git-centered.
