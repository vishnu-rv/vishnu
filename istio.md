# Istio Traffic Routing with Kubernetes

This documentation covers the step-by-step process to install Istio, configure a Gateway, set up a VirtualService, deploy an application, and verify traffic routing in Kubernetes.

## Prerequisites
- Kubernetes cluster up and running
- `kubectl` installed
- `helm` installed (for Istio installation)

---

## Step 1: Install Istio

1. **Download Istio CLI**
   ```bash
   curl -L https://istio.io/downloadIstio | sh -
   cd istio-*
   export PATH=$PWD/bin:$PATH
   ```

2. **Install Istio in Kubernetes**
   ```bash
   istioctl install --set profile=demo -y
   ```

3. **Enable automatic sidecar injection**
   ```bash
   kubectl label namespace default istio-injection=enabled
   ```

4. **Verify Installation**
   ```bash
   kubectl get pods -n istio-system
   ```
   Ensure `istiod` and `istio-ingressgateway` are running.

---

## Step 2: Deploy Sample Application

1. **Create a Namespace**
   ```bash
   kubectl create namespace boo
   kubectl label namespace boo istio-injection=enabled
   ```

2. **Deploy Application**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: game-2048
     namespace: boo
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: game-2048
     template:
       metadata:
         labels:
           app: game-2048
       spec:
         containers:
         - name: game-2048
           image: alexwhen/docker-2048
           ports:
           - containerPort: 80
   ```
   Apply this YAML file:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Create a Service**
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: game-2048
     namespace: boo
   spec:
     selector:
       app: game-2048
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```
   Apply this YAML file:
   ```bash
   kubectl apply -f service.yaml
   ```

---

## Step 3: Configure Istio Gateway and VirtualService

1. **Create a Gateway**
   ```yaml
   apiVersion: networking.istio.io/v1alpha3
   kind: Gateway
   metadata:
     name: game-2048-gateway
     namespace: boo
   spec:
     selector:
       istio: ingressgateway
     servers:
     - port:
         number: 80
         name: http
         protocol: HTTP
       hosts:
       - "*"
   ```
   Apply this YAML file:
   ```bash
   kubectl apply -f gateway.yaml
   ```

2. **Create a VirtualService**
   ```yaml
   apiVersion: networking.istio.io/v1alpha3
   kind: VirtualService
   metadata:
     name: game-2048
     namespace: boo
   spec:
     gateways:
     - game-2048-gateway
     hosts:
     - "*"
     http:
     - match:
       - uri:
           prefix: "/"
       route:
       - destination:
           host: game-2048
           port:
             number: 80
   ```
   Apply this YAML file:
   ```bash
   kubectl apply -f virtualservice.yaml
   ```

---

## Step 4: Verify Istio Traffic Routing

1. **Check Gateway and VirtualService**
   ```bash
   kubectl get gateway -n boo
   kubectl get virtualservice -n boo
   ```

2. **Get the External IP of Istio Ingress Gateway**
   ```bash
   kubectl get svc -n istio-system
   ```
   Look for `istio-ingressgateway` service and note the **EXTERNAL-IP**.

3. **Test Access to the Application**
   ```bash
   curl -v http://<EXTERNAL-IP>/
   ```

4. **Check Istio Routing**
   ```bash
   istioctl proxy-config routes <POD_NAME> -n boo
   ```
   Example:
   ```bash
   istioctl proxy-config routes game-2048-f86fbf9fc-lp4lv -n boo
   ```

5. **Check Logs for Debugging**
   - **Gateway Logs:**
     ```bash
     kubectl logs -n istio-system -l istio=ingressgateway
     ```
   - **Service Details:**
     ```bash
     kubectl get svc game-2048 -n boo -o yaml
     ```
   - **Sidecar Proxy Logs:**
     ```bash
     kubectl logs game-2048-f86fbf9fc-lp4lv -n boo -c istio-proxy
     ```

---

## Conclusion
You have successfully set up Istio for traffic routing in Kubernetes using **Gateways** and **VirtualServices**. This setup allows you to control and observe traffic flow within your cluster efficiently.

For troubleshooting, check logs and verify that Istio sidecars are injected correctly.

**Happy Deploying! 🚀**

