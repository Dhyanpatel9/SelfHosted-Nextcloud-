# Backup and Recovery Plan

Backups are a planned next step for this homelab. Increasing the VM to 500 GiB provides capacity, but storage capacity is not the same thing as data protection.

## Current State

The Nextcloud service is operational and the VM has approximately 500 GiB of virtual disk capacity.

A separate, tested backup strategy should be implemented before treating the service as the only copy of important personal data.

## Recommended Backup Layers

### 1. Proxmox VM Backup

Create scheduled backups of the Nextcloud VM from Proxmox.

The backup should be stored separately from the running VM where practical.

### 2. Application Data Backup

Back up the Nextcloud data directory and configuration as appropriate for the deployment.

### 3. Database Backup

Back up the Nextcloud MariaDB database regularly. A file backup alone is not a substitute for a consistent database backup.

### 4. Off-Site Copy

Important data should eventually have a second physical or off-site copy. This protects against disk failure, accidental deletion, and loss of the primary machine.

## Recovery Testing

A backup should be considered useful only after restoration has been tested.

Future recovery test:

1. Restore the VM or rebuild a test VM.
2. Restore Nextcloud configuration.
3. Restore the database.
4. Restore user data.
5. Verify application integrity.
6. Log the recovery result and any issues found.

## Future Improvement

Implement a documented 3-2-1 backup strategy:

- 3 copies of important data
- 2 different storage locations/media
- 1 copy stored off-site
