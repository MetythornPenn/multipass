# 🚀 Multipass VM Setup & SSH Access Guide

This guide helps you create a new Ubuntu VM using [Multipass](https://multipass.run), access it via SSH, and configure a shortcut for easy login.

---

## 🧱 1. Create a Multipass Instance

Launch a new VM (with custom resources):

```bash
multipass launch --name metythorn3 --mem 4G --cpus 2 --disk 20G
```

Check if it's running

```bash
multipass list

```

## 🔐 2. Inject Your SSH Key

Ensure you have a public key (generate one if needed):

```bash
cat ~/.ssh/id_rsa.pub
```

Then copy it into the instance:

```bash
cat ~/.ssh/id_rsa.pub | multipass exec metythorn3 -- bash -c 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh'
multipass exec metythorn3 -- sudo chown -R ubuntu:ubuntu /home/ubuntu/.ssh

```

## 🔎 3. Get the Instance IP Address

```bash
multipass list

```

Example output:

```bash
Name           State    IPv4           Image
metythorn3     Running  10.82.81.10    Ubuntu 24.04 LTS
```

## 🔗 4. Access the VM via SSH

```bash
ssh ubuntu@10.82.81.10

```

## ⚡ 5. Setup SSH Shortcut (Optional)

Edit your SSH config:

```bash
nano ~/.ssh/config
```

Add this:

```bash
Host metythorn3
    HostName 10.82.81.10
    User ubuntu
    IdentityFile ~/.ssh/id_rsa

```

Then simply run:

```bash
ssh metythorn3

```

## 🧱 How to set a password manually

### 1. Access the instance via multipass shell

```bash
multipass shell metythorn3
```

### 2. Set a password for the ubuntu user

```bash
sudo passwd ubuntu
```
