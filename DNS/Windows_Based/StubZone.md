Stub zone in Windows DNS :
-----------------------------

* is a special, read‑only zone that stores only the essential records (SOA, NS, and glue A records) needed to identify the authoritative DNS servers for another zone.
* It’s mainly used to improve name resolution between separate DNS namespaces and to keep delegation information automatically updated. 

---

#  What a Stub Zone Contains
A stub zone includes only three record types:

- **SOA record** – identifies the primary authoritative server.  
- **NS records** – list the authoritative DNS servers for the target zone.  
- **Glue A/AAAA records** – IP addresses of those authoritative servers.  

This makes the stub zone lightweight and automatically updated whenever the authoritative zone changes. 

---

# Why Use a Stub Zone?

Stub zones are useful when:

- Two organizations merge or need cross‑namespace resolution.  
- You want **automatic updates** of authoritative DNS server info (unlike conditional forwarders).  
- You want to avoid manually maintaining forwarders when DNS servers change.  
- You want faster, more direct resolution to another domain’s authoritative servers. 

---

# How to Create a Stub Zone in Windows DNS

**GUI Method (DNS Manager)**

1. Open **DNS Manager**.  
2. Right‑click **Forward Lookup Zones** → **New Zone**.  
3. Choose **Stub Zone**.  
4. Select **Primary** or **AD‑integrated** (if domain‑joined).  
5. Enter the **zone name** (e.g., `corp.example.com`).  
6. Add the **master DNS server(s)** from which the stub zone will pull NS/SOA/glue records.  
7. Finish the wizard.

**PowerShell Method**
```powershell
Add-DnsServerStubZone -Name "corp.example.com" `
  -MasterServers 10.0.0.10,10.0.0.11 `
  -ZoneFile "corp.example.com.dns"
```
This creates a stub zone and configures the master servers that supply the authoritative records. 

---

# Stub Zone vs Conditional Forwarder (Quick Comparison)

| Feature | Stub Zone | Conditional Forwarder |
|--------|-----------|------------------------|
| Auto‑updates NS/A records | ✅ Yes | ❌ No |
| Stores full zone data | ❌ No | ❌ No |
| Depends on master DNS servers | Yes | Yes |
| Best for | Dynamic environments, cross‑namespace resolution | Static environments |

---

