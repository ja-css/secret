# Do I need XFS   
# OR   
# Do I want to write RAW?  
  
I mean who needs this shit right,  
add so much bloody complexity to achieve the same fukken result  
FFS  
Like I don't even need inodes and shit, or a tree structure with folder srsly.  
I do it myself anyway!  
sd  
  
  
https://flylib.com/books/en/1.393.1.118/1/  
RAW disk IO  
  
  
## **https://tldp.org/HOWTO/SCSI-2.4-HOWTO/rawdev.html**  
RAW devices ^  
https://stackoverflow.com/questions/977829/linux-direct-access-to-the-hard-disk-in-c  
How utilities like `fsck` and `mkfs` are implemented?  
  
https://www.linuxquestions.org/questions/linux-software-2/writing-data-to-a-raw-disk-using-dd-928423/  
  
Look at dd source code  
https://github.com/coreutils/coreutils/blob/master/src/dd.c  
functions   
`iread` -> `read`  
`iwrite` -> `write`  
also,   
`iclose`,  `fcntl`, `invalidate_cache`, `skip`, `ifstat`, `lseek`, `iftruncate`, `ifdatasync`, `ifsync`, `set_fd_flags`, `ifd_reopen`.  
  
  
---  
  
  
https://wiki.archlinux.org/title/XFS  
  
  
https://man7.org/linux/man-pages/man8/xfs_spaceman.8.html  
xfs_spaceman - show free space information about an XFS  
       filesystem  
  
  
https://man7.org/linux/man-pages/man5/xfs.5.html  
xfs - layout, mount options, and supported file attributes for  
       the XFS filesystem  
  
https://www.systutorials.com/docs/linux/man/1-fallocate/  
fallocate: preallocate or deallocate space to a file  
  
https://manpages.ubuntu.com/manpages/trusty/man3/xfsctl.3.html  
 xfsctl - control XFS filesystems and individual files  
  
  
  
http://ftp.ntu.edu.tw/linux/utils/fs/xfs/docs/xfs_filesystem_structure.pdf  
XFS Algorithms & Data Structures 3rd Edition  
  
