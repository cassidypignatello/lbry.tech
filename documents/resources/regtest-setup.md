---
title: Regtest Setup
description: Regtest is a parallel testing network for the LBRY blockchain. Learn how to use it in this resource article.
--- 

## Why Use Regtest

A regtest server provides for a way to instantly generate blocks so that transactions can be instantaneous, which ultimately means no waiting for confirmations from the blockchain. Also, it’s not a problem if you accidentally corrupt your wallet, since no real funds are lost! Delete the files and setup a new one.

## Setup

To begin setting up the network, there are a few things you need.

You'll need a Linux or a Mac distribution to run all this. A virtual machine is fine.
Note: These instructions specifically were tested on Ubuntu version 16.04.

### Virtual Environment

First up it's a good idea to create a Python virtual environment. This requires you to have a functional python2.7 setup, with the Python package manager `pip` installed. To create a new virtual environment in a folder `lbry-env`, run this:
`virtualenv -p /usr/bin/python2.7 lbry-env`
To enter the environment, run:
`source lbry-env/bin/activate`.

### lbrycrd

You need to download a build of `lbrycrd` from [here](https://github.com/lbryio/lbrycrd/releases/), no installation required. To configure `lbrycrd` you need to create a file at `~/.lbrycrd/lbrycrd.conf`,
containing the following:
```ini
rpcuser=test
rpcpassword=test
rpcport=18332
regtest=1
server=1
txindex=1
daemon=1
listen=0
discover=0
```

### lbryum-server

To install lbryum-server, you first need to install the package `leveldb`. After that, download the source from [here](https://github.com/lbryio/lbryum-server/releases), and run the following _not_ inside the environment:
```bash
cd lbryum-server
sudo pip2 install -r requirements.txt
```

If you're not running debian/\*buntu or a derivative of those, you need to edit the `configure` file a bit. In line 11, remove the `apt-get` line and manually install the required packages. In line 51, change `adduser` to `useradd` and on the same line, change `--disabled-password` to `-p !`.

```bash
sudo ./configure
sudo python2 setup.py install
```
The `sudo ./configure` command creates a new user in the system by the name "lbryum", which is the user through which we'll be running the server. lbryum-server also needs W/R access to `/var/lbryum-server`.
To do that run:
```bash
sudo chown -R lbryum /var/lbryum-server
```
When installed, append/use the following config options to the `/etc/lbryum.conf` file:
```ini
[lbrycrdd]
lbrycrdd_host = localhost
lbrycrdd_port = 18332
# user and password from lbrycrd.conf
lbrycrdd_user = test
lbrycrdd_password = test

[network]
type=lbrycrd_regtest
```

### lbryum

To install lbryum, first download the source from [here](https://github.com/lbryio/lbryum/releases). To install it, run the following inside the virtual environment:
```bash
cd lbryum
pip2 install -r requirements.txt
pip2 install -e .
```

After installation completes, you must set the config option for lbryum using:
```bash
lbryum setconfig default_servers '{ "localhost": { "t": "50001" }}'
lbryum setconfig chain 'lbrycrd_regtest'
```

Alternatively, you can create a file `touch ~/.lbryum/config` and paste the following config:
```json
{
  "chain": "lbrycrd_regtest",
  "default_servers": {
    "localhost": {
      "t": "50001"
    }
  }
}
```

### lbry

Download source from [here](https://github.com/lbryio/lbry-sdk/releases), and run the following inside the environment:
```bash
cd lbry
pip2 install -r requirements.txt
pip2 install -e .
mkdir ~/.lbrynet
touch ~/.lbrynet/daemon_settings.yml
```

Append the following in the newly created `~/.lbrynet/daemon_settings.yml` file:
```yml
blockchain_name: lbrycrd_regtest
lbryum_servers:
- localhost:50001
reflect_uploads: false
share_usage_data: false
use_upnp: false
```

### Last step
Go to the `lbryum` folder once again and run:
```bash
pip2 install -e .
```
This is to ensure that `lbrynet-daemon` uses the correct wallet.

## Firing up the regtest server

### Wallet backup

To start off, if you've already used LBRY on your machine, you need to backup the wallet by copying the folders `~/.lbrynet` and `~/.lbryum` and then deleting them to start from fresh. Run
`mkdir ~/.lbryum`

Now it should be all set-up. Just execute the commands in the following order, and the regtest server should be good to go.

### 1) lbrycrd

To run the `lbrycrd` daemon, run the following in the `lbrycrd` folder:
`./lbrycrdd`

To generate blocks, run `./lbrycrd-cli generate <num_of_blocks>`
You'll need to generate some blocks to get the network going. Start off by generating at least 100.
`./lbrycrd-cli generate 173`

If you'd prefer a more verbose output from lbrycrdd, run lbrycrd using:
`./lbrycrdd -printtoconsole`

### 2) lbryum-server

To run the server, run:
```bash
sudo runuser -l lbryum -c 'lbryum-server --conf=/etc/lbryum.conf'
```
Note: conf flag can be left out if the config is in the default directory(default: `/etc/lbryum.conf`)

### 3) lbryum

To run the lbryum, run:
```bash
lbryum daemon start
```

Generate some more blocks, get a wallet address by running:
`lbryum getunusedaddress`
and then send some credits to your wallet by doing
`./lbrycrd-cli sendtoaddress <address> <num_of_credits>`

### 4) lbry

You can now run `lbrynet-daemon`, and it should connect to the `lbryum`. Now you can use the regtest stack as you would normally use lbryum.

## Shutdown

To stop the network, run `lbrynet-cli daemon_stop`, `lbryum daemon stop`, and kill the `lbryum-server` process and stop lbrycrd by `lbrycrdd-cli stop`. If you want to use your wallet and the official servers again, backup the new regtest wallet, and replace it with your own.

## Note 1
You need to generate a few blocks every time you make a new transaction in the form of send, receive, claim, update, publish, support, tip, etc. for it to show up in the daemon and lbryum, etc.

## Note 2
If something goes wrong and you get a "Block not found" error, remember to delete `/var/lbryum-server` before trying again.

## Cheatsheet

#### Required processes in the correct order
```bash
lbrycrdd

sudo runuser -l lbryum -c 'lbryum-server --conf=/etc/lbryum.conf'

lbryum daemon start

lbrynet-daemon
```

#### Generate blocks
```bash
lbrycrd-cli generate 5
```

#### Get a wallet address
```bash
lbryum getunsusedaddress
```

#### Send credits from lbrycrd to your wallet
```bash
lbrycrd-cli sendtoaddress <address> <num_of_credits>
```
