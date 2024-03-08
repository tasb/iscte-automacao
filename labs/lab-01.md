# Create Lab Environment for Ansible

## Table of Contents

- [Objectives](#objectives)
- [Guide](#guide)
  - [Step 01: Create a control node for Ansible using VirtualBox](#step-01-create-a-control-node-for-ansible-using-virtualbox)
  - [Step 02: Create a managed node for Ansible using VirtualBox](#step-02-create-a-managed-node-for-ansible-using-virtualbox)
  - [Step 03: Install prerequisites for Ansible on the control node](#step-03-install-prerequisites-for-ansible-on-the-control-node)
  - [Step 04: Install prerequisites for Ansible on the managed node](#step-04-install-prerequisites-for-ansible-on-the-managed-node)
  - [Step 05: Configure SSH keys for passwordless authentication between the control node and the managed node](#step-05-configure-ssh-keys-for-passwordless-authentication-between-the-control-node-and-the-managed-node)
  - [Step 06: Create an inventory file for the managed node](#step-06-create-an-inventory-file-for-the-managed-node)
  - [Step 07: Run an Ansible ad-hoc command on the managed node](#step-07-run-an-ansible-ad-hoc-command-on-the-managed-node)
  - [Step 08: Copy file to managed nodes](#step-08-copy-file-to-managed-nodes)
  - [Step 09: Run more commands on managed nodes](#step-09-run-more-commands-on-managed-nodes)
  - [Step 10: Install a package on the managed node](#step-10-install-a-package-on-the-managed-node)
- [Conclusion](#conclusion)

## Objectives

- Create a control node for Ansible using VirtualBox
- Create a managed node for Ansible using VirtualBox
- Install prerequisites for Ansible on the control node
- Install prerequisites for Ansible on the managed node
- Configure SSH keys for passwordless authentication between the control node and the managed node
- Test the connection between the control node and the managed node
- Create an inventory file for the managed node
- Run an Ansible ad-hoc command on the managed node

## Guide

### Step 01: Create a control node for Ansible using VirtualBox

First step is to create a control node for Ansible using VirtualBox.

You need to use any Linux distribution for the control node.

For this lab, we will use Ubuntu 22.04.4 LTS Desktop edition, which allow you to have a GUI interface.

The direct download link for the ISO file is [here](https://releases.ubuntu.com/22.04.4/ubuntu-22.04.4-desktop-amd64.iso).

Create a new Virtual Machine in VirtualBox with the following settings:

- Name: ansible-control-node
- Type: Linux
- Version: Ubuntu (64-bit)
- Memory size: 4096 MB
- vCPU: 2
- Hard disk: 60 GB

When prompted to select an user account, create a new user with the following details:

- Username: lab-admin
- Password: "Any password that you can remember later"

### Step 02: Create a managed node for Ansible using VirtualBox

Next step is to create a managed node for Ansible using VirtualBox.

For this lab, we will use Ubuntu 22.04.4 LTS Server edition, which is a minimal installation without GUI.

The direct download link for the ISO file is [here](https://releases.ubuntu.com/22.04.4/ubuntu-22.04.4-live-server-amd64.iso).

Create a new Virtual Machine in VirtualBox with the following settings:

- Name: ansible-managed-node
- Type: Linux
- Version: Ubuntu (64-bit)
- Memory size: 2048 MB
- vCPU: 2
- Hard disk: 40 GB

When prompted to select an user account, create a new user with the following details:

- Username: ansible
- Password: P@ssw0rd

Take a note of the IP address of the managed node, as you will need it later to configure the control node.

### Step 03: Install prerequisites for Ansible on the control node

Login to the control node and open a terminal.

Run the following commands to install prerequisites for Ansible:

```bash
sudo apt update
sudo apt install -y software-properties-common
```

These commands update the packages list and install the `software-properties-common` package, which is required to add new repositories.

Then, add the Ansible repository and install Ansible:

```bash
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

### Step 04: Install prerequisites for Ansible on the managed node

Login to the managed node and open a terminal.

Ubuntu 22.04.4 LTS Server edition comes with OpenSSH server installed by default to allow connections from remote systems.

Additionally, you need to have Python on the managed node, which is required by Ansible to run commands on the managed node.

Ubuntu 22.04.4 LTS Server edition comes with Python 3 installed by default.

To verify the Python version, run the following command:

```bash
python3 --version
```

You should see the output similar to the following:

```plaintext
Python 3.10.0
```

Lastly, and to make your life easier, you should add your user and the `sudo` group to the `sudoers` file and make no need for password to run elevated commands.

Run the following command to add your user to the `sudo` group:

```bash
sudo usermod -aG sudo ansible
```

Run the following command to edit the `sudoers` file:

```bash
sudo visudo
```

Add the following line at the end of the file:

```plaintext
ansible ALL=(ALL) NOPASSWD: ALL
```

Save the file and exit the editor.

Now you can run elevated commands without entering the password.

### Step 05: Configure SSH keys for passwordless authentication between the control node and the managed node

On the control node, run the following command to generate a new SSH key pair:

```bash
ssh-keygen -b 2048 -t rsa -f ./ansible-key
```

You need to enter a passphrase for the SSH key pair. You can leave it empty if you want to have passwordless authentication.

Now you need to copy the public key to the managed node.

Run the following command to copy the public key to the managed node:

```bash
ssh-copy-id -i ./ansible-key ansible@<managed-node-ip>
```

You will need to enter the password for the `ansible` user on the managed node.

Finally you need to test the connection between the control node and the managed node.

Run the following command to test the connection:

```bash
ssh -i ./ansible-key ansible@<managed-node-ip>
```

You should be able to login to the managed node without entering the password.

### Step 06: Create an inventory file for the managed node

Create a folder named `ansible` in the home directory of the `lab-admin` user on the control node.

Create a file named `inventory` inside the `ansible` folder with the following content:

```yaml
hosts:
  managed-node:
    ansible_host: <managed-node-ip>
    ansible_user: ansible
    ansible_ssh_private_key_file: /home/lab-admin/ansible-key
```

This inventory file contains the details of the managed node, which is required by Ansible to connect to the managed node.

### Step 07: Run an Ansible ad-hoc command on the managed node

Run the following command to test the connection between the control node and the managed node:

```bash
ansible -i ./ansible/inventory managed-node -m ping
```

You should see the output similar to the following:

```plaintext
managed-node | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### Step 08: Copy file to managed nodes

To copy a file, you need to create a file first.

Run the following command to create a file named `hello.txt` in `/tmp` on the control node:

```bash
echo "Hello World" > /tmp/hello.txt
```

Then, run the following command to copy the previous created file to `/tmp` on all nodes:

```bash
ansible all -m ansible.builtin.copy -a "src=/tmp/hello.txt dest=/tmp/"
```

You should see output similar to the following:

```bash
servidor-0 | CHANGED => {
    "changed": true,
    "checksum": "d41d8cd98f00b204e9800998ecf8427e",
    "dest": "/tmp/hello.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
    "mode": "0644",
    "owner": "root",
    "size": 0,
    "src": "/home/lab-admin/.ansible/tmp/ansible-tmp-1648530733.3-1-1234/AnsiballZ_copy.py",
    "state": "file",
    "uid": 0
}
```

### Step 09: Run more commands on managed nodes

Run the following command to list `/tmp` folder on all nodes:

```bash
ansible all -m ansible.builtin.shell -a "ls /tmp"
```

You should see output similar to the following:

```bash
10.0.0.0 | CHANGED | rc=0 >>
hello.txt
...
```

Let's check if the file was copied correctly:

```bash
ansible all -m ansible.builtin.shell -a "cat /tmp/hello.txt"
```

You should see output similar to the following:

```bash
10.0.0.0  | CHANGED | rc=0 >>
Hello World
```

### Step 10: Install a package on the managed node

Next, let's run an ad-hoc command to install a package on the managed node.

Run the following command:

```bash
ansible -i ./ansible/inventory managed-node -m apt -a "name=nginx state=present" --become
```

After install the package, you can run the following command to verify the status of the `nginx` service:

```bash
ansible -i ./ansible/inventory managed-node -m service -a "name=nginx state=started" --become
```

And you can use a web browser to access the default web page of the `nginx` service by entering the IP address of the managed node in the address bar.

## Conclusion

In this lab, you have created a control node for Ansible using VirtualBox, created a managed node for Ansible using VirtualBox, installed prerequisites for Ansible on the control node, installed prerequisites for Ansible on the managed node, configured SSH keys for passwordless authentication between the control node and the managed node, tested the connection between the control node and the managed node, created an inventory file for the managed node, and run an Ansible ad-hoc command on the managed node.
