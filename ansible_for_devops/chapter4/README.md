# Running playbooks

Some options:

```bash
# Limiting servers
ansible-playbook playbook.yml --limit webservers
ansible-playbook playbook.yml --limit xyz.example.com

# List the servers that would be affected:
ansible-playbook playbook.yml --list-hosts

# Explicitly set the remote user
ansible-playbook playbook.yml --user=johndoe

# Prompt for sudo password
ansible-playbook playbook.yml --become --become-user=janedoe --ask-become-pass
```

Use `-i` for a custom inventory file. `--check` to run a dry run.
