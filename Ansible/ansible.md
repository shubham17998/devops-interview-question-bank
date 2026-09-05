# Ansible Interview Questions

## Core Concepts
1. What are Ansible Roles?
   A standardized directory structure (`tasks/`, `handlers/`, `templates/`, `vars/`, `defaults/`, `files/`, `meta/`) that packages related tasks, variables, templates, and handlers into a reusable, shareable unit — e.g., a "nginx" role you can apply to any playbook.

2. What are Tasks?
   The individual units of work in a playbook — each task calls a module (e.g., `package`, `copy`, `service`) with specific arguments to perform one action, like installing a package or starting a service.

3. What are Handlers?
   Special tasks that only run when notified by another task (via `notify:`), and only if that task actually reported a change. Commonly used to restart/reload a service only when its config file changed, rather than on every run.

4. What is idempotency?
   The property that running the same operation multiple times produces the same end state as running it once — re-running a playbook shouldn't cause unintended changes or errors if nothing actually needs to change. Most Ansible modules are idempotent by design (e.g., the `package` module only installs if not already present).

5. How do you design idempotent automation scripts?
   Use declarative modules that check current state before acting (Ansible's built-in modules already do this) rather than raw shell commands. When you must use `shell`/`command`, guard them with `creates`/`removes` or `when` conditions so they only run when the target state doesn't already hold. Avoid actions with side effects that vary between runs, and prefer "ensure state X" semantics (`state: present/absent`) over imperative one-off commands.

6. What is Jinja2, and why is it used in Ansible?
   Jinja2 is a Python templating engine. Ansible uses it for variable substitution (`{{ var }}`) in playbooks, and for the `template` module to generate config files dynamically (loops, conditionals, filters) from a `.j2` template and a set of variables.

## Playbook Controls
7. What is the purpose of the when keyword?
   Adds a conditional to a task — the task only executes if the expression evaluates to true (e.g., `when: ansible_os_family == "Debian"`), letting one playbook branch behavior based on facts or variables.

8. How do you use conditional execution in Ansible?
   Primarily via `when:` on a task, block, or role. You can combine conditions with `and`/`or`, check registered variable results (`when: result.rc != 0`), or use `failed_when`/`changed_when` to override how a task's success/change status is determined.

9. What does become: yes mean?
   Instructs Ansible to escalate privileges for that task (equivalent to `sudo` by default, configurable via `become_method`) — used when the task needs root/administrative access on the target host.

10. Where do we use Tags?
    Tags label tasks/roles/plays so you can selectively run or skip them at execution time — e.g., tag a task `- config` and run only that subset with `ansible-playbook site.yml --tags config`, useful for large playbooks where you don't want to run everything every time.

11. How do you run only specific tagged tasks?
    `ansible-playbook playbook.yml --tags "tagname1,tagname2"`. Use `--skip-tags` to exclude specific tags instead.

12. How do you use loops in Ansible?
    The `loop:` keyword iterates a task over a list: `loop: [item1, item2]` with `{{ item }}` referencing the current value inside the task. For more complex iteration (looping over dicts, nested data), `loop` with filters like `dict2items`, or the older `with_items`/`with_dict` constructs, can be used.

## Secrets & Security
13. How do you manage secrets in Ansible?
    **Ansible Vault** — encrypts sensitive variables/files (`ansible-vault encrypt vars/secrets.yml`) so they can be committed to version control safely, and decrypted automatically at runtime with a vault password (supplied via `--ask-vault-pass`, a password file, or integrated with a secrets manager). For larger environments, secrets are often pulled dynamically from HashiCorp Vault or AWS Secrets Manager via lookup plugins instead of being stored in the repo at all.

## Deployment & Operations
14. How do you automate Nginx/Apache deployment?
    Write a playbook/role that: installs the package (`package: name=nginx state=present`), deploys a config file from a Jinja2 template (`template:`), ensures the service is enabled and started (`service: name=nginx state=started enabled=yes`), and notifies a handler to reload/restart the service whenever the config template changes.

15. How do you deploy applications on 15 servers?
    Define the 15 hosts in an inventory (statically or via a dynamic inventory plugin, e.g., for AWS), then run a single playbook against that group — Ansible connects to all matching hosts and applies the same set of tasks. Use `serial:` in the play to control rollout batch size (e.g., `serial: 5` to deploy 5 at a time for a rolling deployment), and `forks` in ansible.cfg to control parallelism.

16. How do you handle errors in Ansible?
    Use `ignore_errors: yes` to continue past a failing task when appropriate, `failed_when:`/`changed_when:` to customize what counts as failure, `block`/`rescue`/`always` for try/catch/finally-style error handling, and `any_errors_fatal`/`max_fail_percentage` to control whether a failure on one host halts the whole run.

17. How do you perform rollback in Ansible without downtime?
    Ansible has no built-in "undo" — rollback is achieved by design: keep the previous release/artifact available (e.g., symlink-based releases like Capistrano-style deploys, or blue-green target groups), and re-run the playbook pointing at the previous known-good version/tag. Combined with `serial:` rolling deployment and health checks between batches, a failed rollout can be stopped early and the previous version re-deployed to affected hosts without taking the whole fleet down.

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
