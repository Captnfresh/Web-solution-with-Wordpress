# Web-solution-with-Wordpress

In this project, we will set up the infrastructure for a web solution using WordPress, a popular content management system. We will be using two separate Linux servers: one for the web server and another for the database server.

## Project Overview

### The project consists of two main parts:

1. Configure storage subsystem for Web and Database servers: Gain hands-on experience with disks, partitions, and volumes on Linux.
2. Install WordPress and connect it to a remote MySQL database server: Solidify your skills in deploying web and database tiers of a web solution.

## Three-tier Architecture:

Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

### Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

   ![image 1](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%201.jpg)

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

   ![image 2](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%202.jpg)

   ![image 3](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%203.jpg)

   ![image 4](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%204.jpg)

   ![image 5](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%205.jpg)

   ![image 6](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%206.jpg)

   ![image 7](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%207.jpg)

3. Attach the three volumes one by one to your EC2 web server instance. You can do this from the AWS Management Console by selecting the instance and attaching the newly created volumes.

4. Connect to your instance via ssh

   ```
   ssh -i <key-pair-name> ec2-user@<ip-address>
   ```

5. Check for the newly attached volumes. As seen above as /dev/xvdf, /dev/xvdg, and /dev/xvdh.

   ```
   ls /dev/
   ```

6. Check disk space:

   ```
   df -h
   ```

7. Partition the Disks: Use `gdisk` utility to create a single partition on each of the 3 disks.

   ```
   sudo gdisk /dev/xvdf
   ```

8. Create a new partition:
a) Type n to create a new partition. 
b) Press Enter to select the default partition number 
c) Press Enter to accept the default last sector. 
d) Choose the partition type by typing 8300 for a Linux filesystem (ext4). 
e) To Write changes to the disk, Once the partition is created, type w

9. Install lvm2 package using `sudo yum install lvm2`. Run sudo lvmdiskscan command to check for available partitions.

   ```
     sudo yum install lvm2 -y
   ```

10. Check available partitions with `lvmdiskscan`.

    ```
    sudo lvmdiskscan
    ```

11. Create Physical Volumes by using `pvcreate` to mark the partitions as physical volumes:

    ```
    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1
    ```

12. Verify that Physical volume has been created successfully by running `sudo pvs`.

    ```
    sudo pvs
    ```

13. Create a Volume Group (VG): Add all three PVs to a volume group (VG) named webdata-vg:

    ```
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    ```

14. Verify that your VG has been created successfully by running:

    ```
     sudo vgs
    ```

15. Create Logical Volumes (LVs): Create two logical volumes:

    apps-lv using half of the VG size.

    ```
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    ```

    logs-lv using the remaining space.

    ```
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    ```

16. Confirm logical volumes creation by running:

    ```
    sudo lvs
    ```

17. Verify the entire setup

    ```
    sudo vgdisplay -v 
    sudo lsblk
    ```

18. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

    ```
    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
    ```
    
19. Create directories for the web data and logs:

    ```
    sudo mkdir -p /var/www/html
    sudo mkdir -p /home/recovery/logs
    ```

20. Mount the logical volumes to the directories:

    ```
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/
    sudo rsync -av /var/log/ /home/recovery/logs/
    sudo mount /dev/webdata-vg/logs-lv /var/log
    sudo rsync -av /home/recovery/logs/ /var/log
    ```

21. Update /etc/fstab so that the mount configuration persists after a server reboot:

    ```
    sudo blkid
    sudo vi /etc/fstab
    ```
      ![image 8](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%208.jpg)
   
      ![image 9](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%209.jpg)
    

      ![image 10](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2010.jpg)


23. Verify that the setup is successful:

    ```
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
    ```



## Step 2 - Prepare the Database Server

Launch a second RedHat EC2 instance that will have a role - 'DB Server' Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
Repeat the whole process for the webserver but instead of apps-lv, use db-lv and mount it to /db directory instead of /var/www/html/.

## Step 3 - Install Wordpress on your webserver.

1. Update the package repository:

   ```
   sudo yum -y update
   ```

2. Install required packages:

   ```
    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
   ```

3. Enable and start the Apache web server:

   ```
   sudo systemctl enable httpd
   sudo systemctl start httpd
   ```

4. Install additional PHP dependencies:

   ```
   sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
   sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
   sudo yum module list php
   sudo yum module reset php
   sudo yum module enable php:remi-7.4
   sudo yum install php php-opcache php-gd php-curl php-mysqlnd
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   ```

5. Allow Apache to execute PHP code:

   ```
   sudo setsebool -P httpd_execmem 1
   sudo systemctl restart httpd
   ```

6. Download and install WordPress:

   ```
   mkdir wordpress
   cd wordpress
   sudo wget http://wordpress.org/latest.tar.gz
   sudo tar -xzvf latest.tar.gz
   sudo rm -rf latest.tar.gz
   ```

7. Copy the Configuration File: WordPress comes with a sample configuration file that you need to copy and rename. This file will be used to configure your database settings.

   ```
   sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
   ```

8. Move WordPress Files to the Web Root Directory: Finally, copy the entire WordPress directory into the web server’s root directory, which is typically located at /var/www/html/.

   ```
   sudo cp -R wordpress /var/www/html/
   ```

9. Change Ownership of the WordPress Directory

    ```
    sudo chown -R apache:apache /var/www/html/wordpress
    ```

10. Configure SELinux Context for WordPress Directory

    ```
    sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
    ```

11. Allow Apache to Connect to the Network

    ```
    sudo setsebool -P httpd_can_network_connect=1
    ```




## Step 4 - Install MySQL on the DB Ec2 Server.

1. Install MySQL Server:

   ```
   sudo yum update
   sudo yum install mysql-server
   ```

2. Enable and start the MySQL service:

   ```
   sudo systemctl enable mysqld
   sudo systemctl start mysqld
   ```

3. Check if MySQL is running:

   ```
   sudo systemctl status mysqld
   ```

   ![image 11](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2011.jpg)


   
## Step 5 - Configure the DB for wordpress

1. Login to MySQL:

   ```
   sudo mysql
   ```

2. Create the WordPress database and a user for the web server:

   ```
   CREATE DATABASE wordpress;
   CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY '<your-password>';
   GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
   FLUSH PRIVILEGES;
   SHOW DATABASES;
   exit;
   ```

   ![image 12](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2012.jpg)

3. Modify the inbound rules for the DB server security group to allow MySQL connections (port 3306) only from the Web Server's private IP. For extra security, restrict access using /32 CIDR notation.

   ![image 13](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2013.jpg)


   
## Step 6 - Configure WordPress to Connect to the Remote Database

1. Install the MySQL client on the Web Server:

   ```
   sudo yum install mysql
   ```

2. Test if the Web Server can connect to the remote DB server:

   ```
   sudo mysql -u myuser -p -h <DB-Server-Private-IP-Address>
   ```

3. Verify the connection:

    ```
    SHOW DATABASES;
    ```

   ![image 14](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2014.jpg)
    
4. Edit the wp-config.php file located in /var/www/html/wordpress to include the database credentials using:

   ```
   sudo vi /var/www/html/wordpress/wp-config.php
   ```

5. Input the following:

   ```
      define('DB_NAME', 'wordpress');
      define('DB_USER', 'myuser');
      define('DB_PASSWORD', 'mypass.1');
      define('DB_HOST', '<DB-Server-Private-IP-Address>');
   ```
      
6. Access WordPress Setup from a Browser:

   ```
   http://<Web-Server-Public-IP-Address>/wordpress/
   ```

   ![image 15](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2015.jpg)

7. Follow the on-screen instructions to complete the WordPress setup:
  
   ![image 16](https://github.com/Captnfresh/Web-solution-with-Wordpress/blob/main/3%20tier%20web%20app/image%2016.jpg)

   

# CONCLUSION

This project demonstrates the successful implementation of a scalable and robust web solution using WordPress, the world's most popular content management system. By leveraging Amazon Web Services (AWS) EC2 instances, we've created a flexible and powerful hosting environment for WordPress.

## Key Achievements

1. **Successful Setup of Three-Tier Architecture**:
   - Configured a web server, application server, and database server for a robust application environment.

2. **Efficient Storage Management**:
   - Implemented Logical Volume Management (LVM) for flexible and scalable storage, allowing for easy resizing and management of disk space.

3. **Deployment of WordPress**:
   - Installed and configured WordPress as a content management system, showcasing the ability to create and manage web applications.

4. **Partitioning and Volume Group Creation**:
   - Successfully partitioned block instances and created volume groups to optimize storage utilization.

5. **Error Handling and Troubleshooting**:
   - Developed problem-solving skills by overcoming various challenges during the setup process, such as configuration issues and permissions errors.

6. **Documentation and Version Control**:
   - Documented the entire process on GitHub, enhancing skills in version control and providing a reference for future projects.


## Likely Errors and Solutions

1. **Error: “Error establishing a database connection”**
   - **Cause**: This can occur due to incorrect database credentials, database service not running, or network issues.
   - **Solution**: 
     - Verify database credentials in the `wp-config.php` file.
     - Check if the MySQL/MariaDB service is running using `sudo systemctl status mysql` or `sudo systemctl status mariadb`.
     - Ensure that the security group settings in AWS allow traffic on port 3306.

2. **Error: “Failed to start mysql.service: Unit mysql.service not found”**
   - **Cause**: MySQL may not be installed or the service may not be configured correctly.
   - **Solution**: 
     - Install MySQL using the appropriate package manager (`yum install mysql-server` for RedHat).
     - Ensure the service is enabled and started with `sudo systemctl enable mysql` and `sudo systemctl start mysql`.

3. **Error: “Access denied for user 'root'@'localhost'”**
   - **Cause**: Incorrect password or user permissions.
   - **Solution**: 
     - Reset the root password using the command line by starting MySQL in safe mode.
     - Grant appropriate permissions to the user by executing the necessary GRANT commands after logging in.

4. **Error: “Physical volume is already in volume group”**
   - **Cause**: Attempting to add an existing physical volume to a volume group.
   - **Solution**: 
     - Use the `vgreduce` command to remove the volume from the group if necessary or check the existing configuration with `vgs` and `pvs` to ensure the correct volumes are being used.

5. **Error: “Command not found” when using `apt` or `yum`**
   - **Cause**: The package manager may not be installed or recognized due to system misconfiguration.
   - **Solution**: 
     - Ensure you are using the correct package manager for your distribution (e.g., `yum` for RedHat).
     - Check the system’s PATH variable and configuration to ensure proper setup.

6. **Error: “Insufficient storage space” when creating logical volumes**
   - **Cause**: Not enough free space in the volume group to allocate to a new logical volume.
   - **Solution**: 
     - Check available space using `vgs` or `lvs`.
     - Consider resizing existing volumes or adding more physical volumes to the volume group.


### By documenting these achievements and challenges, I am not tracking just my progress but also creating a valuable resource for others who may encounter similar issues. 
   




























