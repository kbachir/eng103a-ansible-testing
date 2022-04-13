# **Playbooks** 

- Playbooks was used throughout this project as it saved time having to manually enter nodes and carry out commands. 
- Playbooks are one of the core features of Ansible as it tells Ansible what to execute
- They are written in YML/YAML 
- They are repeatable, re-usable and simple to use
- Due to them being repeatable and re-usable it meant less human error 

### **check_status_docs.yml**
```
# name of the host/instance/vm
- hosts: all
# admin access to install any dependencies
  become: yes
# collect logs or gather facts
  gather_facts: yes
  tasks:
  - name: "Get service facts"
# Gathers service state information of all the available services as fact data
    service_facts:    
  - name: "Check the service nginx facts as they are"
# Specifically filters out the state of nginx service and displays it on the console
    debug:
      msg: "{{ansible_facts.services['nginx.service'].state}}"
```
- This playbook gets all the service state information for all available services
- It then displays the service of Nginx as thats the service selected
  
### **check_svc_docks.yml**
```
# name of the host/instance/vm
- hosts: all
# admin access to install any dependencies
  become: true
# collect logs or gather facts
  gather_facts: yes
  tasks:
  - name: "Get service facts"
# Gathers service state information of all the available services as fact data
    service_facts:
  - name: "Check the service facts as they are"
# Displays the state of all the available services on the console 
    debug:
      msg: "{{ansible_facts.services}}"
```
- This playbook displays all the available services on the console

### **nginx_handler_docs.yml**
```
---
# name of the host/instance/vm
- hosts: web
  tasks:
  - name: Install nginx
# Installs nginx and asserts its state to be present
    package:
      name: nginx
      state: present
    notify:
# Triggers the handler 
    - Start nginx
  - name: ensure Nginx is running
# Checks if nginx is running and in the started state
    service:
      name: nginx
      state: started
  handlers:
# Starts nginx service. Performed at the end of the play once all other tasks are finished
    - name: Start nginx
      service:
        name: nginx
        state: started
```
- This playbook is used to install nginx
- After installation, it checks whether nginx is up and running

### **os_required_patching_docs.yml**
```
# os_required_patching.yml
# name of the host/instance/vm
- hosts: all
# collect logs or gather facts
  gather_facts: true
# admin access to install any dependencies
  become: true
  tasks:
  - name: run apt-get
# Installs tree only when the servers are ubuntu 
    command: apt install tree -y
    when: ansible_distribution == "Ubuntu"
  - name: run yum update
# Runs update only for servers which are centOS
    command: yum update
    when: ansible_distribution == "CentOS"
  - name: run yum
# Installs tree only for the servers that are centOS
    command: yum install tree -y
    when: ansible_distribution == "CentOS"
```
- This playbook installs "Tree" only if the servers are "Ubuntu"
- Tree is used for visual purposes
- Runs an updates for servers which are using "centOS" and then install "Tree"

### **test_docs.yml**
```
# test.yml
---
- name: tests
# name of the host/instance/vm
  hosts: all
# admin access to install any dependencies
  become: yes
# collect logs or gather facts
  gather_facts: no
  tasks:
    - name: store date output for timezone check
# Checks the time zone of all the hosts and stores it in a variable check.tz
      command: date
      register: check_tz

    - name: check tz
# Asserts that the variable check.tz is returning UTC in the standard output (stdout)
      assert:
        that: "'UTC' in check_tz.stdout"

    - name: "Check if NGINX is installed"
      package_facts:
        manager: "auto"

    - name: confirm nginx is installed
# Asserts that nginx is in the list of all available packages
      assert:
        that: "'nginx' in ansible_facts.packages"

    - name: Check if port 80 is listening
# Checks for all the ports listening and stores it in the variable port_check
      shell: lsof -i -P -n | grep LISTEN
      register: port_check

    - name: confirm port 80 is listening
# Asserts that the variable port_check is returning port 80 in the standard output (stdout)
      assert:
        that: "'*:80 (LISTEN)' in port_check.stdout"
```
- This playbook checks the current time zone for all the hosts and then stores it in a variable called "check.tz"
- Makes sure the variable is returning the time in the standard output
- Does a check to see if nginx is installed
- Does a check to see if port 80 is listening 

### **test_nginx_service_docs.yml**
```
---
# name of the host/instance/vm
- hosts: all
# admin access to install any dependencies
  become: yes
# collect logs or gather facts
  gather_facts: yes
  tasks:
  - name: "Populate service facts"
# Gathers service state information of all the available services as fact data
    service_facts:
  - name: "Check the service nginx facts as they are"
# Specifically filters out the state of nginx service and displays it on the console
    debug:
      msg: "{{ansible_facts.services['nginx.service'].state}}"
  - name: "Verify if nginx is running!"
# Asserts that nginx service is running in all hosts and prints a message on the console depending on its status
    assert:
      that:
        - "'{{ansible_facts.services['nginx.service'].state}}' == 'running'"
      fail_msg: "nginx is down, please check nginx status or restart!"
      success_msg: "no issues, service is running as expected"
```
- This playbook gathers all the information for all the available services
- Checks if nginx is running and then displays a message on the console depending on the status of nginx

### **test_solution_docs.yml**
```
- name: tests
# name of the host/instance/vm
  hosts: all
# admin access to install any dependencies
  become: yes
# collect logs or gather facts
  gather_facts: no
  tasks:
    - name: set timezone to CEST
# Setting time zone to UTC
      timezone:
        name: UTC

    - name: install nginx
# Installing nginx
      apt:
        pkg: nginx
        state: present

    - name: start nginx
# Start nginx service
      service:
        name: nginx
        state: started
```
- This playbook sets the timezone to UTC
- Installs and starts nginx

### **update_required_servers_docs.yml**
```
---
# name of the host/instance/vm
- hosts: all
# admin access to install any dependencies
  become: true
  tasks:
# only update Ubuntu servers
  - name: Update Ubuntu
    apt: update_cache=yes upgrade=dist
    when: ansible_distribution == "Ubuntu"

# only update RedHat Servers
  - name: Update RedHat
    apt: name=* update_cache=yes
    when: ansible_os_family == "RedHat"
```
- This playbook carries out updates on Ubuntu and RedHat servers
