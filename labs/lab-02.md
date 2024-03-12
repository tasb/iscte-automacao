# Lab 02 - Creating a Static YAML Inventory with Group Variables in Ansible

## Contents

- [Objective](#objective)
- [Prerequisites](#prerequisites)

## Objective

Create an Ansible static inventory file in YAML format that categorizes two servers into azure group and one server into local group and define some example group variables for these groups.

## Prerequisites

- [ ] Create a folder named `ansible-labs` inside your home directory
- [ ] Create a folder named `ssh-keys` inside your home directory
- [ ] Move the key files used on last lab to the `ssh-keys` folder (`ansible-key` and `ansible-key.pub`)
- [ ] Navigate to the `ansible-labs` folder

## Guide

### Step 1: Create the Inventory File

Create a folder named `inventory` inside `ansible-labs` folder.

Inside the `inventory` folder, create a file named `inventory.yml`.

Add the following content to the file:

```yaml
local:
  hosts:
    local-managed-node-001:
      ansible_host: <managed-node-ip>
      ansible_user: ansible
      ansible_ssh_private_key_file: /home/lab-admin/ssh-keys/ansible-key
```

Please replace `<managed-node-ip>` with the IP address of the managed node used on last lab.

### Step 2: Add Servers to the Inventory File

In this step, you will add two server hosted in Azure to your inventory file.

To allow you to login on those servers you need the SSH private key file used to create the VMs.

Donwload this ZIP file: [SSH Keys](https://iscteiul365.sharepoint.com/:u:/s/TClass_UpSk_AdministracaoSistemas/EUONE2H8n8hEotyhYCM0JG4BkFjcntIS5CGPVshdBdch5A?e=XYLoo1).

This zip file is password protected and the password will be shared during the session.

Then extract the file and move the `azureuser.pem` and `azureuser002.pem` files to the `~/ssh-keys` folder.

Now you can update you `inventory.yml` file to add the following content:

```yaml
azure:
  hosts:
    tiberna-azure-001:
      ansible_host: 4.180.157.199
      ansible_user: azureuser
      ansible_ssh_private_key_file: /home/lab-admin/ssh-keys/azureuser.pem
    tiberna-azure-002:
      ansible_host: 4.180.159.23
      ansible_user: azureuser
      ansible_ssh_private_key_file: /home/lab-admin/ssh-keys/azureuser002.pem
```

## Step 3: Test Connectivity

Before using `ansible` commands to manage your servers, you should test the connectivity to them.

Run the following command to test the connectivity to the local server:

```bash
ssh -i /home/lab-admin/ssh-keys/azureuser.pem azureuser@4.180.157.199
```

You should be able to login on the server.

Now, let's test the connectivity to the other server:

```bash
ssh -i /home/lab-admin/ssh-keys/azureuser002.pem azureuser@4.180.159.23
```

You should be able to login on the server.

## Step 4: Review your Inventory

Review your inventory file using the `ansible-inventory` command:

```bash
ansible-inventory -i inventory/inventory.yml --list
```

This will display your inventory in a JSON format, showing the groups, hosts, and variables.

You can also use the `--graph` option to display the inventory in a more human-readable format:

```bash
ansible-inventory -i inventory/inventory.yml --graph
```

On this option, you see clearly the automatic groups created by Ansible: `all`, `ungrouped`.

## Step 5: Add a last group to the inventory

Edit the `inventory.yml` file and add the following content:

```yaml
linux-debian:
  children:
    azure:
    local:
```

If you used a different distribution when creating the local server, you should add a new group for that distribution.

### Step 6: Run a Simple Ansible Command

To test, run a simple Ansible command like ping:

```bash
ansible -i inventory/inventory.yml all -m ping
```

This will ping both servers in your inventory. You should see output similar to the following:

```bash
local-managed-node-001 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
tiberna-azure-001 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
tiberna-azure-002 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Now let's try to run a command on a specific group. Run the following command:

```bash
ansible -i inventory/inventory.yml local -m command -a "hostname"
```

You should see output similar to the following:

```bash
local-managed-node-001 | CHANGED | rc=0 >>
<VM hostname>
```

Now let's try to run a command on the other group. Run the following command:

```bash
ansible -i inventory/inventory.yml linux-debian -m command -a "hostname"
```

You should see output similar to the following:

```bash
tiberna-azure-001 | CHANGED | rc=0 >>
tiberna-azure-001
```

Finally let's try to run on a specific host. Run the following command:

```bash
ansible -i inventory/inventory.yml tiberna-azure-002 -m command -a "hostname"
```

### Step 7: Create Group Variables

Create one directory named `group_vars` inside `inventory` folder.

Inside the `group_vars` folder, create a file named `azure.yml`

Add the following content to the `azure.yml` file:

```yaml
ansible_user: azureuser
http_port: 80
```

Now create a file named `local.yml` and add the following content:

```yaml
ansible_user: ansible
http_port: 8080
```

Please consider that if you used a different user on your local server, you should change the value of `ansible_user` variable.

Go to your `inventory.yml` file and add remove the `ansible_user` variable from the hosts.

### Step 8: Test Group Variables

Test your inventory file using the `ansible-inventory` command:

```bash
ansible-inventory -i inventory/inventory.yml --list
```

Now run an ansible command to check the variables on `azure` group:

```bash
ansible -i inventory/inventory.yml azure -m ansible.builtin.debug -a "var=http_port"
```

Now run an ansible command to check the variables on `local` group:

```bash
ansible -i inventory/inventory.yml db -m ansible.builtin.debug -a "var=http_port"
```

Finally, run an ansible command to check the variables on `all` group:

```bash
ansible -i inventory/inventory.yml all -m ansible.builtin.debug -a "var=ansible_user"
```

Check that the values are mapped by what you have added to the `group_vars` files.

## Step 9: Add host variables

Create a folder named `host_vars` inside `inventory` folder.

Inside the `host_vars` folder, create a file named `tiberna-azure-001.yml`.

Add the following content to the `tiberna-azure-001.yml` file:

```yaml
ansible_ssh_private_key_file: /home/lab-admin/ssh-keys/azureuser.pem
```

Now create a file named `tiberna-azure-002.yml` and add the following content:

```yaml
ansible_ssh_private_key_file: /home/lab-admin/ssh-keys/azureuser002.pem
```

Finally, create a file named `local-managed-node-001.yml` and add the following content:

```yaml
ansible_ssh_private_key_file: /home/lab-admin/ssh-keys/ansible-key
```

Remove all `ansible_ssh_private_key_file` variables from the `inventory.yml` file.

## Step 10: Test Host Variables

Test your inventory file using the `ansible-inventory` command:

```bash
ansible-inventory -i inventory/inventory.yml --list
```

Since you have added the SSH private key you may run a command on all hosts to check if the key is being used:

```bash
ansible -i inventory/inventory.yml all -m command -a "hostname"
```

You may get the same reply as before, but now you are using the SSH private key file defined on the `host_vars` files.

### Conclusion

By completing this lab, you will have created a static YAML inventory file for Ansible, categorized two servers into different groups, and defined custom variables for each group.
This setup is fundamental for organizing your managed nodes and configuring them based on their roles in your infrastructure.
