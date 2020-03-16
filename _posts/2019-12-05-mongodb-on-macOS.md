---
layout: post
title:  "mongoDB on macOS"
sub_title: "Mongo meet Mac, Mac meet Mongo"
date:   2019-12-05 12:00:00 -0600
categories: mongodb macOS MacPro
---

This is the first of several posts where I walk through installing MongoDB on different substrates.  This post looks at installing and configuring on macOS without Homebrew which entails the following high-level steps.

1. [Create Host Directories](#host-directories)
1. [Create _mongod Service Account](#create-service-account)
1. [Download MongoDB and Configure](#download-and-configure)
1. [Create mongod.conf](#create-mongod-conf-file)
1. [Configure MONGODB_HOME and PATH](#configure-mongodb_home-and-path)
1. [Set Permissions](#set-permissions)
1. [Initial Startup](#initial-startup)
1. [Configure Admin User](#configure-admin-user)
1. [Startup with Auth](#startup-with-auth)
1. [Connect with Mongo Shell](#connect-with-mongo-shell)
1. [Create macOS Launch Daemon](#create-macos-launch-daemon)
1. [mongoDB Lifecycle Management](#mongodb-lifecycle-management)
1. [Uninstall all of this](#uninstall-all-of-this)

Let's get cracking...

### Host Directories

Create directories on the macOS host for MongoDB Server bits, Config, Log and Data.

```bash
# create directory for MongoDB Server bits
sudo mkdir -p /usr/local/mongodb
# create directory for MongoDB config
sudo mkdir -p /etc/mongodb/conf
# create directory for MongoDB log
sudo mkdir -p /var/log/mongodb
# create directory for MongoDB data
sudo mkdir -p /Volumes/ssdraid/mongodb-data0
```

### Create Service Account

Create a Service Account that owns operating MongoDB as we don't want to do this with our osx User account.  First we create a group for our Service Account by running the osx [Directory Service command](https://ss64.com/osx/dscl.html).  **Note** that macOS Service Accounts (i.e. daemons) are prefixed with an underscore ``_``.

List existing groups and pick a groupId that's NOT in this list

```bash
# list existing groups and groupIds
dscl . list /Groups PrimaryGroupID | sort -n -t ' ' -k2
```

Create the ``_mongod`` group with groupId 400 (_or whatever you choose_), **Note** macOS will prompt you to authorize these actions.

```bash
sudo dscl . -create /Groups/_mongod
sudo dscl . -create /Groups/_mongod PrimaryGroupID 400
sudo dscl . -create /Users/_mongod UniqueID 400
sudo dscl . -create /Users/_mongod PrimaryGroupID 400
sudo dscl . -create /Users/_mongod UserShell /usr/bin/false
```

### Download and Configure

Download, unpack and install MongoDB into ``/usr/local/mongodb`` then create a ``latest`` symlink which we'll configure as ``MONGODB_HOME`` a bit later.

```bash
# download from mongodb cdn
curl -O https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-4.2.1.tgz
# unpack into /usr/local/mongodb
sudo tar -xzvf mongodb-macos-x86_64-4.2.1.tgz --directory /usr/local/mongodb
# create latest symlink
sudo ln -s /usr/local/mongodb/mongodb-macos-x86_64-4.2.1 /usr/local/mongodb/latest
```

### Create Mongod Conf file

Configure MongoDB properties in ``mongod.conf`` and copy to ``/etc/mongodb/conf`` directory.  See [MongoDB docs](http://docs.mongodb.org/manual/reference/configuration-options/) for all options.

**Basic mongod.conf**

```yaml
# Where and how to store data.
storage:
  dbPath: /Volumes/ssdraid/mongodb-data0
  journal:
    enabled: true
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1
# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo
```

**Copy to right location**

```bash
sudo cp conf/mongod.conf /etc/mongodb/conf
```

### Configure MONGODB_HOME and PATH

Add exports so we can easily access MongoDB binaries.

```bash
# .bash_profile
export MONGODB_HOME=/usr/local/mongodb/latest
export PATH=$MONGODB_HOME/bin:$PATH
source ~/.bash_profile
```

### Set Permissions

Set proper permissions for our ``_mongod`` Service Account and group.

```bash
# set perms on data directory
sudo chown -R _mongod:_mongod /Volumes/ssdraid/mongodb-data0
# set perms on MongoDB log directory
sudo chown -R _mongod:_mongod /var/log/mongodb/
# then config directory
sudo chown -R _mongod:_mongod /etc/mongodb/
```

### Initial Startup

Now startup Mongod passing it the config file we crafted above.  This will startup ``mongod`` using the ``_mongod`` Service Account.  At this point MongoDB is running wide-open, so next order of business is adding a Root User account.

```bash
sudo -u _mongod mongod --config /etc/mongodb/conf/mongod.conf
```

### Configure Admin User

* Open a new Terminal and start the Mongo Shell ``mongo``
* Run the commands below to create a new root user

```bash
# login
mongo --host localhost --port 27017
# switch to admin database
use admin
# create a new root user
db.createUser(
  {
    user: "corbs",
    pwd: "********",
    roles: [ "root" ]
  }
)
# make sure it took
show users
# kill mongod, oh the power of an admin
db.shutdownServer()
# exit shell
exit
```

### Startup with Auth

Now let's start MongoDB with the ``--auth`` flag to enable client authentication.  Thus far we only have our Root User defined so we'll use that when we connect.

```bash
sudo -u _mongod mongod --config /etc/mongodb/conf/mongod.conf --auth
```

### Connect with Mongo Shell

Since auth is enabled on the MongoDB server we'll need to authenticate before we can see anything.

```bash
# connect as 'corbs' and use the 'admin' database to authenticate against
mongo mongodb://localhost/?authSource=admin --username corbs
MongoDB shell version v4.2.1
Enter password: ***********

> show databases
admin   0.000GB
config  0.000GB
local   0.000GB
```

### Create macOS Launch Daemon

So at this point we have a working native install of mongoDB on macOS but running the server process requires kicking off from the command line.  This works but we'll let macOS start and maintain the server process.  This way on restart or if mongoDB crashes the OS will bring it back.  Also its kinda terse to prefix ``sudo -u _mongod`` anytime we want to start, stop or otherwise use mongoDB server bits.  We can codify these things once and let macOS handle the process life-cycle.

Use the launchctl below to configure macOS to manage mongoDB.  **Note** make sure path values below match what you've previously configured.

**mongod.plist macOS launchctl config**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- mongodb is name we use with launchctl -->
    <key>Label</key>
    <string>mongodb</string>
    <!-- mongod start command and args -->
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/mongodb/latest/bin/mongod</string>
        <string>--config</string>
        <string>/etc/mongodb/conf/mongod.conf</string>
        <string>--auth</string>
    </array>
    <!-- run when macOS starts -->
    <key>RunAtLoad</key>
    <true/>
    <!-- macOS will keep the process in memory -->
    <key>KeepAlive</key>
    <true/>
    <!-- _mongod group name we added with dscl -->
    <key>GroupName</key>
    <string>_mongod</string>
    <!-- _mongod service account we addded with dscl -->
    <key>UserName</key>
    <string>_mongod</string>
    <!-- macOS host directory where mongoDB bits reside -->
    <key>WorkingDirectory</key>
    <string>/usr/local/mongodb/latest</string>
    <!-- error log -->
    <key>StandardErrorPath</key>
    <string>/var/log/mongodb/error.log</string>
    <!-- stdout log -->
    <key>StandardOutPath</key>
    <string>/var/log/mongodb/output.log</string>
</dict>
</plist>
```

Copy ``mongod.plist`` to LaunchDaemons directory ``/Library/LaunchDaemons``.

```bash
sudo cp conf/mongod.plist /Library/LaunchDaemon
```

### mongoDB Lifecycle Management

Since we're letting macOS manage mongoDB we'll need to use macOS tools to manage the server.  With the Launch Daemon in place and loaded you can now use ``mongo`` client to login and start using mongoDB.

```bash
# load the mongod daemon, this just needs to be done once
sudo launchctl load -w /Library/LaunchDaemons/mongod.plist

# check that mongod is running
ps aux | grep mongod

# start mongodb
sudo launchctl start mongodb

# stop mongodb
sudo launchctl stop mongodb

# see if mongodb is in launchctl
sudo launchctl list | grep mongodb

# view log files configured in launchctl
sudo cat /var/log/mongodb/mongod.log
sudo cat /var/log/mongodb/output.log
sudo cat /var/log/mongodb/error.log
```

### Uninstall all of this

```bash
# remove directory for MongoDB binaries
sudo rm -rf /usr/local/mongodb
# remove directory for MongoDB config
sudo rm -rf /etc/mongodb/conf
# remove directory for MongoDB log
sudo rm -rf /var/log/mongodb
# remove directory for MongoDB data
sudo rm -rf /Volumes/ssdraid/mongodb-data0
# remove group and service account
sudo dscl . -delete /Groups/_mongod PrimaryGroupID 400
sudo dscl . -delete /Groups/_mongod
sudo dscl . -delete /Users/_mongod UniqueID 400
sudo dscl . -delete /Users/_mongod PrimaryGroupID 400
sudo dscl . -delete /Users/_mongod UserShell /usr/bin/false
# Unload and remove LaunchDaemon
sudo launchctl unload /Library/LaunchDaemons/mongod.plist
sudo launchctl remove mongodb
```

### References

1. Parts borrowed from [Sydney-o9's gist](https://gist.github.com/Sydney-o9/9a6d4a017539cb8610a5695ae505bb61)
