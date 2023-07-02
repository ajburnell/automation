* Standing up multiple infrastructure servers with Vagrant and running ad-hoc commands.
All credit too: https://www.ansiblefordevops.com/

Example commands:

```bash
ansible multi -a "hostname"

# Only one fork (perform in sequence, no parallel processing):
ansible multi -a "hostname" -f 1

ansible multi -a "df -h"
```

## Using Ansible modules:
```bash
ansible multi -b -m dnf -a "name=chrony state=present"

ansible multi -b -m service -a "name=chronyd state=started enabled=yes"

ansible multi -b -a "chronyc tracking"
```

### Configuring a group of application servers:
```bash
ansible app -b -m dnf -a "name=python3-pip state=present"

ansible app -b -m pip -a "executable=pip3 name=django<4 state=present"

ansible app -a "python3 -m django --version"
```

### Configure the database server:
```bash
ansible db -b -m dnf -a "name=mariadb-server state=present"

# Firewall configuration:
ansible db -b -m dnf -a "name=firewalld state=present"
ansible db -b -m service -a "name=firewalld state=started enabled=yes"
ansible db -b -m firewalld -a "zone=database state=present permanent=yes"
ansible db -b -m firewalld -a "source=192.168.56.0/24 zone=database state=enabled permanent=yes"
ansible db -b -m firewalld -a "port=3306/tcp zone=database state=enabled permanent=yes"

ansible db -b -m dnf -a "name=python3-PyMySQL state=present"
ansible db -b -m mysql_user -a "name=django host=% password=12345 priv=*.*:ALL state=present"
```

### Limiting commands to certain hosts:
```bash
ansible app -b -a "service chronyd restart" --limit "192.168.56.4"

# Limit hosts with a simple pattern (asterisk is a wildcard).
ansible app -b -a "service chronyd restart" --limit "*.4"

# Limit hosts with a regular expression (prefix with a tilde).
ansible app -b -a "service chronyd restart" --limit ~".*\.4"
```

### Users and groups
```bash
ansible app -b -m group -a "name=admin state=present"

ansible app -b -m user -a "name=johndoe group=admin createhome=yes"

# Delete the account
ansible app -b -m user -a "name=johndoe state=absent remove=yes"

### Platform agnostic package management
```bash
ansible app -b -m package -a "name=git state=present"
```

### Files & directories
```bash
# Get information about a file
ansible multi -m stat -a "path=/etc/environment"

# Local to remote copy
ansible multi -m copy -a "src=/etc/hosts dest=/tmp/hosts"
# If you include a trailing slash, only the contents of the directory will be copied into the dest. If you omit the trailing slash, the contents and the directory itself will be copied into the dest.

# Remote to local copy
ansible multi -b -m fetch -a "src=/etc/hosts dest=/tmp"

# Create a directory
ansible multi -m file -a "dest=/tmp/test mode=644 state=directory"

# Symlink
ansible multi -m file -a "src=/src/file dest=/dest/symlink \
state=link"

# Delete
ansible multi -m file -a "dest=/tmp/test state=absent"
```

### Background operations:
* `-B <seconds>`: the maximum amount of time (in seconds) to let the job run.
* `-P <seconds>`: the amount of time (in seconds) to wait between polling the
servers for an updated job status.

```bash
ansible multi -b -B 3600 -P 0 -a "dnf -y update"

ansible multi -b -m async_status -a "jid=169825235950.3572"
```

### Check log files
```bash
ansible multi -b -a "tail /var/log/messages"

# Use shell for redirection
ansible multi -b -m shell -a "tail /var/log/messages | grep ansible-command | wc -l"
```

### Cron jobs
```bash
ansible multi -b -m cron -a "name='daily-cron-all-servers' hour=4 job='/path/to/daily-script.sh'"

# Remove
ansible multi -b -m cron -a "name='daily-cron-all-servers' state=absent"
```

`*` is assumed for values not specified (`day`, `hour`, `minute`, `month`, `weekday`). Other useful values include `special_time`, `user` and `backup=yes`.

### Version controlled application deployment
```bash
ansible app -b -m git -a "repo=git://example.com/path/to/repo.git dest=/opt/myapp update=yes version=1.2.4"
```

