# homelab
My homelab

# Kubernetes 💪🏻

Notes from my own k3s K8s deployment containing boilerplate for a manifest based deployment for truly declaritive infrastructure **(IaC)**
Having experienced bare-metal linux, vm's and containers, I am a big fan of Kubernetes as it accomplishes so many things elegently.
I try to stick to using declaritive infrastructure **(IaC)** as much as possible, so the work below is done with IaC in mind with everything being done using yaml manifests. 

## What is Kubernetes 🤔

Kubernetes is an open-source platform for managing and automating the deployment, scaling, and operation of containerized applications. It simplifies running applications by organizing them into manageable units called pods, distributing them across servers, and handling tasks like scaling, updates, and recovery if something goes wrong.

## What is KubeCTL 🤔

Kubectl is the command-line tool used to interact with Kubernetes clusters, allowing you to manage applications, inspect resources, and troubleshoot issues. It’s essential for getting started because it provides a straightforward way to control Kubernetes with simple commands.

## Concepts 🤔
    
- pod = image
- replica set = copies
- deployment (Multi-Node-Production-Env) = image, copies, labels, mounts, namespaces, container spec, cpu, mem, auto-healing
- self healing = build from manifest

## Links 🔗

- https://docs.k3s.io/quick-start

## Install K3S 💾

Source of installl script
https://docs.k3s.io/quick-start

## Install Script 📜

    curl -sfL https://get.k3s.io | sh -

## Change permissions 🔑

    sudo chmod 644 /etc/rancher/k3s/k3s.yaml

## Directory Structure 📖

    kubernetes/ (Project Name) eg. ProLUG
      ├── namespaces/
      │   ├── dev-namespace.yaml
      │   ├── testing-namespace.yaml
      │   ├── production-namespace.yaml
      ├── pods/
      │   ├── dev-pod.yaml
      │   ├── testing-pod.yaml
      │   ├── production-pod.yaml
      ├── services/
      │   ├── production-service.yaml
      ├── ingress/
      │   ├── ingress-rules.yaml   # Optional if you use Ingress
      ├── configmaps/
      │   ├── dev-config.yaml      # Optional for configuration management
      │   ├── production-config.yaml
      ├── secrets/
          ├── dev-secrets.yaml     # Optional for sensitive data
          ├── prod-secrets.yaml


## YAML Files

    namespaces.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
    ---
    apiVersion: v1
    kind: Namespace
    metadata:
      name: testing
    ---
    apiVersion: v1
    kind: Namespace
    metadata:
      name: production
    
    production-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: production-service
      namespace: production
    spec:
      type: NodePort
      selector:
        environment: production
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          nodePort: 30080

## pods

### Prod Env

    production-environment.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: production-pod
      labels:
        environment: production
        app: commercial-site
    spec:
      containers:
        - name: production-container
          image: rockylinux:9
          command: ["/bin/sh", "-c"]
          args: ["while true; do echo 'Production Environment - Serving Customers'; sleep 10; done"]
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: "2"
              memory: "2Gi"
            requests:
              cpu: "1"
              memory: "1Gi"
              
### Testing Env

    testing-environment.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: testing-pod
      labels:
        environment: testing
        app: commercial-site
    spec:
      containers:
        - name: testing-container
          image: rockylinux:9
          command: ["/bin/sh", "-c"]
          args: ["while true; do echo 'Testing Environment - Validating Content'; sleep 10; done"]
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: "1"
              memory: "1Gi"
            requests:
              cpu: "0.5"
              memory: "512Mi"
              
### Dev Env

    dev-environment.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: dev-pod
      labels:
        environment: dev
        app: commercial-site
    spec:
      containers:
        - name: dev-container
          image: rockylinux:9
          command: ["/bin/sh", "-c"]
          args: ["while true; do echo 'Dev Environment - Serving Content'; sleep 10; done"]
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: "0.5"
              memory: "512Mi"
            requests:
              cpu: "0.25"
              memory: "256Mi"

## Important Commands (Red)

    - Check/Manage Pods
    - kubectl get pods
    - kubectl describe pod <pod-name>
    - kubectl cluster-info
    - kubectl describe pod <pod-name>
    - kubectl logs <pod-name>
    - kubectl attach <pod-name>
    - kubectl exec -it <pod-name> -- /bin/sh

### Changing Contexts (namespace)

    kubectl config use-context <context-name>
    kubectl config current-context

### Apply env specific Namespace

    kubectl apply -f dev-environment.yaml --namespace=dev
    kubectl apply -f testing-environment.yaml --namespace=testing
    kubectl apply -f production-environment.yaml --namespace=production

### Verify namespaces

    kubectl get pods --namespace=dev
    kubectl get pods --namespace=testing
    kubectl get pods --namespace=production

### Apply pods by specific Directory

    kubectl apply -f pods/
    kubectl apply -f services/
    kubectl apply -f namespaces/

### List pods by Namespace

    kubectl get pods --namespace=dev
    kubectl get pods --namespace=testing
    kubectl get pods --namespace=production

## Configure KUBECONFIG for the non root user
This is needed as once k3s restarts, permissions are lost. One must create persistent permissions.

    mkdir -p ~/.kube 

    sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config 

    echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc 

    source ~/.bashrc

    sudo chown $(id -u):$(id -g) ~/.kube/config

### Create alias for kubectl to save typing

    echo 'alias k=kubectl' >> ~/.bashrc

### Connect 'k' alias to 'kubectl' auto complete

    complete -o default -F __start_kubectl k

### Send Kubectl Tab Completion to .bashrc

    echo "source <(kubectl completion bash)" >> ~/.bashrc

### Restart config

    source ~/.bashrc 

---

## When shit hits the fan 💩🪭

    kubectl exec -it dev-pod -c dev-container -- /bin/bash
