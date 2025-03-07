# Multipass on Ubuntu – Comprehensive Guide

This documentation provides a complete overview of installing, configuring, and using Multipass on Ubuntu. It also covers generating SSH keys and incorporating a cloud-init file for automated VM provisioning.

---

## Table of Contents

1. What is Multipass?  
2. Prerequisites  
3. Install Multipass  
4. Basic Usage  
   - Launch a New Instance  
   - List Instances  
   - Shell Access  
   - Stop, Start, Suspend, and Delete  
5. Advanced Usage  
   - Customizing Resources  
   - Mounting Directories  
   - Copying Files  
   - Networking Notes  
   - Working with Images  
6. Generating an SSH Key Pair  
7. cloud-init Configuration  
   - Example cloud-init File  
   - Launching with cloud-init  
8. Cleanup and Removal  
9. Conclusion  

---

## What is Multipass?

Multipass is a tool from Canonical that simplifies the creation and management of Ubuntu virtual machines (instances). It streamlines the process of spinning up quick, disposable Ubuntu environments. These VMs are great for development, testing, and continuous integration workflows.

---

## Prerequisites

- Ubuntu 18.04 or later (on your host machine).  
- Snapd installed and enabled (it comes by default on most modern Ubuntu installations).  
- Sufficient free disk space to allocate for your VMs.  
- Internet access to pull Ubuntu images (unless you use pre-cached images).

---

## Install Multipass

1. Update your system:
    
    sudo apt update  
    sudo apt upgrade

2. Install Multipass from the Snap Store:

    sudo snap install multipass --classic

   The `--classic` flag ensures proper confinement for virtual machine management.

3. Verify installation:

    multipass version

   This should return the installed Multipass version.

---

## Basic Usage

### Launch a New Instance

    multipass launch --name my-instance

- This launches a VM with a specified name (latest Ubuntu LTS by default).
- If you omit `--name`, Multipass will assign a random name.
- To launch a specific release (e.g., Ubuntu 20.04):

      multipass launch 20.04 --name my-focal-vm

### List Instances

    multipass list

- Shows all instances with their names, states, and IP addresses.

### Shell Access

    multipass shell my-instance

- Gives you a shell directly into the VM (using the default `ubuntu` user).

### Stop, Start, Suspend, and Delete

- Stop a running instance:

      multipass stop my-instance

- Start a stopped instance:

      multipass start my-instance

- Suspend (save state):

      multipass suspend my-instance

- Delete (mark for deletion):

      multipass delete my-instance

  Then remove it permanently with:

      multipass purge

  *Note: `multipass purge` permanently deletes all previously deleted instances.*

---

## Advanced Usage

### Customizing Resources

When launching, you can specify CPU cores, memory, and disk size:

    multipass launch --name my-instance --cpus 2 --mem 2G --disk 10G

- `--cpus 2`: Assign 2 CPU cores.  
- `--mem 2G`: Assign 2GB of RAM.  
- `--disk 10G`: Allocate a 10GB disk.

### Mounting Directories

Share host directories with the VM:

    multipass mount ~/projects my-instance:/home/ubuntu/projects

- This makes `~/projects` on your host visible inside the VM.

### Copying Files

Transfer files to/from the VM:

    # Copy from host to VM
    multipass transfer ~/script.sh my-instance:

By default, the file lands in `/home/ubuntu/script.sh`.

    # Copy from VM to host
    multipass transfer my-instance:/home/ubuntu/hello.txt ~/

### Networking Notes

- Each VM is assigned its own IP address, accessible from the host.  
- Find the IP with `multipass list`.  
- If you run a service on port 80 inside the VM, access it at `http://<vm-ip>:80` from your host.

### Working with Images

- List available images:

      multipass find

  This shows official releases, daily builds, and more.

---

## Generating an SSH Key Pair

If you’d like to use key-based authentication (commonly needed for cloud-init), you need a public/private SSH key pair.

1. Generate the key pair:

       ssh-keygen -t ed25519 -C "your_email@example.com"

2. Accept defaults or specify a custom file path. Optionally set a passphrase.  
3. Locate the generated keys:
   - Private key: `~/.ssh/id_ed25519`  
   - Public key: `~/.ssh/id_ed25519.pub`

4. View/Copy your public key:

       cat ~/.ssh/id_ed25519.pub

   This line is what you’ll use in `ssh_authorized_keys`.

---

## cloud-init Configuration

With cloud-init, you can automate initial setup of the instance—install packages, create users, and run commands on first boot.

### Example cloud-init File

Save the following as `cloudinit.yaml`:

    #cloud-config

    ############################
    # 1. Set Hostname & Timezone
    ############################
    hostname: my-ubuntu-vm
    timezone: Etc/UTC

    ############################
    # 2. Create a User
    ############################
    users:
      - name: devuser
        shell: /bin/bash
        sudo: ['ALL=(ALL) NOPASSWD:ALL']
        ssh_authorized_keys:
          - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDEm6Jy+... your_email@example.com

    ############################
    # 3. Upgrade & Install Packages
    ############################
    package_upgrade: true
    packages:
      - curl
      - vim
      - git
      - apache2

    ############################
    # 4. Write Files
    ############################
    write_files:
      - path: /etc/motd
        content: |
          **********************************
          * Welcome to my Ubuntu VM!       *
          * Provisioned by cloud-init.     *
          **********************************

    ############################
    # 5. Run Commands on First Boot
    ############################
    runcmd:
      - systemctl enable apache2
      - systemctl start apache2
      - echo "Hello from cloud-init!" > /home/devuser/hello.txt
      - echo "<h1>Hello World from Cloud-Init!</h1>" | tee /var/www/html/index.html

> Note: Replace the `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDEm6Jy+...` part with your own public key.

### Launching with cloud-init

    multipass launch --name my-ubuntu-vm --cloud-init cloudinit.yaml

- After creation, check the instance:

      multipass list
      multipass shell my-ubuntu-vm

- Confirm the packages (`curl`, `vim`, `git`, `apache2`) are installed. Apache should be running, and `devuser` should be an admin with passwordless sudo.

---

## Cleanup and Removal

1. Stop all instances:

       multipass stop --all

2. Delete all instances:

       multipass delete --all

3. Purge deleted instances:

       multipass purge

4. Uninstall Multipass (if you no longer need it):

       sudo snap remove multipass

---

## Conclusion

Multipass provides a lightweight and straightforward way to run Ubuntu virtual machines on your local system. By combining it with cloud-init, you can streamline the provisioning process—configuring users, installing packages, and running commands automatically on first boot. With these tools, spinning up isolated development or test environments becomes quick, repeatable, and easy to manage.

For more details, refer to the official Multipass documentation (if you have internet access), and explore the built-in help by running:

    multipass --help

Happy virtualizing!
