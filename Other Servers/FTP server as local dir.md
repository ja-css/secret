# MOUNT REMOTE FTP SERVER FOLDER AS LOCAL DIRECTORY ON LINUX / UBUNTU / DEBIAN / RASPBERRY / RASPBIAN

2019-11-16

[Original link](https://gaborhargitai.hu/mount-remote-ftp-server-folder-as-local-directory-on-linux-ubuntu-debian-raspberry-raspbian/#:~:text=1%201.%20Install%20curlftpfs%20apt-get%20install%20curlftpfs%202,FTP%20server%20using%20a%20hostname%20or%20IP%20address)

Have a remote FTP server directory mounted locally is pretty convenient for cross-server copying.

It could also come in handy if you would like to copy content from your NAS to say, and RGH/JTAG Xbox360 using the built-in FTP server of Phoenix or FSD FreeStyle Dashboard.

**1. Install curlftpfs**

```
apt-get install curlftpfs
```

**2. Create a local directory**

```
mkdir /media/ftp
```

**3. Mount the FTP server using a hostname or IP address**

```
curlftpfs ftpuser:ftppass@ftphostname /media/ftp/
```