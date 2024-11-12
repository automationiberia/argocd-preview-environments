# argocd-preview-environments

This repository, **argocd-preview-environments**, is designed to demonstrate and test ArgoCD's ability to automatically create preview environments for pull requests. Using ArgoCD's ApplicationSet with the pull request generator, each new pull request triggers a temporary environment that mirrors the changes in the PR.

## Purpose

The main goal of this repository is to facilitate testing and exploration of ArgoCD’s pull request generator functionality, where every pull request can be deployed as a unique environment for testing.

## How It Works

1. **Automatic Deployment on Pull Requests**: Whenever a new pull request is created, ArgoCD detects the PR and creates a dedicated preview environment based on the branch and pull request number.
2. **ApplicationSet Configuration**: The repository uses an ApplicationSet with a pull request generator that watches for PRs labeled with "preview".
3. **Environment Cleanup**: After a PR is closed or merged, ArgoCD can automatically prune the environment, ensuring only active PRs have live deployments.

## Repository Structure

├── ansible.cfg
├── ansible-navigator.log
├── bootstrap.yaml
├── collections
│   └── requirements.yaml
├── gitops
│   ├── applications
│   │   └── bgd
│   │       ├── base
│   │       │   ├── bgd-deployment.yaml
│   │       │   ├── bgd-route.yaml
│   │       │   ├── bgd-svc.yaml
│   │       │   └── kustomization.yaml
│   │       └── overlays
│   │           └── default
│   │               ├── bgd-deployment-patch.yaml
│   │               └── kustomization.yaml
│   ├── bootstrap
│   │   └── bgd-boostrap.yaml
│   └── installation
│       ├── argocd-app-bootstrap.yaml
│       ├── argocd-server.yaml
│       └── gitops-operator.yaml
├── group_vars
│   ├── all
│   │   └── main.yml
│   └── cluster1
│       └── vault.yaml
├── inventory
├── README.md

## Setup Instructions

1. **Install the required packages**

```
$  python3 -m pip install ansible-navigator --user
$  python3 -m pip install ansible-builder --user
```

2. **Fork and Clone the Repository**

   - Fork this repository to your GitHub account and clone it locally to test it with your own instance.

   ```bash
   git clone https://github.com/your-username/argocd-preview-environments.git
   cd argocd-preview-environments
   ```

   - Update the ApplicationSet configuration in `gitops/bootstrap/bgd-boostrap.yaml` with your GitHub repository details if needed.

   ```yaml
   github:
     owner: user or organization
     repo: repository name
     labels: (Optional)
       - label that must match (e.g., "preview") for creating an Application for each pull request.
   ```
   - Commit and push all changes

   ```bash
   $ git add --all; git commit -am "Update my repository"; git push
   ```

3. **Populate and encrypt your credentials and custom data**

```bash
$ vim group_vars/cluster1/vault.yaml
$ vim group_vars/all/main.yml
$ ansible-vault encrypt group_vars/*/vault.yaml
$ vim .vault_password
```
4. **Create OCP object needed to deploy Openshift GitOps and bootstrap the application**

```bash
$ ansible-navigator run bootstrap.yaml -i inventory -l cluster1 -m stdout --eei quay.io/automationiberia/ee-ocp-aap-iac-casc --vault-password-file .vault_password
```

5. **Verify application is running**

```bash
$ oc get appset -n openshift-gitops
```

6. **Success**

This configuration will create an Application for each [pull request]https://github.com/automationiberia/argocd-preview-environments/pulls labeled as "preview."

```bash
$ oc get app -n openshift-gitops
```

## Variables

|Variable Name|Default Value|Required|Description|Example|
|:---|:---:|:---:|:---|:---|
|`ocp_api_key`|n/a|yes|Token used to authenticate with the Openshift API|'sha256~Po6ydC7CVs12drESQeNiUW9poUT84aFrj7zL3VQfvrS'|
|`ocp_api_host`|n/a|yes|Provide a URL for accessing the Openshift API|'=https://api.cluster-ocp.lab.example.com:6443'|
|`ocp_api_validate_certs`|false|no|Whether or not to verify the API server’s SSL certificates|'true'|
|`aws_access_key`|n/a|yes|Amazon Access Key to access AWS API|'R767AKIFYSF5INA6QKB6'|
|`aws_secret_key`|n/a|yes|Amazon Secret Key to access AWS API|'qKCYpd/jQX6gRhucQwIT1d2lzrapZ/O4lpEKGGqR'|
|`aws_region`|n/a|yes|Amazon region where the S3 will be deploy it|'us-central-3'|



