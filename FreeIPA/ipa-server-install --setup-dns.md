
#  **What Settings Are Required for `ipa-server-install --setup-dns`**


---

#  **1. Hostname**
Installer asks:

```
Enter the fully qualified domain name of the server:
```

### ✔ What it needs  
A **FQDN** that resolves to your server.

Example:
```
ipa.example.local
```

### ✔ Requirements  
- Must be resolvable (via /etc/hosts or DNS)  
- Must not be a short name  
- Must not be an IP  

---

#  **2. Domain Name**
Installer asks:

```
Enter the domain name:
```

### ✔ What it needs  
Your DNS domain.

Example:
```
example.local
```

---

#  **3. IPA Realm Name (Kerberos Realm)**
Installer asks:

```
Enter the Kerberos realm:
```

### ✔ What it needs  
Realm is always **UPPERCASE** version of your domain.

Example:
```
EXAMPLE.LOCAL
```

---

#  **4. Directory Manager (DM) Password**
Installer asks:

```
Directory Manager password:
```

### ✔ What it is  
- Superuser for LDAP  
- Equivalent to “root” for the directory  
- Used rarely (schema changes, recovery)

Choose a strong password.

---

#  **5. IPA Admin Password**
Installer asks:

```
IPA admin password:
```

### ✔ What it is  
Password for the **admin** Kerberos principal.

Used for:
- Web UI login  
- CLI admin commands  
- Managing users, hosts, DNS  

---

#  **6. DNS Forwarders**
Installer asks:

```
Do you want to configure DNS forwarders? [yes/no]:
```

### ✔ Choose **yes**  
Then enter:

```
1.1.1.1
8.8.8.8
```

Or your internal DNS.

### ✔ Why  
Forwarders allow FreeIPA DNS to resolve external domains.

---

#  **7. Allow DNS Forwarding?**
Installer asks:

```
Do you want to allow DNS forwarding? [yes/no]:
```

### ✔ Choose **yes**  
Unless you want a fully isolated DNS.

---

# **8. Reverse Zone Creation**
Installer asks:

```
Do you want to create reverse zone? [yes/no]:
```

### ✔ Choose **yes**  
Then enter:

```
172.16.0.0/24
```

Installer will create:

```
0.16.172.in-addr.arpa
```

---

#  **9. DNSSEC (Optional)**
Installer may ask:

```
Do you want to configure DNSSEC? [yes/no]:
```

### ✔ Choose **no** for labs  
DNSSEC complicates things and is not needed in test environments.

---

# 🕒 **10. NTP Configuration**
Installer asks:

```
Configure NTP? [yes/no]:
```

### ✔ Choose **yes**  
Kerberos requires accurate time.

---

#  **11. Final Confirmation**
Installer shows a summary:

- Hostname  
- Domain  
- Realm  
- DNS settings  
- CA settings  

Then asks:

```
Continue to configure the system with these values? [no]: yes
```

### ✔ Choose **yes**

---

# 🎉 **What Happens After Installation**
FreeIPA automatically configures:

- LDAP (389 Directory Server)  
- Kerberos KDC  
- Dogtag CA  
- BIND DNS with LDAP backend  
- NTP  
- SSSD  
- Web UI  
- Admin accounts  
- DNS zones  
- Kerberos realm  

---

#  **Quick Checklist of Required Settings**

| Setting | Example | Required? | Notes |
|--------|---------|-----------|-------|
| FQDN | ipa.example.local | ✔ | Must resolve |
| Domain | example.local | ✔ | Lowercase |
| Realm | EXAMPLE.LOCAL | ✔ | Uppercase |
| DM Password | StrongPass | ✔ | LDAP superuser |
| Admin Password | StrongPass | ✔ | IPA admin |
| DNS Forwarders | 1.1.1.1 | ✔ | For external DNS |
| Reverse Zone | 172.16.0.0/24 | Optional | Recommended |
| DNSSEC | no | Optional | Avoid in labs |
| NTP | yes | ✔ | Required for Kerberos |

---

**One‑Command Full FreeIPA + DNS Installation**

```
ipa-server-install \
    --unattended \
    --hostname=ipa.example.local \
    --domain=example.local \
    --realm=EXAMPLE.LOCAL \
    --ds-password='DirectoryManagerPass' \
    --admin-password='AdminPass' \
    --mkhomedir \
    --setup-dns \
    --forwarder=1.1.1.1 \
    --forwarder=8.8.8.8 \
    --no-dnssec-validation \
    --auto-reverse

```
