*This project has been created as part of the 42 curriculum by areen-alakaleek.*

# Born2beRoot

## Description

Born2beRoot is a system administration project whose goal is to introduce virtualization by building a first server from scratch inside a virtual machine, while following strict security and configuration rules.

The result is a minimal Debian 13 (trixie) server, with no graphical interface, running in VirtualBox. It includes:

- An encrypted LVM setup (`/boot`, `/`, swap, `/home`)
- An SSH service on port `4242` (root login disabled)
- A UFW firewall allowing only port `4242`
- AppArmor enabled at startup
- A strong password policy (PAM `pwquality` + password aging)
- A hardened `sudo` configuration with input/output logging
- A `monitoring.sh` bash script broadcasting system information every 10 minutes via `wall` and `cron`

## Instructions

### Requirements

- VirtualBox
- Debian stable (netinst ISO)

### Installation summary

1. Create a VM in VirtualBox and install Debian without any desktop environment, using *Guided - use entire disk and set up encrypted LVM*.
2. Set the hostname to `<login>42` and create the user `<login>` in the groups `sudo` and `user42`.
3. Install packages: `sudo ufw openssh-server libpam-pwquality vim cron net-tools`.
4. Configure SSH (`/etc/ssh/sshd_config`): `Port 4242`, `PermitRootLogin no`.
5. Enable UFW: default deny incoming, allow `4242`.
6. Password policy:
   - `/etc/login.defs`: `PASS_MAX_DAYS 30`, `PASS_MIN_DAYS 2`, `PASS_WARN_AGE 7`
   - `/etc/pam.d/common-password`: `minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root`
7. Configure sudo in `/etc/sudoers.d/sudo_config`: 3 attempts, custom bad-password message, logs in `/var/log/sudo/`, `requiretty`, restricted `secure_path`.
8. Install `monitoring.sh` in `/usr/local/bin/` and schedule it in root's crontab:
   ```
   @reboot /usr/local/bin/monitoring.sh
   */10 * * * * /usr/local/bin/monitoring.sh
   ```

### Connecting by SSH (VirtualBox NAT)

Add a port forwarding rule in *Settings > Network > Advanced > Port Forwarding* (Host port `2222` -> Guest port `4242`), then:

```bash
ssh <login>@127.0.0.1 -p 2222
```

### Useful checks

```bash
lsblk
aa-status
ss -tunlp
ufw status
chage -l <login>
sudo -l
```

### Signature

The `signature.txt` file contains the SHA-1 of the virtual disk:

```bash
sha1sum born2beroot.vdi
```

The virtual machine itself is not part of this repository.

## Resources

### References

- [Debian Documentation](https://www.debian.org/doc/)
- [Debian Administrator's Handbook](https://debian-handbook.info/)
- `man sudoers`, `man sshd_config`, `man pam_pwquality`, `man ufw`, `man cron`, `man wall`
- [Arch Wiki - LVM](https://wiki.archlinux.org/title/LVM)
- [Arch Wiki - dm-crypt](https://wiki.archlinux.org/title/Dm-crypt)
- [AppArmor on Debian](https://wiki.debian.org/AppArmor)
- [VirtualBox User Manual](https://www.virtualbox.org/manual/)

### Use of AI

AI (Claude) was used as a learning and troubleshooting assistant for:

- Explaining the purpose of each configuration step (SSH, UFW, PAM, sudo, cron)
- Debugging the VirtualBox port forwarding and SSH access problem
- Drafting a first version of `monitoring.sh`, which was then tested, read and understood line by line
- Helping structure this README

All configurations were typed, tested and verified on the VM, and I can explain each of them.

## Project description

### Design choices

- **Operating system:** Debian 13 (stable). It is beginner-friendly, very well documented, and uses AppArmor, which is enabled by default.
- **Partitioning:** one encrypted partition (`sda5_crypt`) containing an LVM volume group with logical volumes for `/` (about 8 GB), swap (about 1.3 GB) and `/home` (about 15 GB), plus an unencrypted `/boot`. Sizes were chosen to leave room for the system and user data without wasting disk space.
- **Security policies:** SSH on a non-default port with root login disabled, a default-deny firewall, a strong password policy with expiration, and sudo restricted and fully logged.
- **User management:** the `<login>` user belongs to `sudo` and `user42`. Root is not used for remote access.
- **Services installed:** SSH, UFW, cron, AppArmor. No graphical server is installed.

### Debian vs Rocky Linux

| | Debian | Rocky Linux |
|---|---|---|
| Family | Independent, community-driven | RHEL-compatible, community rebuild |
| Package manager | `apt` / `dpkg` (`.deb`) | `dnf` / `rpm` (`.rpm`) |
| Mandatory access control | AppArmor | SELinux |
| Default firewall tool | UFW (optional) | firewalld |
| Release model | Stable releases, long-lived | Follows RHEL, enterprise-oriented |
| Advantages | Easy to learn, huge package repository, great documentation | Enterprise-grade, long support, strong SELinux integration |
| Disadvantages | Older package versions in stable | Steeper learning curve, SELinux is complex to configure |

Debian was chosen because it is recommended for beginners and simpler to configure.

### AppArmor vs SELinux

| | AppArmor | SELinux |
|---|---|---|
| Approach | Path-based profiles | Label-based policies (contexts on every file and process) |
| Complexity | Simpler, easier to write and read | More granular but much more complex |
| Default on | Debian, Ubuntu | RHEL, Rocky, Fedora |
| Strength | Quick to deploy | Very fine-grained control |

### UFW vs firewalld

| | UFW | firewalld |
|---|---|---|
| Purpose | Simple front-end for iptables/nftables | Dynamic firewall manager using zones |
| Usage | `ufw allow 4242` | `firewall-cmd --add-port=4242/tcp` |
| Strength | Very simple syntax | Runtime changes without restart, zones and services |
| Default on | Debian/Ubuntu (optional) | RHEL/Rocky |

### VirtualBox vs UTM

| | VirtualBox | UTM |
|---|---|---|
| Platforms | Windows, Linux, macOS (Intel) | macOS (Intel and Apple Silicon) |
| Technology | Hypervisor with extension features | Built on QEMU and Apple virtualization |
| Disk format | `.vdi` | `.qcow2` |
| Advantages | Mature, free, snapshots, easy networking | Works on Apple Silicon (M1/M2) |
| Disadvantages | Limited support on Apple Silicon | Fewer features, more manual setup |

VirtualBox was used for this project.
