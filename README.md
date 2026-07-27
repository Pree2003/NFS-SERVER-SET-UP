# NFS-SERVER-SET-UP
A scalable AWS DevOps project that deploys a PHP Tooling application using three Apache web servers, an NFS server for shared storage, and a MariaDB database server. Configured with Linux, Git, Apache, NFS, and AWS EC2.

<img width="553" height="49" alt="image" src="https://github.com/user-attachments/assets/13c30164-646b-4501-a0eb-67b979598052" />

To set up my NFS server I logged into my AWS console log to launch an instance, the status showed that it is running. This time I used Redhat as my username my key name was nfsserver.key. My instance had 3/3 passchecks meaning it functions well. Our instance has public IP address 51.21.181.186 a public IP address is important because It is used to connect your local network to the global internet, allowing devices to access websites, stream content, and communicate directly with servers worldwide. 
 and private IP address is important because It allows your computer, smartphone, and printer to communicate internally without being directly exposed to or accessible from the public internet.

First volume
<img width="554" height="275" alt="image" src="https://github.com/user-attachments/assets/5c0873d5-1789-467c-9f48-39fba941be27" />

Second Volume
<img width="554" height="208" alt="image" src="https://github.com/user-attachments/assets/c44dc92a-4f15-4c1c-9095-8eca04de8e11" />

Third Volume
<img width="554" height="272" alt="image" src="https://github.com/user-attachments/assets/0fb12d3d-a6c2-4a88-bd59-76507d79af84" />

Attaching my volumes
<img width="554" height="115" alt="image" src="https://github.com/user-attachments/assets/128c3e75-553b-4aa9-8b82-8af79fb3411c" />
<img width="554" height="202" alt="image" src="https://github.com/user-attachments/assets/48d71c60-e3e7-46da-9010-1e93d8b7fada" />
<img width="554" height="200" alt="image" src="https://github.com/user-attachments/assets/5b17b1b6-43fc-4d8d-9516-2cbaa60efc17" />
<img width="554" height="201" alt="image" src="https://github.com/user-attachments/assets/5606f21b-32e6-4d81-b42c-11cc1468de3a" />

The screenshots above show how I attached the three EBS volumes to my NFS server instance. The first volume was attached as **/dev/sdb**, the second as **/dev/sdc**, and the third as **/dev/sdd**. Assigning unique device names ensured that each volume could be correctly identified and configured during the storage setup process.

<img width="553" height="144" alt="image" src="https://github.com/user-attachments/assets/3d906ddd-9c3e-465c-90ed-33ab9fc78012" />

This screenshot confirms that all three EBS volumes were successfully attached to the NFS server instance. Each volume is listed with an "In-use" status, indicating that the attachment process was completed successfully and the volumes were ready to be configured for storage. 

<img width="554" height="257" alt="image" src="https://github.com/user-attachments/assets/8fe2c289-fcb4-4940-8fa9-7e0babcb8fc3" />

The next step was to establish an SSH connection to my NFS server using its key pair. I connected to the instance with the private IP address 172.31.43.133, which allowed me to securely access the server and continue configuring the storage and NFS services.

UPDATING THE PACKAGES
<img width="553" height="193" alt="image" src="https://github.com/user-attachments/assets/ca4066ee-cee4-4507-ba4e-97d781d95f86" />
<img width="553" height="243" alt="image" src="https://github.com/user-attachments/assets/c4d557b1-ef04-49ba-aa25-a435edd3cf7a" />

Next step was using the command sudo yum -y update  to update all the software packages installed on a Linux system to their latest versions. The sudo part gives you administrator permission to make changes, yum is the tool that manages software, -y automatically answers "yes" to any confirmation prompts, and update checks for and installs available updates. Running this command helps keep the system secure, stable, and ready before installing new software. 

CHECKING THE DISKS

<img width="534" height="173" alt="image" src="https://github.com/user-attachments/assets/ef343be9-c0c2-4c40-a2eb-cc873b743ff1" />

The next step was to verify that the attached EBS volumes were available to the NFS server. I checked the available disks to confirm that all three newly attached volumes had been detected by the operating system and were ready to be partitioned and configured for use with Logical Volume Manager (LVM).

PARTITION OF DISKS

<img width="440" height="549" alt="image" src="https://github.com/user-attachments/assets/b236e4f5-677d-4dcc-a221-4ddaf228c956" />

The next step was to partition each of the three attached EBS volumes using the `parted` utility. I used the following commands to access each disk:

sudo parted /dev/nvme1n1` (Disk 1)
sudo parted /dev/nvme2n1` (Disk 2)
sudo parted /dev/nvme3n1` (Disk 3)

Within the `parted` utility, I created a GPT partition table and a primary partition that utilized the entire disk by running the following commands:

mklabel gpt
mkpart primary xfs 0% 100%
quit

This process prepared each disk with a GPT partition table and a single primary partition formatted for the XFS file system, making the volumes ready for the next stage of the LVM configuration.

INSTALLING LVM 
<img width="553" height="146" alt="image" src="https://github.com/user-attachments/assets/009fe4e9-2162-4b14-9a01-d032b377285a" />

The next step was to install Logical Volume Manager (LVM). Installing LVM is essential because it provides a flexible and efficient way to manage disk storage compared to traditional disk partitioning. It allows multiple physical disks to be combined into a single storage pool, from which logical volumes can be created, resized, or extended as storage requirements change.

In this project, LVM was installed to create and manage the logical volumes lv-apps, lv-logs, and lv-opt. These logical volumes were used to organize and store different types of data on the NFS server, making the storage infrastructure easier to manage and more scalable.

SCANNING SYSTEMS FOR AVAILABLE DISKS

<img width="462" height="194" alt="image" src="https://github.com/user-attachments/assets/b31ea035-44d0-4d26-8ab6-7c5ff7e80fc8" />

The `sudo lvmdiskscan` command was used to scan the system for all available disks and partitions that could be managed by Logical Volume Manager (LVM). This command verifies that the newly attached storage devices have been successfully detected by the operating system before they are configured as physical volumes, volume groups, and logical volumes. Confirming the availability of the disks at this stage ensures that they are ready for the LVM configuration process.

CREATION OF PHYSICAL VOLUMES

<img width="458" height="401" alt="image" src="https://github.com/user-attachments/assets/fd75226a-0591-454d-a449-9ad2be15d82c" />

The next step was to create **physical volumes** from the three disk partitions using the following commands:

sudo pvcreate /dev/nvme1n1p1
sudo pvcreate /dev/nvme2n1p1
sudo pvcreate /dev/nvme3n1p1

These commands initialized each partition as an LVM physical volume, making them available for inclusion in a volume group.

After creating the physical volumes, I verified that they had been successfully created by running the following command:

sudo pvs


The `sudo pvs` command displays all configured physical volumes, allowing me to confirm that each disk partition had been successfully initialized and was ready to be added to a volume group.

CREATING VOLUME  GROUPS

<img width="554" height="198" alt="image" src="https://github.com/user-attachments/assets/808c0022-4082-4309-b34d-f2d446558885" />

The next step was to create a **volume group** named **webdata-vg** using the first physical volume with the following command:

sudo vgcreate webdata-vg /dev/nvme1n1p1


Since the volume group had already been created, I did not create additional volume groups. Instead, I added the remaining physical volumes to the existing **webdata-vg** volume group using the following commands:

sudo vgextend webdata-vg /dev/nvme2n1p1
sudo vgextend webdata-vg /dev/nvme3n1p1


These commands extended the webdata-vg volume group by incorporating the second and third physical volumes. As a result, the storage capacity of the volume group increased, providing sufficient space for creating the logical volumes required for the project.

<img width="455" height="445" alt="image" src="https://github.com/user-attachments/assets/89c40adc-f503-4460-8a93-14a66ca258e2" />

The next step was to use the command sudo vgs command to display a summary of all Volume Groups (VGs) on the system. It shows information such as the volume group name, its size, and how much free space is available. This helps you confirm that your volume group was created successfully. Next I used The sudo vgdisplay webdata-vg command to show detailed information about the volume group named webdata-vg. It displays details such as the total size, available free space, number of physical volumes, logical volumes, and other properties. This command is useful for verifying the configuration and status of a specific volume group.

CREATING 3 LOGICAL VOLUMES
<img width="517" height="154" alt="image" src="https://github.com/user-attachments/assets/4d0caa7c-511f-4d57-82d8-f1fe7c3b88d0" />

The next step is creating logical volumes using these commands: 
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
Creating logical volumes allows you to divide the storage space in a volume group into separate, manageable sections for different purposes. Each logical volume acts like its own disk and can be formatted with a file system, mounted, and used to store data. In this project, logical volumes (lv-apps, lv-logs, and lv-opt) are created to organize and store different types of files on the NFS server, making storage easier to manage and expand when needed. 

XFS file system
<img width="554" height="461" alt="image" src="https://github.com/user-attachments/assets/b2485e11-897f-4311-a3ab-6396b87f1064" />
<img width="554" height="258" alt="image" src="https://github.com/user-attachments/assets/955eacb1-bf45-4f34-8313-83d3ac5b4340" />

CREATING MOUNT DIRECTORIES
<img width="554" height="272" alt="image" src="https://github.com/user-attachments/assets/fb08722e-1911-49f1-8de6-bf9490ef7e1c" />
The `mkdir` commands were used to create the mount point directories /mnt/apps, /mnt/logs, and /mnt/opt. The `mount` commands then attached the logical volumes to these directories, making the storage accessible to the operating system and preparing it for use by the NFS server.


INSTALLING NFS
<img width="553" height="244" alt="image" src="https://github.com/user-attachments/assets/73b1fa28-91e7-4fba-94a4-9b6c41101d19" />

The next step was to install the Network File System (NFS) package using the following command:

sudo yum install nfs-utils -y
This command installed the NFS utilities required to configure the server as an NFS server, enabling it to share directories with the web servers over the network.

<img width="554" height="180" alt="image" src="https://github.com/user-attachments/assets/834f93d7-50ac-4f2e-85f5-e9646c2efef3" />

CONFIGURING NFS PERMISIONS 
<img width="554" height="69" alt="image" src="https://github.com/user-attachments/assets/bdc1fa13-b9f0-4f67-b40e-a267e03ff1af" />
<img width="447" height="571" alt="image" src="https://github.com/user-attachments/assets/1dc9e592-7038-4008-a1a1-86f60c835f47" />
<img width="409" height="92" alt="image" src="https://github.com/user-attachments/assets/19012e5c-f470-43c6-b7b9-fcc77730fe02" />

EDITING INBOUND RULES
<img width="554" height="192" alt="image" src="https://github.com/user-attachments/assets/29f3ec17-7fbb-4c2b-9444-427c490b22e4" />
The next step was to configure the Security Group by adding four inbound rules. These rules allowed communication between the NFS server and the web servers by opening the required ports for NFS services. This ensured that the web servers could securely access the shared directories hosted on the NFS server.

CREATING DATABASE SERVER INSTANCE
<img width="554" height="229" alt="image" src="https://github.com/user-attachments/assets/9a6f95ba-1300-48a9-bcab-47ed230018b7" />

<img width="554" height="257" alt="image" src="https://github.com/user-attachments/assets/abe203e2-cc76-4911-b219-b0c7c6b3236f" />

After the database server instance was running, I established a secure SSH connection to the server using its key pair. This allowed me to access the instance and begin installing and configuring the MariaDB database for the Tooling application.

UPDATING SERVER
<img width="554" height="293" alt="image" src="https://github.com/user-attachments/assets/f5b0bea0-e746-4791-b082-981f0a4beb27" />
The next step was to update the system packages using the following command:

sudo yum -y update

This command updated all installed packages to their latest available versions, ensuring that the database server had the latest security patches, bug fixes, and software updates before installing MariaDB.

<img width="553" height="278" alt="image" src="https://github.com/user-attachments/assets/b3c5824b-1e44-49f0-a7d2-440cfd26ff2b" />

INSTALLING MySQL SERVER

<img width="554" height="110" alt="image" src="https://github.com/user-attachments/assets/08fa17b2-88ad-460a-add3-83f8b6030d55" />

When I attempted to install MySQL using the command `sudo yum install mysql-server -y`, the installation failed because the package was not available on my system. To identify the operating system version, I ran the command `cat /etc/redhat-release`, which showed that I was using Red Hat Enterprise Linux 10.2 (Coughlan). Since MySQL Server is not available by default on RHEL 10.2, I installed MariaDB, which is the default and fully compatible database server for this project.

<img width="380" height="59" alt="image" src="https://github.com/user-attachments/assets/cd9cc0a8-9cb8-4ddb-8815-52e3225649bd" />

INSTALLING MARIADB

<img width="554" height="283" alt="image" src="https://github.com/user-attachments/assets/1792fc21-16f2-4396-9f9b-95bbc083e270" />

Since I could not install MySQL using the `sudo yum install mysql-server -y` command due to the version of my RHEL operating system, I installed **MariaDB** instead by running:

sudo yum install mariadb-server -y

MariaDB is the default database server for Red Hat Enterprise Linux 10.2 and is fully compatible with MySQL, making it a suitable choice for this project.

<img width="553" height="252" alt="image" src="https://github.com/user-attachments/assets/c9af2f87-9894-49b8-afe5-939c796f4bb7" />

MariaDB services were now running 
<img width="538" height="621" alt="image" src="https://github.com/user-attachments/assets/8e4a8abd-6c2c-4c71-9781-90a95ed74c63" />

After installing MariaDB, the next step was to secure the database server by creating a password for the **root** user. Setting a strong password helped protect the database from unauthorized access and ensured that only authorized users could perform administrative tasks.

CREATING DATABASE
<img width="512" height="580" alt="image" src="https://github.com/user-attachments/assets/8d5e30bc-6d6a-485a-9553-f889b1fa3abc" />
<img width="554" height="64" alt="image" src="https://github.com/user-attachments/assets/9fb1b901-0976-4e8e-b5a7-43008d041611" />


The next step was to verify that the MariaDB server was listening on port 3306 by running the following command:
sudo ss -tlnp | grep 3306
This command confirmed that the MariaDB service was active and listening for database connections on port 3306, which is the default port used by MySQL and MariaDB.

<img width="554" height="115" alt="image" src="https://github.com/user-attachments/assets/0624938c-54d7-48c9-8808-31f13356318d" />

LAUNCHING WEBSERVER INSTANCES

<img width="553" height="49" alt="image" src="https://github.com/user-attachments/assets/65fb0e13-ce3a-4a61-9bf6-c2867d8053d4" />
The next step was to launch two EC2 instances, **Web Server 1** and **Web Server 2**. These instances were configured to host the Tooling application and access the shared files stored on the NFS server while connecting to the MariaDB database server.


<img width="554" height="241" alt="image" src="https://github.com/user-attachments/assets/737cbfd0-ba0b-4cab-8c31-d594deae8121" />

After connecting to **Web Server 1** and **Web Server 2** in separate terminals, I installed the required NFS packages using the following command:
sudo yum install nfs-utils nfs4-acl-tools -y
This command installed the utilities required for the web servers to connect to and access the shared directories hosted on the NFS server.

<img width="553" height="256" alt="image" src="https://github.com/user-attachments/assets/41e31692-fc47-4ee6-ac6d-2ba3f54db61d" />

I installed the **NFS client** packages on both **Web Server 1** and **Web Server 2** using the `nfs-utils` and `nfs4-acl-tools` packages. This enabled both servers to connect to the NFS server and access the shared application files over the network.

CREATING WEB ROOT DIRECTORY& MOUNTING NFS SHARE

<img width="554" height="162" alt="image" src="https://github.com/user-attachments/assets/3c0ac619-698a-4acc-9a90-295c87bee5d8" />

The sudo systemctl status nfs-server.service command was used to check the current status of the NFS server service. It confirmed whether the service was active and running correctly, ensuring that the NFS server was ready to share files and directories with the web servers.

<img width="553" height="277" alt="image" src="https://github.com/user-attachments/assets/a7c36816-4fd3-4924-af18-ab708189c0f3" />

The sudo systemctl status firewalld command was used to check whether the firewall service was running. This helped verify the current firewall status and ensure that the required network services, such as NFS communication, were not being blocked.

<img width="554" height="371" alt="image" src="https://github.com/user-attachments/assets/c0374c08-c823-4fe6-a185-3805f28e5682" />

The sudo ss -tulnp | grep -E "111|2049" command was used to check that the required NFS ports were active and listening. Port 111 is used by the RPC service (port mapper), while port 2049 is the default port used by the NFS service. Verifying these ports confirmed that the NFS server was ready to accept connections from the web servers.

<img width="554" height="92" alt="image" src="https://github.com/user-attachments/assets/0279a94c-5c4b-4419-b054-3f6744286fde" />

The next step was to configure the NFS exports file by adding the private IP address of the web servers' network. The private IP address was added to define which clients were allowed to access the shared NFS directory. This ensured that only authorized servers within the private network could mount and use the shared storage.

<img width="554" height="472" alt="image" src="https://github.com/user-attachments/assets/a3f59bda-1f99-4ca7-91b4-308acc03c5cf" />

 The rpcinfo -p 172.31.43.133 command was used to verify the RPC services running on the NFS server. It displayed the available RPC programs, including NFS-related services, and confirmed that the NFS server was reachable and properly configured for remote file sharing.

<img width="554" height="83" alt="image" src="https://github.com/user-attachments/assets/942804a6-caf9-4ac1-a91d-1360cdc6c4fc" />

The mount | grep /var/www command was used to confirm that the NFS share was successfully mounted to the /var/www directory. The output verified that the web server was connected to the NFS server and could access the shared application files stored on the NFS storage
 
INSTALLING APACHE 
<img width="554" height="224" alt="image" src="https://github.com/user-attachments/assets/f7633575-86c5-485b-8d0d-b7cce0cd534c" />

The next step was to install Apache HTTP Server on Web Server 1 using the package manager. Apache was installed to act as the web server responsible for serving the Tooling application files to users through the browser.

<img width="554" height="266" alt="image" src="https://github.com/user-attachments/assets/79289e94-f769-4b03-8d20-7022b5bd92f7" />

The sudo dnf list php command was used to check the availability of PHP packages in the system repositories. This helped verify that PHP was available for installation before configuring the web server to support PHP-based application files.

 <img width="554" height="266" alt="image" src="https://github.com/user-attachments/assets/5021c728-49b2-4c38-9172-1e7004654d82" />





























