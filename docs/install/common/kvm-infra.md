# Ubuntu Cloud-Init Setup with KVM

## Overview

This guide describes how to automatically deploy Ubuntu cloud images on a KVM hypervisor using cloud-init and a simple script.

---

## Requirements

- A Linux host with:
- KVM / QEMU installed:
`virt-install`, `genisoimage`, `cloud-utils`
- A configured user with sudo privileges
- DHCP network setup
- Automatic NTP time synchronization
- Ubuntu Cloud Image (cloud-init enabled)

Install the required packages:

```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager genisoimage cloud-utils
```

or for Fedora:

```bash
sudo dnf install qemu-kvm libvirt virt-install genisoimage cloud-utils-growpart
```

---

## Download Ubuntu Cloud Image

Download the cloud image for your desired Ubuntu version:

```bash
wget https://cloud-images.ubuntu.com/plucky/current/plucky-server-cloudimg-amd64.img -O ubuntu-cloud.img
```

(Replace "plucky" with "focal", "noble" etc. if needed.)

---

## Deploy the VMs

Create script `install_vm.sh`

---

```bash
#!/bin/bash
# install_vm.sh - Automated Cloud-Init Ubuntu VM creation script for KVM
#
# Usage:
#   sudo /home/user/install_vm.sh <vm-prefix> <count> <network-mode> <network-source>
#
# Example:
#   sudo /home/user/install_vm.sh vm 2 direct eno1
#   Will create two VMs (vm1 and vm2) directly connected to KVM interface eno1
#
# Parameters:
#   <vm-prefix>       Prefix for VM names (e.g., 'vm' -> vm1, vm2, ...)
#   <count>           Number of VMs to create
#   <network-mode>    Network connection type: 'direct', 'bridge', or 'network'
#   <network-source>  Network source interface or bridge name (e.g., eno1, br0, default)
#
# Notes:
#   - Requires KVM, libvirt, virt-install, and cloud-init to be installed
#   - Must be run with sudo/root privileges
#   - Backing image must exist at configured location
#
# === Configuration Parsing ===
function parse_arguments() {
  VM_PREFIX="$1"
  VM_COUNT="$2"
  NETWORK_MODE="$3"    # direct, network, bridge
  NETWORK_SOURCE="$4"  # eno1, lo, <bridge name>

  VM_MEMORY="3072"
  VM_VCPUS="2"
  VM_DISK_SIZE="10G"
  BACKING_FILE="/backing_files/plucky-server-cloudimg-amd64.img"
  IMAGES_DIR="/var/lib/libvirt/images"
  CSV_REPORT="/tmp/vm_creation_report.csv"
  SSH_PUBLIC_KEY="$HOME/.ssh/id_ed25519.pub"
  OS_VARIANT="ubuntu25.04"    # virt-install --osinfo list

  # Checking if we are root
  if [ "$EUID" -ne 0 ]; then
    echo "Error: This script must be run as root (use sudo)."
    exit 4
  fi

  # Checking for availability of backing file
  if [ ! -f "$BACKING_FILE" ]; then
    echo "Error: Backing file $BACKING_FILE not found!"
    exit 2
  fi

  # SSH key verification
  if [ ! -f "$SSH_PUBLIC_KEY" ]; then
    echo "Error: SSH public key not found at $SSH_PUBLIC_KEY"
    exit 3
  fi

  # Check for mandatory basic parameters
  if [ -z "$VM_PREFIX" ] || [ -z "$VM_COUNT" ] || [ -z "$NETWORK_MODE" ] || [ -z "$NETWORK_SOURCE" ]; then
    echo "Usage: $0 <vm-prefix> <count> <network-mode> <network-source>"
    exit 1
  fi
}

function initialize_report() {
  echo "VM Name,IP Address,Network Mode,Status" > "$CSV_REPORT"
}

function prepare_disk() {
  echo "Creating disk for $VM_NAME..."
  qemu-img create -f qcow2 -F qcow2 -b "$BACKING_FILE" "$IMAGES_DIR/${VM_NAME}.qcow2" "$VM_DISK_SIZE"
}

function create_cloudinit_iso() {
  echo "Creating cloud-init ISO for $VM_NAME..."

  CLOUDINIT_DIR="/tmp/${VM_NAME}-cloudinit"
  mkdir -p "$CLOUDINIT_DIR"

  SSH_KEY_CONTENT=$(cat "$SSH_PUBLIC_KEY")

  cat > "$CLOUDINIT_DIR/user-data" <<EOF
#cloud-config
hostname: $VM_NAME
users:
  - name: clouduser
    gecos: Cloud User
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash
    lock_passwd: false
    ssh_authorized_keys:
      - $SSH_KEY_CONTENT
chpasswd:
  list: |
    root:\$6\$uy9k8c33eOOnaerZ\$2QJ1xaBi.7KxVECzo.GOiE/yfhcEiDAjn2Zmpkm3.za/AslJjf8ZTFcQtFJuxWgQhQt2y1M4c.mcCLBws6q4Y0
    clouduser:\$6\$q0y5NM6jS0hvrAPq\$GAxSuOaeh.aTzxP7vTjN3G06YQZRULraVOi0Gql3T6HuFLP6ayoOpewws1Rgz5yfsrxu2jxhWUIn4bHFt7uz4.
  expire: false
  encrypt: false
ssh_pwauth: false
package_update: true
package_upgrade: true
packages:
  - qemu-guest-agent
runcmd:
  - systemctl start qemu-guest-agent
EOF

  echo "instance-id: $VM_NAME" > "$CLOUDINIT_DIR/meta-data"
  echo "local-hostname: $VM_NAME" >> "$CLOUDINIT_DIR/meta-data"
}

function install_vm() {
  echo "Installing VM $VM_NAME..."
  virt-install \
    --connect qemu:///system \
    --name "$VM_NAME" \
    --memory "$VM_MEMORY" \
    --vcpus "$VM_VCPUS" \
    --disk path="$IMAGES_DIR/${VM_NAME}.qcow2",format=qcow2,bus=virtio \
    --cloud-init user-data="$CLOUDINIT_DIR/user-data",meta-data="$CLOUDINIT_DIR/meta-data" \
    --os-variant "$OS_VARIANT" \
    --graphics none \
    --import \
    $NETWORK_PARAM \
    --noautoconsole
}

function enable_vm_autostart() {
  echo "Enabling autostart for $VM_NAME..."
  virsh autostart "$VM_NAME"
}

function finalize_vm_setup() {
  echo "Cleaning up temporary files for $VM_NAME..."
  rm -rf "$CLOUDINIT_DIR"
}

function main() {
  parse_arguments "$@"
  initialize_report

  for i in $(seq 1 $VM_COUNT); do
    VM_NAME="${VM_PREFIX}${i}"

    echo "\n=== Creating VM: $VM_NAME ==="

    prepare_disk
    create_cloudinit_iso

    if [ "$NETWORK_MODE" == "direct" ]; then
      NETWORK_PARAM="--network type=direct,source=$NETWORK_SOURCE,source_mode=bridge,model=virtio"
    elif [ "$NETWORK_MODE" == "network" ]; then
      NETWORK_PARAM="--network network=$NETWORK_SOURCE,model=virtio"
    elif [ "$NETWORK_MODE" == "bridge" ]; then
      NETWORK_PARAM="--network bridge=$NETWORK_SOURCE,model=virtio"
    else
      echo "Error: Unsupported network mode $NETWORK_MODE"
      exit 4
    fi

    install_vm
    enable_vm_autostart
    finalize_vm_setup
  done

  echo "\n✅✅✅ All VMs created, configured, and should be visible in Cockpit!"
}

# === Execute Main ===
main "$@"

```

---

!!! note "Note"
    **1. How to generate a secure encrypted password for cloud-init:**
    
    Run the following command to generate a SHA-512 (recommended) password hash:
    
    ```bash
    openssl passwd -6
    ```
    
    Example output:
    ```
    $6$Tg8r0q2XibH8O$yImySK8p2iC2R5L/KHKoXtfClh5UzQGkvz6YtD63At4V3BhUEH7uZ8fwS4vzw5dhB4Z53slYh7eUl6B2yRcE6/
    ```
    
    When inserting the password into a cloud-init `user-data` YAML file, **you must escape** every `$` symbol with a backslash (`\`) if you are inserting it through Bash scripts:
    
    ```yaml
    chpasswd:
      list: |
        ubuntu:\$6\$Tg8r0q2XibH8O\$yImySK8p2iC2R5L/...
      expire: false
      encrypt: false
    ``` 
    
    **2. How to generate an SSH Ed25519 key pair:**
    
    To create a modern and secure SSH key pair, run:
    
    ```bash
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
    
    - Save the key files in the default location (`~/.ssh/id_ed25519`).
    - Public key is located at: `~/.ssh/id_ed25519.pub`
    
    Use the public key content in your `cloud-init` under `ssh_authorized_keys`:
    
    ```yaml
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB... your_email@example.com
    ```

---