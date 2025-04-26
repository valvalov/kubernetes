# My first Kubernetes Cluster

## Generate kubeadm config

```bash
kubeadm config print init-defaults
```

Then create a file `kubeadm-config.yaml` with your configuration.
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

## Initialize the Kubernetes control-plane

```bash
sudo kubeadm init --config kubeadm-config.yaml
```

Configure kubectl:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install CNI:

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

## Join a worker node

Example:

```bash
kubeadm join 192.168.1.37:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

To generate a new token and get the full command:

```bash
kubeadm token create --print-join-command
```