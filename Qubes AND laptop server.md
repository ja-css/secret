# Laptop Debian server

## Add sbin to PATH
nano /etc/profile
add
`/usr/local/sbin:/usr/sbin:/sbin`

## Disable suspend
`systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target`

## Turn screen off

`sudo vbetool dpms off`

??? on after key press
problem here: 'vbetool dpms on' also turns display off
`sudo sh -c 'vbetool dpms off; read ans; vbetool dpms on'` 



# Qubes OS

## Volume control Fedora
`amixer`



## Persistence and Inheritance

| Qube Type | Inheritance [1] | Persistence [2] | 
| -------------------------------------------------------|--------------| ------------|
| [template](https://www.qubes-os.org/doc/glossary/#template) | N/A (templates cannot be based on templates) | everything | [app qube](https://www.qubes-os.org/doc/glossary/#app-qube)[3] |
| [app qube](https://www.qubes-os.org/doc/glossary/#app-qube)(3) | `/etc/skel`  to  `/home`;  `/usr/local.orig`  to  `/usr/local` | `/rw`  (includes  `/home`,  `/usr/local`, and  `bind-dirs`)
| [disposable](https://www.qubes-os.org/doc/glossary/#disposable) | `/rw`  (includes  `/home`,  `/usr/local`, and  `bind-dirs`) | nothing

(1) Upon creation  
(2) Following shutdown  
(3) Includes  [disposable templates](https://www.qubes-os.org/doc/glossary/#disposable-template)


## Tools for VPN
- VPN Reconnector (Flower-based?)
- Terminal frontend 
	- list servers
	- update server/reconnect

## Audio VM

https://forum.qubes-os.org/t/debian-minimal-template-for-sys-audio/11108/47

-   Detach the audio device:

> `qvm-pci detach sys-audio dom0:00_1f.3`