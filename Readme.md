## SRE Projects - Deploy REST API & Its Dependent Services in K8s (Assignment 7)

### Problem Statement / Expectation

In this assignment, we will deploy our REST API and PostgreSQL database on Kubernetes. Instead of using `.env` files for storing sensitive information, we will utilize HashiCorp Vault and External Secrets Operator to securely manage and inject secrets.

The diagram below shows the deployment outputs:

![Deployment Diagram](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9ce3a364-243d-4bf8-803e-331bbc517340%2F49ccfffd-fdc8-4ae8-86f8-5f15c60e7af1%2Fk8s-deployment.drawio.png?table=block&id=86f1021a-c29a-4a02-90a9-3088118cabb4&cache=v2)

---

## Requirements

- Docker
- Kind
- Kubectl
- Helm (For installing Vault and ESO)
- Kubernetes cluster (Refer to the previous assignment for details) and complete the necessary steps to proceed directly with Vault configuration.

---

## Installation / Quick Start

### 1. Install Helm

Before installing Vault and ESO, we need to install Helm, a Kubernetes package manager that simplifies the installation process. Follow the [official Helm installation guide](https://helm.sh/docs/intro/install/).

### 2. Clone and Navigate

```bash
git clone https://github.com/venk404/Deploying-in-k8s.git
cd Deploying-in-k8s
```

---

## Vault and External Secrets Operator Setup

### 3. Install Vault

```bash
cd Dependend_Services/Vault/
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault -n vault --create-namespace --values Vault_Value.yml
kubectl get pods -n vault -owide
```

### 4. Initialize and Unseal Vault

```bash
# Wait for the Vault-0 pod to be ready
kubectl exec vault-0 -n vault -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
export VAULT_UNSEAL_KEY=$(cat cluster-keys.json | jq -r ".unseal_keys_b64[]")
kubectl exec vault-0 -n vault -- vault operator unseal $VAULT_UNSEAL_KEY
```

### 5. Configure Vault Secrets

```bash
# Login to Vault using root token from cluster-keys.json
kubectl exec -it vault-0 -n vault -- /bin/sh
vault login
vault secrets enable -path=secrets kv-v2
vault kv put secrets/DBSECRETS POSTGRES_PASSWORD=<your_password> POSTGRES_DB=<your_db> POSTGRES_USER=<your_user>
```

### 6. Install External Secrets Operator

```bash
cd 'External Secrets/'
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace --set installCRDs=true -f ExternalSecret_Value.yml
```

### 7. Create Vault Token Secret

Go to `ClusterSecretStore.yml`, update the secret token, and save the changes.

### 8. Setup Secret Store

```bash
kubectl apply -f ClusterSecretStore.yml
kubectl get clustersecretstore -n external-secrets
kubectl get externalsecrets -n external-secrets
# You should see the status as SecretSynced
```

---

## Deploy Applications

### 9. Deploy PostgreSQL

```bash
cd ../DB
kubectl apply -f Postgres.yml
```

### 10. Deploy Application

```bash
cd ../Application
kubectl apply -f Application.yml
```

### 11. Access the API

```bash
http://127.0.0.1:30007/docs
```

---

## Conclusion

By following these steps, you have successfully deployed a REST API with its dependencies in Kubernetes, securely managing secrets using Vault and ESO.
