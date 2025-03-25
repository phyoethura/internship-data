# Traefik Setup Instructions

## Step 1: Add Traefik Helm Repository

1. Add the Traefik Helm repository:

    ```bash
    helm repo add traefik https://traefik.github.io/charts
    ```

    - **Explanation**: This command adds the Traefik Helm chart repository to your Helm configuration, allowing you to install charts from this repository.

2. Update your Helm repositories to ensure you have the latest information:

    ```bash
    helm repo update
    ```

    - **Explanation**: This command updates the local Helm chart repository cache with the latest information from the added repositories.

## Step 2: Install Traefik using Helm

1. Install Traefik using Helm:

    ```bash
    helm install traefik traefik/traefik
    ```

    - **Explanation**: This command installs the Traefik Ingress controller in your Kubernetes cluster using the default settings.

## Step 3: Create Deployments and Services

1. Create a file named `playground-deployment-svc.yaml` with the following content:

    ```yaml name=playground-deployment-svc.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: playground-deploy
      name: playground-deploy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: playground-deploy
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: playground-deploy
        spec:
          containers:
          - image: nginx
            name: nginx
            ports:
            - containerPort: 80
            resources: {}
    status: {}

    ---

    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: playground-service
      name: playground-service
    spec:
      ports:
      - port: 8080
        protocol: TCP
        targetPort: 80
      selector:
        app: playground-deploy
      type: ClusterIP
    status:
      loadBalancer: {}
    ```

2. Apply the `playground-deployment-svc.yaml` file:

    ```bash
    kubectl apply -f playground-deployment-svc.yaml
    ```

    - **Explanation**: This command creates a Deployment and Service for the `playground-deploy` application.

3. Create a file named `childplay-deployment-svc.yaml` with the following content:

    ```yaml name=childplay-deployment-svc.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: childplay-deploy
      name: childplay-deploy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: childplay-deploy
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: childplay-deploy
        spec:
          containers:
          - image: nginx
            name: nginx
            ports:
            - containerPort: 80
            resources: {}
    status: {}

    ---

    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: childplay-service
      name: childplay-service
    spec:
      ports:
      - port: 8080
        protocol: TCP
        targetPort: 80
      selector:
        app: childplay-deploy
      type: ClusterIP
    status:
      loadBalancer: {}
    ```

4. Apply the `childplay-deployment-svc.yaml` file:

    ```bash
    kubectl apply -f childplay-deployment-svc.yaml
    ```

    - **Explanation**: This command creates a Deployment and Service for the `childplay-deploy` application.

## Step 4: Create IngressRoute

1. Create a file named `ingressroute.yaml` with the following content:

    ```yaml name=ingressroute.yaml
    apiVersion: traefik.io/v1alpha1
    kind: IngressRoute
    metadata:
      name: ingressroute
      namespace: default
    spec:
      entryPoints:
        - web
      routes:
        - match: Host(`playground.com`)
          kind: Rule
          services:
            - name: playground-service
              port: 8080
        - match: Host(`childplay.com`)
          kind: Rule
          services:
            - name: childplay-service
              port: 8080
    ```

2. Apply the `ingressroute.yaml` file:

    ```bash
    kubectl apply -f ingressroute.yaml
    ```

    - **Explanation**: This command creates an IngressRoute to route traffic to the `playground-service` and `childplay-service` based on the hostnames `playground.com` and `childplay.com`.

## Step 5: Modify Local DNS

1. Edit your local `/etc/hosts` file to map the hostnames to the cluster IP address:

    ```plaintext
    10.111.0.XX playground.com
    10.111.0.XX childplay.com
    ```

    - **Explanation**: This step ensures that your local machine can resolve the hostnames `playground.com` and `childplay.com` to the IP address of your Kubernetes cluster.

---

By following these steps, you will set up Traefik as an Ingress controller, deploy the `playground` and `childplay` applications, and create IngressRoutes to route traffic to these applications based on the specified hostnames.

[Back to Main README](../README.md)