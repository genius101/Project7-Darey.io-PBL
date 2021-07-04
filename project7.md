<h1>In this project, we would be implementing a Devops Tooling Website Solution</h1>

<p>This project would consist a 3-tier Web Application Architecture with a single Database and a NFS Server as a shared file storage</p>

<p>We would implement this solution with the following components:</p>

- [ ] Infrastructure: AWS

- [ ] Webserver Linux: Red Hat Enterprise Linux 8

- [ ] Database Server: Ubuntu 20.04 + MySQL

- [ ] Storage Server: Red Hat Enterprise Linux 8 + NFS Server

- [ ] Programming Language: PHP

- [ ] Code Repository: GitHub


<h2>Step 1 - Prepare NFS Server</h2>

<p>Spin up 4 EC2 instance in RedHat for the three Webservers and NFS Server</p>

<p>Spin up one EC2 Instance for the Database Server with Ubuntu on it</p>

<p>Create a logical volume and attach to the NFS server</p>

<p>Create a partition for the disk created, and create a physical volume; you might need to install the logical volume manager because it is not installed by default</p>

<p>Create 3 logical Volume on the disk:</p>

- [x] sudo lvcreate -n lv-opt -L 3G vg-webdata

- [x] sudo lvcreate -n lv-logs -L 3G vg-webdata

- [x] sudo lvcreate -n lv-apps -L 3G vg-webdata

<p>Format the three Volume groups as xfs file format:</p>

- [x] sudo mkfs.xfs /dev/vg-webdata/lv-opt
	
- [x] sudo mkfs.xfs /dev/vg-webdata/lv-apps
	
- [x] sudo mkfs.xfs /dev/vg-webdata/lv-logs

<p>Create directory for apps, logs and opt and mount the logical volume on each directory:</p>

- [x] sudo mkdir /mnt/apps

- [x]  sudo mkdir /mnt/logs

- [x]   sudo mkdir /mnt/opt

- [x] sudo mount /dev/vg-webdata/lv-apps /mnt/apps
	
- [x] sudo mount /dev/vg-webdata/lv-opt /mnt/opt

- [x] sudo mount /dev/vg-webdata/lv-logs /mnt/logs

<p>Confirm if mount is successful:</p>

- [x] df -h

<p>Copy UUID for mount for NFS Servers into /etc/fstab:</p>

	UUID=2515b95d-6ea5-4ea5-8ade-ee26244b5b25 /mnt/opt                xfs     defaults,nofail 0 0
	UUID=c80ebe3f-986b-4864-8100-a69d68a2ccd6 /mnt/apps               xfs     defaults,nofail 0 0
	UUID=26c93fcd-1558-4d7a-9e94-d897e344b388 /mnt/logs               xfs     defaults,nofail 0 0
  
<p>Run command to confirm if previous action was successful:(no error means succesful)</p>

- [x] sudo mount -a

<p>Then run the command:</p>

- [x] systemctl daemon-reload

<p>Install NFS server, configure it to start on reboot and make sure it is up and running:</p>

- [x] sudo yum -y update
	
- [x] sudo yum install nfs-utils -y
	
- [x] sudo systemctl start nfs-server.service
	
- [x] sudo systemctl enable nfs-server.service
	
- [x] sudo systemctl status nfs-server.service

<p>Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:</p>

- [x] sudo chown -R nobody: /mnt/apps
	
- [x] sudo chown -R nobody: /mnt/logs
	
- [x] sudo chown -R nobody: /mnt/opt

- [x] sudo chmod -R 777 /mnt/apps
	
- [x] sudo chmod -R 777 /mnt/logs
	
- [x] sudo chmod -R 777 /mnt/opt

- [x] sudo systemctl restart nfs-server.service

<p>Configure access to NFS for clients within the same subnet (My Subnet CIDR: 172.31.0.0/20):
  
- [x] sudo nano /etc/exports

<p>Copy below in the exports file</p>

	/mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
	/mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
	/mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)

<p>Save and exit, then run the following command</p>

- [x] sudo exportfs -arv

<p>After this configuration it is important to restart the NFS Service:</p>

- [x] sudo systemctl restart nfs-server.service

![1 n](https://user-images.githubusercontent.com/10243139/124374738-ca778380-dc95-11eb-9b00-5973887e61b3.jpg)

<p>Check which port is used by NFS and open it using Security Groups and add to new Inbound Rule:</p>
	
- [x] rpcinfo -p | grep nfs
	
<p>(Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049)</p>



<h2>Step 2 — Configure the database server</h2>


<p>Install MySQL server:</p>
	
- [x] sudo apt install mysql-server -y

- [x] sudo mysql_secure_installation

<p>mysql_secure_installation is used to provide more security for MYSQL, its adviceable to always run it after MYSQL installation</p>

<p>Log into MYSQl and Create a database and name it tooling:</p>
  
- [x] sudo mysql
  
      CREATE DATABASE `tooling`;
    
<p>Create a database user and name it webaccess:</p>

	CREATE USER 'webaccess'@'172.31.0.0/20' IDENTIFIED WITH mysql_native_password BY 'password';
  
<p>Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr:</p>

	GRANT ALL ON tooling.* TO 'webaccess'@'172.31.0.0/20' WITH GRANT OPTION;
  
<p>Reload the grant table by running:</p>

	FLUSH PRIVILEGES;

<p>Update the bind address on mysql conf file from 127.0.0.1 to 0.0.0.0 so it can connect from anywhere:</p>

- [x] sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

<p>Restart mysql service after the configuration:</p>

- [x] sudo systemctl restart mysql

![2 g](https://user-images.githubusercontent.com/10243139/124374971-458d6980-dc97-11eb-9235-56e7502ce50c.jpg)


<h2>Step 3 — Prepare the Web Servers</h2>


<p>Launch and Install NFS client on WebServer1:</p>

- [x] sudo yum install nfs-utils nfs4-acl-tools -y

<p>Start the NFS-Server service:</p>

- [x] sudo systemctl start nfs-server

<p>Create /var/www and Mount and target the NFS server’s export for apps:</p>

- [x] sudo mkdir /var/www
	
- [x] sudo mount -t nfs -o rw,nosuid 172.31.3.128:/mnt/apps /var/www

<p>Update fstab to make sure the mount point will persist after reboot:</p>

- [x] sudo nano /etc/fstab
	
<p>(with the following line)</p>

	172.31.3.128:/mnt/apps /var/www nfs defaults 0 0
  
<p>Install Apache and start service:</p>

- [x] sudo yum install httpd -y
	
- [x] sudo systemctl start httpd

Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps

- [x] sudo ls -l /mnt/apps

- [x] sudo ls -l /var/www

![3 f](https://user-images.githubusercontent.com/10243139/124375185-bc773200-dc98-11eb-9d30-c24dc06c1570.jpg)

<p>Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs:</p>
	
<p>Before we mount we need to backup the log folder:</p>

- [x] sudo mv /var/log/httpd /var/log/httpd.bak

<p>Create a fresh log folder:</p>

- [x] sudo mkdir /var/log/httpd

<p>Then Mount with the following command:</p>

- [x] sudo mount -t nfs -o rw,nosuid 172.31.3.128:/mnt/logs /var/log/httpd

<p>Update fstab to make sure the mount point will persist after reboot:</p>

- [x] sudo nano /etc/fstab

<p>(with the following line)</p>

	172.31.3.128:/mnt/logs /var/log/httpd nfs defaults 0 0

<p>Confirm mount is successful by running:</p>
  
- [x] sudo mount -a
  
<p>Reload to update the systemd unit:</p>

- [x] sudo systemctl daemon-reload

<p>Fork the tooling source code from Darey.io Github Account to your Github account:</p>

<p>Instead of using Fork we can install git and run:</p>

- [x] git clone https://github.com/darey-io/tooling.git

<p>Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html:</p>

<p>(First copy back the old content of /var/log/httpd back to its original file)</p>

- [x] sudo cp -R /var/log/httpd.bak/. /var/log/httpd

<p>Then inside the tooling folder, run the following command:</p>

- [x] sudo cp -R html/. /var/www/html/

<p>Confirm Apache is running by opening the public ip address from the browser (if it shows error, run from the terminal the following command):</p>

- [x] sudo setenforce 0

<p>Update the website’s configuration to connect to the database (in functions.php file):</p>

<p>Replace from Before details and change to After details</p>
	Before: $db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');
  
	After: $db = mysqli_connect('<db-ipaddress>', 'webaccess', 'password', 'tooling');

o)	Apply tooling-db.sql script:
	(cd into the tooling folder)
	sudo mysql -h 172.31.22.39 -u webaccess -p tooling < tooling-db.sql
	(this can be done after installing mysql-server on the web server)

p)	Along the line, I discovered that login.php was not coming up automatically rather it was showing /index.
	So I installed PHP on the webserver and restarted http service afterwards to see effect; And restart the httpd service for it to effect changes

q)	Create in MySQL a new admin user with username: myuser and password: password:
	INSERT INTO `users` (`id`, `username`, `password`, `email`, `user_type`, `status`) VALUES ('2', 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
  
  
  
