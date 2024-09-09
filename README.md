# Ingress-Controller-demo

This example will guide you through setting up an NGINX Ingress Controller and creating an Ingress resource to route traffic to two different services.

### **Step-by-Step Guide to Setting Up an Ingress Controller**

#### **Step 1: Prerequisites**

Before we start, ensure you have the following:

- A Kubernetes cluster (e.g., Minikube, Kubernetes on Docker Desktop, or any cloud provider like GKE, EKS, or AKS).
- `kubectl` command-line tool configured to interact with your cluster.

#### **Step 2: Deploy NGINX Ingress Controller**

You can deploy the NGINX Ingress Controller using the official Kubernetes manifests. Run the following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

This command deploys the NGINX Ingress Controller in your cluster. It creates a pod, service, and necessary resources to handle incoming traffic.

#### **Step 3: Deploy Sample Applications**

Deploy two sample applications in your Kubernetes cluster. We'll use two simple NGINX web servers as services.

**1. Deploy the first application:**

```yaml
# app1.yaml
apiVersion: v1
kind: Service
metadata:
  name: app1
  labels:
    app: app1
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: app1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Apply this configuration:

```bash
kubectl apply -f app1.yaml
```

**2. Deploy the second application:**

```yaml
# app2.yaml
apiVersion: v1
kind: Service
metadata:
  name: app2
  labels:
    app: app2
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: app2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Apply this configuration:

```bash
kubectl apply -f app2.yaml
```

#### **Step 4: Create an Ingress Resource**

Create an Ingress resource to define the routing rules for incoming traffic.

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2
            port:
              number: 80
```

Apply this configuration:

```bash
kubectl apply -f ingress.yaml
```

#### **Step 5: Test the Ingress Routing**

To test this setup, you need to modify your local `/etc/hosts` file (or equivalent) to point `myapp.local` to the IP address of your Ingress Controller. To find the IP, run:

```bash
kubectl get services -o wide -w -n ingress-nginx
```

Edit your `/etc/hosts` file and add an entry like:

```plaintext
<Ingress Controller IP> myapp.local
```

#### **Step 6: Access Your Applications**

Open your browser and navigate to:

- `http://myapp.local` → This should route to **app1**.
- `http://myapp.local/app2` → This should route to **app2**.

### **Summary of What Happened**

1. **Ingress Controller**: NGINX Ingress Controller was deployed to handle incoming requests.
2. **Applications**: Two NGINX-based applications were deployed as separate services (`app1` and `app2`).
3. **Ingress Resource**: The Ingress resource defined routing rules:
   - Requests to `/` go to `app1`.
   - Requests to `/app2` go to `app2`.
4. **Routing**: The Ingress Controller handled the routing based on the Ingress resource configuration.

This setup demonstrates how Ingress Controllers manage and route incoming traffic to different services within a Kubernetes cluster, providing flexible and scalable ways to expose your applications.
