# Initialize Kubernetes Cluster

## Introduction

This guide walks you through initializing the Kubernetes control plane and joining worker nodes using `kubeadm`.

---

## Generate kubeadm config

!!! note "Note"
    You can generate a default configuration file using `kubeadm config print init-defaults` and use it as a template.

Create a file `kubeadm-config.yaml` with your configuration.
Example:

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.1.37
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  imagePullSerial: true
  name: node
  taints: null
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: 1.32.0
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
controllerManager:
  extraArgs:
    node-cidr-mask-size: "22"
```

---

## Initialize the Control Plane

Initialize your Kubernetes master node:

```bash
sudo kubeadm init --config=kubeadm-config.yaml
```

---

## Configure kubectl Access

Set up your user to control the cluster:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

!!! tip "Tip"
    Repeat this setup for any additional users who need access to `kubectl`.

---

## Install a CNI Plugin

Deploy a networking plugin, such as Flannel:

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

!!! warning "Warning"
    Your cluster will not function properly until a CNI plugin is installed!

---

## Join Worker Nodes

To add a worker node, use the command displayed after `kubeadm init` or create a new one:

```bash
kubeadm token create --print-join-command
```

Example usage:

```bash
sudo kubeadm join 192.168.1.37:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

!!! note "Note"
    The `--token` and `--discovery-token-ca-cert-hash` ensure that worker nodes securely join the cluster.

---

## Conclusion

You have successfully initialized your Kubernetes cluster and connected worker nodes. 
You are now ready to deploy applications and manage your cluster using `kubectl`.
