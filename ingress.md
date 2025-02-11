Kubernetes Service and Load Balancing Mechanisms
In Kubernetes (K8s), services provide a way to expose applications to the outside world. There are three primary ways to expose services:

NodePort

Allows access to a service using <NodeIP>:<NodePort>.
Works within the organization but not ideal for public access.
Can cause port conflicts as each service must use a unique high-range port (30000-32767).
Not preferred for production due to lack of scalability and security limitations.
LoadBalancer (LB)

Relies on the cloud provider to provision a public Load Balancer (e.g., AWS ALB, GCP LB, Azure LB).
The Cloud Controller Manager (CCM) in managed Kubernetes handles this process.
Drawbacks:
High cost since each service gets a separate Load Balancer.
Limited security control in managed K8s (e.g., cannot use custom WAFs, firewalls like Cisco, NGINX, F5, Palo Alto).
In managed K8s, only the provider’s Load Balancer (like AWS ALB) is supported.
Ingress (Recommended for Advanced Load Balancing)

Ingress provides external access to multiple services via a single Load Balancer.
Supports advanced traffic management, such as:
Path-based and host-based routing.
SSL termination.
Rate limiting and authentication.
Requires an Ingress Controller (e.g., NGINX, Traefik, HAProxy) to function.
Works independently of cloud providers, making it more flexible and cost-effective.
Understanding Ingress and Ingress Controller
Ingress: A Kubernetes resource (YAML configuration) that defines rules for routing external traffic to internal services.
Ingress Controller: A component that watches Ingress resources and configures the Load Balancer accordingly.
It is not a Load Balancer itself but automates LB configuration.
Written in Go and deployed as a separate service in the cluster.
Examples:
NGINX Ingress Controller (Open-source, widely used).
Traefik (Dynamic and lightweight).
AWS ALB Ingress Controller (Integrates with AWS).
Why Use Ingress Over LoadBalancer in Managed K8s?
In managed Kubernetes, the default LoadBalancer method is round-robin, which may not suit enterprise needs.
Managed Kubernetes does not support external custom LBs (Cisco, NGINX, etc.).
With Ingress, we can:
Use a single public Load Balancer instead of multiple LBs (cost-efficient).
Implement custom routing policies (e.g., sticky sessions, weighted routing).
Integrate custom security tools (WAFs, authentication layers).
Conclusion
If deploying on cloud (AWS, GCP, Azure) and using managed Kubernetes, Ingress is the best option for cost savings and flexibility.
If deploying on VMs or bare-metal Kubernetes, Security Groups (SGs), WAFs, and custom firewalls can be used instead.
Ingress Controller simplifies advanced Load Balancing while still allowing custom configurations based on business needs.
