# Exercise 7 – ALB Ingress Failure (AWS Load Balancer Controller)

## Objective

Deploy an application using AWS Load Balancer Controller and investigate an ALB Ingress issue. Learn how the controller discovers VPC subnets, creates an Application Load Balancer, and registers Kubernetes targets.

---

## Scenario

### Incident

Users reported that the application was inaccessible through the ALB.

**Reported Error**

```
504 Gateway Timeout
```

### Ingress Configuration

```yaml
alb.ingress.kubernetes.io/target-type: ip
```

### Expected Symptoms

```
Target registration failed
Unable to discover subnets
```

---

# Technologies Used

* Amazon EKS
* Kubernetes
* AWS Load Balancer Controller
* IAM Roles for Service Accounts (IRSA)
* Helm
* AWS CLI
* eksctl

---

# Project Structure

```
ALB-Ingress-Failure/
│
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── screenshots/
└── README.md
```

---

# Architecture

```
                    Internet
                        │
                        ▼
          AWS Application Load Balancer
                        │
      AWS Load Balancer Controller
                        │
                Kubernetes Ingress
                        │
                ClusterIP Service
                        │
                  Nginx Deployment
                        │
                      Pods
```

---

# Steps Performed

## 1. Created EKS Cluster

Created an Amazon EKS cluster using eksctl.

```
eksctl create cluster
```

---

## 2. Associated IAM OIDC Provider

Enabled IAM OIDC Provider for IRSA.

```
eksctl utils associate-iam-oidc-provider
```

---

## 3. Created IAM Policy

Created AWS Load Balancer Controller IAM policy.

```
aws iam create-policy
```

---

## 4. Created IAM Service Account

Configured IAM Role for Service Account (IRSA).

```
eksctl create iamserviceaccount
```

Verified:

```
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```

---

## 5. Installed AWS Load Balancer Controller

Installed using Helm.

```
helm install aws-load-balancer-controller
```

Verified controller pods.

```
kubectl get pods -n kube-system
```

---

## 6. Deployed Sample Application

Created

* Deployment
* Service
* Ingress

Applied using

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

---

## 7. Verified Ingress

```
kubectl get ingress
```

Verified ALB DNS Name.

---

## 8. Investigated Controller Logs

```
kubectl logs deployment/aws-load-balancer-controller -n kube-system
```

Observed successful reconciliation of resources.

---

# Investigation

The exercise scenario expected:

```
504 Gateway Timeout
Target registration failed
Unable to discover subnets
```

However, during implementation the controller successfully discovered the required VPC subnets because the cluster was created using **eksctl**, which automatically applied the required subnet tags.

Controller logs showed:

* Successful subnet discovery
* Target group creation
* Target registration
* Load Balancer creation
* Successful reconciliation

No subnet discovery errors were observed.

---

# Root Cause (Expected Failure Scenario)

The "Unable to discover subnets" error generally occurs when:

* Required subnet tags are missing
* Incorrect VPC is configured
* ALB Controller IAM permissions are missing
* Incorrect AWS Region is configured
* Ingress configuration is invalid

---

# Resolution

Verified:

* IAM OIDC Provider
* IAM Policy
* IRSA Configuration
* AWS Load Balancer Controller Installation
* VPC Configuration
* Ingress Resource
* Service Resource
* Deployment
* Controller Logs

The Application Load Balancer was created successfully and the application became accessible.

---

# Useful Commands

Check Ingress

```
kubectl get ingress
```

Describe Ingress

```
kubectl describe ingress nginx-ingress
```

View Controller Logs

```
kubectl logs deployment/aws-load-balancer-controller -n kube-system
```

Check Service

```
kubectl get svc
```

Check Pods

```
kubectl get pods
```

Check Controller

```
kubectl get pods -n kube-system
```

---

# Key Learnings

* Working with AWS Load Balancer Controller
* Installing Controller using Helm
* IAM Roles for Service Accounts (IRSA)
* OIDC Provider configuration
* ALB Ingress architecture
* Target Group registration
* Controller log analysis
* Troubleshooting subnet discovery issues
* Understanding AWS networking for Kubernetes

---

# Outcome

Successfully configured the AWS Load Balancer Controller on Amazon EKS, deployed an application using Kubernetes Ingress, validated automatic ALB creation, and investigated the expected failure scenario. Although the subnet discovery issue did not occur because the environment was correctly configured, the exercise demonstrated the deployment workflow and the troubleshooting steps required to diagnose ALB Ingress issues in production environments.
