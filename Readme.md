
## SRE Projects - Deploy REST API & its dependent services in K8s(Assignment 7)

### Problem statment/ Expectation :- 

In this assignment, we will deploy our REST API and PostgreSQL database on Kubernetes. Instead of using .env files for storing sensitive information, we will utilize HashiCorp Vault and External Secrets Operator to securely manage and inject secrets.

The diagram below shows the deployment outputs:

![Image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9ce3a364-243d-4bf8-803e-331bbc517340%2F49ccfffd-fdc8-4ae8-86f8-5f15c60e7af1%2Fk8s-deployment.drawio.png?table=block&id=86f1021a-c29a-4a02-90a9-3088118cabb4&cache=v2)

## Requirments
- Docker
- Kind
- Kubectl
- Helm (For installing Vault and ESO)
- Kubernates cluster (refer to the previous assignment for details) and complete the necessary steps to proceed directly with Vault configuration.

## Installation / Quick Start

1) ### Clone and Navigate:

```bash
git clone https://github.com/venk404/SRE--RestAPI.git
cd .\Assignment 7
```

2) ### Before installing Vault and ESO, we need to install Helm, a Kubernetes package manager that simplifies the installation process. Installation is straightforward, and you can refer to the docs for details.

```bash
https://helm.sh/docs/intro/install/
```

## Vault and External Secrets Operator Setup

3) ### Install Vault:

```bash
cd Dependend_Services/Vault/
kubectl create -f Namespace.yml
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault --namespace vault --values Vault_Value.yml
kubectl get pods -n vault -owide
```

4) ### Initialize and unseal Vault:
```bash
# Please wait for a while, as the Vault-0 pod may take some time to reach the ready state.
kubectl exec vault-0 -n vault -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
$VAULT_UNSEAL_KEY=$(cat cluster-keys.json | jq -r ".unseal_keys_b64[]")
kubectl exec vault-0 -n vault -- vault operator unseal "$VAULT_UNSEAL_KEY"
```

5) ### Configure Vault secrets:

```bash
# Login to vault using root token from cluster-keys.json
kubectl exec -it vault-0 -n vault -- /bin/sh
vault login
vault secrets enable -path=secrets kv-v2
vault kv put secrets/DBSECRETS POSTGRES_PASSWORD=pass POSTGRES_DB=postgres POSTGRES_USER=postgres
```

6) ### Install External Secrets Operator:

```bash
cd '.\External Secrets\'
kubectl apply -f Namespace.yml
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets --set installCRDs=true -f ExternalSecret_Value.yml

# Create vault token secret
kubectl create secret generic vault-token --namespace=external-secrets --from-literal=token=<token>

# Setup secret store
kubectl apply -f ClusterSecretStore.yml
kubectl get clustersecretstore -A
#You should see the status as Valid
kubectl apply -f ExternalSecret.yml
kubectl get externalsecrets -A
#You should see the status as SecretSynced
```

7) Deploy applications:
```bash
cd ../DB

kubectl apply -f Postgres.yml

cd ../Application
kubectl apply -f Application.yml
```

8) Access the API at http://127.0.0.1:30007/docs

