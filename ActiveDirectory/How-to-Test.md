
* Joining a **Windows client** to a **Windows Active Directory Domain** is straightforward, but it only works if the Domain Controller is correctly configured.
* Below is a **pure GUI‑based, step‑by‑step guide** that takes you from **client preparation → domain join → testing**.

This is exactly how you would do it in a real enterprise environment.

---

# ⭐ PART 1 — What MUST be ready on the Domain Controller (Server Side)

Before touching the client, confirm these on the DC:

### ✔ AD DS installed  
### ✔ DNS installed  
### ✔ Domain created (e.g., `lab.local`)  
### ✔ DC has a static IP  
### ✔ SYSVOL and NETLOGON shares exist  
### ✔ DNS zone exists (`lab.local` and `_msdcs.lab.local`)

If these are correct, the client can join.

---

# ⭐ PART 2 — Prepare the Windows Client (GUI)

### 1. Set DNS to the Domain Controller’s IP  
This is the **most important** step.

1. Open **Control Panel**  
2. Go to **Network and Internet → Network and Sharing Center**  
3. Click **Change adapter settings**  
4. Right‑click your active network adapter → **Properties**  
5. Select **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**  
6. Choose **Use the following DNS server addresses**  
   - **Preferred DNS:** IP of your DC (example: `192.168.10.10`)  
   - Leave Alternate DNS empty  
7. Click **OK → Close**

If DNS is wrong → domain join will fail.

---

# ⭐ PART 3 — Join the Client to the Domain (GUI)

1. Right‑click **This PC** → **Properties**  
2. Click **Advanced system settings**  
3. Go to the **Computer Name** tab  
4. Click **Change…**  
5. Under **Member of**, select **Domain**  
6. Enter your domain name, e.g.:  
   ```
   lab.local
   ```
You will be prompted for domain credentials:

- Username: `LAB\Administrator`  
- Password: (your domain admin password)

If successful, you will see:

**“Welcome to the lab.local domain.”**

7. Click **OK**  
8. Restart the computer when prompted
---

Important Note for VMware Labs
---

- If you clone VMs → ALWAYS run Sysprep

- Use VMware’s Guest Customization (it runs Sysprep automatically).
Otherwise, every clone will have the same SID and fail domain join.

- confirm SID

        wmic useraccount get name,sid
- to change SID:

       cd C:\Windows\System32\Sysprep\
      .\sysprep.exe
  
-  In the Sysprep window:
  
• 	System Cleanup Action:

• 	Generalize:
✔ Check this box (this generates a new SID)
• 	Shutdown Options:

Click OK.
The system will:
• 	Generalize
• 	Remove SID
• 	Reboot
• 	Create a new unique SID
7. Click **OK**



---

# ⭐ PART 4 — Log in Using a Domain Account (GUI)

After reboot:

1. At the login screen, click **Other user**  
2. Enter:

```
Username: LAB\testuser
Password: <domain password>
```

If login works → Kerberos authentication is successful.

---

# ⭐ PART 5 — Test the Domain Join (GUI + simple commands)

## ✔ Test 1 — Check Domain Membership (GUI)

1. Right‑click **This PC** → **Properties**  
2. Under **Computer name, domain, and workgroup settings**, you should see:

```
Domain: lab.local
```

This confirms the client is joined.

---

## ✔ Test 2 — Access SYSVOL and NETLOGON (GUI)

Press **Win + R**, type:

```
\\dc01\sysvol
```

Then:

```
\\dc01\netlogon
```

If both open →  
DNS, AD DS, DFS‑R, and authentication are working.

---

## ✔ Test 3 — Test Group Policy

1. Open **Command Prompt** as Administrator  
2. Run:

```
gpupdate /force
```

3. Then:

```
gpresult /r
```

You should see:

- The domain name  
- Applied GPOs  
- Logon server (your DC)

This confirms GPO + SYSVOL + Kerberos are working.

---

## ✔ Test 4 — Check the Client Appears in AD (GUI on DC)

On the Domain Controller:

1. Open **Server Manager → Tools → Active Directory Users and Computers**  
2. Expand your domain  
3. Click **Computers** (or your custom OU)

You should see your client computer name (e.g., `CLIENT01`).

This confirms the DC created the computer object.

---

## ✔ Test 5 — Check DNS Registration (GUI on DC)

1. Open **DNS Manager**  
2. Expand **Forward Lookup Zones → lab.local**  
3. Look for:

```
client01   A   192.168.10.x
```

If the A record exists → DNS dynamic update is working.

---

# ⭐ PART 6 — Optional but Recommended Tests

### ✔ Test Kerberos Tickets (GUI + command)

Open Command Prompt:

```
klist
```

You should see:

- A TGT (Ticket Granting Ticket)
- Service tickets

This confirms Kerberos is fully functional.

---

# ⭐ Summary — What You Achieved

After following these steps, you have:

- Joined a Windows client to AD  
- Verified DNS, Kerberos, SYSVOL, NETLOGON  
- Confirmed GPO processing  
- Confirmed the computer object exists in AD  
- Confirmed DNS dynamic updates  
- Confirmed authentication works  




---

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
