# 02-Service Mesh with HashiCorp Consul

> **Student ID:** 6510545624 

> **Name:** Pichayanon Toojinda

This project demonstrates setting up a Kubernetes-based multi-cloud environment using HashiCorp Consul as a service mesh for secure service-to-service communication, cross-cluster federation, and automatic failover.

It is based on:
- [GoogleCloudPlatform - microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
- [TechWorld with Nana - Consul Crash Course](https://www.youtube.com/watch?v=s3I1kKKfjtQ)

---

## Project Structure

| Path                       | Description |
|:---------------------------|:------------|
| `/microservices-demo-gcp/` | Source code of each microservice (copied from GCP microservices-demo) |
| `/kubernetes/`             | Consul Helm chart values, mesh gateway configs, and federation configurations |
| `/terraform/`              | Terraform scripts for provisioning Kubernetes clusters |
| `README.md`                | Documentation for building, deploying, and testing the project |

---

## Setup Infrastructure Using Terraform

### 1. Clone the Repository

```bash
git clone https://github.com/Pichayanon/02-ServiceMeshwithHashiCorpConsul.git
cd 02-ServiceMeshwithHashiCorpConsul
```

---

### 2. Prepare `terraform.tfvars`

Create a file named `terraform.tfvars` inside the `terraform/` directory with the following content:

```hcl
aws_region         = "<your-aws-region>"
aws_access_key_id  = "<your-aws-access-key-id>"
aws_secret_access_key = "<your-aws-secret-access-key>"
```
---

### 3. Initialize Terraform

```bash
cd terraform/
terraform init
```

---

### 4. Apply Terraform to Create the EKS Cluster

```bash
terraform apply -var-file=terraform.tfvars
```

Confirm with `yes` when prompted.  
Terraform will create the Kubernetes cluster in your selected AWS region.

---

### 5. Move Back to Root Directory

```bash
cd ..
```

---

## Deploy Microservices App on EKS

### 1. Update Kubeconfig to Connect to EKS Cluster

Update your local kubeconfig (replace with your own AWS region):

```bash
aws eks update-kubeconfig --region <your-aws-region> --name myapp-eks-cluster
```
Example:

```bash
aws eks update-kubeconfig --region ap-southeast-1 --name myapp-eks-cluster
```


Verify the connection:

```bash
kubectl get nodes
```

---

### 2. Deploy the Microservices Application

Deploy the application using the provided configuration:

```bash
cd kubernetes
kubectl apply -f config.yaml
```
> Note: This deploys microservices from the GCP microservices-demo using publicly available Docker images. No need to build from source.

Verify that pods and services are running:

```bash
kubectl get pods
kubectl get svc
```

---

## Deploy Consul on EKS

### 1. Install Consul on EKS

Add HashiCorp Helm Repository:

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
```

Install Consul using Helm:

```bash
helm install eks hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=eks
```

Verify Consul installation:

```bash
kubectl get pods
kubectl get all
```

---

### 2. Connect Microservices with Consul

Remove previous services:

```bash
kubectl delete -f config.yaml
```

Apply Consul federation configurations:

```bash
kubectl apply -f config-consul.yaml
```

Check logs of the microservice pod (replace with your service pod name:

```bash
kubectl logs <service-pod-name>
kubectl logs <service-pod-name> -c consul-dataplane
```
Example:

```bash
kubectl logs adservice-5f8c767f66-fmg8b
kubectl logs adservice-5f8c767f66-fmg8b -c consul-dataplane
```

---


## Create and Connect to 2nd Kubernetes Cluster

### 1. Set KUBECONFIG to connect to second cluster

Open a new terminal (LKE Cluster Terminal)

```bash
cd kubernetes
```

Set the kubeconfig path for the second Kubernetes cluster (replace with your path to kubeconfig.yaml):

```bash
export KUBECONFIG=<path-to-lke-kubeconfig>
chmod 700 <path-to-lke-kubeconfig>
kubectl get nodes
```
> Note: Ensure that your kubeconfig file for LKE is downloaded correctly

Example:

```bash
export KUBECONFIG=~/Downloads/lke-consul-kubeconfig.yaml
chmod 700 ~/Downloads/lke-consul-kubeconfig.yaml
kubectl get nodes
```

---

### 2. Deploy Consul and Microservices on Second Cluster

Install Consul:

```bash
helm install lke hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=lke
```

Verify Consul installation:

```bash
kubectl get all
kubectl get crd
kubectl get crd | grep consul
```

Deploy Microservices Application on LKE:

```bash
kubectl apply -f config-consul.yaml
```

Verify that pods and services are running:

```bash
kubectl get pods
kubectl get svc
```

---

## Connect the Clusters - Add Peer Connection

Apply mesh gateway configuration:

```bash
kubectl apply -f consul-mesh-gateway.yaml
kubectl get mesh
```

---

## Configure Failover to Other Cluster

On EKS Terminal Simulate service failure:

```bash
kubectl delete deployment shippingservice
kubectl get pods
```

On EKS Terminal Re-apply configuration:

```bash
kubectl apply -f config-consul.yaml
kubectl get pods
```

On LKE Terminal Configure exported service:

```bash
kubectl apply -f exported-service.yaml
```

On EKS Terminal Apply service resolver for failover:

```bash
kubectl apply -f service-resolver.yaml
```

On EKS Terminal Simulate failover again:

```bash
kubectl delete deployment shippingservice
```
> Note: After deleting shippingservice, requests should automatically failover to the service running in LKE Cluster
---


## Acknowledgments

This project incorporates and adapts work from:
- [GoogleCloudPlatform - microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
- [TechWorld with Nana - Consul Crash Course](https://www.youtube.com/watch?v=s3I1kKKfjtQ)

Credits to the original authors for their contributions.

---

## References

- [HashiCorp Consul Documentation](https://www.consul.io/docs)
- [GCP Microservices Demo Documentation](https://github.com/GoogleCloudPlatform/microservices-demo)
- [TechWorld with Nana - YouTube](https://www.youtube.com/c/TechWorldwithNana)

---
