
## Instance Specs
```
OS: Ubuntu 18-04 LTS
Disk size: 200Gb
RAM: 8Gb
CPU: 2Cores
```

Offical Docs: https://xrpl.org/install-rippled-on-ubuntu.html#main_content_body

## 1: install utils + update system
```
$ sudo apt-get update -y
$ sudo apt-get upgrade -y
$ sudo apt -y install apt-transport-https ca-certificates wget gnupg
```

## 2: Add Ripple's package-signing GPG key to your list of trusted keys:
```
$ wget -q -O - "https://repos.ripple.com/repos/api/gpg/key/public" | \
  sudo apt-key add -
```

## 3: Check the fingerprint of the newly-added key:
```
$ apt-key finger

The output should include an entry for Ripple such as the following:

pub   rsa3072 2019-02-14 [SC] [expires: 2021-02-13]
      C001 0EC2 05B3 5A33 10DC 90DE 395F 97FF CCAF D9A2
uid           [ unknown] TechOps Team at Ripple <techops+rippled@ripple.com>
sub   rsa3072 2019-02-14 [E] [expires: 2021-02-13]
In particular, make sure that the fingerprint matches. (In the above example, the fingerprint is on the second line, starting with C001.)
```

## 4: Add the appropriate Ripple repository for your operating system version:
```
$ echo "deb https://repos.ripple.com/repos/rippled-deb bionic stable" | \
    sudo tee -a /etc/apt/sources.list.d/ripple.list
```
    
## 5: Fetch the Ripple repository.
```
$ sudo apt -y update
```


## 6: Install the rippled software package:
```
$ sudo apt -y install rippled
```

## 7: Check the status of the rippled service:
```
$ systemctl status rippled.service
$ sudo systemctl start rippled.service
$ sudo systemctl enable rippled.service
```

## 8: Check the rippled status:
#### Service status:
```
$ sudo systemctl status rippled
===============
● rippled.service - Ripple Daemon
   Loaded: loaded (/usr/lib/systemd/system/rippled.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/rippled.service.d
           └─nofile_limit.conf
   Active: active (running) since Sun 2018-12-16 20:58:51 UTC; 6min ago
 Main PID: 909 (rippled: #1)
    Tasks: 23 (limit: 4704)
   CGroup: /system.slice/rippled.service
           ├─ 909 /opt/ripple/bin/rippled --net --silent --conf /etc/opt/ripple/rippled.cfg
           └─1155 /opt/ripple/bin/rippled --net --silent --conf /etc/opt/ripple/rippled.cfg

```
#### Service ports:
```
$ sudo netstat -ntpl
===============
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:6006          0.0.0.0:*               LISTEN      1155/rippled        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1003/sshd           
tcp        0      0 0.0.0.0:51235           0.0.0.0:*               LISTEN      1155/rippled        
tcp        0      0 127.0.0.1:5005          0.0.0.0:*               LISTEN      1155/rippled        
tcp6       0      0 :::22                   :::*                    LISTEN      1003/sshd     

```
