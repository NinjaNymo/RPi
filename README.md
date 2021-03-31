# RPi
Notes for initializing and maintaning my RPi server.

## Sources
[[1] Copying an operating system image to an SD card using Mac OS](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)

[[2] Initial Server Setup with Debian 10](https://www.snel.com/support/initial-server-setup-with-debian-10/)

[[3] External storage configuration](https://www.raspberrypi.org/documentation/configuration/external-storage.md)

[[4] Plex installation on Pi 4B 4GB Raspbian Buster](https://www.reddit.com/r/raspberry_pi/comments/e5l70l/plex_installation_on_pi_4b_4gb_raspbian_buster/)

[[5] How to Install Python 3.8 on Debian 10](https://linuxize.com/post/how-to-install-python-3-8-on-debian-10/)

# Creating the SD-card image:
Download the latest Raspberry Pi OS image .zip from [raspberrypi.org](https://www.raspberrypi.org/software/operating-systems/) and **extract the image .zip** to get the image .img.

## macOS `dd` method [[1]](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md):

**Identify the SD-card** using `diskutil list`:
```
% diskutil list

/dev/disk2 (external, physical):
   #:     TYPE NAME                                      SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.6 GB    disk2
   1:     Windows_FAT_32 boot                            268.4 MB   disk2s1
   2:     Linux                                          31.3 GB    disk2s2
```

In this case we see the 32GB SD-card as `/dev/disk2`.

**Unmount the disk** with `diskutil unmountDisk`:
 ```
% diskutil unmountDisk /dev/disk2

Unmount of all volumes on disk2 was successful
```

**Copy the image** with `dd`:
```
% sudo dd bs=1m if=input_file_path.img of=/dev/rdisk2; sync

1776+0 records in
1776+0 records out
1862270976 bytes transferred in 453.667018 secs (4104929 bytes/sec)
```
Note that this might take a while. You can press ctrl + T to check `dd` progress.

**Eject the disk** with `diskutil`:
```
% sudo diskutil eject /dev/rdisk2

Disk /dev/rdisk2 ejected
```

**Plug the SD-card back in** cus you are gonna need ssh enabled. Navigate to the SD-card:
```
% cd /Volumes/boot
```
And `touch` a file called ssh:
```
% touch ssh
```
This will enable ssh on port 22 on the RPi.

# Fix SSH host key:
After installing a fresh OS on the RPi, you might experience a **Host key verification failed** error when trying to SSH.

Reset the keys with `ssh-keygen` to fix the issue:
```
% ssh-keygen -R RPi_IP_ADDRESS

# Host RPi_IP_ADDRESS found: line 3
/Users/USER/.ssh/known_hosts updated.
Original contents retained as /Users/USER/.ssh/known_hosts.old
```

# First time setup:
SSH into the RPi as the pi user. The default password will be raspberry.

**Change the pi password** using `passwd`:
```
$ passwd

passwd: password updated successfully
```

**Set the time-zone** with `timedatectl`:
```
$ sudo timedatectl set-timezone Europe/Oslo
```

You can list the avaliable ones with `timedatectl list-timezones`.

**Update** everything:
```
$ sudo apt-get update
```
If you get a warning about "no space left on device" try rebooting with `sudo reboot`.

**Upgrade** everything (should terminate with `done.`):
```
$ sudo apt-get upgrade
```
# Customize the hostname [[2]](https://www.snel.com/support/initial-server-setup-with-debian-10/):
Use `hostnamectl`to set the hostname to "rpi4" or whatever:

```
$ sudo hostnamectl set-hostname rpi4
```
Edit /etc/hosts with ´nano´ and replace "raspberry" following "127.0.1.1" with "rpi4":
```
$ sudo nano /etc/hosts

127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1       rpi4
```
Reboot the system with `sudo reboot`.

# External Storage [[3]](https://www.raspberrypi.org/documentation/configuration/external-storage.md):
**Create mount points** with `mkdir`:
```
$ sudo mkdir /mnt/ext_hdd
$ sudo mkdir /mnt/nas_hdd
```
**Identify** storage with `fdisk`:
```
$ sudo fdisk -l

Disk /dev/sda: 1.8 TiB, 2000398933504 bytes, 3907029167 sectors
...
Disk /dev/sdb: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
```
Find the **UUID** of the storage using `blkid`:
```
$ sudo blkid

/dev/sda1: UUID="UUID_A" TYPE="ext4" PARTUUID="PUUID_A"
/dev/sdb1: UUID="UUID_B" TYPE="ext4" PARTLABEL="primary" PARTUUID="PUUID_B"
```

**Edit fstab** with `nano`. Add two new entries for the storage using the UUIDs (w/o " "):
```
$ sudo nano /etc/fstab

# 2TB Ext HDD:
UUID=UUID_A /mnt/ext_hdd ext4 defaults,auto,users,rw,nofail 0 0
# 4TB NAS HDD:
UUID=UUID_B /mnt/nas_hdd ext4 defaults,auto,users,rw,nofail 0 0
```

Reboot and verify that storage was mounted with `df`:
```
$ df -h

Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       1.8T  1.7T   91G  95% /mnt/ext_hdd
/dev/sdb1       3.6T  1.8T  1.7T  52% /mnt/nas_hdd
```

# Plex [[4]](https://www.reddit.com/r/raspberry_pi/comments/e5l70l/plex_installation_on_pi_4b_4gb_raspbian_buster/):

Add the https transport protocol:
```
$ sudo apt-get install apt-transport-https
```

Add the official Plex package keys:
```
$ curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
```

Add the Plex repository:
```
$ echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list

deb https://downloads.plex.tv/repo/deb public main
```

Update repositories:
```
$ sudo apt-get update

...
Reading package lists... Done
```

And install Plex:
```
$ sudo apt-get install plexmediaserver
```
# Transmission:
Do the regular update:
```
$ sudo apt-get update
```

Install `transmission-daemon`:
```
$ sudo apt-get install transmission-daemon
```

Stop the service:
```
$ sudo service transmission-daemon stop
```

Edit the config with `nano`:
```
$ sudo nano /etc/transmission-daemon/settings.json
```

Here we can leave most things default, but change:

1. Set `"dht-enabled": false` to disable DHT.
2. Set `"pex-enabled": false` to disable PEX.
3. Set `"download-dir"` to the default download dir.
4. Set `"rpc-whitelist-enabled": false` to allow anyone to connect remotely.
   - Be extremely aware that the RPC port should NOT be opened on your firewall/router.
5. Set `"rpc-authentication-required": false` to disable the RPC password.

Start transmission back up:
```
$ sudo service transmission-daemon start
```


# Installing Python 3.8 [[5]](https://linuxize.com/post/how-to-install-python-3-8-on-debian-10/):
Update and install dependencies for **building Python**:
```
$ sudo apt update

$ sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev
```

Go to the [Python website](https://www.python.org/downloads/source/) to find a link for the source tarball such as:
```
https://www.python.org/ftp/python/3.8.8/Python-3.8.8.tgz
```

Download the tarball with `wget`:
```
$ sudo wget https://www.python.org/ftp/python/3.8.8/Python-3.8.8.tgz
```

Extract it with `tar`:
```
$ sudo tar xzf Python-3.8.8.tgz
```

`cd` into the Python folder:
```
$ cd Python-3.8.8
```

Enable some optimizations:
```
$ ./configure --enable-optimizations
```

Run the alternative install script to avoid overwriting excisting Python versions:
```
$ sudo make altinstall
```
Note that this might take a while.

Verify the installation:
```
$ python3.8 -V

Python 3.8.8
```

Do some cleanup:
```
$ cd ..

$ sudo rm -R Python-3.8.8

$ sudo rm Python-3.8.8.tgz 
```
