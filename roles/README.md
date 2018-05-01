# Roles

## Directory Structure

- Files: This directory contains regular files that need to be transferred to the hosts you are configuring for this role. This may also include script files to run.
- Handlers: All handlers that were in your playbook previously can now be added into this directory.
- Meta: This directory can contain files that establish role dependencies. You can list roles that must be applied before the current role can work correctly.
- Templates: You can place all files that use variables to substitute information during creation in this directory.
- Tasks: This directory contains all of the tasks that would normally be in a playbook. These can reference files and templates contained in their respective directories without using a path.
- Vars: Variables for the roles can be specified in this directory and used in your configuration files.
- Creating a new role's directories: `mkdir -p NEWROLENAME/{defaults,files,handlers,meta,templates,tasks,vars}`

From Ansible Docs:

This designates the following behaviors, for each role ‘x’:
- If roles/x/tasks/main.yml exists, tasks listed therein will be added to the play.
- If roles/x/handlers/main.yml exists, handlers listed therein will be added to the play.
- If roles/x/vars/main.yml exists, variables listed therein will be added to the play.
- If roles/x/defaults/main.yml exists, variables listed therein will be added to the play.
- If roles/x/meta/main.yml exists, any role dependencies listed therein will be added to the list of roles (1.3 and later).
- Any copy, script, template or include tasks (in the role) can reference files in roles/x/{files,templates,tasks}/ (dir depends on task) without having to path them relatively or absolutely.

## Task Strategy

When creating new tasks, it is best to plan from an OS agnostic point of view. For example, `tasks\main.yml` should use include logic based on the `ansible_os_family` variable. OS specific actions should be performed in a separate YAML file called by an include. The following example is from the `datadog` role:

```
---
- include_tasks: pkg-debian.yml
  when: ansible_os_family == "Debian"

- include_tasks: pkg-redhat.yml
  when: ansible_os_family == "RedHat"

- include_tasks: agent5.yml
  when: not datadog_agent6 and ansible_os_family != "Windows"

- include_tasks: agent6.yml
  when: datadog_agent6 and ansible_os_family != "Windows"

- include_tasks: centos-main.yml
  when: ansible_distribution == "CentOS"

- include_tasks: windows-main.yml
  when: ansible_os_family == "Windows"
...

```
