#This document describes installing and running redis on Azure VM.

Create the below VM instance in Azure

Expected configuration is
```
1 B2S (2 vCPU(s), 4 GB RAM)
Linux – Ubuntu; 
0 managed OS disks – S4, ?? 
100 transaction units ??

```

Current configuration is
```
VM details:
Size Standard B1ms
vCPUs 1
RAM 2 GiB
```
After successful creation of VM, Install latest version of ubuntu on the VM.
After successful installation of ubuntu, connect to the VM from a remote desktop using cmd or terminal or gitbash.
```
ssh <username>@<instance name/IP>
```
After successful login into the VM, check the ubuntu version.
Check the ubuntu version: 
```
sudo lsb_release -a
```

Output:
```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.5 LTS
Release:        18.04
Codename:       bionic
```

Update the ubuntu:
``` 
sudo apt-get update
sudo apt-get upgrade
```

#Redis Installation

Reference: https://linuxize.com/post/how-to-install-and-configure-redis-on-ubuntu-18-04/

Connect to the VM from a remote desktop using cmd, treminal or gitbash and install redis.

```
sudo apt install redis-server
```
After successful installation redis, please check the satus of redis server.  
```
sudo systemctl status redis-server
```
Output:
```
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor pre
   Active: active (running) since Sat 2020-08-29 07:00:12 UTC; 18s ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 2677 ExecStop=/bin/kill -s TERM $MAINPID (code=exited, status=0/SUCCE
  Process: 26144 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exi
 Main PID: 26162 (redis-server)
    Tasks: 4 (limit: 2232)
   CGroup: /system.slice/redis-server.service
           └─26162 /usr/bin/redis-server 127.0.0.1:6379

Aug 29 07:00:12 vm1 systemd[1]: Starting Advanced key-value store...
Aug 29 07:00:12 vm1 systemd[1]: redis-server.service: Can't open PID file /var/r
Aug 29 07:00:12 vm1 systemd[1]: Started Advanced key-value store.
```
Stop the redis.
```
sudo systemctl stop redis-server
```
After stopping the redis server the status will be,
```
sudo systemctl status redis-server
```
Output:
```
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor pre
   Active: inactive (dead) since Fri 2020-08-28 19:00:31 UTC; 11h ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 2677 ExecStop=/bin/kill -s TERM $MAINPID (code=exited, status=0/SUCCE
 Main PID: 2273 (code=exited, status=0/SUCCESS)

Aug 28 18:54:53 vm1 systemd[1]: Starting Advanced key-value store...
Aug 28 18:54:53 vm1 systemd[1]: redis-server.service: Can't open PID file /var/r
Aug 28 18:54:53 vm1 systemd[1]: Started Advanced key-value store.
Aug 28 19:00:31 vm1 systemd[1]: Stopping Advanced key-value store...
Aug 28 19:00:31 vm1 systemd[1]: Stopped Advanced key-value store.
```

Configure remote access for redis-server. Open the redis.config and change the configuration to allow access from internet **0.0.0.0**.
```
sudo nano /etc/redis/redis.config
```
Find and replace the bind propert **bind 127.0.0.1 ::1** with **bind 0.0.0.0 ::1** and save the changes.

Restart the redis sever. 
```
sudo systemctl restart redis-server
```
Check the status of the server.
```
sudo systemctl status redis-server
```
Output:
```
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor pre
   Active: active (running) since Sat 2020-08-29 07:17:38 UTC; 3min 6s ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 26706 ExecStop=/bin/kill -s TERM $MAINPID (code=exited, status=0/SUCC
  Process: 26710 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exi
 Main PID: 26722 (redis-server)
    Tasks: 4 (limit: 2232)
   CGroup: /system.slice/redis-server.service
           └─26722 /usr/bin/redis-server 0.0.0.0:6379

Aug 29 07:17:38 vm1 systemd[1]: Stopped Advanced key-value store.
Aug 29 07:17:38 vm1 systemd[1]: Starting Advanced key-value store...
Aug 29 07:17:38 vm1 systemd[1]: redis-server.service: Can't open PID file /var/r
Aug 29 07:17:38 vm1 systemd[1]: Started Advanced key-value store.
```
Also check if the redis server is listening on 6379 using the below command. 
```
ss -an | grep 6379
```
Output:
``` 
tcp   LISTEN      0       128   0.0.0.0:6379         0.0.0.0:*
tcp   LISTEN      0       128   [::1]:6379  
```
Connect and check the redis-server using redis-cli from the same VM instance. 
```
redis-cli ping
```
Output:
```
PONG
```
#Remote connection

Now try to connect to the redis-server from a remote client like a windows or linux machine. To do that download the redis-server from: https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100.
After successful installation and setting the redis-cli path in the environment variable try the below command.
  
Check connecting to remote redis server using: 
```
redis-cli -h <IP or Host Name> ping
```
-h is remote host

Output:
```
PONG
```
#Configure authetication on the redis server.

Open the redis.config.
``` 
sudo nano /etc/redis/redis.config
```
Find and replace **requirepass foobared** with **requirepass <new password>** in the redis.config file and save the changes.
 
Restart the redis sever. 
```
sudo systemctl restart redis-server
```
Check the status of the server: 
```
sudo systemctl status redis-server
```
Output:
```
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor pre
   Active: active (running) since Sun 2020-08-30 15:51:29 UTC; 6s ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 18733 ExecStop=/bin/kill -s TERM $MAINPID (code=exited, status=0/SUCC
  Process: 18915 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exi
 Main PID: 18934 (redis-server)
    Tasks: 4 (limit: 2232)
   CGroup: /system.slice/redis-server.service
           └─18934 /usr/bin/redis-server 0.0.0.0:6379

Aug 30 15:51:29 vm1 systemd[1]: Starting Advanced key-value store...
Aug 30 15:51:29 vm1 systemd[1]: redis-server.service: Can't open PID file /var/r
Aug 30 15:51:29 vm1 systemd[1]: Started Advanced key-value store.
```
Now try connecting from the remote client using the below command. 
```
redis-cli -h <IP or Host Name> ping
```
Output:
```
(error) NOAUTH Authentication required.
```
The ping request failed and the redis-server is expecting a password which means the requirepass configuration is working as expected, so provide the password in the ping request command.
```
redis-cli -h <IP or Host Name> -p 6379 -a <password> ping
```
-h is host

-p is port

-a is auth (not sure)

Output:
```
PONG
```


Install and start redis on **windows machine**
```
Download redis on windows from: https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100
```

- After successfully installing the redis server as service please open redis.windows-service.conf and search for requirepass and set the password.

```
requirepass <new password within double quotes>
```

- Note the # infront of requirepass is removed.


To be continued...

- Video on how to install and access redis on azure vm: https://www.youtube.com/watch?v=OPWYgy_ZvjA
- Download redis on windows from: https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100
- Redis on windows refrence: https://medium.com/@binary10111010/redis-cli-installation-on-windows-684fb6b6ac6b
- Configure password: https://stackoverflow.com/questions/7537905/redis-set-a-password-for-redis
