## DNS on RHEL 9: BIND overview

On RHEL 9, the standard DNS server is **BIND** (named). Red Hat’s documentation focuses on using BIND as:

- **Caching DNS server** for a local network.
- **Authoritative DNS server** for one or more zones.
- **Secondary server** for redundancy.  

We’ll walk through a **simple authoritative + caching** setup for a private zone (e.g., `example.local`) on RHEL 9, with both forward and reverse lookup.

---

## Step‑by‑step: Configure a DNS server on RHEL 9 (BIND)

I’ll assume:

- Your DNS server IP: `192.168.1.10`
- Your domain (zone): `example.local`
- Your reverse network: `192.168.1.0/24` → reverse zone `1.168.192.in-addr.arpa`
- You have root or sudo access on RHEL 9.

### 1. Install required packages

```bash
sudo dnf install -y bind bind-utils
```

- **`bind`**: The DNS server (`named`).
- **`bind-utils`**: Tools like `dig`, `nslookup`, `host` for testing.

### 2. Enable and start the `named` service

```bash
sudo systemctl enable named
sudo systemctl start named
sudo systemctl status named
```

- Ensure it’s **active (running)** before proceeding.

### 3. Understand main config files

Key paths on RHEL 9 (default BIND layout):  

- **Main config:** `/etc/named.conf`
- **Zone files directory:** `/var/named/`
- **Default example zone templates:** `/var/named/named.localhost`, `/var/named/named.loopback`

We’ll edit `/etc/named.conf` and create new zone files in `/var/named/`.

---

## Configure BIND as an authoritative + caching server

### 4. Backup and edit `/etc/named.conf`

First, backup:

```bash
sudo cp /etc/named.conf /etc/named.conf.bak
```

Then edit:

```bash
sudo vi /etc/named.conf
```

A typical base config for a small LAN might look like this (simplified):

```conf
options {
    listen-on port 53 { 127.0.0.1; 192.168.1.10; };
    listen-on-v6 { none; };
    directory       "/var/named";
    dump-file       "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";

    allow-query     { localhost; 192.168.1.0/24; };

    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;

    /* If you want to forward queries to upstream DNS (optional) */
    // forwarders { 1.1.1.1; 8.8.8.8; };
};

logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
};

/* Root hints (default) */
zone "." IN {
    type hint;
    file "named.ca";
};

/* Localhost zone (default) */
zone "localhost" IN {
    type master;
    file "named.localhost";
    allow-update { none; };
};

zone "0.0.127.in-addr.arpa" IN {
    type master;
    file "named.loopback";
    allow-update { none; };
};

/* Our custom forward zone */
zone "example.local" IN {
    type master;
    file "example.local.zone";
    allow-update { none; };
};

/* Our custom reverse zone */
zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "1.168.192.in-addr.arpa.zone";
    allow-update { none; };
};
```

Key points:

- **`listen-on`**: Add your server’s IP (`192.168.1.10`) so it listens on the LAN.
- **`allow-query`**: Restrict who can query your server (here: localhost + your subnet).
- **`recursion yes;`**: Allows your server to act as a caching resolver for clients.
- **Zones:** We added `example.local` and `1.168.192.in-addr.arpa`.

---

## Create the forward zone file

### 5. Create `/var/named/example.local.zone`

Start from a template:

```bash
sudo cp /var/named/named.localhost /var/named/example.local.zone
sudo chown root:named /var/named/example.local.zone
sudo chmod 640 /var/named/example.local.zone
```

Edit:

```bash
sudo vi /var/named/example.local.zone
```

Example content:

```dns
$TTL 86400
@   IN  SOA ns1.example.local. admin.example.local. (
        2026012301 ; Serial (YYYYMMDDnn)
        3600       ; Refresh
        900        ; Retry
        604800     ; Expire
        86400 )    ; Minimum

    IN  NS  ns1.example.local.

ns1 IN  A   192.168.1.10
www IN  A   192.168.1.20
db  IN  A   192.168.1.30
```

Notes:

- **SOA line:** `ns1.example.local.` is the primary NS; `admin.example.local.` is the email (replace `@` with `.`).
- **Serial:** Use a date‑based serial like `YYYYMMDDnn` and increment when you change the zone.
- **NS record:** Declares `ns1.example.local` as the authoritative name server.
- **A records:** Map hostnames to IPs.

---

## Create the reverse zone file

### 6. Create `/var/named/1.168.192.in-addr.arpa.zone`

Copy a template:

```bash
sudo cp /var/named/named.loopback /var/named/1.168.192.in-addr.arpa.zone
sudo chown root:named /var/named/1.168.192.in-addr.arpa.zone
sudo chmod 640 /var/named/1.168.192.in-addr.arpa.zone
```

Edit:

```bash
sudo vi /var/named/1.168.192.in-addr.arpa.zone
```

Example content:

```dns
$TTL 86400
@   IN  SOA ns1.example.local. admin.example.local. (
        2026012301 ; Serial
        3600       ; Refresh
        900        ; Retry
        604800     ; Expire
        86400 )    ; Minimum

    IN  NS  ns1.example.local.

10  IN  PTR ns1.example.local.
20  IN  PTR www.example.local.
30  IN  PTR db.example.local.
```

Explanation:

- The zone name `1.168.192.in-addr.arpa` corresponds to `192.168.1.x`.
- The `10` label corresponds to `192.168.1.10`, etc.
- PTR records map IPs back to hostnames.

---

## Check permissions and SELinux context

On RHEL, SELinux can block BIND if contexts are wrong. Use:

```bash
sudo restorecon -Rv /var/named
```

This resets SELinux contexts for zone files to what BIND expects.  

---

## Validate configuration and zone files

### 7. Check `named.conf` syntax

```bash
sudo named-checkconf /etc/named.conf
```

- If there is no output, syntax is OK.
- Fix any line/typo errors if reported.

### 8. Check zone files

Forward zone:

```bash
sudo named-checkzone example.local /var/named/example.local.zone
```

Reverse zone:

```bash
sudo named-checkzone 1.168.192.in-addr.arpa /var/named/1.168.192.in-addr.arpa.zone
```

You should see something like:

```text
zone example.local/IN: loaded serial 2026012301
OK
```

If there are errors (e.g., missing `.` at the end of FQDNs), fix them and re‑run.

---

## Restart BIND and configure firewall

### 9. Restart `named`

```bash
sudo systemctl restart named
sudo systemctl status named
```

Ensure it’s running without errors.

### 10. Open DNS service in firewalld

```bash
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
```

This opens UDP/TCP port 53.

---

## Configure clients to use your DNS server

On your LAN clients (or your RHEL server itself for testing):

- Set **DNS server** to `192.168.1.10` (your BIND server).
- On RHEL, you can use `nmcli` or edit NetworkManager connection settings to set the DNS.

Example with `nmcli` (interface name may differ):

```bash
nmcli connection modify "System eth0" ipv4.dns "192.168.1.10"
nmcli connection up "System eth0"
```

---

## Test DNS resolution

### 11. Test from the DNS server itself

Use `dig`:

```bash
dig @127.0.0.1 ns1.example.local
dig @127.0.0.1 www.example.local
dig @127.0.0.1 -x 192.168.1.20
```

You should see:

- **ANSWER SECTION** with A records for forward lookups.
- **PTR** record for reverse lookup.

### 12. Test from a client machine

From another host using your DNS server:

```bash
dig @192.168.1.10 www.example.local
dig @192.168.1.10 -x 192.168.1.20
```

If everything is correct, you’ll get the same answers.

---

## Optional: Use BIND as a pure caching/forwarding resolver

If you only want caching/forwarding (no local zones):

1. Keep only the default zones in `/etc/named.conf`.
2. Set:

```conf
options {
    recursion yes;
    allow-query { localhost; 192.168.1.0/24; };
    forwarders { 1.1.1.1; 8.8.8.8; };
};
```

3. Restart `named` and point clients to your server. It will cache and forward queries to upstream DNS.

---
