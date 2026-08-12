# Self-Hosted Nextcloud ☁️

> **A homelab project I built to learn systems administration — and ended up using as my own private cloud.**

## Why I built this

This is one of my self-study projects around **self-hosting and private cloud infrastructure**.

I wanted to understand what it actually takes to host and protect my own data instead of treating cloud storage as a black box. Personal data such as photos, documents, and files is an important asset, so I wanted to learn how I could keep that data in a private environment that I control while still being able to access it from anywhere.

The goal isn't simply to avoid cloud services. The bigger goal is to understand the infrastructure behind a private cloud: **where the data lives, how users access it, how the server is secured, how storage is managed, and how the data can be backed up and recovered.**

I already had a Proxmox machine at home, so I turned it into a proper learning project. I deployed Ubuntu Server, installed Nextcloud, exposed it securely over HTTPS, locked down SSH, configured a firewall and Fail2Ban, and eventually expanded the VM from 50 GB to 500 GB.

The result is a personal cloud that I can use from my **iPhone, laptop, or any web browser**.

---

## 📸 What I can do with it

- Upload photos from my iPhone
- Access my files from my laptop through a browser
- Access Nextcloud remotely when I'm away from home
- Store documents and personal files on my own server
- Manage the server remotely through SSH
- Back up the VM through Proxmox
- Keep learning Linux and infrastructure by actually running the service myself

---

##  What the setup looks like

```text
                    Internet
                       │
                  Public DNS
                       │
              cloud.dhyan09.com
                       │
                 Home Router
                       │
                  Proxmox VE
                       │
              ┌───────────────┐
              │ Ubuntu Server │
              │     VM 100    │
              ├───────────────┤
              │   Nextcloud   │
              │    Apache     │
              │    MariaDB    │
              │     UFW       │
              │   Fail2Ban    │
              └───────────────┘
                  │       │
                iPhone   Laptop
```

I intentionally keep Nextcloud inside its own VM rather than installing it directly on the Proxmox host. That gives me a cleaner separation between the hypervisor and the application.

---

##  Technology used

| Technology | What I used it for |
|---|---|
| **Proxmox VE** | Virtualization and VM management |
| **Ubuntu Server** | Nextcloud server OS |
| **Nextcloud** | Private cloud / file storage |
| **Apache** | Web server |
| **PHP** | Nextcloud application runtime |
| **MariaDB** | Nextcloud database |
| **UFW** | Host firewall |
| **Fail2Ban** | SSH brute-force protection |
| **Let's Encrypt** | HTTPS/TLS certificate |
| **DNS** | Public hostname / remote access |
| **LVM + ext4** | Linux storage management |
| **SSH** | Remote administration |

---

## 🔐 Security

Because the server is reachable from the Internet, I didn't want to simply expose a web server and hope for the best.

The main protections I configured were:

- HTTPS/TLS for web traffic
- SSH public-key authentication
- SSH password authentication disabled
- Root SSH login disabled
- UFW with **deny-by-default** inbound traffic
- Only required inbound services exposed
- Fail2Ban protecting SSH authentication
- Limited SSH authentication attempts
- X11 forwarding disabled
- TCP forwarding disabled
- Regular service/status verification

For example, I verify the firewall with:

```bash
sudo ufw status verbose
```

and Fail2Ban with:

```bash
sudo fail2ban-client status sshd
```

I also learned an important lesson here: **a configuration file saying something is enabled doesn't necessarily mean the running service is using that setting.** I used `sshd -T` to check the effective SSH configuration.

---

##  The storage problem I ran into

The VM originally had a **50 GB** virtual disk.

That was fine for installing Nextcloud, but once I started thinking about photos and documents, I knew I needed more space.

I expanded the virtual disk in Proxmox to **500 GB**.

But Ubuntu still reported the old size.

That's where the Linux storage layers became a real-world learning exercise.

I had to expand them one at a time:

```text
Proxmox virtual disk
        ↓
/dev/sda
        ↓
/dev/sda3
        ↓
LVM physical volume
        ↓
ubuntu-vg
        ↓
ubuntu-lv
        ↓
ext4 filesystem
```

The commands I used were:

```bash
sudo growpart /dev/sda 3
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

Final result:

```text
Filesystem       Size   Used   Avail
/                ~490G  ~8.5G  ~462G
```

This was one of the most useful parts of the project because it made the difference between **knowing what LVM is** and actually having to use it.

---

##  Remote access

I wanted to be able to use the cloud even when I wasn't at home.

The final flow is roughly:

```text
Phone / Laptop
      ↓
HTTPS
      ↓
Public DNS
      ↓
Home Router
      ↓
Proxmox
      ↓
Ubuntu VM
      ↓
Nextcloud
```

I tested the setup from a different network using a phone hotspot rather than only testing it from my home Wi-Fi.

That caught issues that wouldn't have been obvious from inside the LAN.

---

##  iPhone integration

One of the reasons I wanted this project in the first place was to have an easy place for my phone photos and files.

I configured the Nextcloud iPhone app so I can upload photos directly to my server.

I can then access those files from the Nextcloud web interface on my laptop.

So instead of thinking about the server as just a Linux VM, it has become something I actually use day-to-day.

---

##  Troubleshooting I worked through

This project wasn't a one-command installation. A lot of the learning came from fixing things when they didn't work.

### MariaDB authentication

The Nextcloud database account initially rejected the credentials used by the backup command. I checked the MariaDB user, authentication plugin, and grants, corrected the credentials, and verified access with:

```bash
mysql -u nextcloud -p -e "SELECT 1;"
```

### Interrupted backup archive

An interrupted `tar` operation left an incomplete archive. Trying to inspect it produced an EOF error.

I recreated the archive and verified it with:

```bash
sudo tar -tzf /backup/nextcloud/nextcloud-files.tar.gz >/dev/null && echo "Archive OK"
```

### SSH hardening

Cloud-init had its own SSH configuration, so I checked the **effective** configuration rather than relying on one file:

```bash
sudo sshd -T
```

### Remote access

I tested DNS, public IP resolution, port forwarding, firewall rules, and SSH access from outside my home network.

These issues were useful because they forced me to troubleshoot the entire path instead of assuming the application itself was the problem.

---

##  Backup and recovery

I created backups at several levels during the project:

- MariaDB database dump
- Nextcloud file/application archive
- Proxmox VM backup

I also learned that **having a backup isn't the same as having a recovery plan**.

The current Proxmox backup is still tied to the same physical host, so my next major improvement is a separate backup location.

My eventual goal is a simple 3-2-1 approach:

```text
Primary server
     │
     ├── Local backup
     │
     ├── Separate physical backup
     │
     └── Off-site copy
```

---

##  What I learned

This project gave me practical experience with:

- Linux administration
- Ubuntu Server
- Proxmox virtualization
- VM storage management
- LVM
- ext4
- Apache
- PHP
- MariaDB
- SSH
- Public-key authentication
- DNS
- NAT / port forwarding
- HTTPS / TLS
- UFW
- Fail2Ban
- Backup and recovery
- Troubleshooting
- Remote administration

More importantly, I learned how these pieces depend on each other.

A problem that looks like a Nextcloud problem can actually be DNS, NAT, Apache, TLS, firewall, Linux permissions, storage, or the database.

That's the part of systems administration I wanted to learn.

---

##  What's next

This project is still evolving. Some things I want to add:

- [ ] Separate backup disk
- [ ] Test a full VM restore
- [ ] Off-site backup
- [ ] Storage monitoring and alerts
- [ ] Centralized logging
- [ ] Automated maintenance/security updates
- [ ] Better disaster recovery documentation
- [ ] More self-hosted services
- [ ] Ansible automation
- [ ] A proper network diagram

---

##  Documentation

More detailed notes are available here:

- [Architecture](docs/ARCHITECTURE.md)
- [Security](docs/SECURITY.md)
- [Backup & Recovery](docs/BACKUP-AND-RECOVERY.md)

---

## License

This project is released under the **MIT License**. See [LICENSE](LICENSE).

The license applies to the original documentation, scripts, and configuration examples in this repository. It does **not** change the licenses of third-party software used by the homelab, such as Nextcloud, Ubuntu, Proxmox, Apache, MariaDB, or other dependencies.

---

##  Important

This repository documents a personal homelab. Real credentials, private keys, tokens, database passwords, and other sensitive information are intentionally **not** included.

If you build something similar, don't copy credentials or expose services without understanding the security implications first.

---

**Status:**  Running and actively used

Built as a hands-on systems administration project.
