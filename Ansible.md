# Ansible

## Concepts

### Control Node

A Control node is a Virtual or Physical machine on which Ansible is installed and which manages the required VMs/physical machines or network devices known as managed nodes or simply nodes. 
Any [*nix machine](https://en.wikipedia.org/wiki/Unix-like) with python can be used as a Ansible Control node. Install Ansible and you're ready to go with a few configurations.

### Managed Node

Any network devices or servers that is managed with Ansible is called as Managed Node. They are referred to as hosts. No Ansible agent needs to be installed on the worker nodes.

### Inventory

An inventory file is a  list of the nodes that are to be managed by Ansible from control node. 
An inventory is also known as "hostfile"

The default inventory file is located at `/etc/ansible/hosts`

### Module
A module is a unit of code that Ansible executes and each module has a particular use.
User Management to managing VLAN to provisioning infrastructure, there is a module for each of the activities. 
Ansible come with a lot of modules for each activity. A Few examples are:

  
  |      Module        | Use Case              |
  |--------------------|-----------------------|
  | setup              | Gather facts from the managed node|
  | shell | Run shell script on the desired managed nodes| 
  | apt/yum| Install/uninstall/update packages on the system. |  
  |service |Module to manage services on a system |
  | copy |  Copy files/directories to the desired managed nodes | 
  | file | Create/delete files/directories on managed nodes | 
  |lineinfile | Edit the existing files |
  

All the pre-defined modules can be found [here](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html) 

There are modules for programming languages such as `pip` for python and `npm` for Node pacakges.

You could develop your own module and publish it or use it within your environment(s). Refer [developing your own module](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html) for more information. 

### Tasks
A module in use is called as a task. 
For example, using `yum` module to install a certain package is a task.
Tasks can be run ad-hoc

Exmaple task1:

```yaml
- name: install apache pacakge       # Name for your Task - a free text field
  yum:                               # OS Package Module(usually apt/yum - depends on the operating system)
    name: httpd                      # Package that you'd like to manage with the yum module.
    state: present                   # State tells ansible whether you'd like to install/remove or update a package.
```

Example task2: 

```yaml
- name: Copy  fles                   # Name for your Task - a free text field
  copy:                              # Copy module to copy files or directories.
    src: /opt/myfiles/file_to_copy   # Path to the files  to be copied - usually this is a path on the controller node or a url
    dest: /var/www/html              # Destination path - the path where the files need to be placed on the managed nodes.
    owner: '1001'                    # which user owns the files in the nodes - User ID or usename can be specified
    group: '1001'                    # which group owns the files in the nodes - group ID or group name can be specified
    mdoe: '0755'                     # Permissions for the file.   
```


### Handlers 

Handles are a special type of tasks. They can be defined like a regular task but they'll be executed only if there is a change noticed in one of the tasks where `notify` is used 

An example of a handler being used in a playbook:

```yaml
tasks:
  - name: update configuration file
    copy:
      src: files/config.conf
      dest: /etc/myapp/config.conf
    notify: restart myapp service

handlers:
  - name: restart myapp service
    service:
      name: myapp
      state: restarted
```

### Using Roles

Roles are a way to organize and package multiple tasks, handlers, and other files into reusable components. Roles can be thought of as building blocks for playbooks, allowing you to abstract common tasks and configurations into separate components that can be shared and reused.

Roles are defined in a specific directory structure and can include tasks, handlers, files, templates, and other resources. A typical directory structure for a role looks like this:

```yaml
roles/
  webserver/
    tasks/
      main.yml
    handlers/
      main.yml
    files/
      index.html
    templates/
      nginx.conf.j2
```

In this example, the "webserver" role includes a main task and handler file, as well as a file and template for configuring the nginx web server. This role can be included in a playbook by referencing the role directory:

```yaml
- name: install and configure web server
  hosts: webservers
  become: true
  roles:
    - webserver
```

This will include the tasks, handlers, and resources from the "webserver" role into the playbook, allowing you to easily reuse and share common configurations and tasks.

### Playbooks

A list of tasks together is known as a Playbook. 
They are written in YAML.
Playbooks can also include variables along with tasks.



An example playbook. The blow playbook 

```yaml
---
- hosts: webservers              # List of serves on which the below tak
  become: yes                    # Privilege escalation - yes means tasks would be executed in sudo mode
  vars:                          # Variables for this play
    newdir: "test123"
  vars_files:                    # Variables from external file are included in this play.
    - vars.yaml
  tasks:
    - name: Installing '{{ webvars.webserver }}'  # The name of the task is dynamically generated. with the variables called 'webserver' which is a part of the variable group 'webvars' in vars.yaml file 
      apt:
        name: '{{ webvars.webserver }}'           # The package to be installed . The value is poplulated from the var 'webserver' which is part of the variable group 'webvars' in vars.yaml file 
        state: present
        update_cache: true                        # This is to ensure the sudo apt-get update is run before installing the packages. Not requried Redhat systems. 
    - name: Copy html files
      copy:                                       # Copy module to copy files from controller to the desired nodes. In this case, the desired nodes are 'webservers' in hosts file
        src:  '{{ webvars.htmlfiles }}'           # Source files - value is populated from the variable 'htmlfiles' under 'webvars' group in vars.yaml file 
        dest: /var/www/html/
        owner: www-data
        group: www-data
        mode: 775
      notify:
        - restart nginx server  
    - name: Create a directory if it does not exist
      file:
        path: "/home/azadmin/'{{ newdir }}'"
        state: directory
        mode: '0755'
  handlers:  
    - name: restart nginx server
      service:
        name: '{{ webvars.webserver }}'
        state: restarted
- hosts: dbservers
  become: yes
  vars_files:
    - vars.yaml
  tasks:
    - name: Install MariaDB
      apt: 
        name: '{{ dbvars.sqlserver }}'
        update_cache: yes
  ```





























## Installing Ansible.

```bash
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

## Setting up inventory

By default ansible's inventory file or hostfile is located at `/etc/ansible/hosts`
This file can be updated with the list of your managed nodes and they can be grouped into different groups.
For instance you can have a group of web servers and another group for DB servers 
Inventory file is a simple text file. 

Example inventory file:

```bash
[web]
192.168.10.1
192.168.10.2

[db]
192.168.10.5
192.168.10.6
```

We could use a custom inventory file by using the `-i` switch

`ansible -i <inventory-file> <commands>`
When adding control node to the inventory add the below line so that ansible wouldn't try to ssh into the control machine as well.
local 
`ansible_collection=local` 



### Working with playbooks

Playbooks are a set of steps that can be performed in sequence or in parallel.
Playbooks are written in YAML.

Read more about Yaml syntax here: https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html#yaml-syntax

### Handlers

















