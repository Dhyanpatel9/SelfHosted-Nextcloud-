# Security Notes

## Security Objectives

The goal is to expose only the services required for remote Nextcloud access while reducing common brute-force and unauthorized-access risks.

## Controls Used

### SSH

SSH is used for administrative access to the Ubuntu server. Fail2Ban monitors SSH authentication activity.

Verification:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### Fail2Ban

The `sshd` jail was enabled and verified during the project. Fail2Ban provides automated temporary bans after repeated authentication failures.

### UFW

Ubuntu's firewall is used to control inbound traffic. Rules should be limited to the ports and services actually required by the deployment.

### HTTPS / TLS

The public Nextcloud service uses HTTPS. TLS certificates are managed using Let's Encrypt.

### Secrets Management

Credentials, private keys, API tokens, database passwords, and other secrets must never be committed to this repository.

## Security Verification Approach

Security controls are checked using direct service/status commands rather than assuming a configuration change succeeded.

Examples:

```bash
sudo systemctl status ssh
sudo fail2ban-client status sshd
sudo ufw status verbose
```

## Future Security Improvements

- Move SSH administration to key-based authentication.
- Review and minimize exposed ports.
- Enable automated security updates where appropriate.
- Add centralized logging and monitoring.
- Test backup restoration regularly.
- Document incident-response and recovery procedures.
