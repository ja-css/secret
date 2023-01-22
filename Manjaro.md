Add rc-local service

1. Create `/etc/rc.local` script
```
#!bin/sh
echo hello
exit 0
```

2. Create service
Create a file `/etc/systemd/system/rc-local.service`
```
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```
Then, just run `systemctl enable rc-local.service` as root to enable it. You can also test it/run it now by running `systemctl start rc-local.service`.


## DNS

`/etc/NetworkManager/system-connections/{connection}.conf`

```
[ipv4]
dns=127.0.0.1;
ignore-auto-dns=true
method=auto

```