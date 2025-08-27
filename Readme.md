
## SRE Projects - Deploy REST API & its dependent services in K8s(Assignment 7)

### Problem statment/ Expectation :- 

In this assignment, we will deploy our REST API and PostgreSQL database on Kubernetes. Instead of using .env files for storing sensitive information, we will utilize HashiCorp Vault and External Secrets Operator to securely manage and inject secrets.

The diagram below shows the deployment outputs:

![Image](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F9ce3a364-243d-4bf8-803e-331bbc517340%2F49ccfffd-fdc8-4ae8-86f8-5f15c60e7af1%2Fk8s-deployment.drawio.png?table=block&id=86f1021a-c29a-4a02-90a9-3088118cabb4&cache=v2)

## Requirments
- Docker
- Kind
- Kubectl
- Kubernates cluster (refer to the previous assignment for details) and complete the necessary steps to proceed directly with Vault configuration.

## Installation / Quick Start

1) ### Clone and Navigate:

```bash
git clone https://github.com/venk404/Deploying-in-k8s.git
cd Deploying-in-k8s/

```

## Vault and External Secrets Operator Setup

2) ### Install Vault:

```bash
cd Dependend_Services/Vault/

kubectl apply -f .

kubectl get pods -n vault -owide
```

3) ### Initialize and unseal Vault:
```bash
# Please wait for a while, as the Vault-0 pod may take some time to reach the ready state.
kubectl exec vault-0 -n vault -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
$VAULT_UNSEAL_KEY=$(cat cluster-keys.json | jq -r ".unseal_keys_b64[]")
kubectl exec vault-0 -n vault -- vault operator unseal "$VAULT_UNSEAL_KEY"
```

4) ### Configure Vault secrets:

```bash
# Login to vault using root token from cluster-keys.json
kubectl exec -it vault-0 -n vault -- /bin/sh
# copy the root toek from cluster-keys.json
vault login <root_token>
vault secrets enable -path=secrets kv-v2
# Update the details for creating secrets in vault
vault kv put secrets/DBSECRETS POSTGRES_PASSWORD=<password> POSTGRES_DB=<DB> POSTGRES_USER=<user>
exit
```

5) ### Setting up External Secrets Operator

```bash
cd ..

cd '.\External Secrets\'

kubectl apply -f .\namespace.yaml

kubectl apply --server-side -f .\crds\ 

echo -n "your-vault-token-here" | base64 

# Update secret.yaml manually: replace <token> with the above base64-encoded token 

kubectl apply -f . 
kubectl get secret vault-token -n external-secrets

kubectl get clustersecretstore
kubectl get externalsecrets -A
# You should see the status as SecretSynced
```

6) ### Deploy Postgres and Applications:

```bash
cd ../..

cd ../DB

# Please wait still all the pods are ready in external-secrets ns

kubectl apply -f Postgres.yml

cd ../Application

kubectl apply -f Application.yml
```

7) ### Access the API at http://127.0.0.1:30007/docs


## Conclusions

All the expectations have been met for **Milestone 7**:

- ✅ Kubernetes manifests created for all components (application, DB, dependent services).  
- ✅ Each component has a single manifest file containing all necessary resources (namespace, configMap, secret, deployment, service, etc.).  
- ✅ DB DML migrations run as init container before application pod startup.  
- ✅ Components deployed in proper namespaces and nodes.  
- ✅ Environment variables passed via ConfigMaps, secrets injected using Kubernetes External Secrets Operator with HashiCorp Vault.  
- ✅ REST API exposed via Kubernetes service and tested successfully (status 200).  
- ✅ README updated with deployment instructions.  