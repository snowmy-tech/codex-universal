# Azure DevOps setup

The root `azure-pipelines.yml` validates the repository on an ARM64 Azure
DevOps agent and builds a `linux/arm64` image inside Azure Container Registry
(ACR). Pull requests run validation only. Non-PR builds publish an immutable
commit tag, a branch tag, and `latest` for `main`.

## Required Azure resources

1. An existing Azure Container Registry.
2. An Azure Resource Manager service connection in the Azure DevOps project.
3. The service connection identity must have permission to inspect the registry
   and run ACR builds. Assign the narrowest roles supported by the registry's
   permission mode; do not store credentials in this repo.
4. A Linux ARM64 self-hosted Azure DevOps agent pool. Agents must advertise
   `Agent.OSArchitecture=ARM64`; the pipeline will not fall back to an x86_64
   Microsoft-hosted agent.

## Create the pipeline

1. In Azure DevOps, select **Pipelines → New pipeline**.
2. Select the repository and **Existing Azure Pipelines YAML file**.
3. Select `/azure-pipelines.yml`.
4. Set the runtime parameters:
   - `azureServiceConnection`: the Azure DevOps service connection name.
   - `acrName`: defaults to the existing `k8scontainer.azurecr.io` registry.
   - `imageRepository`: defaults to `codex-universal`.
   - `armAgentPool`: the ARM64 self-hosted agent pool name; defaults to
     `codex-universal-arm64`.
   - `pushImage`: keep enabled for branch builds; disable for validation-only runs.

The pipeline does not contain tokens, passwords, client secrets, subscription
IDs, or tenant IDs. Azure DevOps supplies authentication through the named
service connection at runtime.

## Published output

Successful non-PR builds publish a `container-image-reference` pipeline
artifact containing the immutable image reference:

```text
<acr-name>.azurecr.io/codex-universal:<git-commit-sha>
```

Use that immutable reference in downstream deployment pipelines.
