# Ansible Interview Questions

## Core Concepts
1. What are Ansible Roles?
2. What are Tasks?
3. What are Handlers?
4. What is idempotency?
5. How do you design idempotent automation scripts?
6. What is Jinja2, and why is it used in Ansible?

## Playbook Controls
7. What is the purpose of the when keyword?
8. How do you use conditional execution in Ansible?
9. What does become: yes mean?
10. Where do we use Tags?
11. How do you run only specific tagged tasks?
12. How do you use loops in Ansible?

## Secrets & Security
13. How do you manage secrets in Ansible?

## Deployment & Operations
14. How do you automate Nginx/Apache deployment?
15. How do you deploy applications on 15 servers?
16. How do you handle errors in Ansible?
17. How do you perform rollback in Ansible without downtime?

## Hands-on Exercises

### Exercise 1: My First Task
**Objective:** Write a single Ansible task.
**Task:** Write a task to create the directory `/tmp/new_directory`.

**Solution:**
```yaml
- name: Create a new directory
  file:
    path: "/tmp/new_directory"
    state: directory
```

### Exercise 2: My First Playbook
**Objective:** Write and run a full playbook against a remote host.
**Task:**
1. Write a playbook that installs the `zlib` package and creates the file `/tmp/some_file`.
2. Run the playbook against a remote host.

**Solution:**
```yaml
# first_playbook.yml
- name: Install zlib and create a file
  hosts: some_remote_host
  tasks:
    - name: Install zlib
      package:
        name: zlib
        state: present
      become: yes
    - name: Create the file /tmp/some_file
      file:
        path: '/tmp/some_file'
        state: touch
```
Inventory (`/etc/ansible/hosts`):
```
[some_remote_host]
some.remote.host.com
```
Run it:
```
ansible-playbook first_playbook.yml
```
