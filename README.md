

# Provisioning Amazon EKS using Terraform and Deploying a Kubernetes Application

---

## 1. Introduction

This project demonstrates the end-to-end process of provisioning cloud infrastructure using Infrastructure as Code and deploying a containerized application on a managed Kubernetes cluster.

The workflow includes:

* Creating AWS infrastructure using **Terraform**
* Provisioning a Kubernetes cluster using **Amazon EKS**
* Deploying and exposing an application using **Kubernetes**

---

## 2. Key Concepts

### 2.1 Terraform

**Definition:**
Terraform is an open-source Infrastructure as Code (IaC) tool that allows users to define and provision cloud infrastructure using declarative configuration files.

**Explanation:**
Instead of manually creating resources such as virtual machines, networks, and clusters through a cloud console, Terraform enables automation by defining the desired infrastructure in code.

**Example:**
A VPC, subnets, and an EKS cluster can be created using a single `.tf` file, and Terraform will provision them automatically.

---

### 2.2 Kubernetes

**Definition:**
Kubernetes is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Explanation:**
It ensures applications run reliably by distributing workloads across multiple nodes and handling failures automatically.

**Example:**
If a container crashes, Kubernetes restarts it automatically without manual intervention.

---

### 2.3 Docker vs Kubernetes

**Docker**

**Definition:**
Docker is a platform used to build, package, and run applications inside containers.

**Example:**
Running an NGINX container locally using:

```bash
docker run nginx
```

---

**Kubernetes**

**Definition:**
Kubernetes is used to manage multiple containers across a cluster of machines.

**Example:**
Deploying multiple replicas of an application and automatically balancing traffic between them.

---

**Comparison:**

| Feature        | Docker            | Kubernetes              |
| -------------- | ----------------- | ----------------------- |
| Purpose        | Container runtime | Container orchestration |
| Scaling        | Manual            | Automatic               |
| Fault Recovery | Limited           | Self-healing            |
| Scope          | Single host       | Multi-node cluster      |

---

### 2.4 Amazon EKS

**Definition:**
Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service provided by AWS that simplifies running Kubernetes clusters without managing the control plane.

**Explanation:**
AWS handles cluster management tasks such as updates, availability, and scaling.

**Example:**
A Kubernetes cluster can be created without setting up master nodes manually.

---

## 3. Architecture Overview

```
Terraform → AWS Infrastructure → EKS Cluster → Kubernetes → Application Deployment
```

Components:

* VPC and Subnets for networking
* IAM Roles for permissions
* EKS Cluster for orchestration
* Node Group (EC2 instances) for workload execution
* Kubernetes Deployment and Service for application hosting

---

## 4. Prerequisites

* AWS Account
* AWS CLI configured
* Terraform installed
* kubectl installed

---

## 5. Implementation Steps

### Step 1: Configure AWS CLI

```bash
aws configure
```

Provide:

* Access Key
* Secret Key
* Region: ap-south-1

---

### Step 2: Create IAM Roles

Create two roles in AWS IAM:

**Cluster Role**

* AmazonEKSClusterPolicy

**Node Role**

* AmazonEKSWorkerNodePolicy
* AmazonEC2ContainerRegistryReadOnly
* AmazonEKS_CNI_Policy

---

### Step 3: Terraform Configuration

Create a file named `main.tf`:

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "sub1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "ap-south-1a"
}

resource "aws_subnet" "sub2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "ap-south-1b"
}

resource "aws_eks_cluster" "eks" {
  name     = "simple-eks"
  role_arn = "YOUR_CLUSTER_ROLE_ARN"

  vpc_config {
    subnet_ids = [
      aws_subnet.sub1.id,
      aws_subnet.sub2.id
    ]
  }
}

resource "aws_eks_node_group" "nodes" {
  cluster_name    = aws_eks_cluster.eks.name
  node_group_name = "worker-nodes"
  node_role_arn   = "YOUR_NODE_ROLE_ARN"

  subnet_ids = [
    aws_subnet.sub1.id,
    aws_subnet.sub2.id
  ]

  instance_types = ["t3.micro"]

  scaling_config {
    desired_size = 1
    max_size     = 2
    min_size     = 1
  }
}
```

---

### Step 4: Initialize and Apply Terraform

```powershell
"D:\terraform\terraform.exe" init
```

```powershell
"D:\terraform\terraform.exe" apply
```

Type:

```
yes
```

---

### Step 5: Configure kubectl

```bash
aws eks --region ap-south-1 update-kubeconfig --name simple-eks
```

---

### Step 6: Verify Cluster

```bash
kubectl get nodes
```

Expected output:

* Node status should be **Ready**

---

### Step 7: Deploy Application

**deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

**service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

---

### Step 8: Deploy to Kubernetes

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

### Step 9: Access Application

```bash
kubectl get svc
```

Open the external IP in a browser using HTTP.

---

## 6. Output

The deployed application displays the default NGINX page or custom content served by the container.

---

## 7. Conclusion

This project demonstrates:

* Infrastructure provisioning using Terraform
* Managed Kubernetes deployment using Amazon EKS
* Application deployment and exposure using Kubernetes services

It highlights how modern cloud systems achieve scalability, automation, and reliability.

---

## 8. Screenshots (To Add)

* Terraform apply success
* <img width="1364" height="711" alt="Screenshot 2026-04-16 090817" src="https://github.com/user-attachments/assets/0cb86efb-a9e5-4cf5-93f5-7acdd1a85f48" />

* 
* <img width="1345" height="689" alt="Screenshot 2026-04-16 091722" src="https://github.com/user-attachments/assets/dae91f26-6fc6-4b06-8644-125af3eb6604" />

* Browser output (application page)
* <img width="1126" height="546" alt="Screenshot 2026-04-16 091332" src="https://github.com/user-attachments/assets/31f74018-f5bd-41ec-9a0e-9d0a0bcbc7ea" />
<img width="1345" height="689" alt="Screenshot 2026-04-16 091722" src="https://github.com/user-attachments/assets/cd3989ad-51f1-441d-b072-f743a9c54b57" />
<img width="1312" height="495" alt="Screenshot 2026-04-16 092200" src="https://github.com/user-attachments/assets/333193f3-c84d-45b8-ad8c-83a92da144f3" />
<img width="1356" height="698" alt="Screenshot 2026-04-16 104516" src="https://github.com/user-attachments/assets/4972af90-03d3-43c9-b64b-3aa787250395" />

<img width="1312" height="495" alt="Screenshot 2026-04-16 092200" src="https://github.com/user-attachments/assets/1b59b666-bc70-4b11-923c-4e3eed843b99" />





---


