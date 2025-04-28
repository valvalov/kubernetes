# Kubernetes Prerequisites

## Enable IPv4 Forwarding

IPv4 forwarding is required for Kubernetes to route network traffic between Pods and Services.

```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

!!! tip "Tip"
    This change persists across reboots and applies immediately without requiring a restart.

---

## Check Bridge Network Settings

Verify that netfilter settings are correctly applied:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables
sudo sysctl net.bridge.bridge-nf-call-ip6tables
```

!!! note "Note"
    Both values should return `1`.

---

## Update System Packages

First, update your local package cache to ensure you install the latest available version of containerd.

```bash
sudo apt update
sudo apt upgrade -y
```

!!! tip "Tip: Update Regularly"
    Keeping your package lists updated ensures that you pull secure and stable versions of software packages.

---

## Install containerd

`containerd` is a core container runtime used by Kubernetes for managing container lifecycle operations.
Installing and configuring containerd correctly is crucial for setting up a stable Kubernetes environment.

Install containerd using your system package manager.

```bash
sudo apt install -y containerd
```

!!! note "Note"
    No need to add additional repositories for containerd if you're using recent Ubuntu or Debian versions.

---

## Configure containerd

Create a default configuration file and modify it if needed.

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Modify `/etc/containerd/config.toml` to ensure that containerd uses systemd for managing cgroups:

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

!!! warning "Warning: Check config carefully"
    Incorrect settings in `config.toml` can cause Kubernetes components like kubelet to fail starting.

---

## Enable and Start containerd

Enable and start containerd to ensure it runs on boot:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## Conclusion

You have now installed and configured containerd successfully!
This runtime will act as the foundation for Kubernetes to manage and schedule containers across your cluster.

Proceed to install Kubernetes components like `kubeadm`, `kubelet`, and `kubectl` in the next steps.
