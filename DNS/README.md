## Overview of DNS

- The Domain Name System (DNS) is a distributed, hierarchical naming system that translates human‑readable domain names (like `www.example.com`) into IP addresses (like `192.0.2.10` or `2001:db8::10`). 
- Computers communicate using IP addresses, but humans prefer names—DNS is the “phonebook” that connects the two.  

Key ideas:

- **Hierarchical:** Root → TLD (`.com`, `.org`, `.sa`) → domain (`example.com`) → host (`www.example.com`).
- **Distributed:** No single central server; many DNS servers cooperate globally.
- **Caching:** Results are cached to speed up lookups and reduce traffic.

How DNS works
--------------

* DNS servers convert URLs and domain names into IP addresses that computers can understand and use.
*  They translate what a user types into a browser into something the machine can use to find a webpage.
*   This process of translation and lookup is called DNS resolution.
  
**The basic process of a DNS resolution follows these steps:**

- 1.	The user enters a web address or domain name into a browser.
-	2.	The browser sends a message, called a recursive DNS query, to the network to find out which IP or network address the domain corresponds to.
- 3.	The query goes to a recursive DNS server, which is also called a recursive resolver, and is usually managed by the internet service provider (ISP). If the recursive resolver has the address, it will return the address to the user, and the webpage will load.
- 4.	If the recursive DNS server does not have an answer, it will query a series of other servers in the following order: DNS root name servers, top-level domain (TLD) name servers and authoritative name servers.
- 5.	The three server types work together and continue redirecting until they retrieve a DNS record that contains the queried IP address. It sends this information to the recursive DNS server, and the webpage the user is looking for loads. DNS root name servers and TLD servers primarily redirect queries and rarely provide the resolution themselves.
- 6.	The recursive server stores, or caches, the A record for the domain name, which contains the IP address. The next time it receives a request for that domain name, it can respond directly to the user instead of querying other servers.
- 7.	If the query reaches the authoritative server and it cannot find the information, it returns an error message.

- The entire process querying the various servers takes a fraction of a second and is usually imperceptible to the user.
DNS servers answer questions from both inside and outside their own domains. When a server receives a request from outside the domain for information about a name or address inside the domain, it provides the authoritative answer.
When a server gets a request from within its domain for a name or address outside that domain, it forwards the request to another server, usually one managed by its ISP


DNS Resolution Flow (Visual Diagram)
------------------------------------

                ┌──────────────────────────┐
                │      User's Device       │
                │  (Browser / Application) │
                └─────────────┬────────────┘
                              │
                              ▼
                ┌──────────────────────────┐
                │   Local DNS Resolver     │
                │ (Usually ISP or OS stub) │
                └─────────────┬────────────┘
                              │
                Does it have the answer?
                     │ Yes          │ No
                     ▼              ▼
             Return cached IP   Continue lookup
                                   │
                                   ▼
                ┌──────────────────────────┐
                │     Root DNS Servers     │
                │          (.)             │
                └─────────────┬────────────┘
                              │
                              ▼
                ┌──────────────────────────┐
                │   TLD DNS Servers        │
                │ (.com, .net, .org, etc.) │
                └─────────────┬────────────┘
                              │
                              ▼
                ┌──────────────────────────┐
                │ Authoritative DNS Server │
                │ (Holds A, AAAA, CNAME…)  │
                └─────────────┬────────────┘
                              │
                              ▼
                ┌──────────────────────────┐
                │     Final DNS Answer     │
                │   (e.g., A = 93.184.216.34) │
                └─────────────┬────────────┘
                              │
                              ▼
                ┌──────────────────────────┐
                │  Local Resolver Caches   │
                │      the response        │
                └─────────────┬────────────┘
                              │
                              ▼
                ┌──────────────────────────┐
                │   User's Device Gets IP  │
                │ Browser connects to site │
                └──────────────────────────┘

---

What Are DNS Records?
---------
- DNS records are entries inside a DNS zone file that map human‑readable domain names (like ) to technical information such as IP addresses, mail servers, or service endpoints.
- They act like a directory that helps browsers, email servers, and applications know where to go and how to connect.

  #  DNS Record Types and What They Do

##  **1. A Record (Address Record)**
- Maps a domain name to an **IPv4 address**.
- Example: `example.com → 93.184.216.34`
- Used by browsers to reach a server.

---

##  **2. AAAA Record (IPv6 Address Record)**
- Same purpose as an A record but for **IPv6 addresses**.
- Example: `example.com → 2606:2800:220:1:248:1893:25c8:1946`

---

##  **3. CNAME Record (Canonical Name Record)**
- Points one domain to another domain (not to an IP).
- Useful for aliases.
- Example: `www.example.com → example.com`

---

##  **4. MX Record (Mail Exchange Record)**
- Specifies mail servers responsible for receiving email for a domain.
- Includes **priority** values.
- Example: `10 mail1.example.com`

---

## **5. TXT Record (Text Record)**
- Stores arbitrary text.
- Common uses:
  - SPF (email sender validation)
  - DKIM (email signing)
  - Domain verification (Google, AWS, etc.)

---

##  **6. NS Record (Name Server Record)**
- Defines which name servers are authoritative for a domain.
- Example: `ns1.example.com`, `ns2.example.com`

---

##  **7. SOA Record (Start of Authority)**
- Contains administrative information about the zone:
  - Primary name server
  - Contact email
  - Serial number
  - Refresh/Retry/Expire TTL values
- Required for every DNS zone.

---

##  **8. PTR Record (Pointer Record)**
- Used for **reverse DNS lookup**.
- Maps an IP address back to a domain name.
- Example: `93.184.216.34 → example.com`

---

##  **9. SRV Record (Service Record)**
- Specifies the location of specific services.
- Format includes:
  - Priority
  - Weight
  - Port
  - Target
- Example: `_sip._tcp.example.com`

---

## **10. SPF Record (Sender Policy Framework)**
- Technically stored as a TXT record.
- Defines which mail servers are allowed to send email for the domain.

---

## **11. CAA Record (Certification Authority Authorization)**
- Controls which Certificate Authorities can issue SSL certificates for the domain.
- Helps prevent unauthorized certificate issuance.

---

##  **12. NAPTR Record (Naming Authority Pointer)**
- Used for advanced service discovery.
- Common in VoIP (SIP) and ENUM systems.

---

##  **13. AFSDB Record**
- Used for AFS (Andrew File System) and OSF DCE.
- Rarely used today.

---

##  **14. HINFO Record (Host Information)**
- Provides CPU and OS details.
- Mostly deprecated for security reasons.

---

##  **15. RP Record (Responsible Person)**
- Lists the email of the person responsible for the domain.

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

## How many DNS root servers exist?

Two different numbers matter when talking about DNS root servers, and people often mix them up.  

### **1. There are 13 *named* DNS root servers**
These are the famous **A–M root servers**.  
They appear as 13 entries in the DNS root zone because of historical and technical limits (mainly UDP packet size).  


###  **2. But there are *thousands* of physical root server instances**
Thanks to **anycast routing**, each of the 13 root server names is actually backed by many servers distributed worldwide.

As of **January 2026**, the global root server system consists of **1,959 physical instances**, operated by 12 organizations.  


---

## Summary

| Concept | Count | Meaning |
|--------|-------|---------|
| **Named root servers** | **13** | Logical DNS entries (A–M) in the root zone |
| **Physical root server instances** | **~1,959** | Real servers distributed globally using anycast |



👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a

👉Youtube Channel: http://www.youtube.com/@engrm.shaban5099
