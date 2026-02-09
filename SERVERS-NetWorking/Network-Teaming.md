
#  What Network Bonding/NIC-teaming  Is?

Network bonding (also called NIC teaming) combines **multiple physical network interfaces** into **one logical interface**. The goal is either:

### **1. High Availability (Failover)**
If one NIC dies, traffic continues on the other.

### **2. Load Balancing / Throughput Aggregation**
Traffic is distributed across multiple NICs.

### **3. Both**, depending on the bonding mode.

The bonded interface appears as a single device (e.g., `bond0`) to the OS.

---

#  Bonding Modes (Most Common)

| Mode | Name | Purpose | Notes |
|------|------|---------|-------|
| **0** | balance-rr | Load balancing + failover | Switch must support it |
| **1** | active-backup | High availability | Most common; no switch config needed |
| **2** | balance-xor | Load balancing | Switch must support LACP-like behavior |
| **4** | 802.3ad (LACP) | Load balancing + failover | Requires switch LACP |
| **5** | balance-tlb | Transmit load balancing | No switch config |
| **6** | balance-alb | Adaptive load balancing | No switch config |

For most servers, you’ll use **mode 1** (simple failover) or **mode 4** (LACP).

---

#  How RHEL 9.5 Handles Bonding

RHEL 9 uses **NetworkManager**, not the old `/etc/sysconfig/network-scripts/` system.

You configure bonding using:

- `nmcli` (recommended)
- Keyfiles under `/etc/NetworkManager/system-connections/`
- Cockpit (GUI)

Below is the cleanest CLI method.

---

#  Step-by-Step: Configure Network Bonding on RHEL 9.5

Let’s assume your NICs are:

- `ens33`
- `ens34`

And you want a bond called `bond0`.

---

## **1. Create the bond interface**

Example: **active-backup mode (mode 1)**

```bash
sudo nmcli connection add type bond ifname bond0 con-name bond0 \
    bond.options "mode=active-backup,miimon=100"
```

For **LACP (mode 4)**:

```bash
sudo nmcli connection add type bond ifname bond0 con-name bond0 \
    bond.options "mode=802.3ad,miimon=100,lacp_rate=fast"
```

---

## **2. Assign IP settings to the bond**

Static IP example:

```bash
sudo nmcli connection modify bond0 ipv4.addresses "192.168.1.10/24"
sudo nmcli connection modify bond0 ipv4.gateway "192.168.1.1"
sudo nmcli connection modify bond0 ipv4.dns "8.8.8.8"
sudo nmcli connection modify bond0 ipv4.method manual
```

Or DHCP:

```bash
sudo nmcli connection modify bond0 ipv4.method auto
```

---

## **3. Add slave interfaces to the bond**

```bash
sudo nmcli connection add type ethernet ifname ens33 con-name ens33-bond0 master bond0
sudo nmcli connection add type ethernet ifname ens34 con-name ens34-bond0 master bond0
```

---

## **4. Bring the bond up**

```bash
sudo nmcli connection up bond0
sudo nmcli connection up ens33-bond0
sudo nmcli connection up ens34-bond0
```

---

## **5. Verify bonding status**

```bash
cat /proc/net/bonding/bond0
```

You’ll see:

- Bonding mode
- Active slave
- Link status
- LACP details (if mode 4)

---

#  Example Bond Configuration File (for reference)

NetworkManager stores connections here:

```
/etc/NetworkManager/system-connections/bond0.nmconnection
```

A typical bond file looks like:

```ini
[connection]
id=bond0
type=bond
interface-name=bond0

[bond]
mode=active-backup
miimon=100

[ipv4]
method=manual
address1=192.168.1.10/24,192.168.1.1
dns=8.8.8.8;
```

---

#  Testing Your Bond

### **Simulate NIC failure**
```bash
sudo nmcli device disconnect ens33
```

Check if traffic continues through `ens34`.

### **Check active interface**
```bash
cat /proc/net/bonding/bond0 | grep "Currently Active Slave"
```

---
Thanks

Just tell me what direction you want to take next.
