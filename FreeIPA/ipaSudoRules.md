Sudo Rules For IPA users:
--------------------------

**1. To run ALL commands on ALL IPA hosts/clients applicable ALL IPA users:**

```
ipa sudorule-add --hostcat='all' --usercat='all' --cmdcat='all' sudorule1
ipa sudorule-add-runasuser --users=root sudorule1
ipa sudorule-show sudorule1 --all
```
Note : sudorule1 is name of rule, it can be any name.

This will allow  all users to become sudo users .

- To allow a specific users running following commonds and then access serever on broweser and add sudo rule for specific user.
```
ipa sudorule-add --hostcat='all' --cmdcat='all' sudorule1
```


**2. To run specific commands on specific hosts/hostgroups applicable to specific IPA users/groups:**
	
	ipa sudorule-add sudorule2
	ipa sudorule-add-user --users=ipauser --groups=admins sudorule2
	ipa sudorule-add-host --hosts=`hostname` --hostgroups=hostgroup sudorule2
	ipa sudocmd-add '/usr/bin/systemctl * sshd.service'
	ipa sudorule-add-allow-command --sudocmds='/usr/bin/systemctl * sshd.service' sudorule2
	ipa sudorule-add-runasuser --users=root sudorule2
	ipa sudorule-show sudorule2 --all



**3. To run ALL commands on specific hosts/hostgroups as LOCAL user(external user) on IPA client:**

	ipa sudorule-add --cmdcat='all'  sudorule3
	ipa sudorule-add-user --users=localuser sudorule3
	ipa sudorule-add-host --hosts=`hostname` sudorule3
	ipa sudorule-add-runasuser --users=root sudorule3
	ipa sudorule-show sudorule3 --all

**4. To run commands without being asked for password prompt (similar to NOPPASSWD option in sudoers) run following command:**
```
ipa sudorule-add-option sudorule1 --sudooption='!authenticate
```

--------
