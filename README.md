# Home Lab — Kali VM (VirtualBox)

**Owner:** Freddy Morgan Smith  
**Purpose:** Personal home lab for learning cyber security / digital forensics and producing portfolio writeups.

---

## Summary (what we completed today)
- Created a Kali Linux VM from the **x86_64 installer (Graphical)** in VirtualBox 7.x  
- VM specs:
  - Name: `Kali`
  - CPU: 4 vCPUs
  - RAM: 8 GB
  - Disk: 40 GB VDI (dynamically allocated)
  - Desktop: XFCE
- Networking:
  - **Adapter 1:** NAT (internet access) → `eth0` in guest
  - **Adapter 2:** Host‑Only (isolated lab network) → `eth1` in guest
  - Host (Windows) host‑only IP: `192.168.56.1`
  - Kali host‑only IP (static): `192.168.56.10`
- Installed VirtualBox **Guest Additions**
- Enabled shared folders & clipboard (optional)
- Enabled SSH on Kali
- Created snapshots:
  - `baseline` (clean install + networking)
  - `post-guest-additions` (after GA + extras)

---

## Goals / Design decisions
- Use **NAT + Host‑Only** combination:
  - NAT for safe internet access on the VM (downloads/updates).
  - Host‑Only for isolated lab traffic (VM ↔ VM ↔ Host) without exposing vulnerable targets to the home LAN.
- Use XFCE for a lightweight desktop.
- Use a static IP on the host-only network so lab IPs are predictable for testing and documentation.

---

## How to reproduce:

### VirtualBox (create VM)
1. `New` → Name: `Kali` → Type: Linux → Version: Debian (64-bit)  
2. RAM: `8192 MB`  
3. CPUs: `4`  
4. Create disk: VDI, Dynamically allocated, `40 GB`  

### VM settings (before boot)
- **System → Motherboard**: EFI disabled  
- **System → Processor**: 4 CPUs  
- **Display**: Video Memory `64–128 MB`  
- **Storage**: attach `kali-x86_64-installer.iso` to optical drive  
- **Network**:
  - Adapter 1 → `Attached to: NAT` (Adapter Type: Intel PRO/1000 MT Desktop)
  - Adapter 2 → `Attached to: Host‑Only Adapter` (choose the VirtualBox Host‑Only adapter bound to `192.168.56.1`)

### Install Kali (Graphical)
- Boot VM → choose **Graphical install**  
- Language / Location / Keyboard → Hostname: `kali-lab`  
- Create user: `kali_user` (use a lab-only password)  
- Partitioning: **Guided — use entire disk** → All files in one partition (no LVM, no encryption)  
- Desktop: **XFCE**; include **Top 10 & default tools** (optional)
- Install GRUB to /dev/sda → finish → reboot → unmount ISO

---

## Post‑install configuration (commands run in Kali)
- Run these after first login in a terminal.

- Update + essentials
```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt -y autoremove
sudo apt install -y build-essential dkms linux-headers-$(uname -r) \
  net-tools iputils-ping nmap wireshark tcpdump openssh-server isc-dhcp-client
```
## Bring up interfaces (NetworkManager)
# ensure both interfaces are up
```bash
sudo nmcli device connect eth0    # NAT (internet)
# create persistent static host-only connection
sudo nmcli connection add type ethernet ifname eth1 con-name hostonly \
  ipv4.addresses 192.168.56.10/24 ipv4.method manual
sudo nmcli connection up hostonly
```
### Verify networking
```bash
ip -o -4 addr show
ip route
ping -c 3 8.8.8.8
ping -c 3 google.com
ping -c 3 192.168.56.1
```
### Install Guest Additions
```bash
# in VirtualBox GUI: Devices -> Insert Guest Additions CD Image...
sudo apt install -y build-essential dkms linux-headers-$(uname -r)   # prerequisites
sudo mkdir -p /mnt/cdrom
sudo mount /dev/cdrom /mnt/cdrom
cd /mnt/cdrom
sudo ./VBoxLinuxAdditions.run
sudo reboot
```
### Enable SSH
```bash
sudo systemctl enable --now ssh
# then SSH from host:
# ssh kali-lab@192.168.56.10
```


