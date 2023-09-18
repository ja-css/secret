
## 1. Find noisy (network) processes on MacOS

### 1.1. Install iftop
`brew install iftop`

### 1.2. sometimes it's not on the path immediately, locate it
`brew link iftop`

With output like:
```
Warning: Already linked: /usr/local/Cellar/iftop/1.0pre4
To relink, run:
brew unlink iftop && brew link iftop
```

### 1.3. run iftop on main interface

`sudo /usr/local/Cellar/iftop/1.0pre4/sbin/iftop -ni en8`

## 2. Find noisy process ID
e.g. for destination address
`sudo lsof -niTCP | grep '3.92.93.117'`

With output like:
```
user@host ~ % sudo lsof -niTCP | grep '3.92.93.117'
Password:
repmgr     8122            root   19u  IPv4 0x6254f84c89ebc83      0t0  TCP 192.168.1.40:49659->3.92.93.117:https (ESTABLISHED)
repmgr     8122            root   28u  IPv4 0x6254f84c812e423      0t0  TCP 192.168.1.40:49660->3.92.93.117:https (ESTABLISHED)
```

## 3. Find process path by id
`ps axwww -o pid,command | grep 8122`

With output like:
```
user@host ~ % ps axwww -o pid,command | grep 8122
8122 repmgr
66215 grep 8122
```

Well wtf is this `repmgr` shit right.  
Supposed to show a normal path like this:
```
user@host ~ % ps axwww -o pid,command
...
92177 /usr/libexec/remindd
92340 /System/Library/PrivateFrameworks/IntelligencePlatformCore.framework/Versions/A/knowledgeconstructiond
98451 /System/Library/PrivateFrameworks/Noticeboard.framework/Versions/A/Resources/nbagent.app/Contents/MacOS/nbagent
98457 /System/Library/PrivateFrameworks/Noticeboard.framework/Versions/A/Resources/nbstated
```

OR

`ps xuwww -p 8122`

With output like:
```
john@CSS-C02FM602MD6R ~ % ps xuwww -p 8122          
USER   PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
root  8122   5.6  1.2 37417772 810724   ??  S<s  Wed11AM  32:19.76 repmgr
```

---

Let's try killing it anyway:
```
john@CSS-C02FM602MD6R ~ % sudo kill -9 8122
kill: 8122: Operation not permitted
```
Kinda expected.
One thing's clear is that it (highly likely) ain't no fucking PostgreSQL replication manager process.

## 4. Locate the executable file and send it to virustotal

Can try mc file search, it showed this:
```
/Applications/VMware Carbon Black Cloud                                                                                                                                                                                                  │ 3 11:00│
     repmgr.bundle                                                                                                                                                                                                                        │ 1  2022│
/Applications/VMware Carbon Black Cloud/repmgr.bundle/Contents/MacOS                                                                                                                                                                     │ 4  2021│
     repmgr                                                                                                                                                                                                                               │ 4 13:51│
```

---
