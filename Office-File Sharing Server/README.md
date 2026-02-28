
Here is **full, clean, detailed, professional guide** for building a **Windows Server 2022 office file‑sharing server** with **upload‑only (no edit/no delete)** permissions.

Office File‑Sharing Server Architecture Diagram
-----------------
```
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
```


# **Windows Server 2022 – Office File‑Sharing Server (Complete SOP)**  
**Scenario:**  
- 15 office users on the same LAN  
- Users can **upload / read / download**  
- Users **cannot edit, rename, or delete**  
- Only Admin has full control  
- No VPN, no cloud  

---

## **1. Install Windows Server 2022**
1. Boot from the installation USB.  
2. Select **Windows Server 2022 Standard (Desktop Experience)**.  
3. Install on the SSD.  
4. Set Administrator password.  
5. Rename the server:

`Server Manager → Local Server → Computer Name → Change`

**Name:** `OFFICE-SERVER`  
Restart.

---

## **2. Configure Static IP**
`Control Panel → Network and Sharing Center → Change adapter settings → Ethernet → IPv4`

Example:
```
IP Address: 192.168.1.10
Subnet Mask: 255.255.255.0
Gateway: 192.168.1.1
DNS: 192.168.1.1
```

---

## **3. Create Local User Accounts**
`Computer Management → Local Users and Groups → Users`

Create one user per employee:
```
user1
user2
user3
...
```

Uncheck:
```
User must change password at next logon
```

---

## **4. Create Security Groups**
`Computer Management → Local Users and Groups → Groups`

Create:
| Group Name | Purpose |
|-----------|----------|
| FS_Admins | Full control |
| FS_Users  | Regular employees |

---

## **4.1 Add Users to Groups**
This step must happen **before** configuring the shared folder.

### **Add employees to FS_Users**
`Groups → FS_Users → Add → enter user1, user2, user3…`

### **Add IT/Admin to FS_Admins**
`Groups → FS_Admins → Add → enter admin accounts`

This ensures permissions work correctly later.

---

## **5. Create the Shared Folder**
Create the data directory on the data disk:

```
D:\CompanyData
```

---

## **6. Configure Share Permissions**
Right‑click folder → **Properties → Sharing → Advanced Sharing**

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
| Group | Share Permission |
|-------|------------------|
| FS_Admins | Full Control |
| FS_Users  | Change + Read |

Click **OK**.

---

## **7. Configure NTFS Security (Critical)**  
This enforces **upload allowed**, but **no edit**, **no rename**, **no delete**.

---

### **Step 1 — Disable Inheritance**
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

Allow ONLY:
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
Change Type → **DENY**

Deny:
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

## **8. User Experience (Final Behavior)**
| Action | Result |
|--------|--------|
| Upload new file | ✔ Allowed |
| Download | ✔ Allowed |
| Copy | ✔ Allowed |
| Modify existing file | ❌ Blocked |
| Rename | ❌ Blocked |
| Delete | ❌ Blocked |
| Admin delete | ✔ Allowed |

---

## **9. How Users Access the Shared Folder**
On each PC:

Press:
```
WIN + R
```
Enter:
```
\\OFFICE-SERVER\CompanyData
```

### **Map Network Drive**
Right‑click → **Map Network Drive**

```
Drive letter: Z:
Reconnect at sign-in: ✔
```

---

## **10. Enable File Recovery (Shadow Copies)**
`This PC → Right‑click D: → Configure Shadow Copies`

Enable and schedule:
```
Twice per day
```

Admins can restore deleted/overwritten files instantly.

---

# **Your SOP is now complete and corrected.**  
The missing step (adding users to groups) is now properly included and placed in the correct order.

---



👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
