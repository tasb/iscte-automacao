# Lab 08 - Using Ansible Vault and create a complex scenario

## Table of Contents

- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 01: Use template for `redis.conf`](#step-01-use-template-for-redisconf)
  - [Step 02: Use Ansible Vault to encrypt sensitive data](#step-02-use-ansible-vault-to-encrypt-sensitive-data)
  - [Step 03: Run the playbook](#step-03-run-the-playbook)
- [Conclusion](#conclusion)

## Objectives

- Create Ansible Roles from existing playbook
- Use Ansible Vault to encrypt sensitive data
- Use Templates with Jinja2 instead of regex replace

## Prerequisites

- [ ] Create a folder named `lab08` inside `ansible-labs` on your home directory
- [ ] Navigate to `lab08` folder
- [ ] Copy to `lab08` folder all roles and playbooks you created on `lab06`

## Guide

You will use Ansible Vault to encrypt sensitive data and use Jinja2 templates to update `redis.conf` file.

### Step 01: Use template for `redis.conf`

On the `lab.redis` role, you should have a task with the following content:

```yaml
- name: Configure Redis
  ansible.builtin.replace:
    path: /etc/redis/redis.conf
    regexp: '^# requirepass foobared'
    replace: 'requirepass {{ redis_password }}'
    backup: yes
  notify:
    - Restart Redis
```

This is a great opportunity to use a template instead of regex replace.

Create a template file named `redis.conf.j2` inside `templates` folder with the content of this file: [redis.conf](https://raw.githubusercontent.com/tasb/ansible-training/main/labs/lab07/redis.conf).

Find the line with the `# requirepass foobared` and replace it with `requirepass {{ redis_password }}`.

Then update the task to use the template file instead of regex replace.

After doing this, the task should look like this:

```yaml
- name: Configure Redis
  template:
    src: "templates/redis.conf.j2"
    dest: /etc/redis/redis.conf
  notify:
    - Restart Redis
```

Find the `ansible-playbook` command to run the playbook and test if everything is working as expected.

### Step 02: Use Ansible Vault to encrypt sensitive data

You should have a variable file named `defaults/main.yml` inside `lab.redis` role with the following content:

```yaml
---
redis_password: ansible
```

Now, let's create an encrypted version of this variable at playbook level.

First, let's generate an encrypted password using Ansible Vault.

Run the following command:

```bash
ansible-vault encrypt_string 'ansible' --name 'redis_password'
```

You need to set a password to encrypt the variable. Use `password` as password.

You need to enter the password twice.

After that, you should get an output similar to this:

```bash
redis_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          663736336336313062....
```

Now you need to create a new `vars` block inside the playbook with the encrypted variable.

After doing this, your playbook should look like this:

```yaml
---
- name: Install Redis, PostgreSQL and Apache
  hosts: all
  become: yes
  roles:
    - lab.redis
    - lab.postgresql
    - lab.apache
  vars:
    redis_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          663736336336313062....
```

### Step 03: Run the playbook

Find the appropriate command to run the playbook and test if everything is working as expected.

You need to enter the password you've set to encrypt the variable.

## Conclusion

Congratulations! You've created roles for each part of the playbook, used a template for `redis.conf` and encrypted sensitive data using Ansible Vault.
