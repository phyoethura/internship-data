# Kubernetes and MetalLB Setup Instructions

## Edit kube-proxy ConfigMap

1. Edit the `kube-proxy` ConfigMap in the `kube-system` namespace to set `strictARP` to `true`:

    ```bash
    kubectl edit configmap -n kube-system kube-proxy
    ```

    - **Explanation**: This command opens the `kube-proxy` ConfigMap in your default editor. You need to add the line `strictARP: true` to ensure that ARP replies are only sent for IP addresses that are local to the node.

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: kube-proxy
      namespace: kube-system
    data:
      config.conf: |
        ...
        strictARP: true
        ...
    ```

## Install MetalLB

2. Apply the MetalLB namespace and deployment manifests:

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.1/manifests/namespace.yaml
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.1/manifests/metallb.yaml
    ```

    - **Explanation**: These commands download and apply the MetalLB manifests from the specified URLs, creating the necessary resources in your cluster.

3. Verify the MetalLB pods are running in the `metallb-system` namespace:

    ```bash
    kubectl get po -n metallb-system
    ```

    - **Explanation**: This command lists the pods in the `metallb-system` namespace to ensure that MetalLB is running correctly.

## Configure MetalLB

4. Navigate to the directory containing your MetalLB configuration file:

    ```bash
    cd path/to/metallb-old
    ```

5. Edit the `metalLBconfigmap.yml` file to define the IP address range:

    ```bash
    vi metalLBconfigmap.yml
    ```

    - **Explanation**: This command opens the `metalLBconfigmap.yml` file in the `vi` editor. Update the file with your specific IP address range.

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 10.111.0.33-10.111.0.59
    ```

6. Apply the MetalLB configuration:

    ```bash
    kubectl apply -f metalLBconfigmap.yml
    ```

    - **Explanation**: This command applies your updated MetalLB configuration to the cluster.

## Update Service Type

7. Edit the service type of your Kubernetes services to "LoadBalancer":

    - **Explanation**: Change the `spec.type` field of your service definitions to `LoadBalancer` to enable them to use the IPs managed by MetalLB.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
      namespace: default
    spec:
      type: LoadBalancer
      ...
    ```

## Add a 100 GB Hard Disk to Each VM

8. Add a 100 GB hard disk to each VM using the VM UI:

    - **Explanation**: Follow the UI instructions of your VM management tool to add a new hard disk to each node.

## Install ZFS on Each Node

9. Install ZFS on each node:

    ```bash
    apt-get install zfsutils-linux -y
    ```

    - **Explanation**: This command installs ZFS utilities on each node, which are required for creating ZFS pools.

10. Create a ZFS pool on each node:

    ```bash
    zpool create data-pool /dev/sdb
    ```

    - **Explanation**: This command creates a ZFS pool named `data-pool` using the newly added hard disk `/dev/sdb`.

## Deploy OpenEBS ZFS Operator

11. Apply the OpenEBS ZFS operator on one node:

    ```bash
    kubectl apply -f https://openebs.github.io/charts/zfs-operator.yaml
    ```

    - **Explanation**: This command deploys the OpenEBS ZFS operator, which manages ZFS-based storage classes in your Kubernetes cluster.

## Create the OpenEBS ZFS StorageClass

12. Create a StorageClass for OpenEBS ZFS:

    ```yaml
    allowVolumeExpansion: true
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
      name: openebs-zfs-data
    parameters:
      compression: "off"
      dedup: "off"
      fstype: xfs
      poolname: data-pool
      recordsize: 4k
    provisioner: zfs.csi.openebs.io
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
    ```

    - **allowVolumeExpansion: true**: Allows resizing of persistent volumes.
    - **apiVersion: storage.k8s.io/v1**: Specifies the API version.
    - **kind: StorageClass**: Indicates this resource is a StorageClass.
    - **metadata**:
      - **annotations**: Marks this StorageClass as the default for the cluster.
      - **name: openebs-zfs-data**: The name of the StorageClass.
    - **parameters**:
      - **compression: "off"**: Disables compression for volumes.
      - **dedup: "off"**: Disables deduplication for volumes.
      - **fstype: xfs**: Specifies the filesystem type as XFS.
      - **poolname: data-pool**: The name of the ZFS pool created on the nodes.
      - **recordsize: 4k**: Sets the record size for the ZFS pool.
    - **provisioner: zfs.csi.openebs.io**: The CSI driver for provisioning volumes.
    - **reclaimPolicy: Delete**: Deletes the persistent volume when it is released.
    - **volumeBindingMode: WaitForFirstConsumer**: Waits for a pod to request the volume before binding it.

13. Apply the StorageClass configuration:

    ```bash
    kubectl apply -f openebs-storageclass.yaml
    ```

    - **Explanation**: This command applies the StorageClass configuration to your Kubernetes cluster, enabling dynamic provisioning of persistent volumes using the OpenEBS ZFS CSI driver.

---

These instructions guide you through setting up MetalLB for load balancing, configuring ZFS storage on your nodes, deploying the OpenEBS ZFS operator, and creating a StorageClass for dynamic volume provisioning.

[Back to Main README](../README.md)