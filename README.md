# Evil-WinRM Lab: Kali ↔ Windows VM (VirtualBox)

> Practice Windows remote management and post-exploitation techniques **safely** in your own offline lab using Evil-WinRM.

![status](https://img.shields.io/badge/status-stable-brightgreen) ![platform](https://img.shields.io/badge/platform-Kali%20%7C%20Windows-blue) ![hypervisor](https://img.shields.io/badge/hypervisor-VirtualBox-orange)

## ✨ Features

* Step-by-step **VirtualBox networking** so VMs can talk to each other
* One-shot **Windows WinRM enable** script (HTTP, lab-only)
* **Kali usage** cheatsheet for Evil-WinRM (password & hash)
* Connectivity **health checks** (ping / nc / WinRM listener)
* Comprehensive **troubleshooting** for common errors (e.g., `ENETUNREACH`)
* Security **disclaimer** & safe-lab defaults

---

## 📚 Table of Contents

1. [Lab Topology](#-lab-topology)
2. [Prerequisites](#-prerequisites)
3. [VirtualBox Networking Setup](#-virtualbox-networking-setup)
4. [Windows Target Configuration](#-windows-target-configuration)
5. [Kali Attacker Setup & Usage](#-kali-attacker-setup--usage)
6. [Connectivity Checks](#-connectivity-checks)
7. [Troubleshooting](#-troubleshooting)
8. [Scripts](#-scripts)
9. [FAQ](#-faq)
10. [Security Notes](#-security-notes)
11. [License](#-license)

---

## 🗺️ Lab Topology

* **Attacker**: Kali Linux (VM)
* **Target**: Windows 10/11 or Server 2016/2019/2022 (VM)
* **Network**: Host-Only (recommended) or Bridged
* **Protocol**: WinRM over HTTP (`TCP 5985`, *lab-only*)

> In NAT mode, VMs **cannot** reach each other by default—use Host-Only or Bridged.

---

## ✅ Prerequisites

* Oracle **VirtualBox** (or VMware—similar steps)
* **Kali** VM (2023+ recommended)
* **Windows** VM (local admin access)
* Internet for Kali package install (optional if already installed)

---

## 🔌 VirtualBox Networking Setup

**Recommended: Host-Only + (optional NAT for Internet)**

1. **Shut down** both VMs.
2. VirtualBox → **File → Tools → Network Manager**:

   * Ensure **Host-Only Network** exists (e.g., `vboxnet0`) with DHCP enabled.
3. For **Kali VM** → **Settings → Network**:

   * **Adapter 1**: `Host-Only Adapter` (e.g., `vboxnet0`)
   * **Adapter 2** *(optional)*: `NAT` (for updates)
4. For **Windows VM** → **Settings → Network**:

   * **Adapter 1**: `Host-Only Adapter` (same `vboxnet0`)
   * **Adapter 2** *(optional)*: `NAT`

> Alternative: **Bridged Adapter** for both VMs to be on your LAN (e.g., `192.168.x.x`). Do **not** mix modes unintentionally.

---

## Step 1: Install Evil-WinRM

On Kali Linux (or Ubuntu):
```

sudo apt update
sudo apt install evil-winrm -y
```


Check:
```

evil-winrm -h
```

## Step 2: Configure WinRM on the Target Windows VM

By default, WinRM is often disabled. On your Windows target VM, open PowerShell (as Administrator) and run:
```

winrm quickconfig
```


Then allow basic authentication:
```

Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
```


Enable unencrypted traffic (only in lab, not production):
```

Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true
```


Also ensure firewall allows WinRM (TCP 5985):
```

netsh advfirewall firewall add rule name="WinRM" dir=in action=allow protocol=TCP localport=5985
```

## Step 3: Add a Test User

On Windows VM, create a test account:
```

net user eviltest Passw0rd! /add
net localgroup administrators eviltest /add
```


(In real-world, you wouldn’t be admin, but for practice this helps.)

## Step 4: Connect from Kali with Evil-WinRM

From your Kali machine:

```
evil-winrm -i <Windows_VM_IP> -u eviltest -p 'Passw0rd!'
```


If successful, you’ll land in a PowerShell session on the target.

## Step 5: Practice Common Commands

Inside Evil-WinRM, you can:

Run PowerShell commands:
```

whoami
hostname
ipconfig
```


Upload/download files:
```

upload /path/to/file C:\Users\eviltest\Desktop\file.txt
download C:\Windows\System32\drivers\etc\hosts /tmp/hosts
```


Run scripts:
```

Invoke-Expression -Command (Get-Content script.ps1 | Out-String)
```

## 🛠️ Troubleshooting

### 1) `ping: connect: Network is unreachable` (Kali)

**Cause:** VMs aren’t on the same L2/L3 network (NAT isolation).
**Fix:** Use **Host-Only** or **Bridged** on both VMs. Confirm both IPs are in the same subnet (e.g., `192.168.56.0/24`).

### 2) Evil-WinRM: `Errno::ENETUNREACH` or `connect(...) port 5985`

**Cause:** Same as above—no route or firewall.
**Fix:**

* VirtualBox adapters → both Host-Only or both Bridged
* On Kali: `nc -vz <IP> 5985`
* On Windows: open firewall and ensure listener exists.

### 3) PowerShell: *“WinRM firewall exception will not work… network connection type is Public”*

**Cause:** Network profile is **Public**.
**Fix:**

```powershell
Set-NetConnectionProfile -InterfaceAlias "Ethernet" -NetworkCategory Private
```

Then re-run WinRM steps.

### 4) No listener on 5985

```powershell
winrm quickconfig -q
Restart-Service WinRM
winrm enumerate winrm/config/listener
netstat -ano | findstr 5985
```

### 5) Creds rejected / account locked

* Verify user & password (`net user eviltest`)
* Ensure not expired/locked
* Try local admin first for lab simplicity

### 6) Path completion warning in Evil-WinRM

```
Warning: Remote path completions is disabled...
```

Harmless. Functionality unaffected.

---

## ❓ FAQ

**Q: Why not NAT?**
A: VirtualBox NAT isolates VMs—Kali cannot directly reach Windows. Use Host-Only or Bridged.

**Q: Can I use HTTPS (5986)?**
A: Yes, but requires certificate setup. For quick lab practice, HTTP is fine (offline).

**Q: VMware steps?**
A: Use a **Host-Only** network (VMnet1) or **Bridged** for both VMs; the remaining steps are identical.

**Q: Do I need admin privileges?**
A: For initial WinRM enable and firewall changes—yes. For learning post-ex, admin is convenient.

---

## 🔐 Security Notes

* **Lab-only** settings (Basic auth + AllowUnencrypted) are intentionally insecure for simplicity.
* Keep the lab **offline** or restrict to a Host-Only network.
* Never point Evil-WinRM at systems you don’t own or have explicit permission to test.

