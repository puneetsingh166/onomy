# How to run a Onomy testnet full node

A Onomy chain full node is just like any other Cosmos chain full node and unlike the validator flow requires no external software

## What do I need?

A Linux server with any modern Linux distribution, 2gb of ram and at least 20gb storage. Requirements are very minimal.

### Download/install Onomy chain binaries
To download binary follow these commands
```
cd $HOME
mkdir binaries
cd binaries
wget https://github.com/onomyprotocol/onomy/releases/download/v0.0.1/onomyd
wget https://github.com/onomyprotocol/onomy/releases/download/v0.0.1/gbt
wget https://github.com/onomyprotocol/onomy/releases/download/v0.0.1/geth
cd ..
chmod -R +x binaries
export PATH=$PATH:$HOME/binaries/
```
or If you have Fedora (Fedora 34) or Redhat (Red Hat Enterprise Linux 8.4 (Ootpa))
and you want to make binaries yourself, then follow these steps
```
sudo yum install -y git
git clone -b dev https://github.com/onomyprotocol/onomy.git
cd onomy/deploy/testnet
bash bin.sh
```
The second way may be unsafe because it used the latest version of the artifacts.

### Init the config files

```
cd $HOME
onomyd --home $HOME/onomy-testnet1/onomy init {validator moniker} --chain-id onomy-testnet1
```

### Copy the genesis file

```
rm $HOME/onomy-testnet1/onomy/config/genesis.json
wget http://147.182.190.16:26657/genesis? -O $HOME/raw.json
jq .result.genesis $HOME/raw.json >> $HOME/onomy-testnet1/onomy/config/genesis.json
rm -rf $HOME/raw.json
```

### Update toml configuration files

Change in $HOME/onomy-testnet1/onomy/config/config.toml to contain the following:

```

replace seeds = "" to seeds = "5e0f5b9d54d3e038623ddb77c0b91b559ff13495@147.182.190.16:26656"
replace "tcp://127.0.0.1:26657" to "tcp://0.0.0.0:26657"
replace "tcp://127.0.0.1:26656" to "tcp://0.0.0.0:26656"
replace addr_book_strict = true to addr_book_strict = false
replace external_address = "" to external_address = "tcp://0.0.0.0:26656"
```

Change in $HOME/onomy-testnet1/onomy/config/app.toml to contain the following:

```
replace enable = false to enable = true
replace swagger = false to swagger = true
```
## Increasing the default open files limit
If we don't raise this value nodes will crash once the network grows large enough
```
sudo su -c "echo 'fs.file-max = 65536' >> /etc/sysctl.conf"
sysctl -p

sudo su -c "echo '* hard nofile 94000' >> /etc/security/limits.conf"
sudo su -c "echo '* soft nofile 94000' >> /etc/security/limits.conf"

sudo su -c "echo 'session required pam_limits.so' >> /etc/pam.d/common-session"
```
For this to take effect you'll need to (A) reboot (B) close and re-open all ssh sessions.
To check if this has worked run
```
ulimit -n
```
If you see 1024 then you need to reboot
### Start your full node in another terminal and wait for it to sync

```
onomyd --home $HOME/onomy-testnet1/onomy start
```
### Check the status of the Onomy chain

You should be good to go! You can check the status of the three
Onomy chain by running.
```
curl http://localhost:26657/status
```
if catching_up is false means your node is fully synced