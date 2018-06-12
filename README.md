# ansible-script--deploy-tomcat
Install Ansible Packages                                          # sudo apt-get install software-properties-common
Install all reposirories                                          # sudo apt-add-repository ppa:ansible/ansible
Install Ansible                                                   # sudo apt-get install ansible
create rsa-keys for ansible use                                   # ssh-keygen -t rsa -b 4096 -C "user@address"
add rsa key to ssh                                                # ssh-agent bash && ssh-add ~/.ssh/id_rsa
copy ssh-key to all servers to which ansible connects             # ssh-copy-id user@192.168.50.2     *user= server user example: john
configure ansible hosts file                                      # vi /etc/ansible/hosts
add your servers IP address or hostname under group name like     # [webserver]
                                                                     EX: [webservers]
                                                                          192.168.50.2
                                                                  
Ping all Servers                                                  # sudo ansible- m ping all
If you get an error "host unreachable"
configure hosts file                                              # vi /etc/ansible/hosts
                                                                      add  [web-servers:vars]
                                                                      ansible_password=**********  (your server root passowrd)
Now ping servers                                                   # sudo ansible- m ping all

INSTALL TOMCAT on server 192.168.50.2
create a directory name cevos                                       # sudo mkdir cevos
change to cevos directory                                           # cd cevos
Create another directory named  'roles'                             # sudo mkdir cevos/roles
cd direcoty to roles                                                # cd roles
Under roles create directory apachetomcat and text-file tomcat.yml  # sudo mkdir apachetomcat and # sudo touch tomcat.yml
change direcoty apachetomcat                                        # cd apachetomcat
create two (tasks and file) driectories under apachetomcat          # mkdir apachetomcat/tasks
                                                                    # mkdir apachetomcat/file
Under file create four text or xml files 1.tomcat.service           # touch file/tomcat.service
                                           2.context.xml            # touch file/context.xml
                                           3.tomcat-user.xml        # touch file/tomcat-user.xml
                                           4.index.html             # touch file/index.html
                                          
Under task create main.yml file                                      # touch tasks/main.yml

Then your tree structure will be
── cevos
│   └── roles
│       ├── apachetomcat
│       │   ├── file
│       │   │   ├── context.xml
│       │   │   ├── index.html
│       │   │   ├── tomcat.service
│       │   │   └── tomcat-users.xml
│       │   └── tasks
│       │       └── main.yml
│       └── tomcat.yml


Open tomcat.service file                                              #sudo vi /cevos/roles/apachetomcat/file/tomcat.service
Paste the following code
***********************************************************************************************************************
# Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/apache-tomcat-8.5.31/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat/apache-tomcat-8.5.31
Environment=CATALINA_BASE=/opt/tomcat/apache-tomcat-8.5.31
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/apache-tomcat-8.5.31/bin/startup.sh
ExecStop=/opt/tomcat/apache-tomcat-8.5.31/bin/shutdown.sh -15 $MAINPID

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
*****************************************************************************************************************************


open tomcat-users.xml file                                              #s udo vi /cevos/roles/apachetomcat/file/tomcat-users.xml
 Paste the following script

**************************************************************************************************************************


<tomcat-users>
    <user username="username" password="password" roles="manager-gui,admin-gui"/>
</tomcat-users>


************************************************************************************************************************************

Open context.xml file                                                   # sudo vi /cevos/roles/apachetomcat/file/context.xml

Paste the fillowing script

************************************************************************************************************************

<Context antiResourceLocking="false" privileged="true" >
  <!--<Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />-->
</Context>


**************************************************************************************************************************


Open index.html file                                                  # sudo vi /cevos/roles/apachetomcat/file/index.html
Paste the following code

****************************************************************************************************************

<html>
<head> welcome </head>
<p> HELLO WORLD </p>
</html>

********************************************************************************************************

Open file mail.yml                                                    # vi /cevos/roles/apachetomcat/tasks/main.yml
Paste the following script
*************************************************************************************************************************
 ---
- name: Install Java
  shell: yum install -y java

- name: create tomcat group
  group:
    name: tomcat
    state: present

- name: create a user tomcat and add to group tomcat
  user:
    name: tomcat
    shell: /bin/bash
    groups: tomcat
    append: yes

- name: download tomcat files
  get_url:
    url: http://mirror.cc.columbia.edu/pub/software/apache/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.tar.gz
    dest: /opt
    mode: 0755

- name: create a directory /opt/tomcat
  file:
    path: /opt/tomcat
    state: directory
    mode: 0755
   
- name: Extract tomcat tar file
  unarchive:
    src: /opt/apache-tomcat-8.5.31.tar.gz
    dest: /opt/tomcat
    remote_src: yes
   
- name: change user to tomcat for webapps,works, logs,temp
  file:
    path: /opt/tomcat/apache-tomcat-8.5.31/
    owner: tomcat
    group: tomcat
    mode: 0777
    recurse: yes

- name: Copy systemd unit file
  copy:
    src: /home/ashishreddy/cevos/roles/apachetomcat/file/tomcat.service
    dest: /etc/systemd/system/tomcat.service
    owner: tomcat
    group: tomcat
    mode: 0755

- name: tomcat reload
  shell: systemctl daemon-reload

- name: start tomcat service
  service:
    name: tomcat
    state: started

- name: Enable tomcat
  systemd:
    name: tomcat
    enabled: yes

- name: open port 8080
  firewalld:
   port: 8080/tcp
   state: enabled
   permanent: true
      
- name: index file
  copy:
    src: /home/ashishreddy/cevos/roles/apachetomcat/file/index.html
    dest: /opt/tomcat/apache-tomcat-8.5.31/webapps/examples/index.html

- name: open port 22
  firewalld:
    port: 22/tcp
    permanent: true
    state: disabled

- name: edit tomcat manager
  copy:
    src: /home/ashishreddy/cevos/roles/apachetomcat/file/tomcat-users.xml
    dest: /opt/tomcat/apache-tomcat-8.5.31/conf/tomcat-users.xml
    owner: tomcat
    group: tomcat
    mode: 0775

- name: edit host-manager
  copy:
    src: /home/ashishreddy/cevos/roles/apachetomcat/file/context.xml
    dest: /opt/tomcat/apache-tomcat-8.5.31/webapps/host-manager/META-INF/context.xml
    owner: tomcat
    group: tomcat
    mode: 0775

- name: restart tomcat
  service:
    name: tomcat
    state: restarted

*******************************************************************************************************************************


Open tomcat.yml file                                                    # sudo vi /cevos/roles/tomcat.yml
and paste the following script

*************************************************************************************************************************

---
- name: Install Tomcat
  hosts: webservers
  remote_user: stackadm
  become: yes
  become_method: sudo
  roles:
    - apachetomcat
    
***************************************************************************************************************************

change to directory  roles                                               # cd /cevos/roles
execute playbook                                                         # sudo ansible-playbook tomcat.yml

NOW Your apache tomcat is installed

                                              




     
