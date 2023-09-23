#### See also, [RasPi disk LUKS encr.md](..%2FRaspberryPi%2FRasPi%20disk%20LUKS%20encr.md)

Below stuff didn't work for me.  
Easy mode - got to Disks,   
choose settings for the drive,  
format  
erase -> internal disk, LUKS encrypted  
  
  
  
# USB stick encryption using Linux  
  
[1. Install cryptsetup](#install-cryptsetup)  
[2. Partition a USB stick](#partition-a-usb-stick)  
[3. Encrypt USB stick partition](#encrypt-usb-stick-partition)  
[4. Desktop mount of an encrypted USB partition](#desktop-mount-of-an-encrypted-usb-partition)  
[5. Mounting USB partition and decryption](#mounting-usb-partition-and-decryption)  
  
## 1. Install cryptsetup  
  
To install cryptsetup:  
`sudo apt install cryptsetup`  
  
## 2. Partition a USB stick  
  
**WARNING**    
Before proceeding, bear in mind that you will lose all data currently stored on your flash drive. If there’s anything important on it, be sure to move the files to your computer for the time being, then you can put them back onto the USB stick after you’ve completed the guide.  
  
1.  Let’s start with partitioning of our USB stick. Insert your USB stick into PC’s USB slot and as a root user execute:  
    `fdisk -l`  
    Search the output of  `fdisk`  command and retrieve the disk file name of your USB stick. In our case, the device is  `/dev/sdX`.  
      
2.  Once we have a file name of our USB stick we can create partitions to be used for encryption and for storage of non-private data. In this example, we will split the USB stick into two partitions, first with size of 2GB and the rest of the space will be used to create second partition and this will produce  `/dev/sdX1`  and  `/dev/sdX2`  respectively. Use any partition tool you see fit for this purpose; in this article we will use  `fdisk`.  
    `fdisk /dev/sdX`  
      
3.  Execute the following commands within the fdisk interactive mode:  
```      
    # Command (m for help): -> n  
      
    [Press enter twice]  
      
    # Do you want to remove the signature? -> y  
    # Command (m for help): -> n  
      
    [Press enter three times]  
      
    Command (m for help): w  
```      
  
[![Partitioning the USB stick with fdisk](https://linuxconfig.org/wp-content/uploads/2013/03/02-usb-stick-encryption-using-linux.png)](https://linuxconfig.org/wp-content/uploads/2013/03/02-usb-stick-encryption-using-linux.png "Partitioning the USB stick with fdisk")  
  
Partitioning the USB stick with fdisk  
  
4.  We now have two partitions, the first one is 2GB in size and will contain our encrypted files. The other partition consumes the rest of the USB stick and will contain non-sensitive information. The two partitions are represented as  `/dev/sdX1`  and  `/dev/sdX2`, but yours may be different. We will now put a filesystem on the partitions. We’re using FAT32 but you may use whatever you want.  
      
`mkfs.fat /dev/sdX1`  
`mkfs.fat /dev/sdX2`  
      
5.  To avoid pattern based encryption attacks it is advisable to write some random data to a partition before proceeding with an encryption. The following  `dd`  command can be used to write such data to your partition. It may take some time. Time depends on the entropy data generated by your system:  
      
`dd bs=4K if=/dev/urandom of=/dev/sdX1`  
      
## 3. Encrypt USB stick partition  
  
Now it is time to encrypt the newly created partition. For this purpose we will use the  `cryptsetup`  tool. If  `cryptsetup`  command is not available on your system make sure that cryptsetup package installed.  
  
The `cryptsetup`  [Linux command](https://linuxconfig.org/linux-commands)  will encrypt the  `/dev/sdX1`  partiton with 256-bit AES XTS algorithm. This algorithm is available on any kernel with version higher than 2.6.24.  
  
`cryptsetup -h sha256 -c aes-xts-plain -s 256 luksFormat /dev/sdX1`  
  
You will be prompted to set a decryption passphrase on the device, which will be used to unlock it and view the sensitive content on your encrypted partition.  
  
## 4. Desktop mount of an encrypted USB partition  
  
Your desktop may react to an encrypted partition by pop-up dialog to prompt you to enter a password for your encrypted partition.  
  
However, some Linux systems may not provide any facility to mount encrypted partitions and you would have to do it manually ( see section “Mounting USB encrypted partition” for details ). In any case make sure that you have cryptsetup package installed and thus md_crypt module loaded in to the running kernel in order to use your encrypted USB stick.  
  
## 5. Mounting USB partition and decryption  
  
1.  In the next step we will set name of our encrypted partition to be recognized by the system’s device mapper. You can choose any name. For example we can use the name “private”:  
      
`cryptsetup luksOpen /dev/sdX1 private`  
      
2.  After executing this command your encrypted partition will be available to your system as  `/dev/mapper/private`. Now we can create a mount point and mount the partition:  
      
`mkdir /mnt/private`  
`mount /dev/mapper/private /mnt/private`  
`chown -R myusername.myusername /mnt/private`  
      
3.  Now your encrypted partition is available in  `/mnt/private`  directory. If you do not wish to have an access to your USB stick’s encrypted partition anymore you need to first unmount it from the system and then use cryptsetup command to close the connected protection.  
      
`umount /mnt/private `  
`cryptsetup luksClose /dev/mapper/private`  
  