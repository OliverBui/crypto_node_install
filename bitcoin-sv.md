## Instance Specs
```
OS: Ubuntu 18-04 LTS
Disk size: 50Gb
RAM: 8Gb
CPU: 2Cores
```
## Install utils + update system

```
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get install git htop nload curl wget zip unzip nano cron build-essential -y
```

## Install dependent libs

```
sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev software-properties-common python3-pip
```

## Install bitcoin-sv
```
Update here
```

## Create bitcoind.conf content
```
nano bitcoind.conf
rpcuser=bitcoinsv //Add rpc user here
rpcpassword=bitcoinsv_testnetbitcoinsv_testnet //Add rpc pass here
testnet=1
rpcport=8332
rpcallowip=0.0.0.0/0
server=1

# this is for zmq notification
zmqpubrawtx=tcp://127.0.0.1:28332
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubhashtx=tcp://127.0.0.1:28332
zmqpubhashblock=tcp://127.0.0.1:28332
```

## Install supervisor
```
sudo apt install supervisor -y
```

## Create bitcoind supervisor
```
sudo nano /etc/supervisor/conf.d/bitcoind.conf
```

## Create bitcoind.conf supervisor content
```
[program:bitcoind]
command=/usr/bin/bitcoind -maxconnections=500 -conf=/home/ubuntu/bitcoind.conf -datadir=/home/ubuntu/data/
user=ubuntu
autostart=true
autorestart=true
stderr_logfile=/var/log/bitcoind.err.log
stdout_logfile=/var/log/bitcoind.out.log
```

## Update new bitcoind supervisor config file
```
sudo supervisorctl reread;sudo supervisorctl update; sudo supervisorctl restart all
```

## Install zmq lib
```
pip3 install zmq
```

## Create zmq_sub.py
```
nano /home/michaelphan/.bitcoin/zmq_sub.py
```

## Create zmq supervisor
```
[program:zmq]
command=python3 /home/ubuntu/zmq_sub.py
user=ubuntu
autostart=true
autorestart=true
stderr_logfile=/var/log/zmq.err.log
stdout_logfile=/var/log/zmq.out.log
```
