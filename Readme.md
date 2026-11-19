# Linux Server Hardening & Incident Response

## Project Overview

This project demonstrates practical Linux system administration, server hardening and incident response using an Ubuntu virtual machine.

The objective was to assess a Linux server, investigate a simulated SSH brute-force attack using authentication logs, identify evidence of a successful compromise, and implement security controls to prevent similar attacks.

The project follows six phases, covering system reconnaissance, access control, patch management, log analysis, SSH hardening and incident reporting.

## Project Objectives

- Assess the security posture of an Ubuntu Linux system.
- Understand Linux users, groups and file permissions.
- Identify and apply available system updates.
- Investigate suspicious authentication activity.
- Identify the source of a brute-force attack.
- Harden SSH authentication and configure firewall rules.
- Produce an incident report supported by log evidence.

## Technologies & Tools

| Technology | Purpose |
|---|---|
| Ubuntu Linux | Operating system and lab environment |
| Bash | Command-line administration |
| OpenSSH | Secure remote access |
| UFW | Host-based firewall |
| APT | Package management and updates |
| grep | Searching authentication logs |
| awk | Extracting information from logs |
| sort & uniq | Grouping and counting IP addresses |
| systemctl | Managing system services |

## Project Implementation

### Phase 1: System Reconnaissance

Assessed the initial state of the Ubuntu machine before making security changes.

Activities:
- Identified the current user and group memberships.
- Inspected operating system and kernel information.
- Examined running processes.
- Identified listening network ports.
- Inspected the system log directory.

Key commands:

```bash
whoami && id
fastfetch
ps aux | wc -l
ps aux --sort=-%cpu | head -n 6
sudo ss -tlnp
ls -l /var/log
```

### Phase 2: User and Permission Management

Implemented Linux access controls using groups, users and file permissions.

Activities:
- Created an `engineers` group.
- Created a user called `alice`.
- Configured a shared directory at `/srv/shared`.
- Applied ownership and setgid permissions.
- Restricted access to a simulated private key.

Key commands:

```bash
sudo groupadd engineers
sudo useradd -m -s /bin/bash -G engineers alice
sudo mkdir -p /srv/shared
sudo chown root:engineers /srv/shared
sudo chmod 2770 /srv/shared
chmod 600 ~/deploy.key
```

### Phase 3: Patch Management

Assessed available package updates and patched the Ubuntu system.

Activities:
- Refreshed package repository information.
- Counted available package updates.
- Installed available updates.
- Verified the remaining upgrade count.

Key commands:

```bash
sudo apt update
apt list --upgradable | wc -l
sudo apt upgrade -y
```

### Phase 4: Incident Investigation and Log Analysis

Investigated a sample authentication log captured from a previously compromised Linux server.

The investigation focused on identifying failed login attempts, suspicious IP addresses, successful authentication and activity following the compromise.

**Investigation findings:**

- 126 authentication log entries.
- 115 failed password attempts.
- Attacking IP address: `203.0.113.47`.
- 102 failed attempts attributed to this address.
- A successful password login to the `admin` account.
- Subsequent suspicious sudo activity involving `/etc/shadow`.

The investigation established that the attacker successfully gained access and attempted to read the system password hash file.

Key commands:

```bash
wget https://raw.githubusercontent.com/SoftReb00t/Linux-Project/main/auth-sample.log -O ~/auth.log

wc -l ~/auth.log

grep -c "Failed password" ~/auth.log

grep "Failed password" ~/auth.log | grep -oE 'from [0-9.]+' | awk '{print $2}' | sort | uniq -c | sort -rn

grep "Accepted" ~/auth.log

grep "203.0.113.47" ~/auth.log | tail -n 5

grep "sudo" ~/auth.log
```

### Phase 5: SSH Server Hardening and Firewall Configuration

Implemented security controls to prevent the password-based attack identified during the investigation.

**SSH security configuration:**

- Installed OpenSSH Server.
- Disabled remote root login.
- Disabled password authentication.
- Enabled public-key authentication.
- Validated the SSH configuration before restarting the service.
- Tested that password-based authentication was refused.

SSH configuration:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Configuration validation:

```bash
sudo sshd -t
sudo systemctl restart ssh
sudo systemctl status ssh --no-pager
```

**UFW firewall configuration:**

Applied a default-deny inbound policy while allowing outbound traffic and SSH connections.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
```

The resulting firewall policy restricts incoming connections to the explicitly permitted SSH port.

### Phase 6: Incident Reporting

Created an incident report containing:

- Failed authentication statistics.
- Source IP addresses associated with the attack.
- Evidence of successful unauthorised access.
- Suspicious activity following authentication.
- Security remediation implemented.
- Incident conclusions and recommendations.

Final report:

`~/incident-report.txt`

## Security Concepts Demonstrated

- Linux system reconnaissance
- Least privilege and access control
- Linux file permissions and ownership
- Setgid directory permissions
- Patch management
- SSH security and authentication
- Authentication log analysis
- Brute-force attack investigation
- Host-based firewall configuration
- Incident response and reporting
- Defence in depth

## Project Evidence

Screenshots demonstrating each phase of the implementation are available in the `screenshots/` directory.

| Screenshot | Description |
|---|---|
| System reconnaissance | Linux system and network inspection |
| User permissions | Group and directory permissions |
| Patch management | Package updates |
| Incident investigation | Authentication log analysis |
| SSH hardening | SSH security configuration |
| Firewall configuration | UFW firewall rules |

## Key Learning Outcomes

This project provided hands-on experience with Linux security administration and incident investigation.

I developed practical skills in identifying suspicious authentication activity, extracting useful evidence from system logs, managing Linux permissions, hardening SSH services and applying firewall policies.

It also reinforced the importance of defence in depth, least privilege, patch management and evidence-based incident reporting.


