# Active Directory Lab Setup Guide

This guide walks through setting up a Windows Server 2022 Active Directory Domain Controller (DC) and joining a Windows 11 Enterprise client machine.

---

## Part 1: Virtual Machine & Remote Management Setup

### 1. Base Virtual Machine Provisioning
1. **Setup Domain Controller:** Create a new VM and install **Windows Server 2022**.
2. **Setup Domain Client:** Create a second VM and install **Windows 11 Enterprise**.
3. **Disable SConfig Auto-Launch:** (Optional) Configure Windows Server to prevent `sconfig` from launching automatically at logon.
4. **Initial Snapshot:** Take a clean snapshot of the Windows DC VM prior to domain configuration.
5. **(Optional) Base Template Creation:** 
   * Create cloneable templates for quick lab deployment without repeating initial OS installation.
   * *Path:* `Right-Click VM` $\rightarrow$ `Manage` $\rightarrow$ `Clone`.
6. **VMware Tools Installation:**
   * Install VMware Tools on the Windows Server 2022 DC VM.
   * Repeat installation for the Windows 11 Workstation VM.
   * Take post-installation snapshots of both VMs.

---

### 2. Configure Remote PowerShell Management

Set up remote administration from the Management Client Workstation to the Domain Controller.

#### Step A: On Windows Server (DC)
Enable PowerShell Remoting and retrieve the server IP address:

```powershell
Enable-PSRemoting
ipconfig
```

#### Step B: On Management Workstation (Client)
Start the WinRM service, add the Server IP to TrustedHosts, and open a remote PowerShell session:

```powershell
# Start WinRM Service
Start-Service WinRM

# Add Windows Server IP to Trusted Hosts (Replace IP with your DC's IP)
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "192.168.29.135"

# Establish Remote PowerShell Session
New-PSSession -ComputerName 192.168.29.135 -Credential (Get-Credential)
Enter-PSSession 1
```

#### Step C: Set Computer Hostname
Rename the server using `sconfig`:
* Launch `sconfig` $\rightarrow$ Select Option **2** (Computer Name) $\rightarrow$ Set new hostname (e.g., `WinSvr-DC1`).

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
