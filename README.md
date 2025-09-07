
# Flux GitOps Upgrade on AKS

## Overview

This project demonstrates a GitOps approach to upgrade Flux CD components on an Azure Kubernetes Service (AKS) cluster.

Flux CD is a continuous delivery tool for Kubernetes that synchronizes cluster state with configuration in a Git repository. This project specifically enables:

- **Automatic upgrades** of Flux controllers (kustomize-controller, notification-controller, etc.) based on a version file in Git.
- **GitOps workflow** via GitHub Actions.
- **Declarative control:** updating `flux-version.txt` triggers upgrades.

## Architecture

```
GitHub Repository
│
├─ clusters/my-cluster/flux-system/flux-version.txt  # Desired versions of Flux controllers
│
└─ .github/workflows/upgrade-flux.yml               # GitHub Actions workflow
        │
        └─ Runs workflow on push or manual trigger
                │
                └─ Connects to AKS via Azure Service Principal
                        │
                        └─ Upgrades selected Flux controllers in flux-system namespace
```

## Prerequisites

- Azure Account with an AKS cluster (resource group, e.g., `SasinduAKS`).
- Service Principal with Contributor access to AKS resource group.
- GitHub Repository with:
  - `clusters/my-cluster/flux-system/flux-version.txt`
  - `.github/workflows/upgrade-flux.yml` workflow file
- GitHub Secret `creds` containing Service Principal JSON:
  ```json
  {
    "clientId": "<APP_ID>",
    "clientSecret": "<PASSWORD>",
    "subscriptionId": "<SUBSCRIPTION_ID>",
    "tenantId": "<TENANT_ID>"
  }
  ```

## Folder Structure

```
Flux-GitOpsonAKS/
├─ clusters/
│   └─ my-cluster/
│       └─ flux-system/
│           └─ flux-version.txt   # desired versions of controllers
└─ .github/
    └─ workflows/
        └─ upgrade-flux.yml       # GitHub Actions workflow
```

## Step 1: Set Desired Flux Versions

Edit `clusters/my-cluster/flux-system/flux-version.txt`:

```
kustomize-controller:v1.3.1
notification-controller:v1.3.1
```

Only include the controllers you want to upgrade. The left side is the deployment name, right side is the image version.

## Step 2: Push Changes to Git

```bash
git add clusters/my-cluster/flux-system/flux-version.txt
git commit -m "Update Flux controller versions"
git push origin main
```

This triggers the GitHub Actions workflow automatically.

## Step 3: GitHub Actions Workflow

The workflow `.github/workflows/upgrade-flux.yml` does the following:

1. Checks out the repository.
2. Logs in to Azure using `creds`.
3. Retrieves AKS credentials.
4. Optionally installs Flux CLI for verification.
5. Iterates over `flux-version.txt` and upgrades the listed controllers using `kubectl set image`.
6. Verifies the upgrade by listing deployments and pods.

## Step 4: Manual Trigger (Optional)

You can manually trigger the workflow from GitHub:

- Go to **Actions → GitOps Flux Upgrade → Run workflow → main → Run workflow**.

## Step 5: Verify Upgrades

After the workflow completes:

```bash
kubectl -n flux-system get deployments
kubectl -n flux-system get pods
```

You should see the updated versions in the IMAGE column for each upgraded controller.

## Notes

- **Idempotent workflow:** Re-running with the same versions will have no effect.
- **Extending workflow:** You can add other Flux components (e.g., helm-controller) to `flux-version.txt` to upgrade them as well.
- **GitOps advantage:** All upgrades are tracked in Git. You can revert by changing the version file.

## References

- [Flux CD](https://fluxcd.io/)
- [GitOps with Flux](https://fluxcd.io/docs/gitops/)
- [Azure AKS](https://learn.microsoft.com/en-us/azure/aks/)
- [Azure Login GitHub Action](https://github.com/Azure/login)

## License

This repository is licensed under the MIT License.
