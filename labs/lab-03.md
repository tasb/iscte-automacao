# Lab 03 - Author and run your first playbook

## Contents

- [Objective](#objective)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 1: Check your inventory](#step-1-check-your-inventory)
  - [Step 2: Create the Playbook](#step-2-create-the-playbook)
  - [Step 3: Run the Playbook](#step-3-run-the-playbook)
  - [Step 4: Test the Web Server](#step-4-test-the-web-server)
  - [Step 5: Change homepage](#step-5-change-homepage)
  - [Step 6: Add a smoke test](#step-6-add-a-smoke-test)
  - [Step 7: Handle sudo usage](#step-7-handle-sudo-usage)
- [Conclusion](#conclusion)

## Objective

Create an Ansible playbook that installs and configures a web server on a managed node.

## Prerequisites

- [ ] Navigate to `ansible-labs` folder inside your home directory

## Guide

### Step 1: Check your inventory

First, let's check if your inventory file is correctly configured.

Let's list the content of the `inventory` file:

```bash
ansible-inventory -i inventory --list
```

Recall that you have two groups: `local` and `azure`.

For this lab you should only use the `local` group, since the `azure` group is shared by everyone.

Let's test the connection:

```bash
ansible -i inventory/inventory.yml local -m ping
```

```bash
local-managed-node-001 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Step 2: Create the Playbook

Create a file named `webserver.yml` inside `ansible-labs` folder.

Add the following content to the file:

```yaml
---
- name: Install and configure web server
  hosts: local
  become: true
  tasks:
    - name: Install Apache
      ansible.builtin.apt:
        name: apache2
        state: latest
    - name: Start Apache
      ansible.builtin.service:
        name: apache2
        state: started
```

On this playbook, we are:

- Installing Apache web server
- Starting Apache web server
- Setting the `become` option to `true` to run the tasks as root
- Setting the `hosts` option to `local` to run the tasks on the hosts in the `local` group

### Step 3: Run the Playbook

Run the playbook using the `ansible-playbook` command:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

This will run the playbook on the hosts in the `local` group.

You should see output similar to the following:

```bash
PLAY [Install and configure web server] ****************************************************

TASK [Gathering Facts] *********************************************************************
ok: [local-managed-node-001]

TASK [Install Apache] **********************************************************************
changed: [local-managed-node-001]

TASK [Start Apache] ************************************************************************
changed: [local-managed-node-001]

PLAY RECAP *********************************************************************************
local-managed-node-001          : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Step 4: Test the Web Server

Run the following command to test the web server:

```bash
curl http://<IP_FROM_MANAGED_NODE>
```

Please replace `<IP_FROM_MANAGED_NODE>` with the IP address of the managed node.

You should get an HTML page as output.

You can also open the URL on your browser to see the page.

### Step 5: Change homepage

Create a folder called `static` inside `ansible-labs` folder.

```bash
mkdir static
```

Then create a new file named `index.html` inside `static` folder.

Add the following content to the file:

```html
<html>
  <head>
    <title>Ansible Lab</title>
  </head>
  <body>
    <h1>Ansible Lab</h1>
    <p>This is a test page</p>
  </body>
</html>
```

Now, let's update the playbook to copy the file to the web server.

Update the `webserver.yml` file to add a new task to copy the file to the web server.

Add the following content to the file after the last task:

```yaml
    - name: Copy index.html
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
```

Pay attention to the indentation. The task should be at the same level as the `Start Apache` task.

Run the playbook again:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

You should see output similar to the following:

```bash
PLAY [Install and configure web server] **************************************

TASK [Gathering Facts] *******************************************************
ok: [local-managed-node-001]

TASK [Install Apache] ********************************************************
ok: [local-managed-node-001]

TASK [Start Apache] **********************************************************
ok: [local-managed-node-001]

TASK [Copy index.html] *******************************************************
changed: [local-managed-node-001]

PLAY RECAP ************************************************************************************************
local-managed-node-001          : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Now navigate to the URL http://<IP_FROM_MANAGED_NODE> and you should see the new page.

Please replace `<IP_FROM_MANAGED_NODE>` with the IP address of the managed node.

### Step 6: Add a smoke test

Update the `webserver.yml` file to add a new task to run a smoke test.

Add the following content to the `webserver.yml` file after the `Copy index.html` task:

```yaml
    - name: Run smoke test
      ansible.builtin.uri:
        url: http://localhost
        return_content: yes
      register: result
    - name: Debug smoke test
      ansible.builtin.debug:
        msg: "{{ result.content }}"
```

Let's run the playbook again:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

## Step 7: Handle sudo usage

Now that you have your playbook finalized, you should review the `become` option.

Remove the `become` option from the play level and add it to each task that requires root privileges.

The tasks that require root privileges are:

- Install Apache
- Start Apache
- Copy index.html

The playbook file context at the end should look like this:

```yaml
---
- name: Install and configure web server
  hosts: webserver
  tasks:
    - name: Install Apache
      become: true
      ansible.builtin.yum:
        name: apache2
        state: latest
    - name: Start Apache
      become: true
      ansible.builtin.service:
        name: apache2
        state: started
    - name: Copy index.html
      become: true
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
    - name: Run smoke test
      ansible.builtin.uri:
        url: http://local-managed-node-001.seg-social.virt
        return_content: yes
      register: result
    - name: Debug smoke test
      ansible.builtin.debug:
        msg: "{{ result.content }}"
```

Run the playbook again:

```bash
ansible-playbook -i inventory/inventory.yml webserver.yml
```

## Conclusion

In this lab, we created our first playbook to install and configure a web server. You also learned how to run a playbook and how to update it to add new tasks.
