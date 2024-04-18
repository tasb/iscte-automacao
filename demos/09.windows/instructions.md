# Run Ansible in Windows

## If not working

winrm set winrm/config/client '@{TrustedHosts ="[0.0.0.0]"}'

## Install IIS

```bash
ansible-playbook -i inventory.yml iis.yml -k
```

## Install notepad++

```bash
ansible-playbook -i inventory.yml install-notepad.yml -k
```

## Create user and group

```bash
ansible-playbook -i inventory.yml users-groups.yml --ask-vault-pass -k
```
