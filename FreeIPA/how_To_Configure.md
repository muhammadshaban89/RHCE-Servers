Step By Step Guide To Configure FreeIPA on Linux.
------------------------------------------------

**1: Set hostname**

	hostnamectl set-hostname myipa.server.local
	exec bash
  
**2: Make entry in /etc/hosts for name resolution: `vi /etc/hosts`**
	
	192.168.100.30 myipa.server.local
  
**3: Set Time and Date :** 
- Set Time Zone as per your TimeZone:
  
  ```
	timedatectl set-timezone Asia/Karachi
  ```
  
**4: check selinux status and set to permissive  :**

	sestatus 
	setenforce 0
  
- OR  permanantly set it permissive:
  
```
  sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/conf
```
**5: Install packages :**

	yum install -y freeipa-server ipa-server-dns bind-dyndb-ldap freeipa-client ipa-admintools

**✔ What this installs:**

| Package | Purpose |
|--------|---------|
| **freeipa-server** | Core FreeIPA server (LDAP, Kerberos, CA, Web UI) |
| **ipa-server-dns** | DNS integration (BIND with IPA plugin) |
| **bind-dyndb-ldap** | Allows BIND to store DNS data in LDAP |
| **freeipa-client** | Client tools (SSSD, enrollment utilities) |
| **ipa-admintools** | CLI tools for managing users, hosts, policies |

  
**6: Install Ipa server  by runing installer script:**

**Option 1 — Without DNS**
```
	ipa-server-install  --mkhomedir
```
* Automatically creates home directories for IPA users when they log in.
  
- OR
- **Option 2 — With DNS:**
   ```
	ipa-server-install --setup-dns
   ```
 * If you want FreeIPA to manage DNS zones (recommended for labs).

# ✔ What happens during installation:

- Configures **LDAP (389 DS)**
- Configures **Kerberos KDC**
- Sets up **Dogtag Certificate Authority**
- Configures **SSSD**
- Optionally configures **DNS**
- Creates **admin** and **Directory Manager** accounts
- Sets up **NTP**
- Enables **Web UI**
- Creates default realm (EXAMPLE.LOCAL)

   
**7: Add services or ports in firewall  and reload firewall**

	firewall-cmd --permanent --add-service={http,https,ldap,ldaps,kerberos,dns,ntp,kpasswd,freeipa-ldap,freeipa-ldaps}
	firewall-cmd --reload
	
**8: Enable Automatic Home Directory Creation.**

	authconfig --enablemkhomedir  --update
	
  ✔ What this does :
  
- When an IPA user logs into a Linux client for the first time:
  
	 * Their home directory is automatically created
    
	 * Permissions are set correctly
    
	* Without this, IPA users cannot log in unless you manually create their home directories.

**9: Generate Ticket for IPA -Server -- kerberos ticket for admin user**

	kinit admin
	klist 
	
✔ What this does:  
- `kinit admin` → Authenticates the IPA admin user and obtains a Kerberos ticket  
- `klist` → Shows the ticket and its expiration time.
-  Because : FreeIPA uses Kerberos for authentication.
-  You must have a valid ticket to run IPA admin commands.


- Check IPA Server Status :

```
ipactl status 
``` 
* all services must be running.
  
- To restart any single service
  ```
  pactl restart "service name"
  ```
**10: Set Default Shell for All Users:**

	ipa config-mod --defaultshell=/bin/bash
  
**11: Add users:**

	ipa user-add  user1
	ipa user-mod --passord

- To find user :
  ```
	ipa user-find    username 
  ```
- Set or modify password:
```
ipa user-mod user1 --password
```

- Access the FreeIPA Web Interfacen `web:https://myipa.server.local`

```
https://myipa.server.local
```

Important Note
--------------

* To allow ssh integration:
  
- You can configure `SSHD` to fetch users `SSH public key` from the `LDAP` directory by uncommenting these lines in `/etc/ssh/sshd_config`:

`AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys`
`AuthorizedKeysCommandUser nobody`

Then restart `sshd.service.`

- You can add your `ssh` key to your `FreeIPA` user account through the web interface or use the `-sshpubkey='ssh-rsa AAAA...'` argument to the ipa `user-mod` or `ipa user-create` commands.

- Test it:
```
	$ sudo -u nobody sss_ssh_authorizedkeys <username>
```
- You should see your `ssh public key` on standard output and no error message on standard error.

**known_hosts:**

- You can configure `SSH` to fetch hosts public key information from their directory entries in FreeIPA by adding those lines in `/etc/ssh/ssh_config`:
```
GlobalKnownHostsFile /var/lib/sss/pubconf/known_hosts
ProxyCommand /usr/bin/sss_ssh_knownhostsproxy -p %p %h
```
**Kerberos/GSS API Authentication:**

- You can enabled `Kerberos` / GSS API Authentication for the `SSH Client` to FreeIPA member hosts by uncommenting and changing the following lines in `/etc/ssh/ssh_config`:
```
GSSAPIAuthentication yes
GSSAPIDelegateCredentials yes
```


# Client Side Configuration:
---------------------------

**1:set hostname** 

```
hostnamectl set-hostname myipa.client.local
	exec bash
````

**2:make entries in `/etc/hosts` file**
```
vi /etc/hosts
	192.168.100.31 myipa.client.local
```
**3:set selinux to permissive**
	sestatus 
	setenforce 0
  
- or permanantly set it permissive:
```
  sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/conf
```

**4:set time and date** 
```
timedatectl set-timezone Asia/Karachi

```
**5:intall packages**
	
    yum install -y freeipa-client
	
**6:Add firewall rules**
	firewall-cmd  --add-port={464/tcp,464/udp,123/udp}
	firewal-cmd --reload
  
**7: execute client installer script by running:**

	ipa-client-install --mkhomedir

- this commond will propmt you for some input.
- add server domain name 
- add server hostname
- provide admin user
- and password you configured for admin in server.

* after completion you can use
```
su - username ||to verify user loging
```

- OR
```  
ssh user@localhost
```
- it will prompt you for password change. 
- change password  and user will be logged in.
```
id -a 
pwd
```


How To Configure on RHEL 9:
--------------------------

- 1:set host name
- 2:make entries in /etc/host
- 3:intsall packages 
- 4: run installer script
```
ipa-server-install --domain server.local --realm SERVER.LOCAL --reverse-zone=100.168.192.in-addr.arpa --no-forwarders --no-ntp --setup-dns --ds-password strongpassword --admin-password strongpassword --unattended 
```
- Rest process is same as above.

- you can run user add commond as :
```
  ipa user-add newuser1 --first=new --last=user1 --email=newuser1@myipa.server.com --password 

```


- To uninstall IPA Server :
```
ipa-server-install --uninstall
```
-------

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
