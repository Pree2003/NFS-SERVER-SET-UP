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

text
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

The `sudo lvmdiskscan` command was used to scan the system for all available disks and partitions that could be managed by **Logical Volume Manager (LVM)**. This command verifies that the newly attached storage devices have been successfully detected by the operating system before they are configured as physical volumes, volume groups, and logical volumes. Confirming the availability of the disks at this stage ensures that they are ready for the LVM configuration process.

CREATION OF PHYSICAL VOLUMES

<img width="458" height="401" alt="image" src="https://github.com/user-attachments/assets/fd75226a-0591-454d-a449-9ad2be15d82c" />

The next step was to create **physical volumes** from the three disk partitions using the following commands:

```text
sudo pvcreate /dev/nvme1n1p1
sudo pvcreate /dev/nvme2n1p1
sudo pvcreate /dev/nvme3n1p1
```

These commands initialized each partition as an LVM physical volume, making them available for inclusion in a volume group.

After creating the physical volumes, I verified that they had been successfully created by running the following command:

text
sudo pvs


The `sudo pvs` command displays all configured physical volumes, allowing me to confirm that each disk partition had been successfully initialized and was ready to be added to a volume group.

CREATING VOLUME  GROUPS

<img width="554" height="198" alt="image" src="https://github.com/user-attachments/assets/808c0022-4082-4309-b34d-f2d446558885" />

The next step was to create a **volume group** named **webdata-vg** using the first physical volume with the following command:

text
sudo vgcreate webdata-vg /dev/nvme1n1p1


Since the volume group had already been created, I did not create additional volume groups. Instead, I added the remaining physical volumes to the existing **webdata-vg** volume group using the following commands:

text
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
The `mkdir` commands were used to create the mount point directories **/mnt/apps**, **/mnt/logs**, and **/mnt/opt**. The `mount` commands then attached the logical volumes to these directories, making the storage accessible to the operating system and preparing it for use by the NFS server.























