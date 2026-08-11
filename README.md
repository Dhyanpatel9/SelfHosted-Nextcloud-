# Self-Hosted Nextcloud Homelab

A personal systems administration and self-hosting project built to provide a private cloud for files and photos while developing practical Linux, virtualization, networking, storage, security, and troubleshooting skills.

## Project Overview

This project runs Nextcloud inside an Ubuntu Server virtual machine hosted on Proxmox. The server is reachable remotely through a domain name and is protected with Linux security controls.

The project started as a hands-on way to learn systems administration and evolved into a useful personal cloud that can be accessed from an iPhone, laptop, and web browser.

## Architecture

```text
                         Internet
                            |
                     Public DNS record
                            |
                    cloud.dhyan09.com
                            |
                     Home Router/NAT
                            |
                    Proxmox VE Host
                            |
                  Ubuntu Server VM
                         VM 100
                            |
        +-------------------+-------------------+
        |                   |                   |
      Nextcloud          SSH + UFW          Fail2Ban
        |
   User files/photos
```

## Environment

| Component | Role |
|---|---|
| Proxmox VE | Hypervisor / virtualization platform |
| Ubuntu Server | Guest operating system |
| Nextcloud | Private cloud platform |
| Apache | Web server |
| MariaDB | Nextcloud database |
| UFW | Host firewall |
| Fail2Ban | Brute-force protection |
| Let's Encrypt | TLS/HTTPS certificates |
| DNS | Public hostname for remote access |
| iPhone / Laptop | Client devices |

## Storage Expansion

The Nextcloud VM originally had a **50 GiB** virtual disk. The virtual disk was expanded to **500 GiB** in Proxmox.

The guest operating system was then expanded in stages:

1. Expanded the Proxmox VM disk from 50 GiB to 500 GiB.
2. Used `growpart` to expand `/dev/sda3`.
3. Used `pvresize /dev/sda3` to make the additional capacity available to LVM.
4. Expanded the Ubuntu logical volume.
5. Verified the resulting filesystem with `df -h /`.

Final verification showed approximately **490 GiB** available to the root filesystem, with only about **8.5 GiB** used at the time of verification.

Example verification commands:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo pvs
df -h /
```

## Security Hardening

### Fail2Ban

Fail2Ban was configured with an SSH jail and verified with:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

At the time of verification, the SSH jail was active with no failed authentication attempts and no banned addresses.

### Firewall

UFW is used to restrict inbound traffic to services that are intentionally exposed.

### HTTPS

The Nextcloud service is configured for HTTPS using a TLS certificate. The public hostname used during the project is `cloud.dhyan09.com`.

Sensitive values such as passwords, private keys, tokens, and authentication secrets are intentionally excluded from this repository.

## Client Access

### iPhone

Nextcloud is accessed from an iPhone using the mobile application. This provides a convenient way to upload and access photos and files from the phone without depending entirely on third-party cloud storage.

### Laptop / PC

Nextcloud can also be accessed through a normal web browser, making the service usable from computers without installing a dedicated client.

## Skills Demonstrated

This project demonstrates practical experience with:

- Linux server administration
- Ubuntu Server
- Proxmox virtualization
- Virtual machine lifecycle management
- LVM and filesystem/storage expansion
- DNS and public hostname configuration
- HTTP/HTTPS web services
- TLS certificate management
- SSH administration
- Firewall configuration
- Fail2Ban and brute-force protection
- Remote access troubleshooting
- Client configuration
- Capacity planning
- System troubleshooting
- Technical documentation

## Troubleshooting Examples

### VM disk expansion

The Proxmox virtual disk was successfully increased to 500 GiB, but the guest operating system initially still reported the smaller partition and filesystem sizes. The issue was resolved by expanding the partition with `growpart`, resizing the LVM physical volume with `pvresize`, and expanding the logical volume/filesystem.

### Remote access testing

The public IPv4 address was checked from the server with:

```bash
curl -4 ifconfig.me
```

DNS resolution for the cloud hostname was also verified against the public address.

### SSH protection verification

Fail2Ban was inspected directly rather than assuming the service was working:

```bash
sudo fail2ban-client status sshd
```

## Lessons Learned

- Virtual disk size, partition size, LVM physical volume size, logical volume size, and filesystem size are separate layers and may need to be expanded independently.
- A successful Proxmox disk resize does not automatically mean the guest OS can use all of the new capacity.
- Remote services require coordinated configuration across DNS, NAT/firewall rules, the operating system, the web server, TLS, and the application.
- Security controls should be verified with actual status commands instead of being assumed to be active.
- A homelab can be both a useful personal service and a realistic systems administration learning environment.

## Future Improvements

- Automated Nextcloud backups
- Off-site backup strategy
- Proxmox VM backup and restore testing
- Storage monitoring and alerting
- More granular firewall rules
- SSH key-based authentication
- Regular security update automation
- Monitoring with a dedicated observability stack
- Additional self-hosted services
- Disaster recovery documentation

## Project Status

**Active homelab project** — Nextcloud is operational and being used as a personal private cloud.

## Disclaimer

This is a personal homelab and learning project. Configuration details may differ from production environments. The repository intentionally documents the architecture and administrative process without publishing credentials, private keys, tokens, or other sensitive information.
