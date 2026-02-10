## 1. What is Active Directory and why it’s used

**Active Directory (AD)** is Microsoft’s directory service for centralized identity, authentication, and authorization in a Windows environment.

- **Directory:**  
  Stores objects like users, computers, groups, printers, and security policies.
- **Authentication:**  
  Verifies who you are (Kerberos/NTLM).
- **Authorization:**  
  Decides what you can access (files, shares, apps, GPOs).
- **Centralization:**  
  One place to manage accounts, passwords, policies, and access across the network.

**Why it’s used:**

- **Centralized management** of users, computers, and permissions.  
- **Single Sign-On (SSO)** for domain-joined devices.  
- **Group Policy** for security baselines and configuration.  
- **Integration** with many enterprise apps, file servers, print servers, etc.

---

## 2. Advantages and drawbacks of Active Directory

### Advantages

- **Centralized identity:**  
  One account per user for all domain resources.
- **Security & control:**  
  Strong policies, password rules, account lockout, delegation.
- **Scalability:**  
  Works from small labs to large enterprises with multiple sites.
- **Group Policy:**  
  Massive power to configure and harden clients and servers.
- **Integration with DNS:**  
  Service discovery via SRV records, AD-integrated zones.

### Drawbacks

- **Complexity:**  
  Forests, domains, trusts, sites, FSMO roles—needs proper design.
- **Dependency on DNS:**  
  Bad DNS = broken AD.
- **Single security boundary:**  
  If a domain/forest is compromised, impact is huge.
- **Maintenance overhead:**  
  Backups, replication health, DC patching, time sync, etc.
- **On-prem focus:**  
  Needs servers, storage, and care (though it can integrate with cloud).

---

## 3. Forest, domains, and functional levels

### Forest

- **Forest:**  
  The top-level security boundary in AD.  
  It contains one or more domains that share:
  - A common schema  
  - A global catalog  
  - Trust relationships  
- Usually, modern environments use **one forest, one domain** unless there’s a strong reason otherwise.

### Domain

- Logical partition inside a forest.  
- Has its own:
  - Domain controllers  
  - Policies  
  - Security groups  
- Namespace example: `corp.local`, `lab.domain.com`.

### Functional levels

Functional levels define which **AD features** are available, based on the **minimum Windows Server version** of domain controllers.

- **Forest Functional Level (FFL):**  
  Minimum OS level for all DCs in the forest.
- **Domain Functional Level (DFL):**  
  Minimum OS level for all DCs in that domain.

On **Windows Server 2022**, you typically choose:

- **Domain functional level:** Windows Server 2016 or higher  
- **Forest functional level:** Windows Server 2016 or higher  

(There is no separate “2022” functional level; 2016 is the latest as of now.)   

Higher functional levels unlock features and drop support for older DC versions.

---

## 4. What is AD DS (Active Directory Domain Services)

**AD DS** is the **server role** that implements Active Directory on Windows Server.

- Installed as a **role** on Windows Server.  
- Turns the server into a **Domain Controller (DC)**.  
- Provides:
  - Authentication (Kerberos/NTLM)  
  - Directory database (NTDS.DIT)  
  - Replication between DCs  
  - Integration with DNS  
  - Group Policy processing  

So: **Active Directory** is the concept/service; **AD DS** is the Windows Server role that runs it.

---

## 5. Prerequisites to install AD DS on Windows Server 2022

Before promoting a server to a Domain Controller:

- **OS:**
  - Windows Server 2022 (Desktop Experience for GUI).
- **Hostname:**
  - Set a proper name (e.g., `DC01`) before promotion.
- **IP configuration:**
  - **Static IP** (no DHCP for DC).  
  - Preferred DNS: usually itself (its own IP) or an existing DC if joining an existing domain.
- **Time & timezone:**
  - Correct time is critical for Kerberos.
- **Network:**
  - Reachability to clients and other DCs (if any).
- **Credentials:**
  - For **new forest**: local Administrator on that server.  
  - For **new domain in existing forest**: Enterprise Admins.   
- **Disk & resources (lab minimum):**
  - 2 vCPU, 4–8 GB RAM, 60+ GB disk (more in production).
- **Security:**
  - Server should not be exposed directly to the internet.  
  - Prefer running on a secure ESXi/vSphere or similar environment.

---

## 6. Step-by-step: Install AD DS on Windows Server 2022 (GUI)

This is for **creating a new forest** (e.g., `lab.local`) on a fresh Windows Server 2022 with Desktop Experience. Steps are based on the standard Server Manager workflow.   

### Step 0 – Basic prep

- **Rename server:**
  - Open **Server Manager → Local Server → Computer Name → Change**  
  - Set name: e.g., `DC01`  
  - Reboot.
- **Set static IP:**
  - Control Panel → Network and Internet → Network Connections  
  - Right-click your NIC → Properties → IPv4  
  - Set:
    - IP: e.g., `192.168.10.10`  
    - Subnet: `255.255.255.0`  
    - Gateway: your router/firewall  
    - DNS: `192.168.10.10` (itself; will host DNS after promotion).

---

### Step 1 – Add the AD DS role

1. Open **Server Manager** (starts automatically on login).
2. Click **Manage → Add Roles and Features**.
3. **Before You Begin:** Next.
4. **Installation Type:**  
   - Choose **Role-based or feature-based installation** → Next.
5. **Server Selection:**  
   - Select your server (e.g., `DC01`) → Next.
6. **Server Roles:**  
   - Check **Active Directory Domain Services**.  
   - A dialog appears to add required features → click **Add Features**.  
   - Click **Next**.
7. **Features:**  
   - Default is fine → Next.
8. **AD DS page:**  
   - Read info → Next.
9. **Confirmation:**  
   - Click **Install**.  
   - Wait for completion (no reboot yet).

---

### Step 2 – Promote the server to a Domain Controller

Once the role is installed:

1. In **Server Manager**, you’ll see a yellow triangle notification:  
   **“Promote this server to a domain controller”**.  
   Click it.
2. **Deployment Configuration:**
   - Choose **Add a new forest**.  
   - Root domain name: e.g., `lab.local`.
   - Click **Next**.
3. **Domain Controller Options:**
   - **Forest functional level:** Windows Server 2016.  
   - **Domain functional level:** Windows Server 2016.  
   - Check **Domain Name System (DNS) server** (if this is first DC).  
   - Check **Global Catalog (GC)** (default).  
   - Do **not** check RODC for first DC.  
   - Set **DSRM password** (Directory Services Restore Mode).  
   - Click **Next**.
4. **DNS Options:**
   - You may see a warning about delegation (ignore for first DC).  
   - Click **Next**.
5. **Additional Options:**
   - NetBIOS name will be suggested (e.g., `LAB`).  
   - Accept or adjust → Next.
6. **Paths:**
   - Database, log files, SYSVOL paths (default: `C:\Windows\NTDS`, `C:\Windows\SYSVOL`).  
   - For lab, defaults are fine → Next.
7. **Review Options:**
   - Confirm settings.  
   - You can optionally view the PowerShell script (good for automation later).
8. **Prerequisites Check:**
   - Wait for checks to complete.  
   - Warnings are often normal (e.g., DNS delegation).  
   - If no errors, click **Install**.

The server will **automatically reboot** after promotion.

---

### Step 3 – Verify AD DS and DNS

After reboot:

1. Log in using **domain credentials**:
   - `LAB\Administrator` (or `lab\Administrator`) with the same password as before.
2. Open **Server Manager → Tools**:
   - **Active Directory Users and Computers**  
   - **DNS**
3. Check:
   - Domain appears (e.g., `lab.local`).  
   - Default containers: `Users`, `Computers`, `Domain Controllers`.  
   - In DNS, under **Forward Lookup Zones**, you see `lab.local` and `_msdcs.lab.local` with SRV records.

At this point, your **first Domain Controller + DNS** is ready. You can now:

- Join Windows clients to the domain.  
- Create OUs, users, and groups.  
- Start designing Group Policies.

