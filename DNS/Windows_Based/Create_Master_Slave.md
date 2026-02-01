How to Create Primary & 2ndary DNS in Windows Server 2022:
-------------------------------------------------------

| Task | Method | Where |
|------|--------|-------|
| Create primary zone | GUI + PowerShell | DNS01 |
| Create secondary zone | GUI + PowerShell | DNS02 |


Assume:

- **Primary DNS server:** `DNS01` (IP `10.0.0.10`)  
- **Secondary DNS server:** `DNS02` (IP `10.0.0.11`)  
- **Zone name:** `corp.local`  

You can adjust names/IPs as needed.

---

## 1. Create a primary DNS zone (Windows Server 2022)

### 1.1 GUI steps on DNS01

1. **Open DNS Manager**  
   - Start → **Server Manager** → **Tools** → **DNS**

2. **Start New Zone Wizard**  
   - Expand your server → right‑click **Forward Lookup Zones** → **New Zone…**

3. **Zone Type**  
   - Select **Primary zone**  
   - If this is an AD‑joined server and you want AD integration:  
     - Check **Store the zone in Active Directory** (optional, but recommended in domain environments).

4. **Zone Replication Scope** (if AD‑integrated)  
   - Choose:  
     - **To all DNS servers in this forest** (common)  
     - Or **To all DNS servers in this domain**  
   - Click **Next**.

5. **Zone Name**  
   - Enter: `corp.local`  
   - Click **Next**.

6. **Zone File**  
   - Leave default: `corp.local.dns`  
   - Click **Next**.

7. **Dynamic Updates**  
   - Choose one:  
     - **Allow only secure dynamic updates** (best for AD)  
     - **Allow both nonsecure and secure dynamic updates** (lab)  
     - **Do not allow dynamic updates** (static environment)  
   - Click **Next** → **Finish**.

Your **primary zone** `corp.local` is now created on **DNS01**.

---

### 1.2 PowerShell steps on DNS01

#### Basic primary zone (file‑based)

```powershell
Add-DnsServerPrimaryZone `
  -Name "corp.local" `
  -ZoneFile "corp.local.dns"
```

#### AD‑integrated primary zone

```powershell
Add-DnsServerPrimaryZone `
  -Name "corp.local" `
  -ReplicationScope "Forest"
```

Other options for `-ReplicationScope`:

- `Forest`
- `Domain`
- `Legacy`

---

## 2. Allow zone transfers from primary to secondary

Before creating the secondary zone, you must allow **zone transfers** on the primary.

### 2.1 GUI on DNS01

1. In **DNS Manager**, right‑click zone **corp.local** → **Properties**.
2. Go to **Zone Transfers** tab.
3. Check **Allow zone transfers**.
4. Choose one of:
   - **To any server** (lab only)  
   - **Only to servers listed on the Name Servers tab**  
   - **Only to the following servers** → add `10.0.0.11` (DNS02)
5. If you choose **Only to the following servers**:
   - Click **Edit…** → add `10.0.0.11` → **OK**.

---

### 2.2 PowerShell on DNS01

```powershell
Set-DnsServerPrimaryZone `
  -Name "corp.local" `
  -Notify "Specific" `
  -SecondaryServers 10.0.0.11 `
  -SecureSecondaries "TransferToSecureServers"
```

For a lab, you could do:

```powershell
Set-DnsServerPrimaryZone `
  -Name "corp.local" `
  -SecureSecondaries "NoTransferSecurity"
```

---

## 3. Create a secondary DNS zone (Windows Server 2022)

### 3.1 GUI steps on DNS02

1. **Open DNS Manager**  
   - On **DNS02**: Server Manager → **Tools** → **DNS**

2. **Start New Zone Wizard**  
   - Right‑click **Forward Lookup Zones** → **New Zone…**

3. **Zone Type**  
   - Select **Secondary zone** → **Next**.

4. **Zone Name**  
   - Enter: `corp.local` → **Next**.

5. **Master DNS Servers**  
   - Enter IP of primary: `10.0.0.10`  
   - Click **Add** → ensure it resolves and shows a green check.  
   - Click **Next** → **Finish**.

If zone transfers are allowed and network is OK, the secondary zone will populate almost immediately.

---

### 3.2 PowerShell steps on DNS02

```powershell
Add-DnsServerSecondaryZone `
  -Name "corp.local" `
  -MasterServers 10.0.0.10 `
  -ZoneFile "corp.local.dns"
```

To verify:

```powershell
Get-DnsServerZone -Name "corp.local"
```

You should see `ZoneType : Secondary`.

---

## 4. Quick verification steps

### 4.1 From DNS02 (secondary)

```powershell
Get-DnsServerResourceRecord -ZoneName "corp.local"
```

You should see the same records as on DNS01.

### 4.2 From any client

```powershell
nslookup host1.corp.local 10.0.0.10   # query primary
nslookup host1.corp.local 10.0.0.11   # query secondary
```

Both should return the same IP.

---


👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099

