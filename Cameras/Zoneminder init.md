Mostly I'm a Linux wannabe, so these issues may be self evident to others?! There were two separate problems I encountered after a clean install of zoneminder, outlined below.

Conf file permissions
apt-get install zoneminder does not set the permissions on the conf file correctly, so the first error, "Access denied for user 'www-data'@'localhost' (using password: NO)" is a red herring. Changing the group ownership of the conf file will resolve the error:

$ chgrp -c www-data /etc/zm/zm.conf
$ systemctl stop zoneminder.service
$ systemctl start zoneminder.service
See also: https://forums.zoneminder.com/viewtopic.php?p=128894#p128894

However you immediately stumble upon the next problem: "ERR [Error reconnecting to db: errstr:Access denied for user 'zmuser'@'localhost' (using password: YES) error val:]"

Creating the ZM database ZMUSER in MySql
I was surprised to find out that the zoneminder database and database user were not created upon installation. Perhaps this is a result of the conf file permissions above. In any case, this is how I resolved it:

step 1: find your mysql admin user and password:
$ sudo cat /etc/mysql/debian.cnf
see also: https://stackoverflow.com/a/54165621/188474

step 2: create the zoneminder database
Now that you know your admin user, you can create the zoneminder database:

$ mysql -u debian-sys-maint -p < /usr/share/zoneminder/db/zm_create.sql
Enter password:
$ mysql -u debian-sys-maint -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 34
Server version: 8.0.35-0ubuntu0.22.04.1 (Ubuntu)

...

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| zm                 |
+--------------------+
Looking better! We now have the zm database!

step 3: log into mysql and create the zmuser database user
You can then either create the ZM user from the terminal prompt, or login to mysql and create the user. I logged in because it felt more familiar:

$ mysql -u debian-sys-maint -p
Enter password:
Welcome to the MySQL monitor.
...
~mysql> create user zmuser identified by '<strong password here>';~
CREATE USER 'zmuser'@'localhost'; IDENTIFIED BY 'zmpass';
Query OK, 0 rows affected (0.08 sec)

~mysql> grant all on zm.* to zmuser;~
GRANT ALL PRIVILEGES ON zm.* TO 'zmuser'@'localhost';
Query OK, 0 rows affected (0.02 sec)


mysql> exit
Bye
step 4: update the zoneminder conf file
Almost there! MySql is setup with the zmuser, grants and the correct zm database and schema. The last step is to update the zoneminder conf file with the zmuser password you created above:

$ sudo vi /etc/zm/zm.conf
Touching the conf file seemed to restart the zoneminder service, so the website immediately just worked for me. If not, try stopping and restarting zoneminder and/or apache:

sudo systemctl restart apache2
sudo systemctl restart zoneminder


setp 5:
add zoneminder conf to apache:
# For Debian/Ubuntu
sudo ln -s /etc/apache2/conf-available/zoneminder.conf /etc/apache2/conf-enabled/zoneminder.conf


webcam as RSTP
ffmpeg -f v4l2 -input_format mjpeg -i /dev/video0 -pix_fmt yuv420p -r 30 -f rtsp -rtsp_transport tcp rtsp://localhost:8554/stream