# Lab 06 - Create Roles from existing code

## Table of Contents

- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 1: Create roles for each playbook block](#step-1-create-roles-for-each-playbook-block)
  - [Step 2: Create a new playbook](#step-2-create-a-new-playbook)
  - [Step 3: Run the playbook](#step-3-run-the-playbook)
- [Conclusion](#conclusion)

## Objectives

- Create an Ansible role from existing code
- Update playbooks to use the new role

## Prerequisites

- Make sure you finished properly the [Lab 04 - Deep dive into Ansible Playbooks](lab-04.md)

## Guide

### Step 1: Create roles for each playbook block

On this step you need to create a role for Redis, PostgreSQL and Apache.

Take a look on [Lab 05 - Authoring your first Ansible Role](lab-05.md) to remember how to create a role.

### Step 2: Create a new playbook

Create a new playbook named `playbook-with-roles.yml` and use the roles you created on the previous step.

### Step 3: Run the playbook

Find the appropriate command to run your playbook and test if everything is working as expected.

## Conclusion

You should now have a playbook that uses roles to install and configure Redis, PostgreSQL and Apache.
