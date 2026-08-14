# Active Directory Security & Deployment Lab

This repository documents the practical step-by-step setup, administration, and auditing of an Active Directory Domain Services (AD DS) lab environment. It is tailored for **SOC Analysts and Security Engineers**, focusing on core identity management, security baselines, and detection readiness.

---

## 📑 Lab Modules & Roadmap

### 🟢 Phase 1: Foundation & Provisioning
* **`01-theory-and-architecture.md`**  
  * Active Directory hierarchy (Forests, Domains, Trees)
  * Core protocols (LDAP, Kerberos, NTLM, DNS)
  * Domain boundaries, trust concepts, and FSMO roles overview
* **`02-lab-setup-and-remoting.md`**  
  * Windows Server 2022 DC deployment (`blue.com`) & static IP configuration
  * WinRM & PowerShell Remoting setup
  * Workstation (`WS01`) domain-joining
  * Non-domain Management Client setup with RSAT (`dsa.msc`, `gpmc.msc`) via `runas /netonly`

---

### 🟢 Phase 2: Core Object Administration & GPOs
* **`03-core-object-management.md`**  
  * Designing a tiered Organizational Unit (OU) structure (e.g., Tier 0 / Tier 1 / Tier 2)
  * User account provisioning and Service Account configuration
  * Managing Security Groups (Domain Admins, Local Admins) vs. Distribution Groups
  * Understanding Group Scopes (Domain Local, Global, Universal)
* **`04-group-policy-management.md`**  
  * Creating and linking Group Policy Objects (GPOs)
  * GPO processing order (LSDOU: Local, Site, Domain, OU) & Enforce settings
  * Deploying security hardening baselines via GPO

---

### 🟢 Phase 3: Forest Infrastructure & High-Priority AD Security
* **`05-forest-and-domain-infrastructure.md`**
  * Active Directory Schema: Registering schmmgmt.dll, inspecting object classes & attributes, and understanding Schema Admins boundaries.
  * Global Catalog (GC): Inspecting GC placement via dssite.msc, Partial Attribute Set (PAS), and Universal Group membership caching.
  * Operations Master (FSMO) Roles: Understanding the 5 FSMO roles (PDC Emulator, Schema Master, etc.) and inspecting role placement via PowerShell.
* **`06-ad-certificate-services.md`** *(Critical SOC Target)*
  * Active Directory Certificate Services (AD CS) role installation & Enterprise CA setup
  * Certificate template management & Web Enrollment configuration
  * Key revocation procedures and inspecting certificate identity mappings

---

### 🟢 Phase 4: Auditing, Threat Detection & Recovery
* **`07-auditing-and-recovery.md`**  
  * Enabling Advanced Security Audit Policies (Logon events, Account Creation, Group Modifications)
  * Mapping Windows Event IDs for threat detection (e.g., `4624`, `4720`, `4738`)
  * Enabling and using the Active Directory Recycle Bin for incident recovery
