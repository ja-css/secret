Similar to:
https://mariadb.com/kb/en/installing-mariadb-on-macos-using-homebrew/


## 1. Install mysql using brew

`brew install mariadb`


## 2. Setup auto-start for MariaDB Server

Use Homebrew's services functionality, which configures auto-start with the `launchctl` utility from  [launchd](https://mariadb.com/kb/en/launchd/):

`brew services start mariadb`


## 3. How to log in

We've installed your MySQL database without a root password. To secure it run:
`mysql_secure_installation`

mysql.server start

After MariaDB Server is started, you can log in as your user:
`mysql`

??? Or log in as root:
??? `sudo mysql -u root`


## 4. How to set up a root password and a user

## Troubleshooting
Error:
```
john@host local % brew services start mariadb
Error: Permission denied @ rb_sysopen - /Users/john/Library/LaunchAgents/homebrew.mxcl.mariadb.plist
```

`sudo chown john /Users/john/Library/LaunchAgents`
`chmod 755 ~/Library/LaunchAgents`

Error: (in log file):

```
2022-12-24 17:37:22 0 [ERROR] Could not open mysql.plugin table: "Table 'mysql.plugin' doesn't exist". Some plugins may be not loaded
2022-12-24 17:37:22 0 [ERROR] Can't open and lock privilege tables: Table 'mysql.servers' doesn't exist
2022-12-24 17:37:22 0 [ERROR] Fatal error: Can't open and lock privilege tables: Table 'mysql.db' doesn't exist
```

`mysql_install_db`


## Show current user

`Select current_user();`


mysql -u john -p --host=localhost --protocol=tcp --port=3306


SET PASSWORD = password('SECURE_SUPER_PASS');