## Overview of DNS

- The Domain Name System (DNS) is a distributed, hierarchical naming system that translates human‑readable domain names (like `www.example.com`) into IP addresses (like `192.0.2.10` or `2001:db8::10`). 
- Computers communicate using IP addresses, but humans prefer names—DNS is the “phonebook” that connects the two.  

Key ideas:

- **Hierarchical:** Root → TLD (`.com`, `.org`, `.sa`) → domain (`example.com`) → host (`www.example.com`).
- **Distributed:** No single central server; many DNS servers cooperate globally.
- **Caching:** Results are cached to speed up lookups and reduce traffic.

---

## Types of DNS servers

### 1. Recursive resolver

- **Role:** Receives queries from clients (like your laptop or a local DNS server) and does the “full walk” through the DNS hierarchy to find the answer.
- **Behavior:** It queries root, TLD, and authoritative servers as needed, then returns the final IP to the client and caches it for future use.  

### 2. Root name servers

- **Role:** Top of the DNS hierarchy.
- **Behavior:** They don’t know every domain, but they know where to find the TLD servers (like `.com`, `.net`, `.sa`).
- **Note:** There are a small number of root server clusters globally, operated by different organizations.

### 3. TLD name servers

- **Role:** Responsible for top‑level domains such as `.com`, `.org`, `.sa`, `.net`, etc.
- **Behavior:** When asked about `www.example.com`, a TLD server for `.com` will respond with the authoritative name servers for `example.com`.

### 4. Authoritative name servers

- **Role:** Hold the actual DNS records for a zone (e.g., `example.com`).
- **Behavior:** They provide final answers for names in their zones (A, AAAA, MX, NS, etc.), without needing to ask other servers.  

### 5. Caching/forwarding DNS servers (often recursive)

- **Caching server:** Stores answers from previous lookups to respond faster next time.
- **Forwarder:** Instead of doing full recursion, it forwards queries to another DNS server (often an ISP or public resolver) and caches the responses.

---

## Types of DNS queries

### 1. Recursive query

- **Definition:** The client asks the DNS server to return either a final answer or an error—no referrals.
- **Typical use:** Your PC to your local DNS resolver. The resolver then does all the work (root → TLD → authoritative) if needed.

### 2. Iterative query

- **Definition:** The server responds with the best information it has—either the answer or a referral to another DNS server closer to the answer.
- **Typical use:** Between DNS servers (recursive resolver → root → TLD → authoritative).

### 3. Inverse (reverse) query

- **Definition:** Resolves an IP address back to a hostname, using special reverse zones like `10.2.0.192.in-addr.arpa` for IPv4.
- **Use case:** Logging, security tools, and some applications that want to map IPs back to names.

---
