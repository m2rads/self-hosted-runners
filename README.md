1. Install Prerequisites

```bash
# Install Homebrew if not installed (macOS)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install minikube
brew install minikube

# Install kubectl
brew install kubectl

# Install Helm
brew install helm
```

2. Start Minikube

```bash
# Start with more resources for ARC
minikube start --cpus 2 --memory 4096 --kubernetes-version=v1.25.0

# Verify it's running
kubectl get nodes
```

3. Create GitHub Personal Access Token
Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
Generate new token with appropriate scopes:
For repo-level runners: repo scope
For organization-level: admin:org scope

4. Install ARC Controller
```bash
# Create namespace for ARC
kubectl create namespace arc-systems

# Install ARC controller
helm install arc \
  --namespace arc-systems \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

5. Deploy Runner Scale Set
```bash
# Create namespace for runners
kubectl create namespace arc-runners

# Store GitHub PAT in a secret
GITHUB_TOKEN="your_token_here"

# Deploy runner scale set
helm install arc-runner-set \
  --namespace arc-runners \
  --create-namespace \
  --set githubConfigUrl="https://github.com/your-username/your-repo" \
  --set githubConfigSecret.github_token="${GITHUB_TOKEN}" \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

6. Verify Installation
```bash
# Check if controller is running
kubectl get pods -n arc-systems

# Check if listener is running
kubectl get pods -n arc-runners
```

7. Create Test Workflow
In your GitHub repo, create .github/workflows/test.yml:
```bash
name: Test Self-Hosted Runner
on: [push]
jobs:
  test:
    runs-on: arc-runner-set  # Match your runner scale set name
    steps:
      - uses: actions/checkout@v3
      - name: Run test
        run: echo "Hello from self-hosted runner!"
```

8. Additional Commands
```bash
# Scale runners manually
kubectl scale runners arc-runner-set -n arc-runners --replicas=3

# Check logs
kubectl logs -n arc-runners -l app.kubernetes.io/name=arc-runner-set-listener

# Clean up when done
minikube stop
minikube delete
```



