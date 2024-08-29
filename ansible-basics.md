### Basic Ansible
This was done on a home lab running Debian 11. `tesseract` is my control-node.

1. Add Ansible to Sources list
1. Update the OS Sources
1. Install Ansible
1. Create SSH keys
1. Tell Ansible to use `ssh-agent` so you don't have to retype passwords
1. Use Ansible to copy the controle node SSH key to the ansible hosts
1. Use an Ansible playbook to ping the devices
1. Use an Ansible playbook to upgrade the devices

##### Add Ansible to Sources list
```
$ echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu focal main" | sudo tee /etc/apt/sources.list.d/ansible.list
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
$ sudo apt update
````

##### Install Ansible
```
$ sudo apt install ansible
```

##### Define hosts, Create Host file
Do not put special characters (like -) into the group names. Hosts should be FQDNs.

```
ariadne@tesseract:~/ansible$ cat /etc/ansible/hosts 
[proxmox]
<hosts redacted>

[docker]
<hosts redacted>

[k8s]
<hosts redacted>

[linux]
<hosts redacted>
```

##### Define Defaults, Modify ansible.cfg
```
ariadne@tesseract:/etc/ansible$ cat ansible.cfg 
# [output omitted]

[defaults]
host_key_checking = False
remote_user = ariadne
```

##### Create a public SSH key to allow passwordless access

I'm using an internal linux host called `tesseract`. It doesn't use a password, it's a home lab.
```
ariadne@tesseract:~$ ssh-keygen -t rsa -b 4096 -C "ariadne@tesseract.haske.org"
```

##### Write a playbook to copy the SSH keys
```
ariadne@tesseract:~/ansible$ cat copy_ssh_keys_test.yml 
---
- name: Copy SSH key to hosts
  hosts: all
  become: yes

  tasks:
  - name: Set authorized key taken from file
    authorized_key:
      user: ariadne
      state: present
      key: "{{ lookup(file, /home/ariadne/.ssh/id_rsa.pub) }}"
```
##### Run it
```
ariadne@tesseract:~/ansible$ ansible-playbook -k copy_ssh_keys.yml 
SSH password: 

PLAY [Copy SSH key to hosts] ***********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************************************************************
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]

TASK [Set authorized key taken from file] **********************************************************************************************************************************************************************************************************************
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
ok: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]
changed: [hosts-redacted]

PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hosts.redacted    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0     
```
##### Write a Playbook to Upgrade Everything
```
ariadne@tesseract:~/ansible$ cat upgrade-everything.yml 
---
- name: Update and upgrade apt packages
  hosts: all
  become: true
  tasks:
    - name: Update apt cache and upgrade all packages
      apt:
        upgrade: yes
        update_cache: yes
        cache_valid_time: 86400 #One day
```

##### Sources

https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian
https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html
