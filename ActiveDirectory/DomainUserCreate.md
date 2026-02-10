Creating a **domain user** in Windows Active Directory
------------


#  PART 1 — Create a Domain User (GUI on the Domain Controller)

### 1. Open Active Directory Users and Computers (ADUC)

On the Domain Controller:

- Open **Server Manager**
- Click **Tools**
- Select **Active Directory Users and Computers**

---

### 2. Choose where to create the user

You can create the user in:

- The default **Users** container  
OR  
- A custom **OU** (recommended for clean structure)

Example:

```
lab.local
 └── OU=Users
```

Click the OU or container where you want the user.

---

### 3. Create the new user

Inside ADUC:

1. Right‑click the OU → **New → User**
2. Fill in:
   - **First name**
   - **Last name**
   - **User logon name** (example: `testuser`)
3. Click **Next**

---

### 4. Set the password

You will see password options:

- Enter a password  
- Confirm password  

Then choose:

- **User must change password at next logon** (recommended for real users)  
OR  
- **Password never expires** (useful for lab/testing)  

Click **Next → Finish**

Your domain user is now created.

---

#  PART 2 — Verify the User Exists

In ADUC, you should now see:

```
testuser
User
```

Right‑click → **Properties** if you want to check:

- Group membership  
- Profile  
- Account options  
- Description  

---

#  PART 3 — Use This User to Log In From a Domain‑Joined Client

Once your Windows client is joined to the domain:

1. Restart the client  
2. At the login screen, click **Other user**  
3. Enter:

```
Username: LAB\testuser
Password: <password you set>
```

If login succeeds:

- Kerberos authentication is working  
- DNS is correct  
- The client is fully using AD DS  

---

#  PART 4 — Test the User Account (GUI + simple commands)

# Test 1 — Check logon server

On the client, open Command Prompt:

```
set l
```

Look for:

```
LOGONSERVER=\\DC01
```

This confirms the user authenticated against the Domain Controller.

---

# Test 2 — Check Group Policy for the user

Run:

```
gpresult /r
```

Under **User Settings**, you should see:

- Domain name  
- Applied GPOs  
- User’s OU  

---

# Test 3 — Access domain resources

Try:

```
\\dc01\sysvol
\\dc01\netlogon
```

If they open → the user has proper domain access.

---

# ⭐ Summary

To log in with a domain user, you must:

### ✔ Create the user in ADUC  
### ✔ Join the client to the domain  
### ✔ Log in using `DOMAIN\username`  
### ✔ Test authentication and GPO  

This is the standard workflow in every Windows AD DS environment.

---



