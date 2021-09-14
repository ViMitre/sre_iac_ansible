### **What is Ansible?**
Ansible is a software tool that provides simple but powerful automation for cross-platform computer support. It is primarily intended for IT professionals, who use it for application deployment, updates on workstations and servers, cloud provisioning, configuration management, intra-service orchestration, and much more.<br>
**Benefits of Ansible**:
- Free: Ansible is an open-source tool.
- Very simple to set up and use: No special coding skills are necessary to use Ansible’s playbooks (more on playbooks later).
- Powerful: Ansible lets you model even highly complex IT workflows. 
- Flexible: You can orchestrate the entire application environment no matter where it’s deployed. You can also customize it based on your needs.
- Agentless: You don’t need to install any other software or firewall ports on the client systems you want to automate. You also don’t have to set up a separate management structure.
- Efficient: Because you don’t need to install any extra software, there’s more room for application resources on your server.

### **What is Infrastructure as Code (IaC)?**<br>
Infrastructure as Code (IaC) is a combination of standards, practices, tools, and processes to provision, configure, and manage computer infrastructure using code and other machine-readable files.

### **Why should we use Ansible?**
It can automate processes such as configuration management, application deployment, intraservice orchestration, and provisioning on many machines at the same time.

### **Default directory structure for Ansible**
```
/etc/ansible/
             ansible.cfg  
             hosts            
             roles/           
```
### **Inventory/hosts file**
The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. The file can be in one of many formats depending on your Ansible environment and plugins. Common formats include INI and YAML. The default location for the inventory file is /etc/ansible/hosts.
### **What should be added to hosts file to establish secure connection between Ansible controller and agent nodes?**
The connection type, username and password.
Example:<br>
`(IP) ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant`
### **What are Ansible ad-hoc commands?**
Commands that can be executed on any and any number of agent nodes simultaneously.
### **Structure of ad-hoc commands**
Starts with 'ansible', followed by the target agent, then the module and module options. In case of *shell*, these would be the commands.<br>
Example: `ansible all -m shell -a "pwd"`<br>
# **Ansible controller and agent nodes set up guide**
![](img/diagram1.png)

- Clone this repo and run `vagrant up`
- `(double check syntax/intendation)`

### Vagrantfile code:

```vagrant 
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what

# MULTI SERVER/VMs environment 
#
Vagrant.configure("2") do |config|

# creating first VM called web  
  config.vm.define "web" do |web|
    
    web.vm.box = "bento/ubuntu-18.04"
   # downloading ubuntu 18.04 image

    web.vm.hostname = 'web'
    # assigning host name to the VM
    
    web.vm.network :private_network, ip: "192.168.33.10"
    #   assigning private IP
    
    config.hostsupdater.aliases = ["development.web"]
    # creating a link called development.web so we can access web page with this link instread of an IP   
        
  end
  
# creating second VM called db
  config.vm.define "db" do |db|
    
    db.vm.box = "bento/ubuntu-18.04"
    
    db.vm.hostname = 'db'
    
    db.vm.network :private_network, ip: "192.168.33.11"
    
    config.hostsupdater.aliases = ["development.db"]     
  end

 # creating are Ansible controller
  config.vm.define "controller" do |controller|
    
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    config.hostsupdater.aliases = ["development.controller"] 
    
  end

end
```
### After running `vagrant up`, make sure all the vm-s are working and up to date:
- `vagrant status`
- SSH into each of them: `vagrant ssh "machine name"`
- Run `sudo apt-get update -y` and `sudo apt-get upgrade -y`
- Check their connection by tying to `ping` **web** and **db** machines from the **controller** machine

## Install Ansible on controller machine
Install dependencies first:
- `sudo apt-get install software-properties-common -y`
- `sudo apt-add-repository ppa:ansible/ansible`

Run update again: `sudo apt-get update -y` <br>
Install Ansible: `sudo apt-get install ansible -y`<br>
Check if it is installed: `ansible --version`

## Set up SSH passwords
- Run SSH from the controller machine into each machine: `ssh vagrant@IP`<br>
- Enter a password (vagrant)

## Add these into `/etc/ansible/hosts`:

```
[web]
192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant

[db]
192.168.33.11 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
```
### Check if they connect:
`ansible all -m ping`

### Run any shell command like this:
`ansible all -a "command"`<br>
*Note: you can replace 'all' with a machine name*

## Ansible Playbooks
- Ansible playbooks are a completely different way to use Ansible than ad-hoc task execution
- Ansible playbooks are .yml or .yaml files written in YAML (Yet Another Markup Language)
- Playbooks start with 3 dashes `---` 

### Install Nginx
```
# Playbook for [web]
---
- hosts: web # name of the host
  gather_facts: yes  # gather facts about the installation steps
  become: true # admin access

  # instructions to install nginx:
  tasks:
  - name: Install Nginx
    apt: pkg=nginx state=present

```
- Run playbook: `ansible-playbook nginx_playbook.yml`
- Check if it worked with and ad-hoc command: `ansible web -a "sudo systemctl status nginx"`
### Copy App from controller to web
```
---
- hosts: web
  become: true

  tasks:
  - name: Copy app
    ansible.builtin.copy:
      src: /home/vagrant/app
      dest: /home/vagrant/
```
### Install NodeJS
```
---
- hosts: web
  gather_facts: yes
  become: true

  tasks:
  - name: Install NodeJS
    shell: |
      curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
      sudo apt-get install -y nodejs
      cd /home/vagrant/app
      sudo npm install pm2 -g -y
      sudo npm install
```
### Install MongoDB
```
---
- hosts: db
  gather_facts: yes
  become: true

  tasks:
  - name: Install MongoDB
    shell: |
      wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
      echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
      sudo apt-get update -y
      sudo apt-get install -y mongodb-org
      sudo systemctl start mongod
      sudo systemctl enable mongod
```
### Change nginx config and restart
```
---
- hosts: web
  become: true

  tasks:
  - name: Copy nginx config file
    ansible.builtin.copy:
      src: /home/vagrant/config_files/default
      dest: /etc/nginx/sites-available/default

  - name: Restart nginx
    shell: |
        systemctl restart nginx
```
### Change mongod config and restart
```
---
- hosts: db
  become: true

  tasks:
  - name: Copy mongod.conf
    ansible.builtin.copy:
      src: /home/vagrant/config_files/mongod.conf
      dest: /etc/mongod.conf
  - name: Restart mongo
    shell: |
      sudo systemctl restart mongod
```

### Start the app
```
---
- hosts: web
  gather_facts: yes
  become: true

  tasks:
  - name: Start app
    environment:
      DB_HOST: mongodb://192.168.33.11:27017/posts
    shell: |
      cd /home/vagrant/app
      node seeds/seed.js
      pm2 kill
      pm2 start app.js
```
