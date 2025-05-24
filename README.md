# AKS with Terraform and Argo CD sample app

## Table of Contents

- [AKS with Terraform and Argo CD sample app](#aks-with-terraform-and-argo-cd-sample-app)
  - [Sample app screenshot](#sample-app-screenshot)
  - [Sample app URL](#sample-app-url)
  - [Notes](#notes)
  - [Prerequisites](#prerequisites)
  - [Usage](#usage)
  - [Clean up resources](#clean-up-resources)

## Sample app screenshot
[Screenshot](https://github.com/eranoc1/eranoc1.github.io/blob/main/ghscreenshots/aks-tf-argocd-app1.png)
## Sample app URL
http://hellok8s.japps.cloud

## Notes
- This code deploys an AKS cluster using Terraform CLI, ArgoCD, a sample app, and configures access through an ingress controller.
- Configured for a single environment to keep the setup simple.
- The Domain / FQDN may be configured in the file kubernetes/argocd/ingress.yaml
- It is possible to set the KUBECONFIG variable before running the commands, rather than prefixing it.
- This is a demo app and therefore uses HTTP only.
  ArgoCD is deployed in non high availability mode to reduce deployment time and simplify the setup.

```
export KUBECONFIG=~/.kube/azurek8s
```
## Prerequisites
- An [Azure account](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account)
- OS: Linux / macOS / Windows - [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- [Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli#install-terraform)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- [Helm](https://helm.sh/docs/intro/install/)

## Usage
```
mkdir ~/gh
cd ~/gh
git clone https://github.com/eranoc1/aks-tf-argocd.git
cd ~/gh/aks-tf-argocd/terraform
terraform init -upgrade
terraform plan -out "tfplan"
terraform apply "tfplan" # Manual approval. This may take a few minutes
echo "$(terraform output kube_config | grep -Ev 'EOT|<< EOT')" > ~/.kube/azurek8s # Save the AKS kubeconfig file
KUBECONFIG=~/.kube/azurek8s kubectl create namespace argocd
KUBECONFIG=~/.kube/azurek8s kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
KUBECONFIG=~/.kube/azurek8s kubectl get pods -n argocd # Verify all pods are in "ready" state
cd ~/gh/aks-tf-argocd/kubernetes/argocd
KUBECONFIG=~/.kube/azurek8s kubectl apply -f ns.yaml
KUBECONFIG=~/.kube/azurek8s kubectl apply -f app.yaml
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml |\
sed 's/namespace: ingress-nginx/namespace: hello-kubernetes/g' | KUBECONFIG=~/.kube/azurek8s kubectl apply -f -
KUBECONFIG=~/.kube/azurek8s kubectl apply -f ingress.yaml # It might take a short time before it starts working
KUBECONFIG=~/.kube/azurek8s kubectl get svc -n hello-kubernetes | grep -E "NAME|ingress-nginx-controller "  # Get the ingress public IP to be used in the DNS record
grep "host:" ingress.yaml | cut -d : -f 2 | awk '{print $1}' # Get the FQDN for the DNS record
curl http://hellok8s.japps.cloud # Use the relevant FQDN
```
Create the appropriate DNS record.
Open the app URL in your browser.

## Clean up resources
```
terraform plan -destroy -out "tfplan"
terraform apply -destroy  "tfplan" # Manual approval
```
