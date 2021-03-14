# RPi
Notes for initializing and maintaning my RPi server.

## Sources
[[1] Copying an operating system image to an SD card using Mac OS](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)

[[2] Initial Server Setup with Debian 10](https://www.snel.com/support/initial-server-setup-with-debian-10/)

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
 