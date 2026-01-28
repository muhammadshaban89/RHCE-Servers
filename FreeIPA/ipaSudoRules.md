Sudo Rules For IPA users:
--------------------------



# How FreeIPA Sudo Rules Works

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

When you create a sudo rule in FreeIPA, it **does not automatically apply to existing users** unless you explicitly *attach* those users (or their groups) to the rule.  
.

---

**How to Apply a Sudo Rule to an Existing IPA User**

Assume:

- The user already exists → `ipauser`
- The sudo rule already exists → `sudorule1`

You simply run:

```
ipa sudorule-add-user --users=ipauser sudorule1
```

That’s it.  
The rule is now active for that user.

---

**If the rule applies to a group instead of a single user**

If you want to apply the rule to a group (recommended for clean design):

1. Add the user to the group:
   ```
   ipa group-add-member admins --users=ipauser
   ```

2. Attach the group to the sudo rule:
   ```
   ipa sudorule-add-user --groups=admins sudorule1
   ```

This way, any user added to the group automatically inherits the sudo rule.

---

**If the rule applies to a host or hostgroup**

Make sure the host is included:

```
ipa sudorule-add-host --hosts=client1.example.local sudorule1
```

Or add a hostgroup:

```
ipa sudorule-add-host --hostgroups=linuxservers sudorule1
```

---

**How to confirm the rule is applied**

```
ipa sudorule-show sudorule1 --all
```

You should see:

- The user listed under **Users**
- The host under **Hosts**
- Commands under **Allow Commands**
- Run-as user (usually root)

---

**How to test on the client**

On the IPA client machine:

1. Refresh sudo rules:
   ```
   sudo sss_cache -E
   ```

2. Test:
   ```
   sudo -l
   ```

You should see the allowed commands.

---

**Summary**

To apply a sudo rule to an already-created user:

```
ipa sudorule-add-user --users=ipauser sudorule1
```

Or, if using groups:

```
ipa group-add-member admins --users=ipauser
ipa sudorule-add-user --groups=admins sudorule1
```

This is the cleanest and most scalable way to manage sudo access in FreeIPA.

---

**1. How to MODIFY an Existing Sudo Rule**

You modify a rule by changing its components:

- Users  
- Groups  
- Hosts  
- Hostgroups  
- Commands  
- Run‑as users  
- Categories (all users, all hosts, all commands)

Below are the most common modifications.

---

## 🔹 **Add a user to an existing rule**
```
ipa sudorule-add-user --users=ipauser sudorule1
```

**Remove a user from a rule**
```
ipa sudorule-remove-user --users=ipauser sudorule1
```


**Remove a group**
```
ipa sudorule-remove-user --groups=admins sudorule1
```

---

**Add a host**
```
ipa sudorule-add-host --hosts=client1.example.local sudorule1
```

**Remove a host**
```
ipa sudorule-remove-host --hosts=client1.example.local sudorule1
```

---

**Add a command**
```
ipa sudocmd-add '/usr/bin/systemctl restart sshd'
ipa sudorule-add-allow-command --sudocmds='/usr/bin/systemctl restart sshd' sudorule1
```

**Remove a command**
```
ipa sudorule-remove-allow-command --sudocmds='/usr/bin/systemctl restart sshd' sudorule1
```

---

**Change run‑as user**
Add:
```
ipa sudorule-add-runasuser --users=root sudorule1
```

Remove:
```
ipa sudorule-remove-runasuser --users=root sudorule1
```

---

**Change categories (all users, all hosts, all commands)**

- Make rule apply to ALL users:
```
ipa sudorule-mod --usercat=all sudorule1
```

- Make rule apply to ALL hosts:
```
ipa sudorule-mod --hostcat=all sudorule1
```

- Make rule allow ALL commands:
```
ipa sudorule-mod --cmdcat=all sudorule1
```

---
**How to DELETE (REMOVE) a Sudo Rule Completely**

If you want to remove the entire rule:

```
ipa sudorule-del sudorule1
```

This deletes:

- The rule  
- All user associations  
- All host associations  
- All command associations  

Everything is removed in one go.

---


