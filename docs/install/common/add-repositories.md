# Add Kubernetes apt repository (pkgs.k8s.io)

=== "1.32"
   
    ```bash
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl gpg
    sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    ```

=== "1.33"
    
    ```bash
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl gpg
    sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o/etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    ```
