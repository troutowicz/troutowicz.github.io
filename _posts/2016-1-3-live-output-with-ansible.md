---
layout: post
title: Live output with Ansible
---

Before moving to [Ansible](http://www.ansible.com/) for automation and config management I used various ad-hoc scripts for simplifying remote tasks. One script, in particular, iterates over a set of hosts and serially executes a command on each. Due to the system variance in my environment this script has become incredibly helpful. It provides a simple interface for troubleshooting problems quickly and efficiently. Uses include restarting services, opening a file in a text editor, watching a log file in real time, and listing contents of directories. 

When trying to produce a similar script in Ansible, I soon found that it could not be done. This is by [design](https://github.com/ansible/ansible/issues/3887). Ansible executes tasks asynchronously, meaning a task will run on multiple hosts simultaneously. A side effect of this design is that displaying real-time output for a large amount of simultaneous tasks becomes difficult, if not unrealistic. Because of this, Ansible will only provide a remote task's output _after_ the task returns an exit code. This behavior is necessary for Ansible to perform efficiently at scale.

However, in my environment, performing the occasional ad-hoc task serially does make sense. When log monitoring has not reached every running application, or when directory paths vary on each server, having a way to quickly access and manage these nuances can be a lifesaver. I just need a way to _plug_ into Ansible's ecosystem and, luckily, Ansible's Python API provides a trivial solution.

Ansible has a class named [Inventory](https://github.com/ansible/ansible/blob/5af1cda7c93375bc84296c641ace49bca8657e6c/lib/ansible/inventory/__init__.py). This class contains everything needed for parsing Ansible inventory files, obtaining a subset of hosts, and accessing host/group vars. The three functions below cover my needs.

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

One other requirement I have is to provide CLI options similar to the `ansible` executable. `argparse` makes this easy.

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

Great! What I feared would be a headache to integrate was surprisingly straight forward. If you are interested in a generic `ansible`-like script using these functions, I have shared one on GitHub [here](https://github.com/troutowicz/ansible-live).
