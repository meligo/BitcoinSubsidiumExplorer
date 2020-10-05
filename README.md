Iquidus Explorer - 1.7.4
================

An open source block explorer written in node.js.

### See it in action

*  [List of live explorers running Iquidus](https://github.com/iquidus/explorer/wiki/Live-Explorers)


*Note: If you would like your instance mentioned here contact me*

### Requires

*  node.js >= 8.17.0 (12.14.0 is advised for updated dependencies)
*  mongodb 4.2.x
*  *coind

### Create database

Enter MongoDB cli:

    $ mongo

Create databse:

    > use explorerdb

Create user with read/write access:

    > db.createUser( { user: "iquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )

*Note: If you're using mongo shell 4.2.x, use the following to create your user:

    > db.addUser( { user: "username", pwd: "password", roles: [ "readWrite"] })

### Get the source

    git clone https://github.com/iquidus/explorer explorer

### Install node modules

    cd explorer && npm install --production

### Configure

    cp ./settings.json.template ./settings.json

*Make required changes in settings.json*

### Start Explorer

    npm start

*Note: mongod must be running to start the explorer*

As of version 1.4.0 the explorer defaults to cluster mode, forking an instance of its process to each cpu core. This results in increased performance and stability. Load balancing gets automatically taken care of and any instances that for some reason die, will be restarted automatically. For testing/development (or if you just wish to) a single instance can be launched with

    node --stack-size=10000 bin/instance

To stop the cluster you can use

    npm stop

### Syncing databases with the blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorers root directory.

    Usage: node scripts/sync.js [database] [mode]

    database: (required)
    index [mode] Main index: coin info/stats, transactions & addresses
    market       Market data: summaries, orderbooks, trade history & chartdata

    mode: (required for index database only)
    update       Updates index from last sync to current block
    check        checks index for (and adds) any missing transactions/addresses
    reindex      Clears index then resyncs from genesis to current block

    notes:
    * 'current block' is the latest created block when script is executed.
    * The market database only supports (& defaults to) reindex mode.
    * If check mode finds missing data(ignoring new data since last sync),
      index_timeout in settings.json is set too low.


*It is recommended to have this script launched via a cronjob at 1+ min intervals.*

**crontab**

*Example crontab; update index every minute and market data every 2 minutes*

    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1
    */5 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1

### Wallet

Iquidus Explorer is intended to be generic, so it can be used with any wallet following the usual standards. The wallet must be running with atleast the following flags

    -daemon -txindex
    
### Security

Ensure mongodb is not exposed to the outside world via your mongo config or a firewall to prevent outside tampering of the indexed chain data. 

### Known Issues

**script is already running.**

If you receive this message when launching the sync script either a) a sync is currently in progress, or b) a previous sync was killed before it completed. If you are certian a sync is not in progress remove the index.pid and db_index.pid from the tmp folder in the explorer root directory.

    rm tmp/index.pid
    rm tmp/db_index.pid

**exceeding stack size**

    RangeError: Maximum call stack size exceeded

Nodes default stack size may be too small to index addresses with many tx's. If you experience the above error while running sync.js the stack size needs to be increased.

To determine the default setting run

    node --v8-options | grep -B0 -A1 stack_size

To run sync.js with a larger stack size launch with

    node --stack-size=[SIZE] scripts/sync.js index update

Where [SIZE] is an integer higher than the default.

*note: SIZE will depend on which blockchain you are using, you may need to play around a bit to find an optimal setting*

Found this guide https://gist.github.com/zeronug/5c66207c426a1d4d5c73cc872255c572 it's pretty good

    Node / Iquidus Explorer Setup for Dummies
    Pulse Crypto is used in this example.

    This Tutorial is going to create a Daemon (node) and install Explorer.
    THIS IS NOT GOING TO CREATE A GUI CLIENT.

    Follow the instructions in [whatever coin name] docs folder Unix build - some builds are different.

    I setup this up on both Ubuntu 15.10 and 16.04 with no issues.
    You can create an account on vultr and get $50 free to be used in 2 months.
    Non-refferal link:  https://www.vultr.com/freetrial/

    ******************************************
    *** Change Password / Update System
    ******************************************

    Change passwords
    # sudo passwd  //change root password

    Update Packages
    # sudo apt-get update
    # sudo apt-get upgrade -y

    Install git and nano
    # sudo apt-get install nano git -y

    ******************************************
    *** Installing Pulse Node
    ******************************************

    Get the Source Code

    Basic packages to build the coin
    # sudo apt-get install build-essential libssl-dev libdb++-dev libboost-all-dev libqrencode-dev miniupnpc libminiupnpc-dev autoconf pkg-config libtool autotools-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev automake -y

    Create a Swap File - Low Ram Servers
    # sudo fallocate -l 4G /swapfile
    # sudo chmod 600 /swapfile
    # sudo mkswap /swapfile
    # sudo swapon /swapfile
    # sudo nano /etc/fstab

    Add to fstab file
    /swapfile   none    swap    sw    0   0

    Adding the Coin
    # sudo git clone https://github.com/elcrypto/Pulse.git

    Change Directories to src/
    # cd Pulse/src/ 

    Compile the code with flags
    # sudo make -f makefile.unix USE_UPNP=1
    # sudo strip pulsed

    Stat the file with daemon flag so that you can create a config file
    # sudo ./pulsed -daemon

    Create a conf file in .pulse/pulse.conf
    # cd
    # sudo nano .pulse/pulse.conf

    rpcuser=pulserpc
    rpcpassword=Gnfh67gdmRTGvA5YB3UZhV6e6cPTTeneTTdnosLLD3cU
    listen=1
    maxconnections=500

    *make sure to set the file to read only.
    # sudo chmod 400 .pulse/pulse.conf
    # cd Pulse/src
    # sudo ./pulsed -daemon -txindex

    Check it daemon has started.  If it returns information, the daemon is working!
    # sudo ./pulsed getinfo

    Stopping the daemon
    # sudo ./pulsed stop

    ******************************************
    *** Setting up the Explorer
    *** https://github.com/iquidus/explorer
    ******************************************

    Install MongoDB Community Edition
    https://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/

    # sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
    # echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    # sudo apt-get update
    # sudo apt-get install -y mongodb-org

    # Reboot your system
    # sudo reboot
    # sudo service mongod start

    Running MongoDB - reference
    # sudo service mongod start
    # sudo service mongod stop
    # sudo service mongod restart

    Installing Nodejs
    # sudo apt-get update
    # sudo apt-get install nodejs nodejs-legacy -y
    # sudo apt-get install npm -y

    Creating a MongoDB Database
    # sudo mongo
    > use explorerdb
    > db.createUser( { user: "3er22wee3", pwd: "4y2#1iuu34hbbw2", roles: [ "readWrite" ] } )
    > exit

    Installing the Explorer
    # cd /home/
    # git clone https://github.com/iquidus/explorer explorer
    # cd explorer && npm install --production
    # cp ./settings.json.template ./settings.json

    Modify the Settings File
    # sudo nano settings.json

    See if it's working
    # npm start

    Update the databases
    # sudo node scripts/sync.js index update 

    Install Forever to keep the js running
    # sudo npm install forever -g
    # sudo npm install forever-monitor

    Start the Explorer
    # forever start bin/cluster

    ******************************************
    *** Installing Cron
    *** https://help.ubuntu.com/community/CronHowto
    ******************************************

    # sudo apt-get install gnome-schedule -y

    Editing Cron
    # sudo crontab -e

    Add Cron to File to update the explorer
    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1

    Add Cron to make sure the Pulsed runs (NOT WORKING)
    If anyone has a good reboot for this, please post in the comments.
    @reboot cd /root/Pulse/src ./pulsed -daemon -txindex
    @reboot cd /root/explorer forever start bin/cluster e


### License

Copyright (c) 2015, Iquidus Technology  
Copyright (c) 2015, Luke Williams  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Iquidus Technology nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
