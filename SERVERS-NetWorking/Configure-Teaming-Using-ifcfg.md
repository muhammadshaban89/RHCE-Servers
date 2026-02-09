How to configue NIC-Teaming Using `ifcg` connection file in Redhat7
--------------------------------------------------------
- Network bonding in Red Hat Enterprise Linux (RHEL) allows you to combine multiple network interfaces into a single logical interface. 
- This can provide redundancy, load balancing, and increased throughput. 
- Here's a step-by-step guide to set up network bonding in RHEL:

# Step 1: Install Necessary Packages
Ensure that the ifenslave package is installed. This package is typically included by default, but you can install it using:
  
    sudo yum install -y ifenslave

# Step 2: Configure Bonding Interface

- Create a configuration file for the bonding interface, typically named `ifcfg-bond0`. This file should be placed in the `/etc/sysconfig/network-scripts/ directory`.

      vi  /etc/sysconfig/network-scripts/ifcfg-bond0
  
Add the following content to the file:
```
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.100.70
PREFIX=24
GATEWAY=192.168.100.1
BONDING_OPTS="mode=active-backup miimon=100"
```
- **miimon=100**
This sets the link monitoring interval to 100 ms.
• 	Every 100 ms, the bond checks if the active NIC is alive.
• 	If link is down, failover happens immediately.

# Step 3: Configure Slave Interfaces:

- Create or modify the configuration files for the slave interfaces (e.g., `ifcfg-eth0` and `ifcfg-eth1`).

For ifcfg-eth0:

    vi /etc/sysconfig/network-scripts/ifcfg-slave1
    
Add the following content:
```
DEVICE=eth0
NAME=bond0-slave1
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```
For ifcfg-eth1:
  
    vi 	/etc/sysconfig/network-scripts/ifcfg-slav2
    
Add the following content:
```
DEVICE=eth1   
NAME=bond0-slave2
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
````
* Note :Eth0, Eth1  are   interface name in  , it could be any other name in your case.
* And `bond0-slav1` and `bond0-slave2` are slaves names, which could be any name as per your desire or requirement.

# Step 4: Restart Network Service

Restart the network service to apply the changes:
```
  systemctl restart Networkmanager
	nmcli connection up bond0
	nmcli connection up bond0-slave1
	nmcli connection up bond0-slave2
	nmcli connection show 
	nmcli connection show bond0
````
# Step 5: Verify Bonding Status

Check the status of the bonding interface to ensure it is working correctly:

	cat /proc/net/bonding/bond0

- `teamdatectl` is used to chaeck status of controller.
	
       teamdctl bond0 state
  
This command will display detailed information about the bonding interface, including the status of each slave interface.

------

Some Importent Commands 
-----------------------
- These are useful in case your running RHEL7 or older versions.

- The bonding driver is a kernel module named **bonding**.
  
To see if it’s currently loaded:

### **1. Using lsmod**
```bash
lsmod | grep bonding
```

If loaded, you’ll see something like:

```
bonding               163840  0
```

If nothing appears, the module is not loaded.

### **2. Using modinfo (indirect check)**
```bash
modinfo bonding
```

This shows module details, but does **not** confirm it’s loaded — it only confirms the module exists on the system.

---

#  Load the bonding module again

If the bonding module is not loaded, or you removed it using `rmmod bonding`, load it back with:

```bash
sudo modprobe bonding
```

This:

- Loads the bonding kernel module  
- Makes `/proc/net/bonding/` available  
- Enables creation of bond interfaces  

To verify:

```bash
lsmod | grep bonding
```

---

#  Verify bonding support in the kernel

There are several ways to confirm that your kernel supports bonding.

# 1. Check if the bonding module exists**
```bash
modinfo bonding
```

If you get output, bonding support is present.

If you get:
```
modinfo: ERROR: Module bonding not found
```
then bonding is not available (rare on RHEL).

---

# 2. Check kernel config**
RHEL kernels almost always include bonding as a module.  
You can confirm with:

```bash
grep BONDING /boot/config-$(uname -r)
```

You should see:

```
CONFIG_BONDING=m
```

Meaning bonding is supported as a loadable module.

---

# 3. Check if bonding interfaces can be created**
If the module is loaded, this directory exists:

```bash
ls /proc/net/bonding
```

If empty → no bonds exist  
If missing → bonding module not loaded



rmmod bonding 
---------------
- is a **kernel‑level command** used to **unload the bonding kernel module** from Linux.  
- It removes the *bonding driver* itself from the kernel — not just a bond interface.
---

#  What `rmmod bonding` Does

`bonding` is a kernel module that provides the bonding functionality.  
When you run:

```bash
rmmod bonding
```

You are telling the kernel:

- “Unload the bonding driver”
- “Remove all bonding functionality from the kernel”
- “Disable all bond interfaces”

This is a **low‑level operation**.

---

# ⚠️ Important: You Can Only Run `rmmod bonding` If…

### 1. **No bond interfaces exist**
All bonds must be removed first:

```bash
ip link delete bond0
ip link delete bond1
```

or via nmcli:

```bash
nmcli connection delete bond0
nmcli connection delete bond0-slave1
nmcli connection delete bond0-slave2
```

### 2. **No process is using the bonding module**
If any bond is active, `rmmod` will fail with:

```
rmmod: ERROR: Module bonding is in use
```

---

#  When Should You Use `rmmod bonding`?

You rarely need it.  
It is used only in special cases:

- Testing kernel module loading/unloading  
- Resetting bonding behavior  
- Debugging bonding issues  
- Removing bonding support from a minimal system  

For normal admin tasks (creating or deleting bonds), you **do not** use `rmmod`.

---

#  Difference Between Removing a Bond vs Removing the Bonding Module

| Task | Command | Meaning |
|------|---------|---------|
| Remove a bond interface | `nmcli connection delete bond0` | Deletes only the bond |
| Remove bonding module | `rmmod bonding` | Removes the entire bonding driver from kernel |

Removing the module is like uninstalling the engine, not just turning off the car.

---

#  Summary

- `rmmod bonding` unloads the **bonding kernel module**  
- You must delete all bond interfaces first  
- Only used for debugging or resetting bonding  
- Not required for normal bond removal  

---



