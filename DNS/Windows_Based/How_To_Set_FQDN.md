
Setting the **FQDN (Fully Qualified Domain Name)**
---------------------

The FQDN is simply:

**hostname + domain name**

Example:  
`server01.example.local`


---

# 1. Set the Computer Name (Hostname)

### **GUI Method**
1. Open **Run** → type:
   ```
   sysdm.cpl
   ```
2. Go to **Computer Name** tab  
3. Click **Change…**  
4. Enter your **hostname** (e.g., `server01`)  
5. Click **OK** and reboot

---

#  2. Set the DNS Suffix (Domain Name)

This is what completes the FQDN.

### **GUI Method**
1. Open **Run** → type:
   ```
   ncpa.cpl
   ```
2. Right‑click your network adapter → **Properties**  
3. Select **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**  
4. Click **Advanced**  
5. Go to **DNS** tab  
6. Under **DNS suffix for this connection**, enter your domain:
   ```
   example.local
   ```
7. Save everything and reboot

Now your FQDN becomes:
```
hostname.domain
```

---

# 3. Verify the FQDN

Open Command Prompt or PowerShell:

```powershell
hostname
```

```powershell
ipconfig /all
```

Look for:
- **Host Name**
- **Primary DNS Suffix**
- **DNS Suffix Search List**

Your FQDN should appear as:
```
server01.example.local
```

---

#  4. Optional: Join a Domain (Automatically sets FQDN)

If you join a Windows domain:

1. Run:
   ```
   sysdm.cpl
   ```
2. Computer Name → **Change**  
3. Select **Domain**  
4. Enter domain name (e.g., `example.local`)  
5. Enter credentials  
6. Reboot

Windows will automatically set the FQDN.

---

#  Summary

| Component | Purpose |
|----------|---------|
| Hostname | First part of FQDN |
| DNS Suffix | Domain part of FQDN |
| FQDN | hostname + domain |

---

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
