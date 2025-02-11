# Kubernetes Service and Load Balancing Mechanisms

## 1. NodePort
- Allows access to a service using `<NodeIP>:<NodePort>`.
- Works within the organization but not ideal for public access.
- Can cause port conflicts as each service must use a unique high-range port (30000-32767).
- Not preferred for production due to lack of scalability and security limitations.

## 2. LoadBalancer (LB)
- Relies on the cloud provider to provision a public Load Balancer (e.g., AWS ALB, GCP LB, Azure LB).
- The **Cloud Controller Manager (CCM)** in managed Kubernetes handles this process.
- Drawbacks:
  - High cost since each service gets a separate Load Balancer.
  - Limited security control in managed K8s (e.g., cannot use custom WAFs, firewalls like Cisco, NGINX, F5, Palo Alto).
  - In managed K8s, only the provider’s Load Balancer (like AWS ALB) is supported.

## 3. Ingress (Recommended for Advanced Load Balancing)
- Ingress provides external access to multiple services via a **single Load Balancer**.
- Supports advanced traffic management, such as:
  - Path-based and host-based routing.
  - SSL termination.
  - Rate limiting and authentication.
- Requires an **Ingress Controller** (e.g., NGINX, Traefik, HAProxy) to function.
- Works independently of cloud providers, making it more flexible and cost-effective.

### Understanding Ingress and Ingress Controller
- **Ingress**: A Kubernetes resource (YAML configuration) that defines rules for routing external traffic to internal services.
- **Ingress Controller**: A component that watches Ingress resources and configures the Load Balancer accordingly.
  - It is **not** a Load Balancer itself but automates LB configuration.
  - Examples:
    - **NGINX Ingress Controller** (Open-source, widely used).
    - **Traefik** (Dynamic and lightweight).
    - **AWS ALB Ingress Controller** (Integrates with AWS).

## Why Use Ingress Over LoadBalancer in Managed K8s?
- Managed Kubernetes does **not** support external custom LBs (Cisco, NGINX, etc.).
- With **Ingress**, we can:
  - Use **a single public Load Balancer** instead of multiple LBs (cost-efficient).
  - Implement **custom routing policies** (e.g., sticky sessions, weighted routing).
  - Integrate **custom security tools** (WAFs, authentication layers).

## Why Does Kubernetes LoadBalancer Service Create an ALB Instead of an NLB?
### 1. Cloud Provider Defaults to ALB in Managed Kubernetes
- AWS **Elastic Kubernetes Service (EKS)** provisions an **ALB** when a `LoadBalancer` service is created.
- The default integration of `kube-proxy` with AWS assumes that an **ALB** is more suitable for HTTP/HTTPS traffic.

### 2. Difference Between ALB and NLB

| Feature         | **ALB (Application Load Balancer)** | **NLB (Network Load Balancer)** |
|---------------|----------------------------------|--------------------------------|
| **Protocol**   | HTTP, HTTPS, WebSockets         | TCP, UDP, TLS                  |
| **Routing**    | Path-based & Host-based        | Direct TCP/UDP forwarding       |
| **Performance**| Layer 7 (slightly slower)      | Layer 4 (faster, low latency)  |
| **IP Mode**    | Allocates a DNS name          | Supports static IPs            |
| **TLS Termination** | Supported (offloads SSL to ALB) | Not supported (SSL must be handled in the app) |

### 3. How to Force Kubernetes to Use an NLB Instead of ALB?
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nlb-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 4. When Should You Use ALB vs. NLB?
✅ **Use ALB if:**
- You need **path-based routing** (`/api` → Service A, `/web` → Service B).
- You need **TLS termination** at the Load Balancer.
- You are serving **HTTP/HTTPS traffic**.

✅ **Use NLB if:**
- You need **low latency** (Layer 4 direct forwarding).
- You require **static IPs** for firewall rules.
- You are handling **TCP/UDP workloads** (like databases, WebSockets, or gRPC services).

## Conclusion
- By default, Kubernetes `LoadBalancer` service in **AWS creates an ALB** because it's optimized for HTTP/HTTPS traffic.
- If you need an **NLB**, you must explicitly add the annotation `service.beta.kubernetes.io/aws-load-balancer-type: "nlb"`.
- The choice depends on your use case: **ALB for web apps, NLB for low-latency TCP/UDP services**.

