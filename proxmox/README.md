# Proxmox Template 

Create an Ubuntu template to spin up many Ubuntu VMs

1. In Proxmox, select your storage that accepts ISOs and click on Download from URL.  Use one of the following links:
    - [Ubuntu 22.04 cloudinit image](https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64-disk-kvm.img)
    - [Ubuntu 23.04 cloudinit image](https://cloud-images.ubuntu.com/lunar/current/lunar-server-cloudimg-amd64-disk-kvm.img)
    - Or find your [favourite](https://cloud-images.ubuntu.com)

    I'll be using 22.04 for this configuration

1. Take note of where where the ISO storage and VM storage is

    ```console
    ISO Storage = iso
    VM Storage = local-zfs
    ```

1. Configure the following for execution in your cluster

    ```console
    cd /var/lib/vz/template/[ISO Storage]/
    qm create [ID] --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
    qm importdisk [ID] jammy-server-cloudimg-amd64-disk-kvm.img [VM Storage]
    qm set [ID] --scsihw virtio-scsi-pci --scsi0 [VM Storage]:vm-[ID]-disk-0
    qm set [ID] --ide2 [VM Storage]:cloudinit
    qm set [ID] --boot c --bootdisk scsi0
    qm set [ID] --serial0 socket --vga serial0
    ```

    My configuration is:

    ```console
    cd /var/lib/vz/template/iso/
    qm create 5000 --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
    qm importdisk 5000 jammy-server-cloudimg-amd64-disk-kvm.img local-zfs
    qm set 5000 --scsihw virtio-scsi-pci --scsi0 local-zfs:vm-5000-disk-0
    qm set 5000 --ide2 local-zfs:cloudinit
    qm set 5000 --boot c --bootdisk scsi0
    qm set 5000 --serial0 socket --vga serial0
    ```

1. Open the ubuntu-cloud VM
    1. Click on Hardware and modify
        - Memory
        - Processors
        - Resize Hard Disk (also ensure Discard is enabled)
    1. Click on Cloud-Init and set
        - User
        - Password
        - SSH public key
        - IP Config - likely set IPV4 to DHCP

1. Right click on the VM and convert it to Template

1. You can now Right click on the template and clone it
    - Ensure you change Mode to Full Clone
    - Give the new VM a name

1. Open the cloned VM and do any final configuration tweaks then start it
