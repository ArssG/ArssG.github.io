---
title: Implement of a Secure System with RAID1, LVM and directory management on linux.
date: 2024-01-21 23:30:40 -6000
categories: [Linux]
tags: [RAID1, LVM, "Virtual machine", Virtualization]
---


## Objective 
Develop a robust and secure storage system using RAID 1, Logical Volume Management (LVM), and establish a directory structure for public and private files in a Linux environment.  

### Prerequisite
- Have a VM running a linux distribution with 2 or more disks with the same capacity.
- Have mdadm and lvm2 libraries available.

### 1. Disk configuration
Check if the disks are attached.  
```bash
lsblk
```
![](/assets/RAID1/Pasted%20image%2020240129025444.png)  
We are going to partition the disk with the default settings as it will take all the disk space.  
```bash
fdisk /dev/sdb
--> n #for new
--> p #for a primary partition
--> #then we just press enter to select default configurations
--> w #for write the disk configuration
```
![](/assets/RAID1/Pasted%20image%2020240129030351.png)  
We will do the same for the next disk "**/dev/sdc**"
Once we have partitioned both disk, typing "**lsblk**" will seems like this:
![](/assets/RAID1/Pasted%20image%2020240129030826.png)  
### 2. RAID 1 Configuration
Configuration of RAID 1 to provide redundancy and fault tolerance.  
Now, typing the next command will create the RAID 1 using our two disks
```bash
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sd[bc]
```
- **`mdadm`**: The command-line utility for managing Linux software RAID arrays.
- **`--create`**: Specifies that a new array should be created.
- **`/dev/md0`**: Specifies the device name for the RAID array. You can choose a different name if needed.
- **`--level=1`**: Specifies the RAID level, in this case, it's RAID 1, which is a mirrored array. Data is duplicated on each drive for redundancy.
- **`--raid-devices=2`**: Specifies the number of devices in the RAID array. In this case, there are two devices.
- **`/dev/sd[bc]`**: Specifies the devices to be included in the RAID array. It uses a wildcard (`[bc]`) to represent `/dev/sdb` and `/dev/sdc`. These two drives will be part of the RAID 1 array.
Finally we have to validate we want to create it, we type yes.  

![](/assets/RAID1/Pasted%20image%2020240129032000.png)  
To check the status we can type:
```bash
mdadm --misc --detail /dev/md0
```
![](/assets/RAID1/Pasted%20image%2020240129032310.png)  
### 3. Creation of Physical Volume and Logical Group 
Creation of a physical volume on RAID 1.  
```bash
pvcreate /dev/md0
```
![](/assets/RAID1/Pasted%20image%2020240129032531.png)  
To check the status of the physical volume we can type:
```bash
pvdisplay
```
![](/assets/RAID1/Pasted%20image%2020240129032632.png)  
Establishment of a logical group named `storage_group` for flexible space management.  
```bash
vgcreate storage_group /dev/md0
```
![](/assets/RAID1/Pasted%20image%2020240129033210.png)  
To check the status we can type the command:
```bash
vgdisplay
```
![](/assets/RAID1/Pasted%20image%2020240129033346.png)  
### 4. Creation of Logical Volumes 
Creation of logical volumes `public_lv` and `private_lv` for efficient storage organization.  
```bash
lvcreate --name public_lv --size 2Gb storage_group
lvcreate --name private_lv --size 2Gb storage_group
```
- `lvcreate`: This is the command used to create a new logical volume.
- `--name public_lv`: Specifies the name of the logical volume to be created. In this case, it is named `public_lv`.
- `--size 2Gb`: Specifies the size of the logical volume. The size is set to 2 gigabytes in this example. You can adjust the size based on your requirements.
- `storage_group`: Specifies the name of the volume group in which the logical volume should be created.  
![](/assets/RAID1/Pasted%20image%2020240129033934.png)  
To check the status we can type:
```bash
lvdisplay
```
![](/assets/RAID1/Pasted%20image%2020240129034054.png)  
### 5. File System and Mounting  
Formatting of logical volumes as file systems (e.g., ext4).   
```bash
mkfs.ext4 /dev/storage_group/public_lv
mkfs.ext4 /dev/storage_group/private_lv
```
![](/assets/RAID1/Pasted%20image%2020240129035546.png)  
Mounting of logical volumes in specific directories such as `/mnt/public` and `/mnt/private`.
First we need to create the folders with:
```bash
sudo mkdir /mnt/public
sudo mkdir /mnt/private
```
![](/assets/RAID1/Pasted%20image%2020240129035833.png)  
Now, to mount the volume groups we use the command "**mount**"
```bash
mount /dev/storage_group/public_lv /mnt/public
```
```bash
mount /dev/storage_group/public_lv /mnt/rivate
```
![](/assets/RAID1/Pasted%20image%2020240129040325.png)  

### 6. Directory Configuration  
Creation of structured subdirectories in `/mnt/public` and `/mnt/private` to store files based on accessibility.
```bash
mkdir /mnt/public/uploads
mkdir /mnt/public/downloads
mkdir /mnt/private/personal
mkdir /mnt/private/confidential
```

### 7. Security and Permissions 
Establishment of appropriate permissions to ensure the security of stored data.
- Creating groups for every folder.  
	```bash
	sudo addgroup public_group
	sudo addgroup private_group
	```
	![](/assets/RAID1/Pasted%20image%2020240129041444.png)  

- Add users to the group
- Change the groups of the public and private folders  to their respective groups.  
	```bash
	sudo chown :public_group /mnt/public
	sudo chown :private_group /mnt/private
	```
	![](/assets/RAID1/Pasted%20image%2020240129044857.png)  
- Change the permissions of the folder.
	```bash
	sudo chmod 770 /mnt/public
	sudo chmod 770 /mnt/private
	```
	![](/assets/RAID1/Pasted%20image%2020240129045211.png)  
	Users are able to create files and folders in those folders, to ensure that every user can edit and write in those files we can type the command
	```bash
	sudo chmod +s /mnt/public
	```
	Now, every file and folder will get the UID that have the parent folder /mnt/public.  
	
	![](/assets/RAID1/Pasted%20image%2020240129050333.png)  
### 8. Testing 
Creation and manipulation of test files in designated directories to validate functionality.
If I try to create a file with an user that is not in the group an error will be displayed.  
![](/assets/RAID1/Pasted%20image%2020240129050424.png)  

### 9. Automation 
Configuration of entries in `/etc/fstab` for the automatic mounting of logical volumes at system startup.
```bash
sudo nano /etc/fstab
```
The fstab should look like this:  
![](/assets/RAID1/Pasted%20image%2020240129051024.png)  
Finally to test if everything worked correctly, we need to restart the VM and check with lsblk if the logic volumes are linked to the folders.  

![](/assets/RAID1/Pasted%20image%2020240129053644.png)  

