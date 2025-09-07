# Flux-GitOpsonAKS

This repository contains configuration files for managing Kubernetes clusters using [Flux](https://fluxcd.io/). It is structured to support GitOps workflows for cluster management and deployment on AKS (Azure Kubernetes Service).

## Repository Structure

```
clusters/
  my-cluster/
    flux-system/
      flux-version.txt
      gotk-components.yaml
      gotk-sync.yaml
      kustomization.yaml
```

- **clusters/my-cluster/flux-system/**: Contains Flux configuration files for the `my-cluster` Kubernetes cluster.
  - `flux-version.txt`: Specifies the Flux version used. You can upgrade AKS Flux to any version by updating this file with the desired version number.
  - `gotk-components.yaml`: Defines Flux components deployed to the cluster.
  - `gotk-sync.yaml`: Configures synchronization between the cluster and this Git repository.
  - `kustomization.yaml`: Manages resources using Kustomize.

## Getting Started

1. **Install Flux CLI**
   - Follow the [official Flux installation guide](https://fluxcd.io/docs/installation/) to install the CLI.

2. **Bootstrap Flux on AKS**
   - Use the Flux CLI to bootstrap your AKS cluster:
     ```bash
     flux bootstrap azure \
       --resource-group <resource-group> \
       --cluster-name <aks-cluster-name> \
       --git-url=https://github.com/<your-org>/Flux-GitOpsonAKS \
       --git-path=clusters/my-cluster/flux-system
     ```

3. **Manage Cluster Resources**
   - Edit the files in `clusters/my-cluster/flux-system/` to update your cluster configuration.
   - Changes pushed to the repository will be automatically applied to the cluster by Flux.

4. **Configure Azure Service Principal**
   - Paste your Azure Service Principal credentials into a Kubernetes Secret manifest (YAML) inside the Git repository, typically under `clusters/my-cluster/flux-system/`.
   - Example secret manifest:
     ```yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: azure-service-principal
       namespace: flux-system
     type: Opaque
     stringData:
       AZURE_CLIENT_ID: <your-client-id>
       AZURE_TENANT_ID: <your-tenant-id>
       AZURE_CLIENT_SECRET: <your-client-secret>
     ```
   - Commit and push the secret manifest to the repository. Flux will apply it to your cluster.

## GitHub Actions

- The repository may include GitHub Actions workflows (e.g., `.github/workflows/flux-upgrade.yml`) to automate upgrades and maintenance tasks for Flux components.

## References

- [Flux Documentation](https://fluxcd.io/docs/)
- [Kustomize Documentation](https://kubectl.docs.kubernetes.io/pages/app_management/introduction.html)

## License

This repository is licensed under the MIT License.
