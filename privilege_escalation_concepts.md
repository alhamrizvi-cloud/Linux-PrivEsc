# Linux Privilege Escalation Concepts

## What is Privilege Escalation?
Privilege escalation is the process of gaining higher access rights than those originally granted to a user or process. In Linux, it usually means moving from a low-privileged account to a more powerful account, often `root`.

### Types of Privilege Escalation
- **Vertical escalation**: moving from a normal user to an administrator or root account.
- **Horizontal escalation**: moving from one user account to another account with similar privileges.

### Why it matters
Attackers and penetration testers use privilege escalation to:
- access sensitive files and secrets
- control system services
- persist on a compromised host
- exfiltrate data or access other network resources

## Linux Security Fundamentals
Understanding Linux privilege escalation begins with Linux security building blocks.

### User and group identities
- `UID` (user identifier): defines who owns a process or file.
- `GID` (group identifier): defines the group ownership.
- `root` has UID 0 and full system control.

### File permissions
- Owner permissions, group permissions, and world permissions control read, write, and execute access.
- Special bits:
  - `SUID` (set-user-ID): executes a binary with the file owner's privileges.
  - `SGID` (set-group-ID): executes a binary with the file owners group privileges.
  - `sticky bit`: controls deletion in shared directories.

### Linux capabilities
Capabilities allow fine-grained privileges to be assigned to processes without full root access. Examples:
- `CAP_NET_BIND_SERVICE`
- `CAP_DAC_READ_SEARCH`
- `CAP_SETUID`

### Sudo and command allowances
`sudo` controls what a user can run with elevated privileges.
- `sudo -l` lists allowed commands.
- `NOPASSWD` lets the user run commands without a password.
- `env_keep` and `secure_path` affect environment-based privilege strategies.

## Core privilege escalation concepts

### Enumeration is the foundation
The most reliable escalations come from careful discovery.
- inspect sudo rights
- find SUID/SGID binaries
- look for writable scripts and configs
- review cron jobs and services
- examine file permissions and environment variables

### Weaknesses are often configuration-based
Most Linux privilege escalations are not zero-days; they leverage misconfigurations such as:
- overly permissive sudoers entries
- world-writable system files or directories
- poorly configured cron jobs
- writable scripts executed by privileged processes
- stale services running as root

### Exploitation vs discovery
- **Discovery** is finding the weakness.
- **Exploitation** is using that weakness to escalate privileges.
Good methodology separates these steps.

## Common escalation paths
1. **Sudo abuse**: allowed command can spawn a shell or modify root-owned content.
2. **SUID/SGID binaries**: execute binaries that escalate privileges via file owner/group privileges.
3. **Writable files**: modify scripts, config files, or binaries used by root.
4. **Cron jobs / scheduled tasks**: inject commands into jobs that run as root.
5. **PATH or library hijacking**: insert malicious executables or libraries into search paths.
6. **Capabilities abuse**: use granted capabilities to gain additional control.
7. **Container or service misconfigurations**: escape containers or abuse service accounts.

## Defensive concepts
To defend against Linux privilege escalation, administrators should:
- apply the principle of least privilege
- avoid `NOPASSWD` for unnecessary sudo commands
- avoid writable root-owned scripts and service files
- restrict SUID/SGID binaries and remove unnecessary ones
- monitor cron jobs, systemd units, and installed packages
- use centralized configuration management and regular audits

## Reading path for this repository
- `README.md`: fast cheat sheet and practical commands.
- `privilege_escalation_concepts.md`: foundational concepts and methodology.
- `privilege_escalation_techniques.md`: detailed exploitation techniques with examples.
