# phpipam

Ansible playbook para instalar: 

    - phpipam: https://phpipam.net/


# Data
    - http://99.99.99.99/phpipam/
    - login user: admin
    - login pass: Pass_word:1

<pre>
Running ansible-playbook...

ansible-playbook Ansible/phpipam/main.yml --limit=aws

[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and 
make sure become_method is 'sudo' (default). This feature will be removed in a 
future release. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.

PLAY ***************************************************************************

TASK [setup] *******************************************************************
ok: [default]

TASK [paquetes_basicos : Run apt-get update] ***********************************
ok: [default]

TASK [paquetes_basicos : Install list of packages] *****************************
changed: [default] => (item=[u'vim', u'apache2', u'git'])

TASK [paquetes_basicos : hostname] *********************************************
changed: [default]

TASK [apache : Install apache2] ************************************************
ok: [default]

TASK [php : install PHP] *******************************************************
changed: [default] => (item=[u'php5-cli', u'php5-curl', u'php5-fpm', u'php5-intl', u'php5-json', u'php5-mcrypt', u'php-pear', u'libapache2-mod-php5', u'php5-mysql', u'php5-common', u'php5-gmp', u'php-pear', u'php5-ldap'])

TASK [mysql : Install mysql server and client] *********************************
changed: [default] => (item=[u'mysql-server', u'mysql-client', u'python-mysqldb'])

TASK [mysql : Creating database phpipam] ***************************************
changed: [default]

TASK [mysql : mysql_user] ******************************************************
changed: [default]

TASK [mysql : locale_gen] ******************************************************
changed: [default]

TASK [mysql : copy] ************************************************************
changed: [default]

TASK [software : git] **********************************************************
changed: [default]

TASK [software : Import database from a dump] **********************************
changed: [default]

TASK [software : a2enmod rewrite & service apache2 restart] ********************
changed: [default]

TASK [crontab : cron] **********************************************************
changed: [default]

PLAY RECAP *********************************************************************
default                    : ok=15   changed=12   unreachable=0    failed=0


</pre>
<br>
<p align="center">
<br>
  <img src="ip-database.png"/><br>
</p>
