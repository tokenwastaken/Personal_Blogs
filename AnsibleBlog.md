
# ANSIBLE 

 Ansible is an open source automation tool. Ansible Tower is a visualized interface for Ansible.
Ansible is agentless, meaning no additional software is needed unlike Zabbix which has you installing client software to be able to communicate and hosts can only be accessed through SSH, which makes it more secure than the other. This means that any computer that you can administer through SSH, you can also administer through Ansible. Ansible configurations are mainly written in YAML format since it’s more popular.


```sudo apt update```

```sudo apt install software-properties-common```

```sudo apt-add-repository ppa:ansible/ansible```

```sudo apt update```

```sudo apt install ansible```

Get the output of 

```cat ~/.ssh/id_rsa.pub```


Then copy the output using: ~ here is for our home directory for my case /home/isken

```ssh-copy-id -i ~/.ssh/id_rsa.pub ssh_user@192.168.56.101```

If you are using ssh keys to connect to your vm you should go manually and copy since you can’t connect the instance without the key.

For ssh connection with key.
Copy the file into

```nano ~/.ssh/authorized_keys```

Go into Ansible hosts file and present you IP addresses.

```sudo nano /etc/ansible/hosts```

inside the file:

```gentoo ansible_ssh_host=your_server_ip```

```amazonaws ansible_ssh_host=your_server_ip```


The syntax was ansible_host but after ansible 2.0 it is changed into ansible_ssh_host.

```sudo mkdir /etc/ansible/host_vars```

Go into:

```sudo nano etc/ansible/host_vars/amazonaws```

```
---
ansible_ssh_port: 22
ansible_ssh_user: ubuntu
ansible_ssh_private_key_file: /home/isken/Downloads/zabbixkey.pem
```

‘---’ is because the file is YAML.


```sudo nano etc/ansible/host_vars/gentoo```

```
---
ansible_ssh_user: root

```

then test with

```ansible -m ping all```

Example: Install and start nginx on client server

create a YAML file 

```sudo mkdir nginx.yaml```

Inside the YAML add:

```
---
 - hosts: amazonaws
   tasks:
     - name: ensure nginx is at the latest version
       apt: name=nginx
     - name: start nginx
       service:
           name: nginx
           state: started
   ```

To run the file

```ansible-playbook nginx.yaml -b```

‘-b’ option forces the client server to run as root.



Troubleshooting:

- If you get connection refused (publickey) error make sure you add the public key to the CORRECT	user that you will be connecting to.

- If you are connecting with a password make sure the destination host has one set up.

- sometimes changing remote_host in ansible.cfg to /tmp/.ansible-{USER}/tmp can solve some problems.

- ``` ssh-keygen -R ip_addr``` deletes the key.

- to overwrite file:

- name: overwrite file 
 ```
      shell: |
         echo 'success' > /root/ansible
 ```





