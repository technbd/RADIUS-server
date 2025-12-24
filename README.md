## RADIUS Server:

A RADIUS server (**Remote Authentication Dial-In User Service**) is a centralized authentication, authorization, and accounting (AAA) system widely used in network access control. It provides AAA services for a variety of networking protocols, including PPP, VPN, and WiFi. FreeRADIUS is the de-facto RADIUS server for Linux and other Unix-like systems. 

**FreeRADIUS** is an open source, high-performance, scalable, modular and feature-rich RADIUS server. FreeRADIUS has support for request proxy, fail-over and load balancing, as well as access to various database backends.


### RADIUS Server Does (AAA):
- Authentication:
    - Verifies who the user is
    - Username & password
    - Certificates
    - One-time passwords (OTP)
    - Two-factor authentication (2FA)

- Authorization:
    - Determines what the user is allowed to do
    - VLAN assignment
    - IP address allocation
    - Bandwidth limits
    - Access policies

- Accounting:
    - Tracks what the user did
    - Login/logout time
    - Session duration
    - Data usage
    - Connection history



### Where RADIUS is Used:

RADIUS is commonly used in:
- Wi-Fi authentication (WPA2/WPA3-Enterprise)
- VPN access (IPSec, OpenVPN, SSL VPN)
- Network devices login: Routers, Switches, Firewalls
- ISP user management
- Enterprise identity systems


### RADIUS Components: 
- RADIUS Server:
    - Performs authentication & policy decisions

- RADIUS Client:
    - Network device (AP, switch, firewall, VPN)
    - Forwards user credentials

- Backend Identity Source:
    - Local database
    - LDAP
    - Active Directory
    - SQL
    - MFA provider


### Common RADIUS Ports:

| Purpose        | Port        | Protocol |
| -------------- | ----------- | -------- |
| Authentication | 1812        | UDP      |
| Accounting     | 1813        | UDP      |
| Older standard | 1645 / 1646 | UDP      |




### Environment:
- RADIUS Server IP: 192.168.10.193
- RADIUS Client IP: 192.168.10.192




---
---





## Installing FreeRADIUS: 


```
dnf module list -y freeradius
```


```
dnf install -y freeradius freeradius-utils

Or,

apt install -y freeradius
```



```
radiusd -v


radiusd: FreeRADIUS Version 3.0.20, for host x86_64-redhat-linux-gnu
FreeRADIUS Version 3.0.20
Copyright (C) 1999-2019 The FreeRADIUS server project and contributors
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE
You may redistribute copies of FreeRADIUS under the terms of the
GNU General Public License
For more information about these matters, see the file named COPYRIGHT
```



_Directory Structure:_
```
ll /etc/raddb/

drwxrwx--- 2 root radiusd  4096 Dec 24 21:11 certs
-rw-r----- 1 root radiusd  8550 Dec 24 21:22 clients.conf
-rw-r--r-- 1 root radiusd  1440 Jul 31  2024 dictionary
lrwxrwxrwx 1 root radiusd    30 Jul 31  2024 hints -> ./mods-config/preprocess/hints
lrwxrwxrwx 1 root radiusd    35 Jul 31  2024 huntgroups -> ./mods-config/preprocess/huntgroups
drwxr-x--- 2 root radiusd  4096 Dec 24 21:11 mods-available
drwxr-x--- 6 root radiusd    85 Dec 24 21:11 mods-config
drwxr-x--- 2 root radiusd  4096 Dec 24 21:11 mods-enabled
-rw-r----- 1 root radiusd    52 Jul 31  2024 panic.gdb
drwxr-x--- 2 root radiusd   160 Dec 24 21:11 policy.d
-rw-r----- 1 root radiusd 28654 Jul 31  2024 proxy.conf
-rw-r----- 1 root radiusd 38638 Dec 24 21:14 radiusd.conf
-rw-r----- 1 root radiusd 20807 Jul 31  2024 README.rst
drwxr-x--- 2 root radiusd  4096 Dec 24 21:11 sites-available
drwxr-x--- 2 root radiusd    41 Dec 24 21:11 sites-enabled
-rw-r----- 1 root radiusd  3470 Jul 31  2024 templates.conf
-rw-r----- 1 root radiusd  8536 Jul 31  2024 trigger.conf
lrwxrwxrwx 1 root radiusd    29 Jul 31  2024 users -> ./mods-config/files/authorize
```





### Configure FreeRADIUS:


#### Configure FreeRADIUS Server:

_After installation, you need to **configure the server** according to your requirements:_
```
vim /etc/raddb/radiusd.conf


```



#### Configure RADIUS Client (NAS): 

_Next, edit the file on `clients.conf`:_
- `ipaddr` = AP / Firewall / Switch IP
- `secret` = must match on the client device

```
vim /etc/raddb/clients.conf


### Syntax:
#client any_name {
#    ipaddr = <client IP or hostname> 
#    secret = <shared_secret>
#}


### Add your network device:
client FortiGate-VM01 {
       ipaddr = 192.168.10.11
       secret = testing123
}

client RT01 {
       ipaddr = 192.168.10.12
       secret = testing123
}

client linux_vm01 {
       ipaddr = 192.168.10.192
       secret = centos
}
```



#### Create Test User (Local Auth): 

- Add users to the RADIUS server. 

```
vim /etc/raddb/users


### Syntax:
#<username> Cleartext-Password := "<password>"

user1 Cleartext-Password := "testpass1"
user2 Cleartext-Password := "testpass2"
user3 Cleartext-Password := "testpass3"
```



```
systemctl enable radiusd
systemctl start radiusd
systemctl status radiusd
```




### Test RADIUS Locally:

_Stop the `radiusd` service:_
```
systemctl stop radiusd
```



_To run it in debug mode:_
```
radiusd -X


### Output:

Listening on auth address * port 1812 bound to server default
Listening on acct address * port 1813 bound to server default
Listening on auth address :: port 1812 bound to server default
Listening on acct address :: port 1813 bound to server default
Listening on auth address 127.0.0.1 port 18120 bound to server inner-tunnel
Listening on proxy address * port 53137
Listening on proxy address :: port 53512
Ready to process requests
```



#### Check from another terminal or Node:

- Expected output: `Access-Accept`


_On client node:_
```
dnf install -y freeradius-utils
```


_Syntax:_
```
radtest  <user>  <password>  <Radius_server_IP>  <NAS_port>  <Device_password>
```



```
radtest user1 testpass1 127.0.0.1 0 testing123

radtest user3 testpass3 192.168.10.193 0 centos


### Output:
Sent Access-Request Id 100 from 0.0.0.0:51690 to 127.0.0.1:1812 length 79
        User-Name = "testuser1"
        User-Password = "testpass1"
        NAS-IP-Address = 192.168.10.193
        NAS-Port = 0
        Cleartext-Password = "testpass1"
Received Access-Accept Id 100 from 127.0.0.1:1812 to 127.0.0.1:51690 length 38
        Message-Authenticator = 0x7ec8aac910d2862ef750a6b1b5349763
```







