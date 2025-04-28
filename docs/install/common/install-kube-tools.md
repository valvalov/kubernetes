
# Install Kubernetes Tools

## Introduction

In this section, you will install the core Kubernetes tools: `kubeadm`, `kubelet`, and `kubectl`.
These components are essential for managing and operating your Kubernetes cluster effectively.

---

## Check Latest Kubernetes Release

It is recommended to check the latest available Kubernetes release before installing the packages.

```bash
curl -L -s https://dl.k8s.io/release/stable.txt
```

---

## Add Kubernetes Package Repository

First, add the official Kubernetes APT repository to your system.

=== "1.32"
    ```bash
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    ```
=== "1.33"
    ```bash
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    ```

!!! note "Note"
    Make sure `/etc/apt/keyrings/` exists before saving the GPG key.

---

## Install kubeadm, kubelet, and kubectl

Install the essential Kubernetes components:

```bash
sudo apt install -y kubelet kubeadm kubectl
```

After installation, prevent them from being automatically upgraded:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

!!! tip "Tip"
    Holding these packages prevents unexpected version upgrades that could break your cluster.

---

## Enable kubelet Service

Enable the `kubelet` service to start automatically at boot:

```bash
sudo systemctl enable kubelet
```

!!! warning "Warning"
    Note that `kubelet` will remain in a crash loop until you initialize your cluster with `kubeadm init`.

---

## Conclusion

You now have the essential Kubernetes tools installed and configured.
The next step is to initialize your Kubernetes control plane using `kubeadm init`.
