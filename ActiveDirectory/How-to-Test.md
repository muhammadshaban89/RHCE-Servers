
A client “using” Active Directory:

- simply means the client is **joined to the domain** and is successfully **authenticating, receiving policies, resolving DNS, and accessing domain resources**.  


---

# ⭐ Step 1 — Prepare the Client (Windows 10/11)

### Set DNS to the Domain Controller’s IP  
This is the MOST important step.

On the client:

```
Control Panel → Network & Internet → Adapter Settings → IPv4
```

Set:

- **Preferred DNS:** IP of your DC (e.g., 192.168.10.10)
- **Alternate DNS:** leave empty (for now)

If DNS is wrong → domain join will fail.

---

# ⭐ Step 2 — Join the Client to the Domain

On the client:

```
System Properties → Computer Name → Change → Domain
```

Enter your domain name:

```
lab.local
```

You will be prompted for domain credentials:

```
LAB\Administrator
```

If the join succeeds, you’ll see:

- “Welcome to the lab.local domain”
- Reboot required

This confirms:

- DNS is working  
- AD DS is reachable  
- Authentication is working  

---

# ⭐ Step 3 — Log in Using a Domain Account

After reboot, on the login screen:

Click **Other User** and enter:

```
LAB\testuser
```

If login works → Kerberos authentication is successful.

This is the first real proof that the client is using AD.

---

# ⭐ Step 4 — Test Group Policy (GPO)

On the client:

```
gpupdate /force
```

Then check applied policies:

```
gpresult /r
```

You should see:

- The domain name  
- The OU the computer belongs to  
- The GPOs applied  

This confirms:

- SYSVOL is working  
- DFS-R replication is healthy  
- GPO processing is functional  

---

# ⭐ Step 5 — Test Domain Resources

### 1. Access a shared folder on the DC

On the client:

```
\\dc01\sysvol
\\dc01\netlogon
```

If these open → AD replication + DFS-R + permissions are correct.

### 2. Test file share permissions  
Create a shared folder on the DC:

```
C:\Shares\Dept
```

Give access to a domain group:

```
Finance Users → Read/Write
```

On the client, log in as a Finance user and try accessing:

```
\\dc01\Dept
```

This confirms authorization is working.

---

# ⭐ Step 6 — Test DNS from the Client

On the client:

```
nslookup dc01.lab.local
```

Then test SRV records:

```
nslookup
> set type=SRV
> _ldap._tcp.dc._msdcs.lab.local
```

If the client can resolve these → AD service discovery is working.

---

# ⭐ Step 7 — Test Time Sync (Kerberos Requirement)

On the client:

```
w32tm /query /source
```

It should show:

```
PDC.domain.local
```

If time is off → Kerberos fails → logins fail.

---

# ⭐ Step 8 — Test User & Computer Visibility in AD

On the Domain Controller:

Open **Active Directory Users and Computers**:

- Check **Computers** OU → your client should appear  
- Check **Users** OU → your test user should appear  

This confirms the client is registered in AD.

---

# ⭐ Step 9 — Test Remote Management

From the DC:

```
ping client01
```

Then:

```
Enter-PSSession -ComputerName client01
```

If it works → WinRM + AD authentication are functional.

---

# ⭐ Step 10 — Test Login Script or GPO Deployment (Optional)

Create a simple login script:

```
echo "Welcome to the domain" > C:\scripts\login.bat
```

Place it in:

```
\\dc01\netlogon
```

Link a GPO to the user’s OU to run the script.

If the script runs at login → GPO + SYSVOL + permissions are perfect.

---

# ⭐ Summary — How a Client Uses AD (and How You Test It)

| Test | Confirms |
|------|----------|
| Domain join | DNS + authentication |
| Login with domain user | Kerberos |
| gpupdate / gpresult | GPO + SYSVOL |
| Access SYSVOL/NETLOGON | DFS-R + permissions |
| Access shared folders | Authorization |
| nslookup SRV records | AD service discovery |
| w32tm | Time sync |
| Client appears in ADUC | Registration in AD |

If all these work → your AD DS deployment is **healthy and fully functional**.

---

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
