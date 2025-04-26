# Install containerd

## Install containerd package

```bash
sudo apt update
sudo apt install -y containerd
```
## Configure containerd

Set default config.

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Edit the config:
Under the section [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options] add SystemdCgroup = true

```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

## Enable and start containerd service

```bash
sudo systemctl enable containerd
sudo systemctl restart containerd
```