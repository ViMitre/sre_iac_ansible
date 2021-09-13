# What is Ansible?
Ansible is a software tool that provides simple but powerful automation for cross-platform computer support. It is primarily intended for IT professionals, who use it for application deployment, updates on workstations and servers, cloud provisioning, configuration management, intra-service orchestration, and much more.

# What is Infrastructure as Code (IaC)?
Infrastructure as Code (IaC) is a combination of standards, practices, tools, and processes to provision, configure, and manage computer infrastructure using code and other machine-readable files.
# Ansible controller and agent nodes set up guide
![](img/diagram1.png)

- Clone this repo and run `vagrant up`
- `(double check syntax/intendation)`

## We will use 18.04 ubuntu for ansible controller and agent nodes set up 
### Please ensure to refer back to your vagrant documentation

- **You may need to reinstall plugins or dependencies required depending on the OS you are using.**

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
- Enter a password

## Add these into `/etc/ansible/hosts`:

```
[web]
192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant

[db]
192.168.33.11 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
```
### Check if they connect:
`ansible all -m ping`