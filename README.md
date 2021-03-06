# Civitas
Building two Civitas Master Nodes on one server.                                                    
If you only want one Masternode on your server follow this guide: [Here](https://github.com/tokayseo/Civitas/blob/master/README.md)

## Preperation

1. Get a VPS from a provider like DigitalOcean, Vultr, Linode, etc. (only tested on Vultr) 
   - Recommended VPS size at least 1gb RAM 
   - **It must be Ubuntu 16.04 (Xenial)**
   - It must have two IP addresses. (make sure to enable IPv6 when setting up the droplet)
2. Make sure you have a transaction of exactly 10000 CIV in your desktop cold wallet.

## Cold Wallet Setup Part 1

1. Open your wallet on your desktop.

   Click Settings -> Options -> Wallet
   
   Check "Enable coin control features"
   
   Check "Show Masternodes Tab"
   
   Press Ok (you will need to restart the wallet)
   
   ![Alt text](https://github.com/tokayseo/Civitas/blob/master/civitas.PNG "Wallet Settings")

   
   
   
2. Go to the "Tools" -> "Debug console"
3. Run the following command: `masternode genkey`
4. You should see a long key that looks like:
```
3HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg
```
5. This is your `private key`, keep it safe, do not share with anyone.




## VPS Setup

1. Log into your VPS
   - Windows users [follow this guide](https://www.digitalocean.com/community/tutorials/how-to-log-into-your-droplet-with-putty-for-windows-users) to log into your VPS.
2. Copy/paste these commands into the VPS and hit enter: (One Box At A Time)
```
apt-get -y update
```
```
apt-get -y upgrade
```
```
cd /var
touch swap.img
chmod 600 swap.img
dd if=/dev/zero of=/var/swap.img bs=1024k count=2000
mkswap /var/swap.img
swapon /var/swap.img
echo "/var/swap.img none swap sw 0 0" >> /etc/fstab
```
```
cd
```
```
apt-get -y install software-properties-common
```
```
apt-add-repository -y ppa:bitcoin/bitcoin
```
```
apt-get -y update
```
```
apt-get -y install \
    wget \
    git \
    unzip \
    libevent-dev \
    libboost-dev \
    libboost-chrono-dev \
    libboost-filesystem-dev \
    libboost-program-options-dev \
    libboost-system-dev \
    libboost-test-dev \
    libboost-thread-dev \
    libminiupnpc-dev \
    build-essential \
    libtool \
    autotools-dev \
    automake \
    pkg-config \
    libssl-dev \
    libevent-dev \
    bsdmainutils \
    libzmq3-dev \
    nano
```
```
apt-get -y update
```
```
apt-get -y upgrade
```
```
apt-get -y install libdb4.8-dev
```
```
apt-get -y install libdb4.8++-dev
```
```
wget https://github.com/eastcoastcrypto/Civitas/releases/download/v1.2.2/civitas-1.2.2.tar.gz
```
```
tar -xzf civitas-1.2.2.tar.gz
```
```
rm civitas-1.2.2.tar.gz
```
```
cd civitas-1.2.2
```
```
find . -name "*.sh" -exec sudo chmod 755 {} \;
```
```
./autogen.sh
```
```
./configure --without-gui
```
```
make
```
```
make install
```
```
cd src
```
```
strip civitasd
strip civitas-cli
strip civitas-tx
```
```
cp civitasd /usr/bin
cp civitas-cli /usr/bin
cp civitas-tx /usr/bin
```
```
cd
```
Take a snapshot if you want at this point (can be used on a second droplet)

Add the first masternode user and set the password for that user
```
useradd -m -s /bin/bash civitasmn01
```
```
passwd civitasmn01
```
```
mkdir -p .civitas
```
Reboot the VPS machine and log in as civitasmn01
```
civitasd
```
Copy rpcpassword to use in the conf file
```
nano /home/civitasmn01/.civitas/civitas.conf
```
Replace:
1. rpcpassword=WITH_THE_ONE_YOU_COPIED
2. externalip=VPS_IP_ADDRESS
3. masternodeprivkey=WALLET_GENKEY

With your info!
```
rpcuser=civitasrpc
rpcpassword=passhf95uiygr5308h08r3h0249fbgh7389h973
rpcallowip=127.0.0.1
rpcport=18900
listen=1
server=1
logtimestamps=1
maxconnections=256
daemon=1
bind=VPS_IP_ADDRESS
externalip=VPS_IP_ADDRESS
masternodeaddr=VPS_IP_ADDRESS:18843
masternodeprivkey=WALLET_GENKEY
masternode=1
```
CTRL X to save it. Y for yes, then ENTER.
```
civitasd
```
Wait for the blockchain to sync (check with "watch civitas-cli getinfo").

While waiting for the blockchain to sync, go down to "Cold Wallet Setup Part 2" and complete that before proceeding.

Open the Debug Console in your cold wallet.
```
startmasternode alias false mnXX(mn01)
````
Go back to your terminal and run:
```
civitas-cli masternode status
```
This should return : "Masternode successfully started" if everything worked (might take a couple of minutes)

## Setting up second user/masternode

Log in as root
Create a second user and add a password:
```
useradd -m -s /bin/bash civitasmn02
```
```
passwd civitasmn02
```
```
cp -R /home/civitasmn01/.civitas /home/civitasmn02/.civitas
```
```
chown -R civitasmn02:civitasmn02 /home/civitasmn02
```
Exit, then log in as civitasmn02
```
nano /home/civitasmn02/.civitas/civitas.conf
```
Replace:                                                                                                                               
1. rpcpassword=MAKE_SOME_CHANGES_TO_THE_CURRENT_ONE
2. externalip=VPS_IP_ADDRESS
3. masternodeprivkey=WALLET_GENKEY

With your info!
```
rpcuser=civitasrpc
rpcpassword=passhf95uiygr5308h08r3h0249fbgh7389h973
rpcallowip=127.0.0.1
rpcport=18901
listen=1
server=1
logtimestamps=1
maxconnections=256
daemon=1
bind=[VPS_IP_ADDRESS]
externalip=[VPS_IP_ADDRESS]
masternodeaddr=[VPS_IP_ADDRESS]:18843
masternodeprivkey=WALLET_GENKEY
masternode=1
```
Remember to use a second IP for this config file, and make sure to use [ ] around your IP if you are using IPv6.                          

CTRL X to save it. Y for yes, then ENTER.
```
civitasd
```
Wait for the blockchain to sync (check with "civitas-cli getinfo").

While waiting for the blockchain to sync, go down to "Cold Wallet Setup Part 2" and complete that before proceeding.

Open the Debug Console in your cold wallet.
```
startmasternode alias false mnXX(mn02)
````
Go back to your terminal and run:
```
civitas-cli masternode status
```
This should return : "Masternode successfully started" if everything worked (might take a couple of minutes)


## Cold Wallet Setup Part 2 

1. On your local machine open your `masternode.conf` file.
   Depending on your operating system you will find it in:
   * Windows: `%APPDATA%\Civitas\`
   * Mac OS: `~/Library/Application Support/Civitas/`
   * Unix/Linux: `~/.civitas/`
   
   Leave the file open
2. Go to "Tools" -> "Debug console"
3. Run the following command: `masternode outputs`
4. You should see output like the following if you have a transaction with exactly 10000 Civitas:
```
{
    "12345678xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx": "0"
}
```
5. The value on the left is your `txid` and the right is the `vout`
6. Add a line to the bottom of the already opened `masternode.conf` file using the IP of your
VPS (with port 18843), `private key`, `txid` and `vout`:
```
mn01 1.2.3.4:18843 3xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 12345678xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 0 
```
7. Save the file, exit your wallet and reopen your wallet.
8. Go to the "Masternodes" tab
9. Click "Start All"

## Usefull commands to know
* civitas-cli stop (Stops the masternode)
* civitas-cli getinfo (Returns info about version, connections, blocks, etc)
* civitas-cli masternode status (Prints Masternode status)
* civitasd -resync (forces the blockchain to resync, use Civitas-cli stop first)

## Credit
[tokayseo](https://github.com/tokayseo)/Casey for making the original [guide](https://github.com/tokayseo/Civitas/blob/master/README.md)
