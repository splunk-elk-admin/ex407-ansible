4. Implementing Task Control 
 \- Writing loops and conditional tasks 
   - loop keyword
   - the loop variable 'item' holds the value during each iteration 
================================
Simple Loop:
- name: Services are started 
  service:
    name: "{{ item }}"
	state: started 
  loop:
    - postfix 
	- sshd 
================================
Simple Loop 1:
 vars:
   mail_services:
     - posfix 
	 - dovecot 
 tasks:
 - name: Services are started 
   service:
     name: "{{ item }}"
	 state: started 
   loop: "{{ mail_services }}"
====================================
Loops over Dict: 
- name: users exist in correct group
  user:
    name: "{{ item.name }}"
	state: present 
	groups: "{{ item.groups }}"
  loop: 
    - name: jane 
	  groups: wheel 
	- name: joe 
	  groups: root 
====================================
Earlier-Style Ansible Loops:
with_items => Lists 
with_file => item will be set to the content of each file in sequence
with_sequence => with_sequence: start=4 end=16 stride=2
======================================================================
Using Register Variables with Loops
- name: Loop Register Test
  gather_facts: no
  hosts: localhost
  tasks:
  - name: Looping Echo Task
    shell: "echo This is my item: {{ item }}"
    loop:
      - one
	  - two
    register: echo_results
  - name: Show stdout from the previous task.
    debug:
      msg: "STDOUT from previous task: {{ item.stdout }}"
    loop: echo_results['results']
======================================================================
Conditional Task:
- name: Simple Boolean Task Demo 
  hosts: localhost 
  vars:
    #run_my_task: true 
    run_my_task: false
  tasks:
  - name: httpd package is installed 
    yum:
      name: httpd
    when: run_my_task
======================================================================
Conditional Task 2: 
- name: Test Variable is Defined Demo
  hosts: all
  vars:
    my_service: httpd
  tasks:
  - name: "{{ my_service }} package is installed"
    yum:
      name: "{{ my_service }}"
    when: my_service is defined
======================================================================
Conditional Task 3:
- name: Demonstrate the "in" keyword
  hosts: all
  vars:
    supported_distros:
      - RedHat
      - Fedora
  tasks:
  - name: Install httpd using yum, where supported
    yum:
      name: http
      state: present
    when: ansible_distribution in supported_distros
======================================================================
Testing Multiple Conditions:
when: ansible_distribution == "RedHat" or ansible_distribution == "Fedora"

when: ansible_distribution_version == "7.5" and ansible_kernel == "3.10.0-327.el7.x86_64"

When a list is provided to the when keyword, all of the conditionals are combined using the and operation.
when:
  - ansible_distribution_version == "7.5"
  - ansible_kernel == "3.10.0-327.el7.x86_64"
  
More complex conditional statements can be expressed by grouping conditions with parentheses.
when: >
    ( ansible_distribution == "RedHat" and
    ansible_distribution_major_version == "7" )
    or
    ( ansible_distribution == "Fedora" and
    ansible_distribution_major_version == "28" )
======================================================================
Combining loops and conditionals:

- name: install mariadb-server if enough space on root
  yum:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_mounts }}"
  when: item.mount == "/" and item.size_available > 300000000
======================================================================
Condition and Loop : 
- name: Condition and Loop Play 
  hosts: localhost 
  gather_facts: yes 
  tasks:
  - name: install mariadb server if enough space on root 
    yum:
      name: mariadb-server 
      state: present 
    loop: "{{ ansible_mounts }}"
    when: item.mount == "/" and item.size_available > 30000000000
======================================================================
Condition and Register:
---
- name: Restart HTTPD if Postfix is Running
hosts: all
tasks:
- name: Get Postfix server status
command: /usr/bin/systemctl is-active postfix
ignore_errors: yes
register: result
- name: Restart Apache HTTPD based on Postfix status
======================================================================
