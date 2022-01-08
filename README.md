# Ansible Role For SSH

Set Basic security configuration using `rkhunter` and `clamav`.

# Examples

Directory tree.

```
.
├── README.md
├── ansible.cfg
├── group_vars
│   ├── all.yml
│   ├── webserver_centos.yml
│   └── webserver_ubuntu.yml
├── inventory
└── webservers.yml
```

## Inventory file

```
[webservers:children]
webserver_ubuntu
webserver_centos

[webserver_ubuntu]
ubuntu ansible_host=10.10.10.10 ansible_port=22

[webserver_centos]
centos ansible_host=10.10.10.11 ansible_port=22
```

## Group Vars / Common settings(all.yml)

`all.yml` sets common variables.

```
# Common settings
become: yes
ansible_user: root

# Private_key is saved in local host only!
ansible_ssh_private_key_file: ""
```

## Group Vars / Ubuntu(webserver_ubuntu.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

This role references the `rkhunter` variable and `clamav` vairable.

The following example shows that

* rkhunter does not execute propupd as an ansible task.
* rkhunter sends the result of the cron job to root(defined by `rkhunter.mail_to`)
* clamav daemon service and freshclam service's user and group are clamav(defined by `clamav.user` and `clamav.group`)
* Ansible does not run freshclam as a task.
* The command of clamdscan except for some directories(defined by `clamav.exclude_paths`)
* clamdscan sends the scan result to root(defined by `clamav.mail_to`)

```
ansible_user: ubuntu
become: yes
ansible_become_password: 'ThisIsSecret!'

rkhunter:
  propupd: no
  mail_to: root

clamav:
  user: clamav
  group: clamav
  exclude_paths:
    - ^/boot/
    - ^/dev/
    - ^/proc/
    - ^/sys/
    - ~/run/
    - ^/var/log/clamav/virus/
  freshclam: no
  mail_to: root
```

## Group Vars / CentOS(webserver_centos.yml)

`webserver_ubuntu.yml` is `webservers` host's children.
When appling to CentOS, you can define same parameters.

```
rkhunter:
  propupd: no
  mail_to: root

clamav:
  user: clamav
  group: clamav
  exclude_paths:
    - ^/boot/
    - ^/dev/
    - ^/proc/
    - ^/sys/
    - ^/var/log/clamav/virus/
    - ~/var/spool/
  freshclam: no
  mail_to: root
```

# How to DryRun and Apply

DryRun

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -CD webservers.yml --tags security
```

Apply

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -D webservers.yml --tags security
```