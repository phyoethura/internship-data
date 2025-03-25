# Install Ingress-Nginx

## Step 1: Create PersistentVolumeClaim (PVC)

1. Create a file named `createpvc.yaml` with the following content:

    ```yaml name=createpvc.yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: openebs-zfs-data
    ```

2. Apply the PVC:

    ```bash
    kubectl apply -f createpvc.yaml
    ```

    - **Explanation**: This command creates a PersistentVolumeClaim (PVC) named `my-pvc` with a storage request of 1Gi using the `openebs-zfs-data` StorageClass. The PVC allows ReadWriteOnce access mode.

## Step 2: Apply IngressClass

1. Create a file named `ingressclass.yaml` with the following content:

    ```yaml name=ingressclass.yaml
    apiVersion: networking.k8s.io/v1
    kind: IngressClass
    metadata:
      name: nginx
    spec:
      controller: k8s.io/ingress-nginx
    ```

2. Apply the IngressClass:

    ```bash
    kubectl apply -f ingressclass.yaml
    ```

    - **Explanation**: This command creates an IngressClass named `nginx` which uses the `k8s.io/ingress-nginx` controller.

## Step 3: Apply Deployment and Service for deploy-01

1. Create a file named `deployment-svc-01.yaml` with the following content:

    ```yaml name=deployment-svc-01.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: deploy-01
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: deploy-app-01
      template:
        metadata:
          labels:
            app: deploy-app-01
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
            volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: nginx-storage
          volumes:
          - name: nginx-storage
            persistentVolumeClaim:
              claimName: my-pvc

    ---

    apiVersion: v1
    kind: Service
    metadata:
      name: service-no1
      labels:
        app: nginx
    spec:
      selector:
        app: deploy-app-01
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: LoadBalancer
    ```

2. Apply the Deployment and Service:

    ```bash
    kubectl apply -f deployment-svc-01.yaml
    ```

    - **Explanation**: This command creates a Deployment named `deploy-01` with one replica. It uses the `nginx:latest` image and mounts the `my-pvc` PVC to `/usr/share/nginx/html`. The associated Service named `service-no1` exposes the Deployment via a LoadBalancer on port 80.

## Step 4: Apply Deployment and Service for deploy-02

1. Create a file named `deployment-svc-02.yaml` with the following content:

    ```yaml name=deployment-svc-02.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: deploy-02
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: deploy-app-02
      template:
        metadata:
          labels:
            app: deploy-app-02
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
            volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: nginx-storage
          volumes:
          - name: nginx-storage
            persistentVolumeClaim:
              claimName: my-pvc

    ---

    apiVersion: v1
    kind: Service
    metadata:
      name: service-no2
      labels:
        app: nginx
    spec:
      selector:
        app: deploy-app-02
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: LoadBalancer
    ```

2. Apply the Deployment and Service:

    ```bash
    kubectl apply -f deployment-svc-02.yaml
    ```

    - **Explanation**: This command creates a Deployment named `deploy-02` with one replica. It uses the `nginx:latest` image and mounts the `my-pvc` PVC to `/usr/share/nginx/html`. The associated Service named `service-no2` exposes the Deployment via a LoadBalancer on port 80.

## Step 5: Apply Ingress

1. Create a file named `ingress.yaml` with the following content:

    ```yaml name=ingress.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: lucky-ingress
      namespace: default
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      ingressClassName: nginx
      rules:
      - host: luckyy.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-no1
                port:
                  number: 8080
      - host: luckyy-2.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-no2
                port:
                  number: 8080
    ```

2. Apply the Ingress:

    ```bash
    kubectl apply -f ingress.yaml
    ```

    - **Explanation**: This command creates an Ingress resource named `lucky-ingress` with rules for routing traffic to `service-no1` and `service-no2` based on the hostnames `luckyy.com` and `luckyy-2.com`. The rewrite-target annotation ensures that all paths are redirected to the root.

## Final Step: Edit Hosts File

1. Edit your local `/etc/hosts` file to map the hostnames to your cluster's IP address:

    ```plaintext
    <cluster_ip> luckyy.com
    <cluster_ip> luckyy-2.com
    ```

    - **Explanation**: This step ensures that your local machine can resolve the hostnames `luckyy.com` and `luckyy-2.com` to the IP address of your Kubernetes cluster.

---

By following these steps, you can set up PersistentVolumeClaims, IngressClass, Deployments, Services, and Ingress in your Kubernetes cluster and access the Nginx UI using the specified hostnames through Ingress-NGINX.

[Back to Main README](../README.md)