# Project 7

Launched 4 EC2 Instances within the same subnet using Red Hat Linux distribution, 1 for the NFS server & 3 for the Web Servers. 

Launched 1 Ubuntu Linux distro - within the same subnet also - to serve as the Database server.

Attached an 8gb, 8gb & 10gb EBS volumes to the NFS server to be formatted and made into 3 lv's.


## preparing nfs


updating red hat

`$sudo yum update -y`

![](images/nfs1.png)

listing blocks attached to the nfs server

`$lsblk`

![](images/nfslsblk2.png)

creating partitions on the disks:

`$sudo gdisk /dev/xvdf`

`$sudo gidsk /dev/xvdg`

`$sudo gdisk /dev/xvdh`

![](images/nfsxvdf3.png)

![](images/nfsxvdg4.png)

![](images/nfsxvdh5.png)

listing the attched blocks to confirm the configured partitions

`$lsblk`

![](images/nfslsblkconf6.png)

installing lvm2 package

`$sudo yum install lvm2 -y`

![](images/nfslvm7.png)

creating the pv's:

`$sudo pvcreate /dev/xvdf1`

`$sudo pvcreate /dev/xvdg1`

`$sudo pvcreate /dev/xvdh1`

pv creation verification

`$sudo pvs` 

vg's:

adding all pv's to a vg

`$sudo vgcreate webserver-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

vg creation verification

`$sudo vgs`

lv's:

creating 3 lv's - lv-apps, lv-logs & lv-opt

`$sudo lvcreate -n lv-apps -L 8G webserver-vg`

`$sudo lvcreate -n lv-logs -L 8G webserver-vg`

`$sudo lvcreate -n lv-opt -L 10G webserver-vg`

lv creation verification

`$sudo lvs`

verifying entire setup

`$sudo vgdisplay -v`

![](images/nfspvlv8.png)

![](images/nfsvgdisplay9.png)

formatting the disks as xfs 

`$sudo mkfs -t xfs /dev/webserver-vg/lv-apps`

`$sudo mkfs -t xfs /dev/webserver-vg/lv-logs`

`$sudo mkfs -t xfs /dev/webserver-vg/lv-opt`

![](images/nfsxfs10.png)

creating directories to mount the lv's to

`$sudo mkdir /mnt/apps`

`$sudo mkdir /mnt/logs`

`$sudo mkdir /mnt/opt`

mounting the lv's

`$sudo mount /dev/webserver-vg/lv-apps /mnt/apps`

`$sudo mount /dev/webserver-vg/lv-logs /mnt/logs`

`$sudo mount /dev/webserver-vg/lv-opt /mnt/opt`

updating the /etc/fstab file so the mount conf will remain after restarting the server

getting the uuid of the disks

`$sudo blkid`

`$sudo vi /etc/fstab`

![](images/nfsmntfstab11.png)

updating fstab

![](images/nfsfstab12.png)

testing configuration and reloading the daemon

`$sudo mount -a`

`$sudo systemctl daemon-reload`

verifying the setup

`$df -h`

![](images/nfsconfirm13.png)

installing nfs server 

`$sudo yum install nfs-utils -y`

![](images/nfsutils13.png)

configuring nfs

`$sudo systemctl start nfs-server.service`

`$sudo systemctl enable nfs-server.service`

`$sudo systemctl status nfs-server.service`

![](images/nfsservice14.png)

setting r,w,x permissions for web servers on nfs

`$sudo chown -R nobody: /mnt/apps`

`$sudo chown -R nobody: /mnt/logs`

`$sudo chown -R nobody: /mnt/opt`

`$sudo chmod -R 777 /mnt/apps`

`$sudo chmod -R 777 /mnt/logs`

`$sudo chmod -R 777 /mnt/opt`

`$sudo systemctl restart nfs-server.service`

![](images/nfschmodown15.png)

configuring access to nfs for clients within the same subnet cidr

`$sudo vi /etc/exports`

`$sudo exports -arv`

![](images/nfsaccess16.png)

checking the ports nfs is using and adding additional inbound rules

`$rpcinfo -p | grep nfs`

![](images/nfsexportrcp17.png)

opening ports so client can access nfs:

tcp 111, udp 111, tcp 2049, udp 2049

![](images/nfsinbound18.png)


## configuring database server

updating ubuntu

`$sudo apt update -y`

![](images/dbupdate1.png)

installing mysql-server

`$sudo apt install mysql-server -y`

![](images/dbsqlserver2.png)

verifying sql status

`$sudo systemctl status mysql`

![](images/dbsqlstatus3.png)

populating db server with a database, a user and granting permission to the user

`mysql create database tooling;`

`mysql create user 'webaccess'@'subnet-cidr' identified by 'password';`

`mysql grant all on tooling.* to 'webaccess'@'subnet-cidr';`

`mysql flush privileges;`

`mysql show databases;`

`mysql exit`

![](images/dbsqlsetup4.png)


## preparing web servers

### everything done on 1 web server will be replicated on the other 2. they will share the same nfs and database.

updating red hat

`$ sudo yum update`

![](images/wsupdate1.png)

installing nfs client

`$sudo yum install nfs-utils nfs4-acl-tools -y`

![](images/wsinstallnfsclient1.png)

creating /var/www directory
mounting the /mnt/apps on the directory

`$sudo mkdir /var/www`

`$sudo mount -t nfs -o rw,nosuid <nfs-server-private-ip-address>:/mnt/apps /var/www`

verifying the mount was successful

`$ df -h`

editing /etc/fstab to make sure the changes persist after reboot

`$sudo vi /etc/fstab`

![](images/wsmntappsdf2.png)

![](images/wsfstab3.png)

installing remi's repository, apache and php + its dependencies

`$sudo yum install httpd -y`

![](images/wshttpd4.png)

`$sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![](images/wsdnf5.png)

`$sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![](images/wsdnf6.png)

`$sudo dnf module reset php`

![](images/wsmodulereset7.png)

`$sudo dnf module enable php:remi-7.4`

![](images/wsmoduleremi8.png)

`$sudp dnf install php php-opcache php-gd php-curl php-mysqlnd`

![](images/wsphpdep9.png)

`$sudo systemctl start php-fpm`

`$sudo systemctl enable php-fpm`

`$sudo setsebool -P httpd_execmem 1`

![](images/wssystemctl10.png)

verifying that apache files and directories are available on the web server in /var/www and also on the nfs server in /mnt/apps

ran the ls command on both servers to be sure they have the same files

`$ ls /var/www`

`$ ls /mnt/apps`

added a .txt file to one of the web servers and confirmed that it's available on the nfs server too

`$cd /var/www`

`$sudo touch test.txt`

`$ls`

`$ls /mnt/apps`

![](images/wstest11.png)

![](images/wstest111.png)

repeating the steps above for /mnt/logs and /var/log/httpd

`$sudo mount -t nfs -o rw,nosuid <nfs-server-private-ip-address>:/mnt/logs /var/log/httpd`

verifying the mount was successful

`$ df -h`

editing /etc/fstab to make sure the changes persist after reboot

`$sudo vi /etc/fstab`

![](images/wsmnt12.png)

editing fstab

![](images/wslogfstab13.png)

verifying that the process works on the other web servers

![](images/wsmnnt12.png)

![](images/wsmntt12.png)

### cloning the tooling source code from darey.io github account

installing git

`$sudo yum install git -y`

![](images/wsgit14.png)

`$git init`

`$git clone`

![](images/wsgitclonecopy15.png)

deploying the tooling website's code to the webserver.

`$cd tooling`

confirming the html directory is there and copying it into /var/www/html

`$sudo cp -R html/. /var/www/html`

![](images/wsgitclonecopy15.png)

opening tcp port 80 on the web server 

![](images/wshttpinb16.png)

confirming httpd is running

`$ sudo systemctl status httpd`

![](images/wshttpddead17.png)

httpd is inactive so i'll disable selinux and edit the config file

`$sudo sentenforce 0`

`$sudo vi /etc/sysconfig/selinux`

i set selinux=disabled and restarted httpd

![](images/wshhtpdrunning18.png)

![](images/wsselinux19.png)

edited the website's configuration file to connect to the db 

`$sudo vi /var/www/html/functions.php`

![](images/wsviphp20.png)

edited the db access info

`$db = mysqli_connect('subnet-cidr', 'username', 'password', 'db-name');`

![](images/wsfunctionsphp21.png)

installed mysql 

`$sudo yum install mysql -y`

![](images/wsinstallmysql22.png)

i edited the db server's inbound rule to allow access from the subnet-cidr

![](images/dbsubnetciderinboound5.png)

edited bind address to allow access from anywhere

`$sudo vi /etc/mysql/mysql.conf.d/mysql.cnf`

![](images/dbbindaddre7.png)

restarted mysql 

`$sudo systemctl restart mysql`

![](images/dbmysqlconnected6.png)

went back to the web server to confirm that the connection was allowed

`mysql -h <database-private-ip> -u <db-username> -p <db-password> < tooling-db.sql

![](images/wsconnectedmysql23.png)

tested the db server to confirm my db and user are there

`mysql show databases;`

`mysql use tooling;`

`mysql show tables;`

`mysql select * from users;`

![](images/dbtest8.png)

created another user in mysql users table and granted all privileges

![](images/dbfinal8.png)

tested website 

![](images/final1.png)

![](images/final2.png)

