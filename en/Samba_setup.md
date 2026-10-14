# Samba 4 Active Directory Domain Controller

**Complete installation and configuration guide**

Two Ubuntu 24.04.4 LTS VMs • AD DC server • Domain-joined graphical client

Samba 4 AD DC • Kerberos • Internal DNS • realmd • SSSD • Nautilus

---

## Table of Contents

1. [Understanding the Architecture](#1-understanding-the-architecture)
2. [VM Specifications](#2-vm-specifications)
3. [Server Preparation (VM 1)](#3-server-preparation-vm-1)
4. [Provisioning Samba as an Active Directory DC](#4-provisioning-samba-as-an-active-directory-dc)
5. [Managing AD Users and Shares](#5-managing-ad-users-and-shares)
6. [Client Configuration (VM 2)](#6-client-configuration-vm-2)
7. [Accessing Samba Shares from the Client](#7-accessing-samba-shares-from-the-client)
8. [Day-to-Day Active Directory Administration](#8-day-to-day-active-directory-administration)
9. [Troubleshooting](#9-troubleshooting)
10. [Cheat Sheet — Essential Commands](#10-cheat-sheet--essential-commands)

---

## 1. Understanding the Architecture

Before typing the first command, it's essential to understand why Active Directory works the way it does. It's not just an LDAP directory — it's a set of three protocols working together, and ignoring any one of them is the main source of failures during installation.

### 1.1 The Three Pillars of Active Directory

An Active Directory domain rests on three inseparable services.

**First, DNS.** Every machine in the domain locates things through special DNS records called SRV records. Without working DNS, clients cannot find the domain controller, full stop. This is the number one cause of domain-join failures.

**Second, Kerberos.** This is AD's authentication protocol. When a user types their password, the password itself never travels over the network — Kerberos exchanges encrypted "tickets" with a limited lifetime. This makes interception pointless, because an expired ticket is worthless.

**Third, LDAP.** This is the directory protocol that stores and exposes the list of users, groups, computers and policies. When SSSD looks up a user's information, it speaks LDAP. But authentication itself goes through Kerberos, not LDAP.

### 1.2 What Samba 4 AD DC Does

Samba 4 in domain controller mode (AD DC) implements all three protocols above simultaneously in a single service. From the clients' point of view, it behaves exactly like a Windows Server 2008 R2 — an Ubuntu, Windows 10 or Windows 11 client can't tell the difference.

The important difference from the previous guide: you no longer install slapd separately. Samba 4 AD DC includes its own internal LDAP instance (Heimdal for Kerberos, and an LDB-based LDAP). Never install slapd on the same server — the two would conflict on port 389.

### 1.3 Architecture Diagram

The domain will be called MONLABO.LOCAL in this guide. The `.local` suffix is fine for labs — in production you use a real, registered domain name. Every reference to MONLABO.LOCAL must be replaced consistently if you choose a different name.

| **VM 1 - AD DC SERVER**<br><br>Ubuntu 24.04.4 Server (no GUI)<br><br>Static IP: 192.168.56.10<br><br>Hostname: dc1.monlabo.local<br><br>**Samba 4 AD DC**<br><br>Domain's internal DNS<br><br>Kerberos (KDC)<br><br>Active Directory LDAP<br><br>Samba shares (SYSVOL, NETLOGON, Shared) | ⟷ | **VM 2 - DOMAIN CLIENT**<br><br>Ubuntu 24.04.4 Desktop (GNOME)<br><br>Static IP: 192.168.56.20<br><br>Hostname: poste01.monlabo.local<br><br>**realmd + SSSD**<br><br>AD domain join<br><br>Kerberos authentication<br><br>Share access via Nautilus |
| --- | --- | --- |

> **ℹ Note:** Both VMs must share a Host-Only network in VirtualBox (vboxnet0, subnet 192.168.56.0/24). Each VM also has a NAT adapter to reach the internet during installation.

---

## 2. VM Specifications

These specifications are calibrated for Ubuntu 24.04. The Desktop client needs more RAM than on 22.04, because GNOME 46 (included in 24.04) is more demanding than GNOME 42.

### 2.1 VM 1 — Active Directory Server (AD DC)

| **Parameter** | **Recommended value** | **Notes** |
| --- | --- | --- |
| Operating system | Ubuntu 24.04.4 LTS Server | ISO: ubuntu-24.04.4-live-server-amd64.iso — no graphical interface |
| RAM | 2 GB minimum / 4 GB recommended | Samba AD DC + Kerberos + DNS uses more than a plain slapd |
| CPU | 2 vCPUs | 1 vCPU is enough in a lab but slows down Kerberos provisioning |
| Disk | 30 GB, dynamically allocated | The AD LDAP database and SYSVOL can grow over time |
| Network adapter 1 | NAT | To download packages during installation |
| Network adapter 2 | Host-Only — vboxnet0 | Internal VM↔VM network — must be in 192.168.56.0/24 |
| Hostname | dc1 | The FQDN will be dc1.monlabo.local |
| Static IP (adapter 2) | 192.168.56.10/24 | Configured after installation — critical for AD DNS |
| VirtualBox chipset | PIIX3 or ICH9 | ICH9 sometimes gives enp0s8 instead of enp0s3 — check with `ip link` |

### 2.2 VM 2 — Domain-Joined Graphical Client

| **Parameter** | **Recommended value** | **Notes** |
| --- | --- | --- |
| Operating system | Ubuntu 24.04.4 LTS Desktop | ISO: ubuntu-24.04.4-desktop-amd64.iso — includes GNOME 46 |
| RAM | 4 GB minimum / 6 GB recommended | GNOME 46 on 24.04 uses ~1.5 GB at idle |
| CPU | 2 vCPUs | Minimum for a smooth interface |
| Disk | 30 GB, dynamically allocated | Leave room for user profiles |
| Network adapter 1 | NAT | To download packages |
| Network adapter 2 | Host-Only — vboxnet0 | Same network as the server |
| Video memory | 128 MB minimum / 256 MB recommended | Increase if the screen flickers or GNOME crashes |
| 3D acceleration | Enabled (VBoxSVGA) | Improves GNOME 46 performance |
| Hostname | poste01 | FQDN: poste01.monlabo.local |
| Static IP (adapter 2) | 192.168.56.20/24 | Must use dc1 (192.168.56.10) as its DNS |

---

## 3. Server Preparation (VM 1)

This section covers installing Ubuntu Server 24.04, configuring the network, and updating the system before installing Samba. These preparation steps are critical — a misconfigured IP or hostname will make the AD DC unusable.

### 3.1 Installing Ubuntu Server 24.04

When booting from the ISO, the text-mode installer (Subiquity) guides you step by step. Here are the important choices to make.

- Choose "Ubuntu Server" (not the minimized version) to include all the network utilities.
- At the network step, configure only the NAT adapter (enp0s3) with DHCP for now — the static IP comes later.
- Set the hostname to `dc1`.
- Create a local administrator user (e.g. `srvadmin`) — this account is separate from AD users.
- Enable the OpenSSH server if you want to administer the server from your host machine (handy for copy-pasting commands).
- Leave everything else at its default and finish the installation.

### 3.2 Configuring the Static IP on the Host-Only Interface

On Ubuntu 24.04 Server, Netplan is still the default network manager, with systemd-networkd as the backend. The configuration is identical to 22.04, but you first need to identify the exact names of the interfaces.

#### Step 1 — Identify the network interfaces

```bash
ip link show

# You should see two interfaces (besides lo):
# - enp0s3 (or similar) → NAT adapter, already configured via DHCP
# - enp0s8 (or enp0s9)  → Host-Only adapter, to be configured
```

#### Step 2 — Create the Netplan file for the Host-Only interface

```bash
sudo nano /etc/netplan/99-hostonly.yaml
```

File contents (careful: YAML uses spaces, never tabs):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:                  # Replace with your interface name if different
      dhcp4: false
      addresses:
        - 192.168.56.10/24
      nameservers:
        addresses: [127.0.0.1]   # The server is its own DNS once AD is configured
```

```bash
sudo chmod 600 /etc/netplan/99-hostonly.yaml   # Netplan requires strict permissions
sudo netplan apply
ip addr show enp0s8                            # Should show 192.168.56.10/24
```

> **⚠ Important:** The `nameservers: addresses: [127.0.0.1]` line points to the server itself. It will only work once Samba AD DC is started and configured as the DNS. For the initial installation you can temporarily use 8.8.8.8, then switch back to 127.0.0.1 afterwards.

#### Step 3 — Configure the hostname and the hosts file

This is a critical step. The FQDN (Fully Qualified Domain Name) must be resolvable locally even before AD DNS is active, otherwise the Samba provisioning will fail.

```bash
sudo hostnamectl set-hostname dc1.monlabo.local

# Check:
hostname       # Should return: dc1.monlabo.local
hostname -s    # Should return: dc1
hostname -d    # Should return: monlabo.local

sudo nano /etc/hosts
```

Make sure these two lines are present (and ONLY these two for 192.168.56.10), and delete any other line that contains dc1 or monlabo:

```
127.0.0.1 localhost
192.168.56.10 dc1.monlabo.local dc1
```

Test local resolution:

```bash
getent hosts dc1.monlabo.local   # Should return: 192.168.56.10
```

### 3.3 Updating and Installing Dependencies

```bash
sudo apt update && sudo apt full-upgrade -y

# Packages needed before Samba:
sudo apt install -y acl attr build-essential libacl1-dev libattr1-dev \
  libblkid-dev libgnutls28-dev libreadline-dev python3-dev python3-dnspython \
  python3-gpg python3-markdown python3-packaging python3-subunit \
  xsltproc dnsutils krb5-user samba winbind libnss-winbind \
  libpam-winbind ntp ntpdate
```

During the installation of krb5-user, three questions are asked:

- **Kerberos realm** → `MONLABO.LOCAL` (in UPPERCASE, mandatory)
- **Kerberos server** → `dc1.monlabo.local`
- **Admin server** → `dc1.monlabo.local`

> **ℹ Note:** The Kerberos realm MUST be uppercase (MONLABO.LOCAL). This is a strict Kerberos convention, not a preference. A lowercase realm will cause cryptic authentication errors later.

---

## 4. Provisioning Samba as an Active Directory DC

Provisioning is the step that initializes the Active Directory database. It creates the domain, configures Kerberos, generates the DNS zones, and prepares SYSVOL (the share used to distribute group policies). It's a one-time operation — doing it again means wiping everything and starting from scratch.

### 4.1 Stop and Disable Conflicting Services

Samba AD DC includes its own DNS server and its own LDAP implementation. Before provisioning, make sure no existing service tries to occupy the same ports.

```bash
# Stop potentially conflicting services:
sudo systemctl stop smbd nmbd winbind
sudo systemctl disable smbd nmbd winbind

# On Ubuntu 24.04, systemd-resolved listens on port 53 by default.
# Samba AD DC needs this port for its built-in DNS — so you must free it:
sudo systemctl disable --now systemd-resolved

# Temporarily reconfigure DNS resolution (until AD starts):
sudo rm /etc/resolv.conf     # This is a symlink to systemd-resolved
echo 'nameserver 8.8.8.8' | sudo tee /etc/resolv.conf
echo 'nameserver 1.1.1.1' | sudo tee -a /etc/resolv.conf
```

> **⚠ Important:** Disabling systemd-resolved is a step specific to Ubuntu 24.04 for AD servers. Skip it and Samba can't start its internal DNS, which means clients will never find the domain controller.

### 4.2 Remove the Existing Samba Configuration

Ubuntu installs Samba with a default smb.conf. The provisioning command overwrites this file, but it's better to remove it manually to avoid any leftovers.

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak

# Also clear any old Samba database files if present:
sudo find /var/lib/samba -name '*.ldb' -delete
sudo find /var/lib/samba -name '*.tdb' -delete
```

### 4.3 Run the Active Directory Domain Provisioning

The `samba-tool domain provision` command creates the entire AD domain. Each option is explained below. Run this command in one go (the backslashes let the command continue on the next line in the terminal).

```bash
sudo samba-tool domain provision \
  --use-rfc2307 \
  --realm=MONLABO.LOCAL \
  --domain=MONLABO \
  --server-role=dc \
  --dns-backend=SAMBA_INTERNAL \
  --adminpass='YourAdminPassword123!'

# Explanations:
# --use-rfc2307                → Enables POSIX attributes (uidNumber, gidNumber)
#                                Essential for Linux clients to read UIDs/GIDs
# --realm                      → Kerberos realm name in UPPERCASE
# --domain                     → The domain's NetBIOS name (max 15 characters, no dots)
# --server-role=dc             → This server is a domain controller
# --dns-backend=SAMBA_INTERNAL → Samba manages its own DNS (recommended for labs)
# --adminpass                  → Password of the AD Administrator account
#                                Must contain uppercase, lowercase, digits, symbols
```

> **✔ Tip:** Memorize or carefully note the AD Administrator password — it's the domain's equivalent of 'root'. It's used to create all other users and to join machines to the domain.

If provisioning succeeds, you'll see a summary with the domain information. Samba will have created a new smb.conf suited to the AD DC, as well as the SYSVOL structure in `/var/lib/samba/sysvol/`.

### 4.4 Configure Kerberos

Provisioning created a Kerberos configuration file in `/var/lib/samba/private/krb5.conf`. It needs to be copied to the standard location expected by system utilities.

```bash
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf

# Check: the file should look like this:
cat /etc/krb5.conf
# [libdefaults]
#   default_realm = MONLABO.LOCAL
#   dns_lookup_realm = false
#   dns_lookup_kdc = true
```

### 4.5 Point DNS at the Server Itself

Now that Samba is ready to start its internal DNS, reconfigure resolv.conf so the server uses its own AD DNS, which lets it resolve the domain's SRV records.

```bash
sudo bash -c 'cat > /etc/resolv.conf << EOF
search monlabo.local
nameserver 192.168.56.10
EOF'

# Protect the file from automatic overwriting:
sudo chattr +i /etc/resolv.conf

# To modify the file later: sudo chattr -i /etc/resolv.conf
```

### 4.6 Start and Enable Samba AD DC

On Ubuntu 24.04, the Samba AD DC service is called `samba-ad-dc` — this is different from `smbd`, which is standalone Samba. These two services must not run at the same time.

```bash
sudo systemctl unmask samba-ad-dc
sudo systemctl start samba-ad-dc
sudo systemctl enable samba-ad-dc

# Check the status:
sudo systemctl status samba-ad-dc
# Should show: active (running)
```

### 4.7 Post-Provisioning Validation Tests

These tests are essential — don't move on to the next section if any of them fails. Each test checks a different pillar of AD.

#### Test 1 — Domain functional level

```bash
sudo samba-tool domain level show
# Expected: Domain Level: 2008 R2 (or higher)
#           Forest Level: 2008 R2
```

#### Test 2 — Internal DNS (SRV records)

```bash
host -t SRV _kerberos._udp.monlabo.local
# Expected: _kerberos._udp.monlabo.local has SRV record ... dc1.monlabo.local.

host -t SRV _ldap._tcp.monlabo.local
# Expected: _ldap._tcp.monlabo.local has SRV record ... dc1.monlabo.local.

host -t A dc1.monlabo.local
# Expected: dc1.monlabo.local has address 192.168.56.10
```

#### Test 3 — Kerberos authentication

```bash
kinit administrator@MONLABO.LOCAL
# Enter the AD Administrator password

klist
# Expected: ticket with 'krbtgt/MONLABO.LOCAL@MONLABO.LOCAL'

kdestroy    # Clean up the test ticket
```

#### Test 4 — LDAP access

```bash
samba-tool user list
# Should list: Administrator, Guest, krbtgt
```

#### Test 5 — Share access

```bash
smbclient -L localhost -U Administrator
# Should show: SYSVOL, NETLOGON
```

---

## 5. Managing AD Users and Shares

Now that the domain is operational, you'll create users and groups in Active Directory, then configure a Samba share accessible only to members of an AD group.

### 5.1 Create Users in AD

All AD users are managed with `samba-tool` on the server. The syntax is similar to the Active Directory PowerShell tools on Windows Server.

```bash
# Create a user 'alice' with POSIX attributes (UID/GID):
sudo samba-tool user create alice 'AlicePassword123!' \
  --given-name='Alice' \
  --surname='Dupont' \
  --mail-address='alice@monlabo.local' \
  --uid-number=10001 \
  --gid-number=10001 \
  --home-directory='/home/alice' \
  --login-shell='/bin/bash'

# Create a second user:
sudo samba-tool user create bob 'BobPassword123!' \
  --given-name='Bob' \
  --surname='Martin' \
  --uid-number=10002 \
  --gid-number=10001 \
  --home-directory='/home/bob' \
  --login-shell='/bin/bash'

# List the users:
sudo samba-tool user list
```

> **ℹ Note:** The `--uid-number` and `--gid-number` attributes (RFC 2307) are available thanks to the `--use-rfc2307` flag used during provisioning. Without them, Linux clients couldn't map AD users to real Unix UIDs.

### 5.2 Create AD Groups

```bash
# Create the 'staff' group:
sudo samba-tool group add staff \
  --gid-number=10001 \
  --nis-domain=monlabo

# Add alice and bob to the staff group:
sudo samba-tool group addmembers staff alice
sudo samba-tool group addmembers staff bob

# Verify:
sudo samba-tool group listmembers staff
```

### 5.3 Configure a Samba File Share

The smb.conf generated by provisioning only contains SYSVOL and NETLOGON. You'll add a 'shared' share accessible to the staff group.

```bash
# Create the shared folder with the right permissions:
sudo mkdir -p /srv/samba/shared
sudo chgrp staff /srv/samba/shared   # Owning group = AD staff
sudo chmod 2770 /srv/samba/shared    # setgid: new files inherit the group

# Enable POSIX ACLs on the filesystem:
sudo tune2fs -l /dev/sda1 | grep 'Default mount'
# If 'acl' is not listed, add acl to the mount options in /etc/fstab

sudo nano /etc/samba/smb.conf
```

Add this at the END of the file (don't modify the existing sections):

```ini
[shared]
    comment = Common share - Staff group
    path = /srv/samba/shared
    valid users = @staff
    read only = no
    browseable = yes
    create mask = 0660
    directory mask = 0770
    force group = staff
```

```bash
sudo systemctl restart samba-ad-dc

# Access test from the server itself:
smbclient //dc1.monlabo.local/shared -U alice
# Enter alice's password → should show the prompt: smb: \>
```

---

## 6. Client Configuration (VM 2)

You'll now configure the Ubuntu Desktop 24.04 client to join the Active Directory domain. The realmd tool makes this remarkably simple — it automatically discovers the domain via DNS and configures SSSD, PAM and NSS in a single command.

### 6.1 Installing Ubuntu Desktop 24.04

- Boot the VM from the Ubuntu 24.04.4 Desktop ISO.
- Choose "Normal installation" to include GNOME and all the standard applications.
- Set the hostname to `poste01`.
- Create a local administrator user (e.g. `localadmin`) — separate from AD accounts.
- Finish the installation and boot into the GNOME desktop.

### 6.2 Configure the Static IP and DNS

On the Desktop 24.04 client, NetworkManager manages the network by default. You can configure the static IP directly from GNOME, which is cleaner than writing a Netplan file by hand (the two would coexist and could contradict each other).

#### Via the GNOME graphical interface (recommended method)

- Open Settings → Network.
- Click the gear next to the Host-Only interface (enp0s8).
- Go to the IPv4 tab.
- Choose the 'Manual' method.
- Address: 192.168.56.20 — Netmask: 255.255.255.0 — Gateway: (empty).
- In the DNS field, enter only: 192.168.56.10 (the domain controller's address).
- Click Apply, then turn the interface off and on again.

> **⚠ Important:** The client's DNS MUST point to the domain controller (192.168.56.10), not to 8.8.8.8. It's the AD server that resolves the domain's names. Without this, realmd won't find the domain.

#### Checking connectivity

```bash
# From a terminal on the client:
ping -c 3 192.168.56.10                # Ping the AD server
host monlabo.local 192.168.56.10       # AD DNS must answer
host dc1.monlabo.local 192.168.56.10   # Should return 192.168.56.10
```

#### Update /etc/hosts

```bash
echo '192.168.56.10 dc1.monlabo.local dc1' | sudo tee -a /etc/hosts
```

### 6.3 Install the Required Packages

```bash
sudo apt update
sudo apt install -y realmd sssd sssd-tools sssd-ad \
  adcli krb5-user samba-common-bin packagekit \
  libnss-sss libpam-sss oddjob oddjob-mkhomedir
```

During the installation of krb5-user:

- **Default realm** → `MONLABO.LOCAL` (UPPERCASE)
- **KDC server** → `dc1.monlabo.local`
- **Admin server** → `dc1.monlabo.local`

### 6.4 Discovering and Joining the Domain with realmd

realmd is a service that automates joining an AD domain. It queries DNS to find the domain's SRV records, negotiates with the Kerberos KDC, and configures SSSD automatically.

#### Step 1 — Discover the domain

```bash
sudo realm discover monlabo.local

# Expected output:
# monlabo.local
#   type: kerberos
#   realm-name: MONLABO.LOCAL
#   domain-name: monlabo.local
#   configured: no
#   server-software: active-directory
#   client-software: sssd
```

If discovery fails, it's a DNS problem — check that the client really uses 192.168.56.10 as its DNS and that samba-ad-dc is running on the server.

#### Step 2 — Join the domain

```bash
sudo realm join --user=Administrator monlabo.local
# Enter the AD Administrator password when prompted

# Check the join:
sudo realm list
# Should show: configured: kerberos-member
```

> **✔ Tip:** The `realm join` command does everything automatically: it creates a machine account in AD (`poste01$`), generates a complete /etc/sssd/sssd.conf, configures /etc/nsswitch.conf, and enables PAM. This is one of realmd's big advantages over manual configuration.

### 6.5 Configure Automatic Home Directory Creation

By default, when an AD user logs in for the first time, their /home directory doesn't exist yet. The oddjobd service automatically creates this directory at first login.

```bash
sudo pam-auth-update --enable mkhomedir
sudo systemctl enable --now oddjobd

# Alternative if oddjob causes problems: configure PAM manually
# sudo nano /etc/pam.d/common-session
# Add at the end: session optional pam_mkhomedir.so skel=/etc/skel umask=077
```

### 6.6 Fine-Tune the SSSD Configuration

realmd generates a working sssd.conf, but a few tweaks make it nicer to use day to day, notably for the login syntax (typing 'alice' rather than 'alice@monlabo.local').

```bash
sudo nano /etc/sssd/sssd.conf
```

The important settings to check or add in the `[domain/monlabo.local]` section:

```ini
[sssd]
domains = monlabo.local
config_file_version = 2
services = nss, pam

[domain/monlabo.local]
default_shell = /bin/bash
krb5_store_password_if_offline = true
cache_credentials = true
krb5_realm = MONLABO.LOCAL
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u
ad_domain = monlabo.local
use_fully_qualified_names = false   # Lets you type 'alice' instead of 'alice@monlabo.local'
ldap_id_mapping = false             # Uses the RFC 2307 UIDs/GIDs we defined
access_provider = ad
```

```bash
sudo chmod 600 /etc/sssd/sssd.conf   # SSSD refuses to start if permissions are wrong
sudo systemctl restart sssd
```

### 6.7 Validation Tests on the Client

#### Test 1 — Resolving AD users

```bash
getent passwd alice
# Expected: alice:*:10001:10001:Alice Dupont:/home/alice:/bin/bash

id alice
# Expected: uid=10001(alice) gid=10001(staff) groups=10001(staff),...
```

#### Test 2 — Kerberos authentication from the client

```bash
kinit alice@MONLABO.LOCAL
klist
# Should show a valid Kerberos ticket for alice
```

#### Test 3 — Graphical login with an AD account

On the GDM login screen (the GNOME lock screen), there is a 'Not listed?' field at the bottom. Enter 'alice' as the username and her AD password. GNOME should create /home/alice and open the session.

> **ℹ Note:** If the graphical login fails but `id alice` works in the terminal, the problem often comes from PAM. Check with: `sudo pam-auth-update --force`

---

## 7. Accessing Samba Shares from the Client

There are two ways to access the shares from the client: through the Nautilus graphical interface (for users), and from the command line with a permanent mount in /etc/fstab (for administrators or automation needs).

### 7.1 Graphical Access via Nautilus

GNOME Files (Nautilus) natively supports the SMB protocol. Access is particularly smooth when the user is logged in with their AD account, because SSSD can pass the Kerberos credentials automatically.

- Open the Files application (Nautilus) from the GNOME dock.
- In the left sidebar, click Other Locations at the very bottom.
- In the 'Connect to Server' bar at the bottom, type: `smb://dc1.monlabo.local/shared`
- Click Connect. In the authentication dialog:
  - Choose: Registered User
  - Domain: MONLABO
  - Username: alice
  - Password: alice's AD password
- The share opens like a local folder. To bookmark it: Ctrl+D.

> **✔ Tip:** If Nautilus doesn't offer the SMB option, install the gvfs backend: `sudo apt install -y gvfs-backends gvfs-fuse`, then log out and back in to the GNOME session.

### 7.2 Permanent Mount via /etc/fstab with Kerberos Authentication

For an automatic mount at boot that uses a Kerberos ticket rather than a plaintext password in a file, use the `sec=krb5` option of cifs-utils. This is the most secure method.

```bash
sudo apt install -y cifs-utils

# Create the mount point:
sudo mkdir -p /mnt/shared

# Add to /etc/fstab:
sudo nano /etc/fstab
```

Samba mount with Kerberos authentication (add at the end):

```
//dc1.monlabo.local/shared /mnt/shared cifs sec=krb5,uid=10001,gid=10001,_netdev,nofail 0 0
```

Alternative with a credentials file if Kerberos causes problems:

```
//dc1.monlabo.local/shared /mnt/shared cifs credentials=/etc/samba/creds-alice,_netdev,nofail 0 0
```

If you use the credentials option, create the file:

```bash
sudo bash -c 'cat > /etc/samba/creds-alice << EOF
username=alice
password=AlicePassword123!
domain=MONLABO
EOF'

sudo chmod 600 /etc/samba/creds-alice

# Test the mount without rebooting:
sudo mount -a
ls /mnt/shared
```

> **⚠ Important:** The `_netdev` option is essential — it tells the system to wait for the network to be available before mounting this share. Without it, the VM can hang at boot if the server isn't reachable yet.

---

## 8. Day-to-Day Active Directory Administration

Once the domain is in place, here are the common operations you'll perform regularly with `samba-tool` on the server.

### 8.1 User Management

```bash
# List all users:
sudo samba-tool user list

# View a user's details:
sudo samba-tool user show alice

# Reset a password:
sudo samba-tool user setpassword alice

# Disable an account (without deleting it):
sudo samba-tool user disable alice

# Re-enable an account:
sudo samba-tool user enable alice

# Delete a user:
sudo samba-tool user delete alice
```

### 8.2 Group Management

```bash
# List all groups:
sudo samba-tool group list

# List a group's members:
sudo samba-tool group listmembers staff

# Add a member to a group:
sudo samba-tool group addmembers staff bob

# Remove a member from a group:
sudo samba-tool group removemembers staff bob
```

### 8.3 Managing Domain Member Machines

```bash
# List the machines joined to the domain:
sudo samba-tool computer list

# Remove a machine from the domain (from the server):
sudo samba-tool computer delete POSTE01$

# To cleanly remove a machine from the client side:
sudo realm leave monlabo.local
```

### 8.4 Backing Up Active Directory

Backing up a Samba AD is done with `samba-tool domain backup`. It's the recommended method because it handles the transactional aspects of the LDB database.

```bash
# Create a full AD backup:
sudo samba-tool domain backup online \
  --targetdir=/var/backups/samba \
  --server=dc1.monlabo.local \
  -U Administrator

# Schedule a daily backup with cron:
echo '0 3 * * * root samba-tool domain backup online --targetdir=/var/backups/samba --server=dc1.monlabo.local -U Administrator --password=YourAdminPassword123!' | sudo tee /etc/cron.d/samba-backup
```

---

## 9. Troubleshooting

Problems with Active Directory often follow a causal chain: DNS → Kerberos → LDAP → SSSD → Samba. Always work in that order when diagnosing — a DNS problem makes debugging everything else pointless.

| **Symptom** | **Probable cause** | **Solution** |
| --- | --- | --- |
| `realm discover` can't find the domain | Client DNS misconfigured | Check that 192.168.56.10 is the client's DNS and that samba-ad-dc is running on the server |
| `realm join` fails with 'Insufficient permissions' | Wrong Administrator password | Test with: `kinit administrator@MONLABO.LOCAL` on the client |
| `getent passwd alice` returns nothing | `use_fully_qualified_names = true` in sssd.conf | Test with: `getent passwd alice@monlabo.local` or change it to false |
| Can't log in to GNOME with an AD account | PAM or oddjob not configured | Run: `sudo pam-auth-update --force`, check oddjobd |
| samba-ad-dc won't start | systemd-resolved still active on port 53 | `sudo systemctl disable --now systemd-resolved` and retry |
| Kerberos tickets expire too quickly | Clock drift between server and client | Install ntp on both VMs: `sudo apt install -y ntp` |
| Nautilus can't see the SMB shares | gvfs-backends missing | `sudo apt install -y gvfs-backends` then log back in to GNOME |
| fstab blocks the boot | `_netdev` option missing | Add `_netdev` and `nofail` to the fstab mount options |
| SSSD caches stale data | SSSD cache not refreshed | `sudo sss_cache -E` then `sudo systemctl restart sssd` |

### 9.1 Essential Diagnostic Commands

#### On the server

```bash
sudo systemctl status samba-ad-dc
sudo samba-tool domain level show
sudo samba-tool drs showrepl               # Check replication
sudo journalctl -u samba-ad-dc -f          # Real-time logs
host -t SRV _kerberos._udp.monlabo.local   # DNS SRV test
testparm                                   # Validate smb.conf
```

#### On the client

```bash
sudo realm list                            # Join status
sudo systemctl status sssd
sudo sss_cache -E                          # Clear the SSSD cache
sudo journalctl -u sssd -f                 # Real-time SSSD logs
id alice                                   # Test resolving an AD user
kinit alice@MONLABO.LOCAL && klist         # Test Kerberos auth
```

---

## 10. Cheat Sheet — Essential Commands

| **Action** | **Command (on the server unless noted)** |
| --- | --- |
| DC status | `sudo systemctl status samba-ad-dc` |
| Restart Samba AD | `sudo systemctl restart samba-ad-dc` |
| List AD users | `sudo samba-tool user list` |
| Create a user | `sudo samba-tool user create alice 'Pass123!' --uid-number=10003 ...` |
| Reset a password | `sudo samba-tool user setpassword alice` |
| Disable an account | `sudo samba-tool user disable alice` |
| List groups | `sudo samba-tool group list` |
| Add to a group | `sudo samba-tool group addmembers staff alice` |
| List the domain's machines | `sudo samba-tool computer list` |
| Kerberos SRV DNS test | `host -t SRV _kerberos._udp.monlabo.local` |
| Kerberos ticket test (server) | `kinit administrator@MONLABO.LOCAL && klist` |
| Test user resolution (client) | `getent passwd alice` (on the client) |
| Clear SSSD cache (client) | `sudo sss_cache -E` (on the client) |
| Join the domain (client) | `sudo realm join --user=Administrator monlabo.local` |
| Leave the domain (client) | `sudo realm leave monlabo.local` |
| Full AD backup | `sudo samba-tool domain backup online --targetdir=/var/backups ...` |

---

*— End of guide —*

Ubuntu 24.04.4 LTS • Samba 4 AD DC • Kerberos • realmd • SSSD
