Ansible from KodeKloud

Ansible - playbooks , inventory files
Using ansible you can make changes in public cloud like AWS or VMs

VID is nothing but virtual disk

osboxes.org have all the OS images

full clone, linked clone

the host names can be changed from /etc/hosts and /etc/hostname

ansible directory is located at /etc/ansible/ --> here the config file is "ansible.cfg"  and the hosts list file can be "inventory"

ansible is agent less ---> so you just need one server where ansible is installed and for the target hosts, for linux it uses ssh and for windows it uses powershell remoting command

the info about the target systems in available in inventory

ansible target1 -m ping -i inventory.txt 

target1 ansible_host=192.168.209.131 ansible_ssh_pass=osboxes.org --> this is the entry in above inventory.txt

<aliase> ansile_host=<host dns name> 

#### Ansible concepts -->
inventory
playbooks
modules
variables

inventory parameters -->
	ansible_host
	ansible_ssh_pass --> we dont use it generally in prod, it is done using ssh key based authentication
	ansible_port
	ansible_user
	ansible_connection --> ssh/winrm(windows remote)/localhost

eg --> web ansible_host=<name>.com ansible_connection=ssh

##############
# Sample Inventory File

# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# Database Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123! (notice we are using ansible_password for windows)


[web_servers] --> grouping of nodes
web1
web2
web3

[db_servers]
db1

[all_servers:children] --> grouping of groups
web_servers
db_servers
#############




### Playbooks are ansibles orchestration language --> here we write what ansible should do 
all playbooks are written in YAML

different actions run by tasks are called modules --> in ansible there are 100's of modules... like yum, script, service, command, ping...

ansible-doc -l --> contains all the list of modules vailable now..
ansible-playbook <playbook_name> -->
ansible-playbook --help


################# Sample playbook
-
    name: 'Stop the web services on web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Stop the web services on web server nodes'
            command: 'service httpd stop'
-
    name: 'Shutdown the database services on db server nodes'
    hosts: db_nodes
    tasks:
        -
            name: 'Shutdown the database services on db server nodes'
            command: 'service mysql stop'
-
    name: 'Restart all servers (web and db) at once'
    hosts: all_nodes
    tasks:
        -
            name: 'Restart all servers (web and db) at once'
            command: /sbin/shutdown -r
-
    name: 'Start the database services on db server nodes'
    hosts: db_nodes
    tasks:
        -
            name: 'Start the database services on db server nodes'
            command: service mysql start
-
    name: 'Start the web services on web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Start the web services'
            command: service httpd start
            
######################

Modules : --> ansible has 100s of modules and below are few, which help with our automations

system
file
cloud
database
windows
command --> used to execute a command on remote node
script --> runs a local script on remote nodes after copying it to remote nodes
service --> used to maintain services on system---> you can start, stop and restart it
lineinfile --> search for a line in a file and replace it or add it 

/etc/resolv.conf --> This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.

############# sample to use module lineinfile, script , user and service 
-

    name: 'Execute a script on all web server nodes and start httpd service'
    hosts: web_nodes
    tasks:
        -
            name: 'Update entry into /etc/resolv.conf'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver 10.1.250.10'
        -   
            name: 'add new user'
            user:
                name: web_user
                uid: 1040
                group: developers

        -
            name: 'Execute a script'
            script: /tmp/install_script.sh
        -
            name: 'Start httpd service'
            service:
                name: httpd
                state: present


################ 

'{{ variable_name}}' --> this format is called jinja2 templating --> below sample show the usage of varibles

-
    name: 'Update nameserver entry into resolv.conf file on localhost'
    hosts: localhost
    nameserver_ip: 10.24.12.01
    tasks:
        -
            name: 'Update nameserver entry into resolv.conf file'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver {{nameserver_ip}}'


################### Contitionals
Register: --> module is used to register the output of above command into a variable 

  name: 'Execute a script on all web server nodes'
    hosts: all_servers
    tasks:
        -
            service: 'name=mysql state=started'
            when: 
                ansible_host=="server4.company.com"
-
    name: 'Am I an Adult or a Child?'
    hosts: localhost
    vars:
        age: 25
    tasks:
        -
            command: 'echo "I am a Child"'
            when: age<18
        -
            command: 'echo "I am an Adult"'
            when: age>=18

-
    name: 'Add name server entry if not already entered'
    hosts: localhost
    tasks:
        -
            shell: 'cat /etc/resolv.conf'
            register: command_output
        -
            shell: 'echo "nameserver 10.0.250.10" >> /etc/resolv.conf'
            when: command_output.stdout.find("10.0.250.10") == -1

##########################################

############## ansible loops

-
    name: 'Install required packages'
    hosts: localhost
    vars:
        packages:
            - httpd
            - binutils
            - glibc
            - ksh
            - libaio
            - libXext
            - gcc
            - make
            - sysstat
            - unixODBC
            - mongodb
            - nodejs
            - grunt
    tasks:
        -
            yum: 'name={{items}} state=present'
            with_items: "{{ packages }}"
###############################################

Ansible Role --> if we assign a role to server, it will install all the necessary software in that server... to perform that role...> for example if we assign a role as DBServer to a host, ansible will install and and start all the required packages  to perform the DB role.

Ansible galexy is the palce where we can find 1000's of roles created by communities 



ansible-galaxy init <role_name> --> will create all the required directory structure to create ansible

eg: ansible-galaxy init mysql 

ansible-galaxy list --> gives list of roles

common locations of roles is /etc/ansible/roles

roles_path is defined on /etc/ansible/ansible.cfg
########### Sample definition of role

-
	name: "sample for role"
	hosts: myhosts_list
	roles:
		- role : myrole.yml
		- role : nginx
##################################################

######################## Advanced topics of Ansible

Ansible control machine can only be linux, but windows can be targets

Patterns are like regex, used to indentify the hosts name in inventory

dynamic invetory --> we can give inventory on the run

ansible-palybook -i inventory.txt playbook.yml --> here this txt file will contain list of required hosts 
ansible-palybook -i inventory.py playbook.yml --> here this python script will reach out to all the sources of inventory and provide us the list 


modules like, command , script, --etc are nothing but python scripts, if we need another functionality we can write custom modules in python





