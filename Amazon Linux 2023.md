# Instalations on AWS Linux 2023
- Open 8080 on TCP and 22 on SSH into your AWS Security group. **(Scope to your IP to avoid attacks)**
- Create an ssh key and set it to chmod 400 on your computer to avoid read it from outside.
- Get the instance public IP and access to it via ssh *(ssh -i /path/to/key user@ip)*
- Enter su mode *(sudo su -)*
- Change the host name into **/etc/hostname** *(Reboot after modify this)*
- Its important to know, AWS Linux 2023 use dnf instructions to install, so:

## Jenkins sever
- Create a fresh EC2 instance (1GB RAM 10GB HDD)

- Install Java 11:
    ``` 
    dnf install java-11-amazon-corretto -y 
    ```
- Add Jenkins repo:
    ```
    sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
    ```
- Import keys from jenkins:
    ```
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    ```
- Now, we can install Jenkins:
    ```
    dnf install jenkins -y
    ```
- After that, we can check status of service, enable it on startup and access via ***ip:8080***


#### Now we can prepare our Jenkins server to interact and work with other technologies, such as Docker, Ansible, Maven, Git, and many more.

### Git
- Install git on server:
    ```
    dnf install git
    ```

- Go to Manage Jenkins on GUI and install GitHub plugin.
- After this we are gonna Setup git basic information on Jenkins main setup

### Maven
- First of all, we go ahead Maven website and copy download URL.
- Go to /opt folder and then download Maven:
    ```
    wget maven/url
    ```
- Extract it (then you can rename it *mv maven-name maven-new-name*)
    ```
    tar -xvzf maven.tar.gz
    ```
- Now we have to update the .bash_profile to add Java and Maven bins to PATH
    - JAVA_HOME: /usr/lib/jvm/javabin
    - M2_HOME: /opt/maven
    - M2: /opt/maven/bin
- Then, we declare those variables onto PATH and reload the file with
    ```
    source .bash_profile
    ```
- Go to Jenkins GUI and install Maven plugin.
- Now we can setup Java and Maven paths onto our Jenkins GUI global configuration.

### Docker
- Install plugin **Publish Over SSH**
- On global configuration, Add a new SSH server pointing to our Docker server


## Tomcat server
- Get a fresh new EC2 instance (1GB RAM 10GB HDD)
- Install Java:
    ``` 
    dnf install java-11-amazon-corretto -y 
    ```
- Move to /opt and download Tomcat from URL as we did with Maven.
    ```
    wget tomcat/url
    ```
- Extract it (then you can rename it *mv tomcat-name tomcat-new-name*)
    ```
    tar -xvzf tomcat.tar.gz
    ```
- Then we can start Tomcat server launching startup script located on bin folder
- After this, we have to declare which hosts can access to the environment from http. Let's update **context.xml** file.
    ```
    nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
    nano /opt/tomcat/webapps/manager/META-INF/context.xml

    (Comment <!--<Valve className="org.apache...RemoteAddrValve" allow="127\.\d+..." /> -->)
    ```
- Now, we will update which users can do certain actions. This feature can be set on config/tomcat-users.xml:
    ```
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="manager-jmx"/>
    <role rolename="manager-status"/>
    <user username="admin" password="password" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
    <user username="deployer" password="password" roles="manager-script"/>
    <user username="tomcat" password="password" roles="manager-gui"/>
    ```
- Restart server and it's ready!

*(Just like an extra tip, we can set a simbolic link to the scripts right onto /usr/local/bin folder, in order to launch those from wherever)*
```
ln -s /opt/tomcat/bin/startup.sh /usr/local/bin/tomcatup
ln -s /opt/tomcat/bin/shutdown.sh /usr/local/bin/tomcatdown
```

## Docker
- Create a fresh EC2 instance (1GB RAM 16GB HDD)
- Install Docker
    ```
    dnf install docker -y
    ```
- For more information about Docker, read Docs!!

### Integration with Jenkins server
- Create a dockeradmin user and integrate in docker group
    ```
    useradd dockeradmin
    passwd dockeradmin password
    usermod -aG docker dockeradmin
    ```
- Modify users authentication on **/etc/ssh/sshd_config**
    - Uncomment "PasswordAuthentication yes"
    - Comment "PasswordAuthentication no"

- Restart the sshd service:
    ```
    systemctl restart sshd
    ```


### Integration with Ansible
- Create an ansadmin user and add to sudoers file
    ```
    useradd ansadmin
    passwd ansadmin password
    visudo 
    ```
    *(On this file, shift + g to scroll down. Add ansadmin and set "ALL=(ALL) NO PASSWD: ALL")*

## Ansible server
- Create a fresh EC2 instance (1GB RAM 10GB HDD)
- First of all, create an ansadmin user and add to sudoers file
    ```
    useradd ansadmin
    passwd ansadmin password
    visudo 
    ```
    *(On this file, shift + g to scroll down. Add ansadmin and set "ALL=(ALL) NO PASSWD: ALL")*
- Modify users authentication on **/etc/ssh/sshd_config**
    - Uncomment "PasswordAuthentication yes"
    - Comment "PasswordAuthentication no"

- Restart the sshd service:
    ```
    systemctl restart sshd
    ```
- Change to sudo from ansadmin
    ```
    sudo su - ansadmin
    ```
- From there, create a ssh key and copy it into our dockerhost.
    ```
    ssh-keygen -t rsa -b 4096
    ```
- Check if Python is installed, in case not:
    ```
    dnf install python
    ```
- Install pip:
    ```
    dnf install python3-pip
    ```
- Install Ansible
    ```
    pip install ansible
    ```
- Manage hosts whocan access to Ansible server. Add our Dockerhost address. File can be found on **/etc/ansible/hosts**

- Copy the ssh public key to our dockerhost:
    ```
    ssh-copy-id dockerhost-ip
    ```
    *Now we can check if the key has been succesfully copied onto our Dockerhost by checking /home/ansadmin/.ssh/authorized_keys*

- Check the connection beetween ansible and docker hosts:
    ```
    ansible all -m ping
    ansible all -m command -a uptime (Launch a command to our dockerhost)
    ```
