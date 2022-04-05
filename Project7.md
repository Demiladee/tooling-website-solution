#Project 7

Launched 4 EC2 Instances within the same subnet using Red Hat Linux distribution, 1 for the NFS server & 3 for the Web Servers. 

Launched 1 Ubuntu Linux distro - within the same subnet also - to serve as the Database server.

Attached an 8gb, 8gb & 10gb EBS volumes to the NFS server to be formatted and made into 3 lv's.

updating red hat

$`sudo yum update -y`

![](images/nfs1.png)

listing blocks attached to the nfs server

$`lsblk`

![](images/nfslsblk2.png)

creating partitions on the disks:

$`sudo gdisk /dev/xvdf`

$`sudo gidsk /dev/xvdg`

$`sudo gdisk /dev/xvdh`

![](images/nfsxvdf3.png)

![](images/nfsxvdg4.png)

![](images/nfsxvdh5.png)

listing the attched blocks to confirm the configured partitions

$`lsblk`

![](images/nfslsblkconf6.png)

installing lvm2 package

$`sudo yum install lvm2 -y`

![](images/nfslvm7.png)

creating the pv's:

$`sudo pvcreate /dev/xvdf1`

$`sudo pvcreate /dev/xvdg1`

$`sudo pvcreate /dev/xvdh1`

pv creation verification

$`sudo pvs` 

vg's:

adding all pv's to a vg

$`sudo vgcreate webserver-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

vg creation verification

$`sudo vgs`

lv's:

creating 3 lv's - lv-apps, lv-logs & lv-opt

$`sudo lvcreate -n lv-apps -L 8G webserver-vg`

$`sudo lvcreate -n lv-logs -L 8G webserver-vg`

$`sudo lvcreate -n lv-opt -L 10G webserver-vg`

lv creation verification

$`sudo lvs`

verifying entire setup

$`sudo vgdisplay -v`

![](images/nfspvlv8.png)

![](images/nfsvgdisplay9.png)

formatting the disks as xfs 

$`sudo mkfs -t xfs /dev/webserver-vg/lv-apps`

$`sudo mkfs -t xfs /dev/webserver-vg/lv-logs`

$`sudo mkfs -t xfs /dev/webserver-vg/lv-opt`

![](images/nfsxfs10.png)

creating directories to mount the lv's to

$`sudo mkdir /mnt/apps`

$`sudo mkdir /mnt/logs`

$`sudo mkdir /mnt/opt`

mounting the lv's

$`sudo mount /dev/webserver-vg/lv-apps /mnt/apps`

$`sudo mount /dev/webserver-vg/lv-logs /mnt/logs`

$`sudo mount /dev/webserver-vg/lv-opt /mnt/opt`

updating the /etc/fstab file so the mount conf will remain after restarting the server

getting the uuid of the disks

$`sudo blkid`

$`sudo vi /etc/fstab`

![](images/nfsmntfstab11.png)

updating fstab

![](images/nfsfstab12.png)

testing configuration and reloading the daemon

$`sudo mount -a`

$`sudo systemctl daemon-reload`

verifying the setup

$`df -h`

![](images/nfsconfirm13.png)