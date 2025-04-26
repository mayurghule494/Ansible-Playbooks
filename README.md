# Ansible-Playbooks

========================================


**ignore_errors: yes**

----> Allows the task to continue even if it fails.
- name: Try something risky
  command: /bin/false
  ignore_errors: yes

=======================================
**failed_when**
----> Custom condition for failure.
- name: Fail only if output contains "error"
  command: some_command
  register: result
  failed_when: "'error' in result.stdout"
  
======================================
**Basic loop:**

- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - htop
=====================================
**rescue:**
This section only runs if the task inside block fails (i.e., if install_app returns an error).
In this case, it logs a debug message: "Install failed, handling error."

**always:**
This section always runs, whether the block task fails or succeeds.
It ensures that any temporary files are cleaned up by running cleanup_temp_files.

- block:
    - name: Try to install app
      command: install_app
  rescue:
    - name: Handle the failure
      debug:
        msg: "Install failed, handling error."
  always:
    - name: Cleanup
      command: cleanup_temp_files

====================================

**Nested loops (using with_nested):**
- name: Combine users and groups
  debug:
    msg: "User: {{ item.0 }}, Group: {{ item.1 }}"
  with_nested:
    - [ 'alice', 'bob' ]
    - [ 'admin', 'users' ]

=====================================
**when Statements:**

- name: Restart service if OS is Ubuntu
  service:
    name: apache2
    state: restarted
  when: ansible_facts['os_family'] == "Debian"


=====================================
**until with retries:**
Retry a task until it succeeds or times out.

- name: Wait for service to respond
  uri:
    url: http://localhost:8080
    status_code: 200
  register: result
  until: result.status == 200
  retries: 5
  delay: 10

=====================================

**Asynchronous Tasks:**
Run long tasks in the background.

- name: Start a long process
  shell: /usr/bin/long_running_task.sh
  async: 300  # seconds
  poll: 0     # don't wait
  
========================================
**Delegation (delegate_to):**
Run a task on another host, not the one in the play.

- name: Fetch logs from web server to control node
  fetch:
    src: /var/log/app.log
    dest: ./logs/
  delegate_to: localhost

=======================================
