# Join a worker node

Example:

```bash
kubeadm join 192.168.1.37:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

To generate a new token and get the full command:

```bash
kubeadm token create --print-join-command
```