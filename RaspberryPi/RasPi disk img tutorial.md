*Luks tutorial has disk img part as well*  
  
# How to Back Up Your Raspberry Pi as a Disk Image  
https://www.tomshardware.com/how-to/back-up-raspberry-pi-as-disk-image  
  
  
August 13, 2020  
  
Turn your Raspberry Pi microSD card into a small .img file you can share or take anywhere.  
  
  
When you work hard on a Raspberry Pi project, you’ll want to make a complete disk backup of the entire OS and software, not just your code. Even the  [best Raspberry Pi microSD cards](https://www.tomshardware.com/best-picks/raspberry-pi-microsd-cards)  can fail or get lost and you may also want to re-use the project on another card or share it with others. For example, at Tom’s Hardware, we have a  [Raspberry Pi web server](https://www.tomshardware.com/news/raspberry-pi-web-server,40174.html)  we use for battery tests and we have multiple Pis that all have the same exact image.  
  
There are a few ways to backup a Raspberry Pi. You can use Raspberry Pi OS’s SD Card Copier app, which is under the Accessories section of the Start menu, to clone your microSD card directly to another microSD card. But unless you need a second card right away, it’s a better idea to create a disk image: a file you can store on a PC or in the cloud, distribute to others and write to a new microSD card at any time, using Etcher or Raspberry Pi Disk Imager.  
  
Some folks recommend taking your microSD card, sticking it in a Windows PC and copying it sector-for-sector with  [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/), but that creates two problems. First, the card you are writing to has to be exactly the same size as the one you backed up or larger. Because there are subtle differences in the number of sectors on different makes and models of card, a 32GB San Disk card may have a few more sectors than a 32GB Samsung card and, if the destination card is smaller, the copy process won’t work properly and the card won’t boot. Second, your backup file will be huge: the full size of your card, even if you only were using 3 out of 32GB.  
  
Fortunately, there’s a way to create a compressed disk image that’s even smaller than the amount of used space on the source microSD card you’re backing up. To create the disk image, you’ll need an external USB drive to connect to your Raspberry Pi and write it to. If the USB drive is a higher capacity than the source microSD card (ex: a 32GB USB drive to image a 16GB card), you can back up the whole card before shrinking it. However, if you don’t have a USB drive that’s big enough, check out the section at the bottom of this article on shrinking your rootfs partition.  
  
## How to Make a Raspberry Pi Disk Image  
  
1. **Format a USB Flash or the** [**best hard drive**](https://www.tomshardware.com/best-picks/best-hard-drives)  as either NTFS (if you are using Windows on your PC and plan to read this drive on a PC) or EXT4 (for Linux). Make sure the Flash drive is larger than the capacity of the used space. Make sure to give the drive a volume name that you remember (ex: “pibkup” in our case). You can also format the drive directly on the Raspberry Pi if you like.  
  
2. **Connect the USB drive** to your Raspberry Pi.  
  
**3. Install pishrink.sh**  on your Raspberry Pi and copy it to the  **/usr/local/bin** folder by typing:  
  
```bash  
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh  
sudo chmod +x pishrink.sh  
sudo mv pishrink.sh /usr/local/bin  
```  
  
4.  **Check the mount point path** of your USB drive by entering  
  
```bash  
lsblk  
```  
  
You’ll see a list of drives connected to the Raspberry Pi and the mount point name of each. Your USB drive will probably be mounted at  _/media/pi/[VOLUME NAME]_. In our case, it was  _/media/pi/pibkup_. If your drive isn’t mounted, try rebooting with the USB drive connected or you can mount it manually by typing  _sudo mkdir /dev/mysub_  to create a directory and  _sudo mount /dev/sda1 /dev/myusb_  to mount it. However, you can’t and shouldn’t do that if it’s already mounted.  
  
**5. Copy all your data to an img file**  by using the dd command.  
  
```bash  
sudo dd if=/dev/mmcblk0 of=[mount point]/myimg.img bs=1M  
```  
  
However, if you shrank a partition on the source microSD card, you’ll need to use the count attribute to tell it to copy only as many MBs as are in use. For example, in our case, we had had a 16GB card, but after shrinking the rootfs down to 6.5GB, the card only had about 6.8GB in use (when you count the /boot partition). So, to be on the safe side (better to copy too much data than too little), we rounded up and set dd to copy 7GB of data by using count=7000. The amount of data is equal to count * block size (bs) so 7000 * 1M means 7GB.  
  
```bash  
sudo dd if=/dev/mmcblk0 of=[mount point]/myimg.img bs=1M count=7000  
```  
  
Only do this if you have previously shrunk the partition. If you use count and copy less than the full partition you could have an incomplete image that's missing data or won't boot.  
  
6.  **Navigate to the USB drive's root directory**.  
  
```bash  
cd /media/pi/pickup  
```  
  
7. **Use pishrink with the -z parameter**, which zips your image up with gzip.  
  
```bash  
sudo pishrink.sh -z myimg.img  
```  
  
  
(Image credit: Tom's Hardware)  
  
This process will also take several minutes but, when it is done, you will end up with a reasonably sized image file called myimg.img.gz. You can copy this file to your PC, upload it to the cloud or send it to a friend.  
  
## How to Shrink a Partition on Raspberry Pi  
  
If you want to make a disk image of a microSD card, but don’t have an external USB drive of a greater capacity, you have a problem. Even though the eventual .img.gz file you create in the tutorial above should be much smaller than your source card, you still need enough space to accommodate the uncompressed .img file as part of the process.  
  
What’s particularly frustrating is that, by default, the dd file copy process makes an image out of ALL the space on your microSD card, even the unused space.For example, you might have a 64GB microSD card, but only be actually using 6GB of space. If you don’t shrink the rootfs partition, you will end up copying all 64GB over to your external drive, which will take a lot more time to complete and will require that you have at least 65GB of free space.  
  
So the solution is to shrink the rootfs partition of your microSD card down to a size that’s just a little bit bigger than the amount of used space. Then you can copy just your partitions over to the USB drive.  
  
To do the shrinking, you’ll need a USB microSD card reader and a second microSD card with Raspberry Pi OS on it.  
  
1.  **Put your source microSD card** (the one you want to copy)  **in a reader** and connect to your Raspberry Pi.  
  
2.  **Boot your Raspberry Pi** off a different microSD card.  
  
3.  **Install gparted** on your Raspberry Pi.  
  
```bash  
sudo apt-get install gparted -y  
```  
  
4. **Launch gparted** from within the Raspberry Pi OS GUI. It’s in the System Tools section of the start menu.  
  
5. **Select your external microSD card** from the pull down menu in the upper right corner of the gparted window.  
  
6.  **Unmount the rootfs partition** if it is mounted (a key icon is next to it) by right clicking it and selecting Unmount from the menu. If the option is grayed out, it’s not mounted.  
  
7. **Right click rootfs and select Resize / Move.**  
  
8. **Set the new size for the partition** as the minimum size or slightly larger and click Resize.. Note that gparted may overreport the amount of used space (when we unmounted a partition with 4.3GB used, it changed to say 6GB were in use), but you have to go with at least its minimum.  
  
9. **Click the green check mark**  in the gparted window and  **click Apply**  (when warned) to proceed.  
  
10.  **Shutdown**  the Raspberry Pi.  
  
11. **Remove the source microSD card**  from the USB card reader and insert it into the Raspberry Pi to boot from.  
  
12.  **Follow the instructions**  in the section above on creating a disk image. Make sure to use the  _count_ attribute in step 5.  
  
## Writing Your Raspberry Pi Disk Image to a Card  
  
Once you’re done, you’ll have a file with the extension .img.gz and you can write or “burn” it to a microSD card the same way you would any .img file you download from the web. The easiest way to burn a custom image is to:  
  
1. **Launch Raspberry Pi Imager**  on your PC. You can  [download Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)  if you don’t have it already.  
  
2. **Select Use custom** from the Choose OS menu.  
  
3. **Select your .img.gz file**.  
  
4. **Select the microSD card**  you wish to burn it to.  
  
5. **Click Write**.  
