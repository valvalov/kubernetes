# Prerequisites

To manually enable IPv4 packet forwarding:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

Verify:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables
sudo sysctl net.bridge.bridge-nf-call-ip6tables
sudo sysctl net.ipv4.ip_forward
```