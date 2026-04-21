# Ansible Media Server

Automated Proxmox media server: Plex LXC + Docker arr-stack VM.

## Prerequisites

- Proxmox VE installed
- SSH key access to Proxmox host (`ssh-copy-id root@192.168.0.200`)
- Ansible 2.15+ installed locally
- ProtonVPN WireGuard key
- OpenSubtitles.com account

## Setup

1. Install collections:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

2. Create vault with secrets:
   ```bash
   ansible-vault create group_vars/vault.yml
   ```
   Add:
   ```yaml
   vault_vpn_wireguard_private_key: "your-key"
   vault_bazarr_opensubtitles_username: "your-username"
   vault_bazarr_opensubtitles_password: "your-password"
   ```

3. Run:
   ```bash
   ansible-playbook site.yml -e plex_claim_token=claim-xxxx --ask-vault-pass
   ```

## Cleanup (destructive!)

```bash
ansible-playbook cleanup.yml -e ansible_confirm_cleanup=true
```
