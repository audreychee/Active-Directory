# Active Directory Fundamentals

## 1. What is Active Directory (AD)?
Active Directory is a centralized platform used to manage a business's network resources, including computers, user accounts, and other network assets.

## 2. What is the Purpose of AD?
Active Directory simplifies administrative tasks with a centralized management system and enhances organizational security. 

* **Single Sign-On (SSO):** A user authenticates once to access all authorized resources within their domain.
* **Resource Sharing:** Shared folders can be accessed across domains, facilitating seamless collaboration and business continuity.

---

## 3. Key Concepts & Terminology

### a. Forest
A **Forest** is the highest organizational boundary in Active Directory, consisting of a collection of domain trees that share a two-way trust relationship.
* **Schema:** A forest shares a single, common schema.
* **Intra-Forest Migration:** Moving objects between domains within the same forest is permitted by default as they share a security boundary.
* **Inter-Forest Migration:** Accessing objects across different forests is not trusted by default and requires manual trust configuration.

### b. Tree
A **Tree** is a hierarchical collection of one or more domains sharing a contiguous namespace.
* **Structure:** Consists of a parent domain (e.g., `ETS.local`) and child domains (e.g., `PR.ETS.local`, `Sales.ETS.local`).
* **Naming:** Child domains append their name to the parent domain name.

### c. Domain
A **Domain** is a logical group of network objects (user accounts, computer accounts, groups) that share a common security database, administration, and centralized authentication (SSO).
> **Note:** A domain client uses domains temporarily when requiring server access (e.g., browsing websites), whereas a child domain permanently appends its name in AD to establish domain trust.

### d. Organizational Unit (OU)
An **OU** is a container within a domain that acts like a folder for organizing objects.
* Used to apply **Group Policy Objects (GPOs)** (enforcing user/computer security rules).
* Used for delegating administrative authority and pushing software deployments.

### e. Objects
An **Object** is a distinct entity representing a physical or logical network resource (e.g., Users, Computers, Security Groups). Each object possesses unique attributes (e.g., a user account object has attributes like `Name`, `Email Address`, `sAMAccountName`).

### f. Domain Controller (DC)
A **Domain Controller** is a server running Windows Server with Active Directory Domain Services (AD DS) installed.
* **Redundancy:** DCs are typically deployed in pairs or clusters to prevent a single point of failure.
* **Replication:** Multi-master replication ensures changes made on one DC are synchronized across all other DCs in the domain boundary.

### g. Global Catalog (GC)
A **Global Catalog** is a distributed data repository containing a searchable, full copy of all objects in its home domain and a partial copy of all objects in all other domains within the forest.
* Avoids multi-domain query lag by allowing fast object lookups across the entire forest.

### h. Schema
The **Schema** is the formal definition/blueprint of all object classes and attributes that can be created within the Active Directory forest. Each forest contains exactly one schema.

---

## 4. How Active Directory Operates
Active Directory follows a strict hierarchical tree-and-forest structure:

Forest -> Tree -> Domain -> Organizational Unit (OU) -> Objects

This hierarchy enables centralized administrative control, scalable security boundary delegation, and efficient authentication across enterprise environments.
