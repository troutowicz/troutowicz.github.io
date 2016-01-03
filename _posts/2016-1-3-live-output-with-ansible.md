---
layout: post
title: Live output with Ansible
---

Before moving to [Ansible](http://www.ansible.com/) for automation and config management I used various ad-hoc scripts for simplifying remote tasks. One script in particular would iterate over a set of hosts and serially execute a command on each. This script is very helpful due to the system variance in my environment and makes troubleshooting problems easier. Uses include viewing a conf file with a text editor, watching a log file in real time, and listing contents of directories. Ansible, however, does not work serially.

Ansible is designed to execute tasks asynchronously, meaning a task will execute on multiple hosts in parallel. This behavior is necessary for an efficient automation tool, but I still wanted to perform some tasks serially while making use of Ansible's inventory files. Luckily, Ansible's Python API provides a trivial solution.

Ansible has a class named [Inventory](https://github.com/ansible/ansible/blob/5af1cda7c93375bc84296c641ace49bca8657e6c/lib/ansible/inventory/__init__.py). This class contains everything needed for parsing Ansible inventory files, obtaining a subset of hosts, and accessing host/group vars.

**Note: This code snippet references the API of Ansible [1.9.4-1](https://github.com/ansible/ansible/tree/5af1cda7c93375bc84296c641ace49bca8657e6c).**

```python
from ansible.inventory import Inventory

# initialize Inventory instance
inventory = Inventory('/etc/ansible/hosts')

# filter hosts if given a limit
inventory.subset('appservers')

# obtain host variables
inventory.get_variables('appserver01')
```

Wow, that was easy. I now have everything I need to integrate my script with Ansible's inventory system. One other requirement I have is to have CLI options similar to the `ansible` executable.

```
usage: ansible-live [-h] [-u USER] [-i INVENTORY] [-l LIMIT] command

positional arguments:
  command               The command to execute

optional arguments:
  -h, --help            show this help message and exit
  -u USER, --user USER  The user used for auth
  -i INVENTORY, --inventory INVENTORY
                        The PATH to the inventory hosts file
  -l LIMIT, --limit LIMIT
                        Further limits the selected host/group patterns.
```

Great! What I feared would be a headache to integrate was surprisingly straight forward. If you are interested in my finished script, I have shared it on GitHub [here](https://github.com/troutowicz/ansible-live).
