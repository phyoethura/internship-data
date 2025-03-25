# Prometheus and Grafana Setup Instructions

## Step 1: Add Prometheus Community Helm Repository

1. Add the Prometheus Community Helm repository:

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    ```

    - **Explanation**: This command adds the Prometheus Community Helm chart repository to your Helm configuration, allowing you to install charts from this repository.

2. Update your Helm repositories to ensure you have the latest information:

    ```bash
    helm repo update
    ```

    - **Explanation**: This command updates the local Helm chart repository cache with the latest information from the added repositories.

## Step 2: Create values.yaml File

1. Create a file named `values.yaml` with the following content:

    ```yaml name=values.yaml
    alertmanager:
      ## Deploy alertmanager
      ##
      enabled: false

    grafana:
      enabled: true
      #image:
              #tag: 10.1.7
      service:
        type: LoadBalancer

    prometheus:
      prometheusSpec:
        storageSpec:
          volumeClaimTemplate:
            spec:
              accessModes: ["ReadWriteOnce"]
              resources:
                requests:
                  storage: 10Gi
    ```

    - **Explanation**: This file customizes the Helm chart values for deploying Prometheus and Grafana. It disables the Alertmanager, enables Grafana with a LoadBalancer service type, and configures Prometheus to use a PersistentVolumeClaim (PVC) with 10Gi of storage.

## Step 3: Install Prometheus and Grafana using Helm

1. Install the Prometheus and Grafana stack using the custom `values.yaml` file:

    ```bash
    helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack --values values.yaml --create-namespace --namespace monitoring
    ```

    - **Explanation**: This command installs the `kube-prometheus-stack` from the Prometheus Community Helm repository using the custom `values.yaml` file. The `--create-namespace` flag ensures that the `monitoring` namespace is created if it does not already exist. Replace `[RELEASE_NAME]` with a name for your Helm release.
    
## Step 4: Access Grafana UI

1. After the installation, you can access the Grafana UI using the external IP address provided by the LoadBalancer service.

    - **Explanation**: The Grafana service is exposed as a LoadBalancer, which means it will be assigned an external IP address. You can use this IP address to access the Grafana UI in your browser.

    ```bash
    kubectl get svc -n monitoring
    ```

    - **Explanation**: This command retrieves the list of services in the `monitoring` namespace. Look for the external IP address of the Grafana service to access the UI.

---

By following these steps, you will set up Prometheus and Grafana in your Kubernetes cluster. The Grafana service is configured as a LoadBalancer, allowing you to access the Grafana UI using the external IP address.

[Back to Main README](../README.md)