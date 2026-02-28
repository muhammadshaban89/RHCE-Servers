
Here is **full, clean, detailed, professional guide** for building a **Windows Server 2022 office file‑sharing server** with **upload‑only (no edit/no delete)** permissions.

Office File‑Sharing Server Architecture Diagram
-----------------

                           ┌──────────────────────────────┐
                           │      Internet (Ignored)       │
                           │   No VPN / No Cloud Access    │
                           └──────────────────────────────┘
                                         │
                                         │
                               (Local LAN Only)
                                         │
                                         ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           Office Local Network (LAN)                         │
│                                                                              │
│   ┌──────────────────────┐         ┌──────────────────────┐                 │
│   │   User PC #1         │         │   User PC #2         │                 │
│   │  (FS_Users group)    │  ...    │  (FS_Users group)    │   ...           │
│   │  Upload / Read Only  │         │  Upload / Read Only  │                 │
│   └──────────────────────┘         └──────────────────────┘                 │
│            │                                   │                            │
│            └─────────────── LAN Switch / Router ────────────────┐           │
│                                                                  │           │
└──────────────────────────────────────────────────────────────────┼───────────┘
                                                                   │
                                                                   ▼
                     ┌──────────────────────────────────────────────┐
                     │         Windows Server 2022 (OFFICE-SERVER)   │
                     │  Static IP: 192.168.1.10                      │
                     │                                              │
                     │  ┌────────────────────────────────────────┐  │
                     │  │              File Services             │  │
                     │  │  Shared Folder: D:\CompanyData         │  │
                     │  │  Share Name: \\OFFICE-SERVER\CompanyData│ │
                     │  └────────────────────────────────────────┘  │
                     │                                              │
                     │  ┌────────────────────────────────────────┐  │
                     │  │          Security Structure            │  │
                     │  │                                        │  │
                     │  │  Groups:                               │  │
                     │  │   - FS_Admins → Full Control           │  │
                     │  │   - FS_Users  → Upload + Read Only     │  │
                     │  │                                        │  │
                     │  │  NTFS Rules:                           │  │
                     │  │   - Allow: Read, Write, Create Files   │  │
                     │  │   - Deny: Delete, Modify, Rename       │  │
                     │  └────────────────────────────────────────┘  │
                     │                                              │
                     │  ┌────────────────────────────────────────┐  │
                     │  │            Shadow Copies               │  │
                     │  │   Automatic snapshots for recovery     │  │
                     │  └────────────────────────────────────────┘  │
                     └──────────────────────────────────────────────┘

# **Complete Step‑by‑Step Guide: Windows Server 2022 Office File‑Sharing Server**  
**Goal:**  
- 15 office users (same LAN)  
- Upload / read / download allowed  
- **No edit**  
- **No delete**  
- Only admin has full control  
- No VPN / no cloud  

---

## **1. Install Windows Server 2022**
1. Boot from the Windows Server USB installer.  
2. Choose **Windows Server 2022 Standard (Desktop Experience)**.  
3. Install on the SSD (recommended for OS).  
4. Set the Administrator password.  
5. After first login, rename the server:

**Path:**  
`Server Manager → Local Server → Computer Name → Change`

**Name:**  
`OFFICE-SERVER`

Restart the server.

---

## **2. Configure Static IP (Critical Step)**
**Path:**  
`Control Panel → Network and Sharing Center → Change adapter settings → Ethernet → IPv4`

**Example configuration:**
```
IP Address: 192.168.1.10
Subnet Mask: 255.255.255.0
Gateway: 192.168.1.1
DNS: 192.168.1.1
```

A static IP ensures all users can always reach the server.

---

## **3. Create Local User Accounts**
**Path:**  
`Computer Management → Local Users and Groups → Users`

Create one user per employee:

Examples:
```
user1
user2
user3
```

Uncheck:
```
User must change password at next logon
```

---

## **4. Create Security Groups**
**Path:**  
`Computer Management → Local Users and Groups → Groups`

Create two groups:

| Group Name   | Purpose |
|--------------|---------|
| FS_Admins    | Full control |
| FS_Users     | Regular employees |

Add:
- All employees → **FS_Users**  
- IT/Admin → **FS_Admins**

---

## **5. Create the Shared Folder**
Create a data folder on the data disk (D: recommended):

```
D:\CompanyData
```

---

## **6. Configure Share Permissions**
Right‑click the folder → **Properties → Sharing → Advanced Sharing**

1. Check **Share this folder**  
2. Share name:
```
CompanyData
```
3. Click **Permissions**

Remove:
```
Everyone
```

Add:

| Group       | Share Permission |
|-------------|------------------|
| FS_Admins   | Full Control     |
| FS_Users    | Change + Read    |

Click **OK**.

---

## **7. Configure NTFS Security (MOST IMPORTANT)**  
This is what enforces **upload allowed**, but **no edit**, **no rename**, **no delete**.

### **Step 1 — Disable Inheritance**
**Path:**  
`Properties → Security → Advanced`

Click:
```
Disable inheritance → Convert inherited permissions
```

Remove all entries **except**:
- Administrators  
- SYSTEM  

---

### **Step 2 — Give Admins Full Control**
Add → **FS_Admins**

Allow:
```
Full Control
```

Apply to:
```
This folder, subfolders and files
```

---

### **Step 3 — Add Employee Permissions**
Add → **FS_Users**

Allow ONLY these permissions:
```
Traverse folder / execute
List folder / read data
Read attributes
Read permissions
Create files / write data
Create folders / append data
Read
Write
```

Apply to:
```
This folder, subfolders and files
```

---

### **Step 4 — Deny Delete (Critical)**
Add → **FS_Users**  
Change **Type** to:  
```
DENY
```

Deny these:
```
Delete
Delete subfolders and files
```

Apply to:
```
This folder, subfolders and files
```

Click **OK → OK → OK**.

---

## **8. User Experience (Result)**
| Action | Result |
|--------|--------|
| Upload new file | ✔ Allowed |
| Download | ✔ Allowed |
| Copy | ✔ Allowed |
| Modify existing file | ❌ Blocked |
| Rename | ❌ Blocked |
| Delete | ❌ Blocked |
| Admin delete | ✔ Allowed |

This matches your exact requirement.

---

## **9. How Users Access the Shared Folder**
On each user PC:

Press:
```
WIN + R
```
Type:
```
\\OFFICE-SERVER\CompanyData
```

Enter their username/password.

### **Map Network Drive (Recommended)**
Right‑click the folder → **Map Network Drive**

Choose:
```
Drive letter: Z:
Reconnect at sign-in: ✔
```

---

## **10. Enable File Recovery (Shadow Copies)**
**Path:**  
`This PC → Right‑click D: → Configure Shadow Copies`

Enable Shadow Copies.

Recommended schedule:
```
Twice per day
```

Admins can restore deleted or overwritten files instantly.

---

# **Your Server Is Now:**
- Local  
- Secure  
- No cloud  
- No VPN  
- No accidental deletion  
- Admin‑controlled  
- Perfect for office environments  

---

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
