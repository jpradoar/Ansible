<p align="center">
  <img src="Ansible_Logo.png"/><br>
</p>

### BASIC INFRASTRUCTURE
<pre>
 Name            ID                      Type              IP
 Srv            i-1eseeacacf1f523a2 	t2.micro 	  172.31.27.65
 httpd_front    i-1eseeacacf1f523a2 	t2.micro 	  172.31.27.66
 httpd_front2   i-1eseeacacf1f523a2 	t2.micro 	  172.31.27.67
 
 - aclarations: In this doc, no account was taken of security.
</pre>


### Install ansible 
<pre>
  yum install ansible python-crypto2.6  python-httplib2 python-jinja2-26 python-keyczar python-paramiko python-pyasn1 python-setuptools python-simplejson sshpass 
</pre>



### Configure ansible hosts
<pre>
  Add the host to administer with ansible.  ( /etc/ansible/hosts )
     echo -e "[Srv] \n172.31.27.65\n" >> /etc/ansible/hosts
     echo -e "[httpd_front] \n172.31.27.66\n" >> /etc/ansible/hosts
     echo -e "[httpd_front2] \n172.31.27.67\n" >> /etc/ansible/hosts
</pre>



### Generate a ssh key (The one that later uses to manage the servers)
<pre>
   ssh-keygen
        Generating public/private rsa key pair.
        Enter file in which to save the key (/root/.ssh/id_rsa):  
        Enter passphrase (empty for no passphrase): 
        Enter same passphrase again: 
        Your identification has been saved in /root/.ssh/id_rsa.
        Your public key has been saved in /root/.ssh/id_rsa.pub.
        The key fingerprint is:
        bb:74:xx:xx:xx:bc:49:f2:1b:2c:xx:xx:xx:xx:xx:xx root@httpd_front
        The key's randomart image is:
        +--[ RSA 2048]----+
        |      o ...      |
        |     x . .       |
        |      x   x      |
        |       . x .     |
        | x      S +      |
        |       x X +     |
        |        x %      |
        |       x x x     |
        | x      x .      |
        +-----------------+
</pre>     



### Copy the public key to servers to administer with ansible
<pre>
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.31.27.65
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.31.27.66
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.31.27.67
</pre>



### Test the key
<pre>
   ssh 'root@172.31.27.65'
   ssh 'root@172.31.27.66'
   ssh 'root@172.31.27.67'
</pre>


### I execute a test ping on all servers using ansible with user root
<pre>
   ansible all -m ping -u root
        172.31.27.65 | SUCCESS => {
            "changed": false, 
            "ping": "pong"
        }
        172.31.27.66 | SUCCESS => {
            "changed": false, 
            "ping": "pong"
        }
        172.31.27.67 | SUCCESS => {
            "changed": false, 
            "ping": "pong"
        }
 </pre>       



### If the access fails it shows an error similar to this, which specifies the type of problem
<pre>
  172.31.27.67 | UNREACHABLE! => {
      "changed": false, 
      "msg": "Failed to connect to the host via ssh: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).\r\n", 
      "unreachable": true
  }
</pre>



### Executing remote commands 
<pre>
   ansible httpd_front -a 'cat /etc/resolv.conf' -u root
      172.31.27.66 | SUCCESS | rc=0 >>
      nameserver 8.8.8.8
      nameserver 8.8.4.4
</pre>



### Copy a file from local host to remote host
<pre>
 ansible ws02 -m copy -a "src=/etc/mysql/my.cnf dest=/tmp/my.cnf_test"
</pre>


### basic PlayBook (cat webserver.yml)
<pre>
     ---
     - hosts: all
       remote_user: root
       become: true
       
       vars:
         user: jprado
         password: jprado
         bash: /bin/bash

       tasks:
     
      - name: create user
        user: name={{user}} shell={{bash}} password={{password}}

     - name: Paquetes | Instalando LAMP.
       action: apt pkg={{ item }} state=installed
       with_items:
      - php5
      - apache2
      - mysql-server
      - mysql-client
      - php5-mysql
      - php-apc
      - php5-xmlrpc
      - php-soap
      - php5-gd
      - unzip
      - python-mysqldb

    - name: Apache2 | enable rewrite, a2enmod and vhost
      action: command rewrite a2enmod vhost_alias

    - name: Restart Apache
      action: service name=apache2 state=restarted    
      
     # ansible-playbook /ansible/ansible_generic/main.yml --limit front01
</pre>


### Output of the above
<pre>
    PLAY [webserver] ************************************************************** 

    GATHERING FACTS ***************************************************************
    ok: [172.31.27.66]

    TASK: [General | Instalación de paquetes requeridos.] ************************
    changed: [172.31.27.66] => (item=php5,apache2,mysql-server,mysql-client,php5-mysql,php-apc,php5-xmlrpc,php-soap,php5-gd,unzip,python-mysqldb)

    TASK: [Apache2 | Habilitar modulos] *******************************************
    changed: [172.31.27.66]

    TASK: [Restart Apache] ********************************************************
    changed: [172.31.27.66]

    PLAY RECAP ********************************************************************
    172.31.27.66            : ok=4    changed=3    unreachable=0    failed=0
</pre>



### Ansible Galaxy
...






### Ansible-vault

Ansible-vault can encrypt your files. It is very interesting for host our files securely on github or another server without compromising files or confidential information

<pre>
# Create a main.yml encrypted
 ansible-vault create new_main.yml

# edit the created file
 ansible-vault edit new_main.yml

# Encrypt or Decrypt file 
 ansible-vault encrypt/decrypt main.yml

# Run an encrypted playbook (without decrypting) only that asks for the encryption step before executing.
 ansible-playbook main.yml --limit front01 --ask-vault-pass


</pre>


### Ansible on aws
<pre>
To deploy our configs and systems, first need add "ansivle:vars" on /etc/ansible/hosts.  
In my case I added my aws key-pair, ssh port and remote user (remember enable security groups!)

[aws]
54.xyz.zyx.12

[aws:vars]
ansible_ssh_user=ubuntu
ansible_ssh_port=22
ansible_ssh_private_key_file=/home/jonathan/.ssh/KP_test.pem

# And later run the  playbook
ansible-playbook /home/jonathan/ansible/webserver/main.yml --limit=aws



# In some cases, you can receive an error as a this:
54.xyz.zyx.12 | FAILED! => {
    "changed": false, 
    "failed": true, 
    "module_stderr": "Shared connection to 34.224.23.16 closed.\r\n", 
    "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n", 
    "msg": "MODULE FAILURE", 
    "rc": 0
}

# If you receive this error, you need add the python interpreter
[aws]
54.xyz.zyx.12  ansible_python_interpreter=/usr/bin/python3

[aws:vars]
ansible_ssh_user=ubuntu
ansible_ssh_port=22
ansible_ssh_private_key_file=/home/jonathan/.ssh/KP_test.pem
ansible_sudo=True

</pre>



### Some playbooks and roles
[PHPIPAM](/phpipam/)

[PHP CRUD](/ansibie_full_crud/)



### Helps
<pre>
   https://www.ansible.com/
   http://docs.ansible.com/ansible/index.html
   https://galaxy.ansible.com/
</pre>
