Sudo Rules For IPA users:
--------------------------



# How FreeIPA Sudo Rules Work**

A FreeIPA sudo rule always has **four components**:

1. **Who** can run sudo  
   - Users or groups  
   - Or `--usercat=all`

2. **Where** they can run sudo  
   - Hosts or hostgroups  
   - Or `--hostcat=all`

3. **What commands** they can run  
   - Specific commands  
   - Or `--cmdcat=all`

4. **As which user** they can run the command  
   - Usually `root`  
   - Added via `ipa sudorule-add-runasuser`

FreeIPA → SSSD → `/etc/sudoers.d/` → sudo applies the rule.

---

**1. Allow ALL IPA Users to Run ALL Commands on ALL IPA Hosts**

```
ipa sudorule-add --hostcat='all' --usercat='all' --cmdcat='all' sudorule1
ipa sudorule-add-runasuser --users=root sudorule1
ipa sudorule-show sudorule1 --all
```

 ✔ What this rule means  
- **All users** (`--usercat=all`)  
- On **all hosts** (`--hostcat=all`)  
- Can run **all commands** (`--cmdcat=all`)  
- **As root** (`--runasuser=root`)  

 ✔ Effect  
This is equivalent to putting this in `/etc/sudoers`:

```
ALL ALL=(root) ALL
```

✔ When to use  
- Small labs  
- Testing environments  
- When you want every IPA user to have full sudo access  

 ⚠ Not recommended for production  
Because it gives **full root access** to everyone.

---

**2. Allow Specific Users/Groups to Run Specific Commands on Specific Hosts**


```
ipa sudorule-add sudorule2
ipa sudorule-add-user --users=ipauser --groups=admins sudorule2
ipa sudorule-add-host --hosts=`hostname` --hostgroups=hostgroup sudorule2
ipa sudocmd-add '/usr/bin/systemctl * sshd.service'
ipa sudorule-add-allow-command --sudocmds='/usr/bin/systemctl * sshd.service' sudorule2
ipa sudorule-add-runasuser --users=root sudorule2
ipa sudorule-show sudorule2 --all
```

✔ What this rule means  
- Applies only to **ipauser** and members of **admins** group  
- Applies only to **this host** or a **hostgroup**  
- Allows only **one command**:

```
/usr/bin/systemctl * sshd.service
```

Meaning they can:

- Start sshd  
- Stop sshd  
- Restart sshd  
- Reload sshd  

✔ Effect  
Equivalent to:

```
ipauser, %admins ALL=(root) /usr/bin/systemctl * sshd.service
```

✔ When to use  
- Delegating limited admin tasks  
- Allowing helpdesk to restart services  
- Allowing developers to restart only their application service  
- Enforcing least privilege  

 ✔ Why this is powerful  
You can restrict:

- Which users  
- Which hosts  
- Which commands  
- Which run-as user  

This is the **proper enterprise way** to manage sudo.

---

**3. Allow a LOCAL (non‑IPA) User to Run ALL Commands on Specific Hosts**

### Commands:
```
ipa sudorule-add --cmdcat='all' sudorule3
ipa sudorule-add-user --users=localuser sudorule3
ipa sudorule-add-host --hosts=`hostname` sudorule3
ipa sudorule-add-runasuser --users=root sudorule3
ipa sudorule-show sudorule3 --all
```

✔ What this rule means  
- Applies to **localuser** (a Linux local account, not IPA)  
- Applies only to **this host**  
- Allows **all commands**  
- As **root**

✔ Why this works  
FreeIPA can manage sudo rules for:

- IPA users  
- IPA groups  
- **Local system users** (external users)

✔ When to use  
- You have a local service account  
- You want to manage sudo centrally  
- You want to avoid editing `/etc/sudoers` manually  

✔ Effect  
Equivalent to:

```
localuser ALL=(root) ALL
```

But controlled centrally from FreeIPA.

---

# ⭐ **Summary Table**

| Rule | Who | Where | What | Run As | Purpose |
|------|-----|--------|--------|---------|----------|
| **sudorule1** | All IPA users | All hosts | All commands | root | Full sudo for everyone |
| **sudorule2** | Specific IPA users/groups | Specific hosts | Specific commands | root | Least privilege, controlled access |
| **sudorule3** | Local Linux user | Specific hosts | All commands | root | Centralized sudo for local accounts |

---


