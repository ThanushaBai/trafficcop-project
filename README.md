# 🚦 TrafficCop: Kubernetes API Gateway & DDoS Simulator

**Stop the bad traffic. Protect the backend.**

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![CloudWatch](https://img.shields.io/badge/CloudWatch-FF4F8B?style=for-the-badge&logo=amazoncloudwatch&logoColor=white)

TrafficCop is a cloud-native API Gateway built to protect backend applications from DDoS attacks and malicious payloads. Built using Nginx, FastAPI, and Kubernetes (K3s/AWS EKS), it sits at the edge of the network, enforcing rate limits and Web Application Firewall (WAF) rules before traffic ever reaches the internal application.

## 🚀 Key Features
- **Reverse Proxy Architecture:** The FastAPI backend is completely hidden from the public internet using a Kubernetes `ClusterIP` service. All external traffic must pass through the Nginx Gateway.
- **DDoS Mitigation (Rate Limiting):** Implemented Nginx's Token Bucket algorithm (`limit_req_zone`) to restrict IPs to 3 requests per second, returning `HTTP 429 Too Many Requests` during traffic floods.
- **Web Application Firewall (WAF):** Configured regex-based Nginx rules to instantly block SQL Injection payloads (e.g., `' OR 1=1 --`) with `HTTP 403 Forbidden`.
- **Production-Ready Containerization:** Both the Gateway and Backend are built as lightweight Docker images.
- **Cloud Monitoring:** Integrated AWS CloudWatch Container Insights to monitor CPU and Memory utilization of the EKS cluster and individual pods in real-time.

## 🏗️ Architecture
![TrafficCop Architecture](./screenshots/00-architecture.png)

---

## 📸 Proof of Concept (DDoS & WAF Drill)

Here is the step-by-step execution of normal traffic, a simulated DDoS attack, and a WAF block:

### 1. Normal Traffic (200 OK)
The user sends a standard request to the Gateway. The Gateway checks the rate limit, verifies the payload is safe, and proxies the request to the backend.

![Normal Request](./screenshots/01-normal-request.png)

### 2. DDoS Attack Mitigated (429 Too Many Requests)
I sent 20 rapid requests to the Gateway. The burst limit allowed 6 requests, while the remaining 14 were instantly blocked with HTTP 429 to prevent backend exhaustion.

![DDoS Attack](./screenshots/02-ddos-attack.png)

### 3. WAF Blocking SQL Injection (403 Forbidden)
I attempted to pass a malicious SQL payload (`?id=1' OR '1'='1`) to the backend. The Gateway's WAF regex instantly blocked it, returning a 403 Forbidden before it could reach the database.

![WAF Block](./screenshots/03-waf-block.png)

### 4. Gateway Logs (Evidence of Block)
The Nginx Gateway logs prove the rate limiter and WAF are actively protecting the backend, showing the attacker's IP and the exact reason for the block.

![Gateway Logs](./screenshots/04-gateway-logs.png)

---

## ☁️ AWS Cloud Deployment & Monitoring (EKS, ECR, ELB, CloudWatch)

This application was successfully deployed to a managed **AWS EKS (Elastic Kubernetes Service)** cluster to simulate a production cloud environment.

**Infrastructure Provisioned:**
*   **AWS EKS:** Managed Kubernetes cluster with 2 `t3.small` worker nodes.
*   **AWS ECR:** Private Docker registry used to store the `trafficcop-backend:v1` image.
*   **AWS ELB:** Provisioned dynamically to expose the Nginx Gateway to the public internet.
*   **AWS CloudWatch:** Integrated Container Insights for real-time observability.

**Live Proof:**
![AWS Curl Proof](./screenshots/aws-curl-proof.png)
![AWS EKS Console](./screenshots/aws-eks-console.png)

**CloudWatch Monitoring Proof:**
*Cluster Overview:*
![AWS CloudWatch Cluster](./screenshots/aws-cloudwatch-cluster.png)

*Pod CPU/Memory Metrics During DDoS Attack:*
![AWS CloudWatch Pods](./screenshots/aws-cloudwatch-pods.png)

*(Note: The infrastructure was torn down using `eksctl delete cluster` after testing to ensure zero ongoing cloud costs).*

---

## 🛠️ Tech Stack
**Gateway:** Nginx (Alpine)  
**Backend:** Python (FastAPI)  
**Orchestration:** Kubernetes (K3s), AWS EKS  
**Networking:** NodePort, ClusterIP, Reverse Proxy, WAF, Rate Limiting, AWS ELB  
**Monitoring:** AWS CloudWatch (Container Insights)  
**Registries:** AWS ECR
