# Permissions
Whitout understanding users, usergroups and their premissions in Linux, nothing is ever going to work properly.

## Resources
[[1] An Introduction to Linux Permissions](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-permissions)

[[2] Linux /etc/passwd file explained](https://www.linuxnix.com/linux-password-file-explained-detail/)

[[3] Linux /etc/group file](https://www.tutorialsandyou.com/linux/linux-group-file-7.html)



# Users
Fist off all, let's view all users by inspecting the `passwd` file:
```
cat /etc/passwd
```
The file contains all users with 6 parameters delimited by `:` :
```
pi:x:1337:420:dank:/home/pi:/bin/bash
|  | |    |   |    |        |--- Default shell
|  | |    |   |    |------------ Home dir aka ~
|  | |    |   |----------------- Comment
|  | |    |--------------------- GID, primary Group ID
|  | |-------------------------- UID, User ID
|  |---------------------------- Password location, x indicating encrypted & shadowed
|------------------------------- User ID, comma-delimted
```

This file can be edited to manage users in place of the `useradd` command.

# Groups
Each user is assigned to at least one group. By default, a group with the same name as the user will be created and assigned respectively when an user is created.


Inspect the `group` file:
```
cat /etc/group
```

As `passwd`, the `group` file delimites with `:` :
```
users:x:100:pi
|     |  |  |----- Group members
|     |  |-------- GID, Group ID
|     |----------- Password location, x indicating encrypted & shadowed
|----------------- Group name
```

# Permissions

View ownership the files in the current directory:
```
ls -l
```

Or for a specific file:
```
ls -l someDir/someFile
```

I coulnd't be bothered to type of the decoding of `ls -l` output, so look at some pictures from
[[1]](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-permissions) instead.

![ls -l output](https://assets.digitalocean.com/articles/linux_basics/ls-l.png "ls -l output")


![Mode decoding](https://assets.digitalocean.com/articles/linux_basics/mode.png "Mode decoding")


Interesting things to note here are:

- The file type: `-` for regular files, `d`for directory. There are apparently other types as well.
- You can assign files and directories only to groups, i.e with mode = `----rw----`.
- "other" indicates everyone except user or group with ownership.

## Editing Permissions:

Changing the user ownership of `foo` to `USER` with `chown`:
```
sudo chown USER foo
```

Changing the group ownership of `foo` and `bar` to `GROUP` with `chgrp`: 
```
sudo chgrp GROUP foo bar
```

Note that if `foo` or `bar` is a directory, you will only change the directory it self. Use the `-R` for recursion:

```
sudo chown -R USER foo_dir
```

Now for the difficult part. Changing the mode with `chmod`. You hgave the following _operators_:

- 


# Setting up groups:
