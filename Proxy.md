## Proxy

Our goal is to traverse through the Active Directory environment and get the flag from the Administrator's Desktop.

Start with an nmap scan on the target.
```
nmap -sC -sV -p- -Pn 10.65.132.214
```
<img width="1920" height="1140" alt="Screenshot 2026-06-30 163630" src="https://github.com/user-attachments/assets/e554793f-5874-4b0a-827c-7372fef74e9d" />

We find an opening in the form of SMB. Using nxc, we can enumerate the SMB shares.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 164116" src="https://github.com/user-attachments/assets/6f6808e7-d5db-4838-a2d9-3c9aa5deba1e" />

We can also enumerate the users by brute-forcing the rid. These will be stored in a file for later, which I named users.txt.
```
nxc smb 10.65.132.214 -u 'guest' -p '' --rid-brute
```
<img width="1920" height="1140" alt="Screenshot 2026-06-30 164922" src="https://github.com/user-attachments/assets/d14e1fb4-8b4f-4db8-b9db-eee15b23346c" />

Using smbclient.py, we can connect to the share.
```
smbclient.py guest:''@10.65.132.214
```
We have access to a few of the shares. Looking at IT-Shared, we find a few txt files of potential interest.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 170022" src="https://github.com/user-attachments/assets/f547a6bb-9447-442b-bdc2-130d6f8ef104" />

Looking at IT-Credentials-Backup.txt, we find two sets of credentials. Unfortunately, the credentials don't actually work, as I tried using them on all accounts listed in users.txt and did not find any success there, so we're stuck as a guest for now. However, looking at IT-Onboarding-Checklist.txt, we learn that svc.scanner runs every 2 minutes, which gives us a potential attack vector. We also learn svc.mssql handles nightly MSSQL backups, so we have a new target.

We can upload a file to IT-Shared that causes the system to dump its hashes, which I have called share.bat.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 172658" src="https://github.com/user-attachments/assets/16775f32-60da-4ea1-b23a-c06036dd3055" />

Uploading this file and setting up an smb server will give us an NTLMv2 hash.
```
responder -I ens5 -v
```
<img width="1920" height="1140" alt="Screenshot 2026-06-30 174910" src="https://github.com/user-attachments/assets/42922ae0-cd38-4505-a13b-14935cdb556a" />

Using hashcat, we can crack this hash.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 175500 (Redacted)" src="https://github.com/user-attachments/assets/993257b0-c74c-4ad0-a0d5-8f806368f7e1" />

With the newly acquired set of credentials, we can now gain access to the shares using the svc.scanner account.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 175816 (Redacted)" src="https://github.com/user-attachments/assets/11588ba5-0ef2-4896-965f-273bc2c714e3" />

We can request an Administrator service ticket as the next step.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 183249 (Redacted)" src="https://github.com/user-attachments/assets/316bacb6-4e08-40b6-b19e-716b4337fa02" />

We can now export that ticket to the KRb5CCNAME environment variable, then use smbclient.py to authenticate to DC01 using that ticket to gain access as the Administrator. Make sure you edited /etc/hosts to include "10.65.132.214 DC01.ctf.local DC01", else this will not work.
```
export KRB5CCNAME=Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```
From here, we can go to the Desktop to find the flag.
<img width="1920" height="1140" alt="Screenshot 2026-06-30 185813 (Redacted)" src="https://github.com/user-attachments/assets/506bf6ce-5b8b-4a8f-8931-3be732eeb6e2" />
