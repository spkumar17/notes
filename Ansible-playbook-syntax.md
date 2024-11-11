# Ansible

## Example Play

```
- name: Playbook to install generic tools
  hosts: pvt
  gather_facts: yes
  become: yes
  become_user: root
  vars:
    TFROM: 1.8.5
  tasks:
    - name: check the hostnames
      shell: cat /etc/lsb-release
    - name: perform apt update
      shell: apt-get update
    - name: install tools
      shell: apt-get install -y unzip jq stress net-tools
```

### Playbook Metadata

**- name:**

Playbook Name: Playbook to install generic tools

**Hosts:** `pvt`  This playbook targets hosts defined under the `pvt` inventory group.

**Gather Facts:**
 `yes` This gathers system information from the target hosts, which can be used within the playbook.
we can understand the use of this when using **when** block

**Privilege Escalation:**
**become: yes** — Enables privilege escalation (similar to sudo) to run tasks with elevated permissions.
**become_user: root** — Specifies that tasks should run as the root user.

**Task:**

**Task Name:** check the hostnames

**Module:** shell (here we are using shell)

**Command:** cat /etc/lsb-release (command used under the shell block)

**vars:**
    TFROM: 1.8.5  # The version of terraform to be installed
```
     - name: Download terraform version {{ TFROM }}
      shell: |
        wget https://releases.hashicorp.com/terraform/{{ TFROM }}/terraform_{{ TFROM }}_linux_amd64.zip -O /tmp/terraform.zip

```

TFROM variable, ensuring the playbook installs terraform at the specified version.

```
vars:
    AWS_ACCOUNT: "211125710812"
- name: Print Variable
  debug:
    msg: System {{ inventory_hostname }} has variable {{ AWS_ACCOUNT }}
```

This task uses the debug module to print a message containing the hostname (inventory_hostname) and the AWS_ACCOUNT variable.
**Output** : `System testserver01 has variable 021891605639`

In Ansible, {{ inventory_hostname }} is a built-in variable that represents the name of the current host as defined in the inventory file. It’s available in any playbook by default


```
vars:
    AWS_ACCOUNT: "211125710812"
 - name: Print Variable Sesitive
      debug:
        msg: System {{ inventory_hostname }} has variable {{ AWS_ACCOUNT }}
      no_log: True
```
This task also prints a similar message but is marked as sensitive using no_log: True.
 The no_log: True option tells Ansible to hide the message in the output logs, treating it as sensitive information. Instead of the actual value, Ansible will log **HIDDEN** in its place to protect sensitive information from being displayed in plain text. 

 In Ansible, {{ inventory_hostname }} is a built-in variable that represents the name of the current host as defined in the inventory file. It’s available in any playbook by default


**Play tasks**

```
  - name: Perform apt update & Install basic packages
      shell: apt update && apt install -y unzip jq net-tools stress
      when: (ansible_facts['distribution'] == "Ubuntu" and ansible_facts['distribution_version'] == "24.04")
      timeout: 30
      tags:
        - tools
```

**when:**

* The when condition ensures that this task is only executed when the host is running Ubuntu 24.04.

* ansible_facts['distribution'] == "Ubuntu" checks if the operating system is Ubuntu.

* ansible_facts['distribution_version'] == "24.04" checks if the Ubuntu version is 24.04.

* If both conditions are true, the task will run; otherwise, it will be skipped.

* This condition uses the facts gathered by Ansible (which comes from gather_facts: yes).

**timeout:**

timeout: 30: Sets a maximum execution time of 30 seconds for the task. If the task exceeds this time, it will be aborted and marked as failed. This can be useful for tasks that might hang or take too long to complete.

### **tags:**

tags: [tools]: Tags allow you to selectively run or skip tasks when executing the playbook. If you run the playbook with the --tags tools option, only tasks with the tools tag will be executed.

**To skip tasks with the tools tag:**

`ansible-playbook your_playbook.yml --skip-tags tools`

**To run only tasks with the tools tag:**

`ansible-playbook your_playbook.yml --tags tools`

**All tasks**

`ansible-playbook your_playbook.yml`
