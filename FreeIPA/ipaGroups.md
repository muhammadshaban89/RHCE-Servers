

FreeIPA makes group creation very straightforward, and you can do it either from the **Web UI** or the **CLI**. 
---

#  Create a FreeIPA Group (CLI)



**Create a simple POSIX group**
```bash
ipa group-add mygroup
```

**Create a POSIX group with a description**
```bash
ipa group-add mygroup --desc="My test group"
```

 **Create a non‑POSIX group**
Useful for access control or HBAC rules.

```bash
ipa group-add mygroup --nonposix
```

**Add a user to the group**
```bash
ipa group-add-member mygroup --users=muhammad
```

**Add multiple users**
```bash
ipa group-add-member mygroup --users=user1,user2,user3
```

**Verify group membership**
```bash
ipa group-show mygroup
```

---

#  Create a FreeIPA Group (Web UI)

1. Log in to the FreeIPA Web UI  
   `https://ipa.example.com/ipa/ui/`
2. Go to **Identity → Groups**
3. Click **Add**
4. Fill in:
   - Group name  
   - Description  
   - POSIX or non‑POSIX
5. Save
6. Open the group → **Members** → Add users

---

#  POSIX vs Non‑POSIX — quick clarity

| Type | Purpose |
|------|---------|
| **POSIX group** | Used for Linux permissions, file ownership, UID/GID mapping |
| **Non‑POSIX group** | Used for FreeIPA policies: HBAC, sudo rules, access control |

- If you’re planning to use the group for **sudo**, **HBAC**, or **access control**, non‑POSIX is fine.  
- If you want it to behave like a normal Linux group, choose POSIX.

---

#  Example: A clean, production‑ready group setup

```bash
ipa group-add devops --desc="DevOps Team" --posix
ipa group-add-member devops --users=muhammad
ipa group-add-member devops --users=ahmed,saad
```

