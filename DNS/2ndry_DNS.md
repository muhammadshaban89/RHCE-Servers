# Step‑by‑Step: Configure Secondary DNS (BIND)

## **A. Configure the Primary DNS to allow zone transfers**

Edit `/etc/named.conf` on **ns1**:

```conf
allow-transfer { 192.168.1.11; };
also-notify { 192.168.1.11; };
```

Inside each zone definition:

```conf
zone "example.local" IN {
    type master;
    file "example.local.zone";
    allow-transfer { 192.168.1.11; };
    also-notify { 192.168.1.11; };
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "1.168.192.in-addr.arpa.zone";
    allow-transfer { 192.168.1.11; };
    also-notify { 192.168.1.11; };
};
```

Restart:

```bash
sudo systemctl restart named
```

---

## **B. Configure the Secondary DNS Server (ns2)**

Install BIND:

```bash
sudo dnf install -y bind bind-utils
```

Edit `/etc/named.conf`:

```conf
options {
    listen-on port 53 { 127.0.0.1; 192.168.1.11; };
    allow-query { localhost; 192.168.1.0/24; };
    recursion yes;
};

zone "example.local" IN {
    type slave;
    masters { 192.168.1.10; };
    file "slaves/example.local.zone";
};

zone "1.168.192.in-addr.arpa" IN {
    type slave;
    masters { 192.168.1.10; };
    file "slaves/1.168.192.in-addr.arpa.zone";
};
```

Create the slave directory:

```bash
sudo mkdir -p /var/named/slaves
sudo chown named:named /var/named/slaves
sudo restorecon -Rv /var/named/slaves
```

Start the service:

```bash
sudo systemctl enable --now named
```

If everything is correct, BIND will automatically pull the zone files from the primary.

---

## **C. Firewall Rules (both servers)**

```bash
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
```

---

## **D. Verify Zone Transfer**

From ns2:

```bash
dig @192.168.1.10 example.local AXFR
```

You should see the full zone output.  
If not, check:

- `allow-transfer` on primary  
- SELinux contexts  
- Firewalld  
- Serial number increments  

---

# 🧪 Testing Redundancy

### Test primary:

```bash
dig @192.168.1.10 www.example.local
```

### Test secondary:

```bash
dig @192.168.1.11 www.example.local
```

### Simulate primary failure

Stop named on ns1:

```bash
sudo systemctl stop named
```

Then test:

```bash
dig @192.168.1.11 www.example.local
```

If the secondary responds correctly, redundancy is working.

-------------------
- 👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a
- 👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
---
