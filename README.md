# Web-solution-with-Wordpress

In this project, we will set up the infrastructure for a web solution using WordPress, a popular content management system. We will be using two separate Linux servers: one for the web server and another for the database server.

## Project Overview

### The project consists of two main parts:

1. Configure storage subsystem for Web and Database servers: Gain hands-on experience with disks, partitions, and volumes on Linux.
2. Install WordPress and connect it to a remote MySQL database server: Solidify your skills in deploying web and database tiers of a web solution.

## Three-tier Architecture:

Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

### Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

image 1

1. Presentation Layer (PL): This is the user interface such as the client server or browser on laptop.
2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver.
3. Data Access or Management Layer (DAL): This is the layer for data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.

In this project, we will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.

### Our 3-Tier Setup Need:

1. A Laptop or PC to serve as a client.
2. An EC2 Linux Server as a web server (This is where you will install WordPress).
3. An EC2 Linux server as a database (DB) server.

We will Use RedHat OS for this project.
By now we should know how to spin up an EC2 instanse on AWS, In previous projects we used 'Ubuntu', but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called 'RedHat' (it also has a fully compatible derivative - CentOS)

#### Note: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. SSH connection will look like ec2-user@.



## Step 1 - Prepare a Web Server

1. Launch an EC2 instance that will serve as "Web Server" and Create 3 volumes in the same AZ as Web Server EC2, each of 10 GiB.

   image 2

   image 3

   image 4

   image 5

   image 6

   image 7

2. Attach the three volumes one by one to your EC2 web server instance. You can do this from the AWS Management Console by selecting the instance and attaching the newly created volumes.

3. Connect to your instance via ssh

   ```
   ssh -i <key-pair-name> ec2-user@<ip-address>
   ```

4. Check for the newly attached volumes. As seen above as /dev/xvdf, /dev/xvdg, and /dev/xvdh.

   ```
   ls /dev/
   ```

5. Check disk space:

   ```
   df -h
   ```

6. Partition the Disks: Use `gdisk` utility to create a single partition on each of the 3 disks.

   ```
   sudo gdisk /dev/xvdf
   ```

7. Create a new partition:
a) Type n to create a new partition. 
b) Press Enter to select the default partition number 
c) Press Enter to accept the default last sector. 
d) Choose the partition type by typing 8300 for a Linux filesystem (ext4). 
e) To Write changes to the disk, Once the partition is created, type w

8. Install lvm2 package using `sudo yum install lvm2`. Run sudo lvmdiskscan command to check for available partitions.

   ```
     sudo yum install lvm2 -y
   ```

9. Check available partitions with `lvmdiskscan`.

    ```
    sudo lvmdiskscan
    ```

10. Create Physical Volumes by using `pvcreate` to mark the partitions as physical volumes:

    ```
    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1
    ```

11. Verify that Physical volume has been created successfully by running `sudo pvs`.

    ```
    sudo pvs
    ```

12. Create a Volume Group (VG): Add all three PVs to a volume group (VG) named webdata-vg:

    ```
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    ```

13. Verify that your VG has been created successfully by running:

    ```
     sudo vgs
    ```

14. Create Logical Volumes (LVs): Create two logical volumes:

    apps-lv using half of the VG size.

    ```
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    ```

    logs-lv using the remaining space.

    ```
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    ```

15. Confirm logical volumes creation by running:

    ```
    sudo lvs
    ```

16. Verify the entire setup

    ```
    sudo vgdisplay -v 
    sudo lsblk
    ```

17. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

    ```
    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
    ```
    
18. Create directories for the web data and logs:

    ```
    sudo mkdir -p /var/www/html
    sudo mkdir -p /home/recovery/logs
    ```

19. Mount the logical volumes to the directories:

    ```
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/
    sudo rsync -av /var/log/ /home/recovery/logs/
    sudo mount /dev/webdata-vg/logs-lv /var/log
    sudo rsync -av /home/recovery/logs/ /var/log
    ```

20. Update /etc/fstab so that the mount configuration persists after a server reboot:

    ```
    sudo blkid
    sudo vi /etc/fstab
    ```

21. Verify that the setup is successful:

    ```
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
    ```

## Step 2 - Prepare the Database Server

Launch a second RedHat EC2 instance that will have a role - 'DB Server' Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
Repeat the whole process for the webserver but instead of apps-lv, use db-lv and mount it to /db directory instead of /var/www/html/.

## Step 3 - Install Wordpress on your webserver.




























