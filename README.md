# 🧱 Active Directory Lab (VirtualBox) — Step-by-Step

Build your own mini Windows domain at home using **Oracle VirtualBox**, **Windows Server 2019**, and **Windows 10**.

You'll set up:
- A **Domain Controller** (AD DS + DNS)
- **DHCP** for IP assignment
- **NAT with RRAS** for Internet access
- **Bulk user creation** with PowerShell

---

## 🗺️ Topology

Familiarize yourself with the lab layout:  
📈 [Active Directory Topology Diagram](https://github.com/JoshMcleods/Active-Directory-Lab/blob/main/Active%20Directory%20Topology%20Diagram.png)

---

## ✅ Prerequisites

- **VirtualBox**: https://www.virtualbox.org/wiki/Downloads  
- **Windows Server 2019 ISO**: https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019  
- **Windows 10 ISO**: https://www.microsoft.com/en-au/software-download/windows10iso  
- **Host System**:
  - Recommended: 16 GB RAM
  - Minimum: 8 GB (with minimal VM specs)

---

## 🖥️ 1. Create the Server 2019 VM (DC1)

**VirtualBox Settings:**
- OS Type: Windows 2019 (64-bit)
- CPU/RAM: 2 vCPU / 4–6 GB RAM
- Disk: 60 GB (VDI, dynamically allocated)

**Network Adapters:**
- Adapter 1: **NAT** (Internet access)
- Adapter 2: **Internal Network** — name it `INTNET`

**Install Windows Server 2019** using the ISO.

### 🔧 Post-Install NIC Configuration:

Rename interfaces (optional):
- Adapter 1 → `WAN`
- Adapter 2 → `LAN`

Configure the **LAN adapter**:
- IP 172.16.0.1
- Mask 255.255.255.0
- DNS 172.168.0.1
- Leave gateway blank on LAN NIC.
- The **WAN** adapter should use DHCP.

---

## 🏢 2. Install AD DS & Create Domain

1. Server Manager → *Add Roles and Features* → **Active Directory Domain Services**
2. Post-install: Click yellow flag → *Promote this server to a domain controller*
3. Choose: **Add a new forest**
   - Root domain: `mydomain.com`
4. Set a DSRM password → Accept defaults → Install & Reboot
5. Sign in as: `LAB\Administrator`

---

## 🌐 3. Verify DNS Setup

DNS is auto-installed with AD DS.

- Open **DNS Manager**
- Confirm:
  - A **Forward Lookup Zone** for `mydomain.com`
  - An **A record** for `DC1`

---

## 🌍 4. Enable NAT via RRAS

1. Server Manager → Add Roles → **Remote Access** → **Routing**
2. Open **Routing and Remote Access**
3. Right-click `DC1` → *Configure and Enable RRAS*
4. Choose: **NAT**
5. Configure:
   - **External Interface** = `WAN`
   - **Internal Interface** = `LAN`
6. Complete wizard.  
Clients in `172.16.0.1/24` will now route Internet traffic via `DC1`.

---

## 📦 5. Install & Configure DHCP

1. Server Manager → Add Roles → **DHCP Server**
2. Open **DHCP Console**
3. Right-click IPv4 → *New Scope*:
- Name: mydomain.com
- Range: 172.16.0.100 – 172.16.0.200
- Mask: 255.255.255.0
- Router (gateway): 172.168.0.1
- DNS: 172.16.0.1

4. Authorize the DHCP server if prompted

---

## 🗂️ 6. (Optional) Create OU Structure

Using **Active Directory Users and Computers (ADUC)**:

- Create:
  - `OU=Workstations`
  - `OU=Users`

---

## 👥 7. Bulk Create Users via PowerShell

1. Copy the CSV (`names.txt`) next to the script:

Import Script: https://github.com/JoshMcleods/Active-Directory-Lab/blob/main/AD_Powershell%20Files/Create_Users.ps1 

Run from an elevated Windows PowerShell on DC1:
Set-ExecutionPolicy RemoteSigned -Scope Process -Force
.\nCreate_Users.ps1

---

## 8. Build a Windows 10 client (Win10-01)
New VM → Windows 10, 2 vCPU / 4 GB RAM, 30 GB disk.

Network: Internal Network = INTNET (only one NIC).

Install Windows 10 normally.

After first boot, run ipconfig — you should receive an address from 172.16.0.100–200.

Test:

ping 172.16.0.1 (DC1)

nslookup mydomain.com

ping 8.8.8.8 and ping google.com (tests NAT + DNS)

---

## 9. Join the client to the domain
GUI method
On Win10‑01: Settings → System → About → Rename this PC (advanced) → Change → Domain = mydomain.com


Enter domain creds (e.g., LAB\Administrator).


Reboot.


PowerShell method
Add-Computer -DomainName mydomain.com -Credential LAB\Administrator -Restart

Log in as one of the newly created users to verify domain auth works.

---

## 10. Nice next steps
Create a GPO to set a desktop wallpaper or disable local admin.

Add another client (Win10‑02) and test roaming logons.

Wire up Windows Event Forwarding or a SIEM (e.g., free Splunk/Elastic) to ingest security logs.

Take VirtualBox snapshots at key milestones.

Troubleshooting quick hits
Client has APIPA (169.254.x.x) → DHCP scope not authorized or wrong NIC on INTNET.

Client resolves names but no Internet → RRAS NAT not enabled / wrong external interface.

Client can’t resolve mydomain.com → Client DNS must be 172.168.0.1 only.

Client cannot find domain → misconfigured static ip of domain controller (typo)

