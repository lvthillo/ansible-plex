# Ansible Media Server

Automated Proxmox media server: Plex LXC + Docker arr-stack VM.

## Prerequisites

- Proxmox VE installed
- SSH key access to Proxmox host (`ssh-copy-id root@192.168.0.200`)
- Ansible 2.15+ installed locally
- ProtonVPN account (paid plan for port forwarding + WireGuard)
- OpenSubtitles.com account

### Proxmox host preparation

The Proxmox host needs the no-subscription repo enabled (enterprise repos require a paid subscription):

```bash
# Disable enterprise repos
mv /etc/apt/sources.list.d/pve-enterprise.sources /etc/apt/sources.list.d/pve-enterprise.sources.disabled
mv /etc/apt/sources.list.d/ceph.sources /etc/apt/sources.list.d/ceph.sources.disabled

# Add no-subscription repo
cat > /etc/apt/sources.list.d/pve-no-subscription.sources << 'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
Enabled: yes
EOF

# Verify
apt update
```

### ProtonVPN WireGuard key

1. Go to https://account.protonvpn.com → **Downloads** → **WireGuard configuration**
2. Platform: **GNU/Linux**
3. Enable **NAT-PMP (Port Forwarding)**, Moderate NAT, VPN Accelerator
4. Select any server → **Create** → **Download** the `.conf` file
5. Copy the `PrivateKey` value (the key works for all ProtonVPN servers)

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

## Post-install: manual steps

### Plex

Open `http://<plex-lxc-ip>:32400/web` and log in with your Plex account to claim the server.

### Bazarr

Open `http://<docker-vm-ip>:6767` and configure:

1. **Sonarr** — Settings → Sonarr → Enable
   - Address: `http://sonarr`
   - Port: `8989`
   - API Key: copy from Sonarr → Settings → General

2. **Radarr** — Settings → Radarr → Enable
   - Address: `http://radarr`
   - Port: `7878`
   - API Key: copy from Radarr → Settings → General

3. **Languages** — Settings → Languages
   - Add New Profile → Name: `Dutch`
   - Add `Dutch` (primary) + `English` (fallback)
   - Default Settings for Series: `Dutch`
   - Default Settings for Movies: `Dutch`

4. **Providers** — Settings → Providers → + → OpenSubtitles.com
   - Username + Password → Test → Save

5. **Subtitle sync** — Settings → Subtitles → Subtitle Auto-Synchronization
   - Enable: on
   - Always use Audio Track as Reference: on

6. **Authentication** (optional) — Settings → General → Security
   - Authentication: Forms
   - Set username/password

## Cleanup (destructive!)

```bash
ansible-playbook cleanup.yml -e ansible_confirm_cleanup=true
```
