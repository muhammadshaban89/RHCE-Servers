 **Caching‑only DNS** :
 ------------------------
**no zones, only forwards + caches**

  
**Caching DNS = Recursive Resolver:**

This server:

• 	Speeds up DNS for all clients

• 	Caches answers for internal + external domains

• 	Reduces load on your master/slave

• 	Reduces internet DNS traffic

• 	Provides fast responses after first lookup

• 	Protects clients if upstream DNS is slow

It is the DNS server your clients actually use.

✔ It handles:

• 	Internal queries (by forwarding to master/slave)

• 	External queries (by recursion or forwarding to public DNS)

• 	Cache hits (instant answers)

⭐ Key Points: Caching‑Only DNS Server:
--------------------
  
🔹 1. No authoritative zones
-----
• 	It does not host any zone files.

• 	It only performs recursion and caching.

🔹 2. Speeds up DNS resolution
---
• 	First query → forwarded upstream

• 	Subsequent queries → served instantly from cache

• 	Reduces latency across the entire network.

🔹 3. Reduces load on authoritative DNS
---
• 	Authoritative server answers fewer queries.

• 	Caching DNS absorbs repetitive traffic.

🔹 4. Reduces external DNS traffic
----
• 	Only the first lookup goes to the internet.

• 	Saves bandwidth and improves reliability.

🔹 5. Provides resilience during outages-
-----
• 	Cached entries continue working even if upstream DNS is slow or unreachable.

🔹 6. Improves security
-----
• 	Internal clients don’t query public DNS directly.

• 	Central point for DNS logging, filtering, and monitoring.



---

# 1. Install BIND on the caching‑only server
```bash
sudo dnf install bind bind-utils -y
```

---

# 2. Configure caching‑only + forwarder mode
Edit:

```bash
sudo nano /etc/named.conf
```

Use this clean configuration:

```conf
options {
    directory "/var/named";

    listen-on port 53 { any; };
    listen-on-v6 port 53 { any; };

    allow-query { localhost; 192.168.0.0/24; };

    // Forward all queries to your authoritative DNS first
    forwarders {
        192.168.10.10;   // your authoritative DNS
        1.1.1.1;         // fallback
        8.8.8.8;         // fallback
    };

    forward first;   // try your authoritative DNS, then fallback
    recursion yes;
    dnssec-validation auto;

    empty-zones-enable no;
};
```

### Why `forward first`?
- If your authoritative DNS knows the answer → it replies
- If not → caching DNS forwards to public resolvers
- This is ideal for lab testing

---

#  3. Remove all authoritative zones
Ensure **no zones** are defined.

Comment out everything in:

```
/etc/named.rfc1912.zones
```

Your caching DNS must not be authoritative for anything.

---

#  4. Fix SELinux context
```bash
sudo restorecon -RFv /var/named
```

---

#  5. Start and enable the service
```bash
sudo systemctl enable --now named
sudo systemctl status named
```

---

#  6. Open firewall
```bash
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```

---

#  7. Testing the caching‑only DNS

## Test 1 — Query your lab domain (should go to authoritative DNS)
```bash
dig @192.168.20.20 host1.lab.local
```

Expected:
- First query: forwarded to authoritative DNS
- Second query: served from cache

Check cache speed:
```bash
dig @192.168.20.20 host1.lab.local | grep "Query time"
```

---

## Test 2 — Query an internet domain (should go to upstream DNS)
```bash
dig @192.168.20.20 google.com
```

---

## Test 3 — Confirm recursion is working
```bash
dig @192.168.20.20 +trace google.com
```

You should see:
- Caching DNS does recursion
- Not authoritative for anything

---

#  8. Optional: Enable query logging
Useful for lab testing.

```bash
sudo rndc querylog on
```

Logs appear in:
```
/var/named/data/named.run
```

---

Test Caching With `Flush`:
------------------------

#  1. Flush the DNS cache on the caching‑only server

BIND on RHEL 9 uses **rndc**.


`rndc`is short for Remote Name Daemon Control.
- It’s the command‑line tool used to control and manage the BIND (named) DNS server while it’s running
- You can use following commands:
```
rndc reload
rndc flush
rndc status
rndc querylog on
```

## Flush everything:
```bash
sudo rndc flush
```


## Flush only one domain:
```bash
sudo rndc flushname example.com
```

## Flush only one view (if using views):
```bash
sudo rndc flush view internal
```

---

## 2. Test caching behavior

## Step A — First query (should be slow)
```bash
dig @<caching-dns-ip> host1.lab.local
```

Check the query time:
```
;; Query time: 45 msec
```

This means it forwarded to your authoritative DNS.

---

## Step B — Second query (should be fast)
Run the same query again:
```bash
dig @<caching-dns-ip> host1.lab.local
```

Expected:
```
;; Query time: 0 msec
```

This confirms the caching‑only DNS is working.

---

##  3. Test internet domain caching

## First query:
```bash
dig @<caching-dns-ip> google.com
```

## Second query:
```bash
dig @<caching-dns-ip> google.com
```

You should see the same pattern:
- First = slow (forwarded)
- Second = fast (cached)

---

##  4. Verify cache flush works

# Step 1 — Query twice (fast second time)
```bash
dig @<caching-dns-ip> yahoo.com
dig @<caching-dns-ip> yahoo.com
```

Second query should be fast.

## Step 2 — Flush cache
```bash
sudo rndc flush
```

## Step 3 — Query again (should be slow again)
```bash
dig @<caching-dns-ip> yahoo.com
```

If the query time increases again → **cache flush confirmed**.

---

##  5. Optional: Enable query logging to watch behavior live

```bash
sudo rndc querylog on
```

Then tail the log:
```bash
sudo tail -f /var/named/data/named.run
```

You’ll see:
- Forwarded queries
- Cached responses

# Summary
Your caching‑only DNS will now:
- Forward lab queries to your authoritative DNS
- Forward internet queries to public DNS
- Cache everything for fast responses
- Never act as an authoritative server

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
