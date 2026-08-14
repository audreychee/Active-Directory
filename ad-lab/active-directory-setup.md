# Active Directory Lab Setup Guide

This guide walks through setting up a Windows Server 2022 Active Directory Domain Controller (DC), joining a Windows 11 Enterprise client machine, and installing Remote Server Administration Tools (RSAT).

---

## Part 1: Virtual Machine & Remote Management Setup

### 1. Base Virtual Machine Provisioning
1. **Setup Domain Controller:** Create a new VM and install **Windows Server 2022**.
2. **Setup Domain Client:** Create a second VM and install **Windows 11 Enterprise**.
3. **Disable SConfig Auto-Launch:** (Optional) Configure Windows Server to prevent `sconfig` from launching automatically at logon.
4. **Initial Snapshot:** Take a clean snapshot of the Windows DC VM prior to domain configuration.
5. **(Optional) Base Template Creation:** 
   * Create cloneable templates for quick lab deployment without repeating initial OS installation.
   * *Path:* `Right-Click VM` -> `Manage` -> `Clone`.
6. **VMware Tools Installation:**
   * Install VMware Tools on the Windows Server 2022 DC VM.
   * Repeat installation for the Windows 11 Workstation VM.
   * Take post-installation snapshots of both VMs.

---

### 2. Configure Remote Management & RSAT

#### Step A: Configure PowerShell Remoting (WinRM)
On the **Windows Server (DC)**, enable PowerShell Remoting and retrieve the server IP address:

```powershell
Enable-PSRemoting
ipconfig
```

On the **Management Workstation (Client)**, start WinRM, add the DC IP to TrustedHosts, and open a remote session:

```powershell
# Start WinRM Service
Start-Service WinRM

# Add Windows Server IP to Trusted Hosts (Replace IP with your DC's IP)
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "192.168.29.135"

# Establish Remote PowerShell Session
New-PSSession -ComputerName 192.168.29.135 -Credential (Get-Credential)
Enter-PSSession 1
```

> **Note on WinRM vs RSAT:** WinRM / PSRemoting provides a command-line terminal session directly on the target server. RSAT (Remote Server Administration Tools) installs graphical management consoles and local AD PowerShell cmdlets on your management workstation.

#### Step B: Install RSAT on Management Workstation
To manage Active Directory and Group Policy natively from your Workstation without logging directly into the DC desktop, run the following in an elevated PowerShell session on your **client machine**:

```powershell
# Install Active Directory Administrative Tools (ADUC, ADAC, ActiveDirectory Module)
Add-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"

# Install Group Policy Management Tools (GPMC)
Add-WindowsCapability -Online -Name "Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0"
```
**Troubleshooting Connection Errors in `dsa.msc`:
** If `dsa.msc` opens with a red "X" or displays the error *"Naming information cannot be located because: The specified domain either does not exist or could not be contacted"*, target the Domain Controller IP directly:
  1.  Click **OK** to dismiss the error dialog inside `dsa.msc`.
  2.  Right-click **Active Directory Users and Computers** at the top-left of the console.
  3.  Select **Change Domain Controller...**
  4.  Select **Type a Domain Controller name or IP address**, enter your DC's static IP (`192.168.29.135`), and click **OK**.

Once installed, you can launch management consoles directly from your workstation:
* **`dsa.msc`** -> Active Directory Users and Computers (ADUC)
* **`gpmc.msc`** -> Group Policy Management Console (GPMC)

#### Step C: Set Computer Hostname
Rename the server using `sconfig`:
* Launch `sconfig` -> Select Option **2** (Computer Name) -> Set new hostname (e.g., `WinSvr-DC1`).

---

## Part 2: Active Directory Domain Services (AD DS) Setup

### 1. Configure Static IP & Loopback DNS on DC
> **Crucial Rule:** Domain Controllers **must** have a static IP address; dynamic DHCP assignment is prohibited for DCs.

Configure the network adapter with a static IP and point its DNS server to itself (`127.0.0.1` / local IP):

```powershell
# Set static IP, Subnet Mask, and Gateway
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.29.135 -PrefixLength 24 -DefaultGateway 192.168.29.2

# Set DNS to local loopback
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("127.0.0.1")
```

Verify configuration remotely from the management workstation.

---

### 2. Install Active Directory Domain Services (AD DS)

Run the following commands on the server to install the AD DS role and deploy a new forest:

```powershell
# Install AD DS Role and Management Tools
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Import ADDS Deployment Module
Import-Module ADDSDeployment

# Install New Active Directory Forest (Example Domain: blue.com)
Install-ADDSForest -DomainName "blue.com"
```

---

### 3. Post-Promotion DNS Verification
After AD promotion completes and the server reboots, verify that the active DNS server address is pointing to the DC's static IP address (`192.168.29.135`):

```powershell
# View DNS Server configuration
Get-DnsClientServerAddress

# Force DNS server to DC IP on Interface Index 6 (adjust index as needed)
Set-DnsClientServerAddress -InterfaceIndex 6 -ServerAddresses "192.168.29.135"
```

Take a VM snapshot of the configured Domain Controller (`WinSvr-DC1`).

---

### 4. Join Windows 11 Workstation (`WS01`) to Domain

1. Boot the Windows 11 Workstation (`WS01`).
2. Update DNS settings on `WS01` to point to the Domain Controller IP (`192.168.29.135`):

```powershell
# Check network adapter details
Get-NetIPAddress
Get-DnsClientServerAddress

# Point Client DNS to the Domain Controller IP
Set-DnsClientServerAddress -InterfaceIndex 3 -ServerAddresses "192.168.29.135"
```

3. Join `WS01` to the `blue.com` domain:

```powershell
Add-Computer -DomainName "blue.com" -Credential blue\Administrator -Force -Restart
```

4. Take a VM snapshot of `WS01` after successfully joining the domain.
