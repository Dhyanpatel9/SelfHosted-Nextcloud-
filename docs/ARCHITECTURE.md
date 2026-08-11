# Architecture

## High-Level Design

```text
Internet
   |
Public DNS
   |
cloud.dhyan09.com
   |
Home Router / NAT
   |
Proxmox VE Host
   |
Ubuntu Server VM (VM 100)
   |
+-------------------------+
| Nextcloud                |
| Apache / HTTPS            |
| MariaDB                   |
| UFW + Fail2Ban            |
+-------------------------+
   |
User data
   |
+-------------------------+
| iPhone | Laptop | Browser |
+-------------------------+
```

## Virtualization Layer

The physical host runs Proxmox VE. Nextcloud is isolated inside an Ubuntu Server virtual machine rather than being installed directly on the hypervisor.

This separation provides a useful administration model:

- Proxmox handles virtualization and VM resources.
- Ubuntu handles the operating system and services.
- Nextcloud handles the application and user data.

## Guest Storage Layout

The Ubuntu VM uses a virtual disk presented as `/dev/sda`.

The storage expansion performed during the project followed this path:

```text
Proxmox virtual disk
        |
      /dev/sda
        |
      /dev/sda3
        |
   LVM physical volume
        |
    ubuntu-vg
        |
 ubuntu-lv (root)
        |
      / filesystem
```

The disk was increased from 50 GiB to 500 GiB. The guest partition and LVM layers were then expanded so Ubuntu could consume the additional capacity.

## Network / Application Flow

A client connects to the public hostname. DNS resolves the hostname to the home connection's public address. Router/NAT rules and host firewall rules then determine whether the request reaches the Nextcloud web service.

HTTPS protects the web session in transit.

## Design Considerations

The architecture deliberately keeps the Proxmox management layer separate from the application VM. This reduces the blast radius of application-level issues and makes the VM easier to back up, restore, resize, or rebuild.
