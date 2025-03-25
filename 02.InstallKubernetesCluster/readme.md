

# Steps to Set Up the Kubernetes Cluster Environment

## On the Ansible VM

### Add Three Nodes with IP Addresses

1. Open the `/etc/hosts` file:

    ```bash
    sudo vi /etc/hosts
    ```

2. Copy and paste the following content:

    ```yaml
    127.0.0.1 localhost
    127.0.1.1 luckyansible
    10.111.0.27 wky-node01
    10.111.0.28 wky-node02
    10.111.0.29 wky-node03

    # The following lines are desirable for IPv6 capable hosts
    ::1     ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ```

## On Each Node VM

### Set IP and Hostname

1. Open the `/etc/hostname` file:

    ```bash
    sudo vi /etc/hostname
    ```

2. Copy and paste the appropriate hostname:

    ```plaintext
    wky-node01
    ```

    Repeat this for each node, changing the hostname to `wky-node02` and `wky-node03` respectively.

3. Open the `/etc/hosts` file:

    ```bash
    sudo vi /etc/hosts
    ```

4. Copy and paste the following content:

    ```yaml
    127.0.0.1 localhost localhost.localdomain
    127.0.1.1 template_20_04
    10.111.0.27 wky-node01
    10.111.0.28 wky-node02
    10.111.0.29 wky-node03
    ```

### Configure Netplan

1. Open the Netplan configuration file:

    ```bash
    sudo vi /etc/netplan/00-installer-config.yaml
    ```

2. Add the following content:

    ```yaml
    # This is the network config written by 'subiquity'
    network:
      ethernets:
        ens160:
          addresses:
          - 10.111.0.27/24  # Change accordingly for each node
          routes:
          - to: default
            via: 10.111.0.254
          nameservers:
            addresses:
            - 8.8.8.8
            - 8.8.4.4
            search: []
      version: 2
    ```

3. Apply the Netplan configuration:

    ```bash
    sudo netplan apply
    ```

### Restart the Node

1. Reboot the node:

    ```bash
    sudo reboot
    ```

2. Restart SSH services:

    ```bash
    sudo systemctl restart ssh
    sudo systemctl restart sshd
    ```

### Configure SSH

1. Open the SSH configuration file:

    ```bash
    sudo vi /etc/ssh/sshd_config
    ```

2. Change the `PermitRootLogin` setting to `yes`:

    ```plaintext
    PermitRootLogin yes
    ```

3. Change the SSH password:

    ```bash
    passwd
    ```

## Return to the Ansible VM

### Generate SSH Keys and Copy to Nodes

1. Generate an SSH key:

    ```bash
    ssh-keygen
    ```

2. Copy the SSH key to each node:

    ```bash
    ssh-copy-id wky-node01
    ssh-copy-id wky-node02
    ssh-copy-id wky-node03
    ```

3. Close the `PermitRootLogin` setting on the Ansible VM:

    ```bash
    sudo vi /etc/ssh/sshd_config
    ```

    Comment out the `PermitRootLogin` line:

    ```plaintext
    # PermitRootLogin yes
    ```

## On Each Node VM

### Prepare for Kubernetes Cluster Installation

1. Disable swap:

    ```bash
    sudo swapoff -a
    ```

2. Stop and disable the firewall:

    ```bash
    sudo systemctl stop ufw
    sudo systemctl disable ufw
    ```

## Return to the Ansible VM

### Install Kubernetes Cluster

1. Disable swap and stop the firewall:

    ```bash
    sudo swapoff -a
    sudo systemctl stop ufw
    sudo systemctl disable ufw
    ```

2. Update and install necessary packages:

    ```bash
    sudo apt update
    sudo apt install python3-pip -y
    pip3 install --upgrade pip
    ```

3. Clone the Kubespray repository:

    ```bash
    git clone https://github.com/kubernetes-sigs/kubespray.git --branch v2.20.0
    cd kubespray
    ```

4. Install the required Python packages:

    ```bash
    pip3 install -r requirements.txt --default-timeout=1000
    pip3 install ansible==2.14 jinja2==2.11
    ```

5. Create the inventory file:

    ```bash
    declare -a IPS=(10.111.0.27 10.111.0.28 10.111.0.29)
    CONFIG_FILE=inventory/kubecluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
    ```

6. Edit the inventory file to match your node IPs and hostnames:

    ```yaml
    all:
      hosts:
        wky-node01:
          ansible_host: 10.111.0.27
          ip: 10.111.0.27
          access_ip: 10.111.0.27
        wky-node02:
          ansible_host: 10.111.0.28
          ip: 10.111.0.28
          access_ip: 10.111.0.28
        wky-node03:
          ansible_host: 10.111.0.29
          ip: 10.111.0.29
          access_ip: 10.111.0.29
      children:
        kube_control_plane:
          hosts:
            wky-node01:
            wky-node02:
            wky-node03:
        kube_node:
          hosts:
            wky-node01:
            wky-node02:
            wky-node03:
        etcd:
          hosts:
            wky-node01:
            wky-node02:
            wky-node03:
        k8s_cluster:
          children:
            kube_control_plane:
            kube_node:
        calico_rr:
          hosts: {}
    ```

7. Run the Ansible playbook to install the Kubernetes cluster:

    ```bash
    ansible-playbook -i inventory/kubecluster/hosts.yaml --become --become-user=root cluster.yml
    ```

## Verify Kubernetes Cluster

On each node VM, run the following commands to verify the Kubernetes cluster installation:

```bash
kubectl get nodes
kubectl get pods
kubectl get namespace
```

[Back to Main README](../README.md)