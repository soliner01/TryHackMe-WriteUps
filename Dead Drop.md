## Dead Drop

Our goal is to take advantage of a web-facing file-sharing application to gain access to the corporate network, and escalate privileges while collecting flags along the way.

We start with an nmap scan.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 123312" src="https://github.com/user-attachments/assets/6ab482c1-6fbe-40e5-af6c-551a9b4f5f70" />

Visiting the webpage leads us to a login page.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 123357" src="https://github.com/user-attachments/assets/727cbade-56ea-478d-a1a7-2e464beaaf19" />

We can try SQL injection. It turns out that ' or 1 or ' works.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 124406" src="https://github.com/user-attachments/assets/0446a842-abf4-45b0-9ae6-413de6148356" />

We gain access to the dashboard, and find a file upload service. It seems like there have been previous .js files uploaded by someone before us.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 124536" src="https://github.com/user-attachments/assets/986b8520-8a07-4a0c-a369-5df4ca1d5a6c" />

We can upload a .js script to start a shell and gain access to the corporate network. I tried to get a reverse shell session with the scripts, but I couldn't get it to work, so I'm instead going to try to use the scripts to get the ssh password myself.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 134938" src="https://github.com/user-attachments/assets/b5ec898a-81f5-4904-8a86-361f5d0d326e" />

Uploading this script and previewing it lets us see the contents of /etc/passwd.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 135014" src="https://github.com/user-attachments/assets/1dcf6745-fa7f-4248-bf5a-711103a04c3e" />

Notably, one of these is named backup, meaning there may be a backup somewhere with credentials.

We can dig around in the directories to find something of interest.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 135302" src="https://github.com/user-attachments/assets/964f3ac3-8563-4cca-b1be-eea4f87e9eec" />

Uploading and previewing this script, we find a file called shadow.bak exists.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 135433" src="https://github.com/user-attachments/assets/6402d908-a828-4127-a27e-c19de3250413" />

We can upload another script to view the contents of this file.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 135718" src="https://github.com/user-attachments/assets/a3a26661-1ded-4d76-a3fd-43e4b662204d" />

Previewing this script, we find a hash for the account svc-drop.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 135755" src="https://github.com/user-attachments/assets/466d85a3-f4a6-4746-8039-3b8ea4c8d65a" />

We can use John The Ripper or something similar to crack the hash.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 142352 (Redacted)" src="https://github.com/user-attachments/assets/c1e9c7b0-28db-4bba-aa67-90f60af7e0d9" />

We can use these newly obtained credentials to ssh into the server.

Looking around, we find an .apk file in the backup directory.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 143337" src="https://github.com/user-attachments/assets/b50e35df-3806-44f8-a85f-8ed5155a1015" />

Download the file onto your AttackBox VM.
```
scp svc-drop@192.168.11.200:/home/svc-drop/backup/deaddrop-mobile.apk .
```

Install and run apktool to decompile the .apk file.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 145225" src="https://github.com/user-attachments/assets/b389f8a4-4073-4aba-9d81-8e9f936aaed7" />

Through a bit of digging, we can find hardcoded credentials in the file Config.smali.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 145357 (Redacted)" src="https://github.com/user-attachments/assets/344a0d36-514f-475e-af1a-d66040b46502" />

Because the corporate network isn't accessible to our VM directly, we need to use a proxy to go any further. Use proxychains to solve this issue.
```
nano /etc/proxychains.conf
```
<img width="1920" height="1140" alt="Screenshot 2026-07-09 150854" src="https://github.com/user-attachments/assets/afc62045-e4e5-4b02-8d62-0f2d2b8298fe" />

```
ssh -D 1080 svc-drop@192.168.11.200 -fN

proxychains enum4linux-ng 192.168.11.100
```

We find the domain name through this process.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 151135" src="https://github.com/user-attachments/assets/0965952d-61ab-41d0-b36d-2385058f5e61" />

We can RDP into the target account that we found.
```
proxychains xfreerdp /v:192.168.11.100 /u:j.harris /p:[REDACTED] /d:deaddrop.loc /cert:ignore
```

Now that we're in, we need to find a way to escalate privileges. Using BloodHound, we can identify that j.harris has the ability to add members to the ITSUPPORT-ADMINS group, which is also a member of DOMAIN ADMINS. Add j.harris to ITSUPPORT-ADMINS.
```
proxychains net -dc-ip 192.168.11.100 -target-ip 192.168.11.100 'deaddrop.loc/j.harris:[REDACTED]@DEADDROP-DC.deaddrop.loc' group -name "Domain Admins" -join j.harris
```
<sub>(Note: Because networks are shared amongst other users, it is possible that j.harris already has been added to ITSUPPORT-ADMINS by someone else. This was the case for me, so I didn't need to run a command to do that. I placed what I believe is the command to do so above. I cannot guarantee it will work, though.)</sub>

Go to the Administrator Desktop and open flag.txt to find the final flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-09 155027 (Redacted)" src="https://github.com/user-attachments/assets/f466c56e-5e94-48d6-ae04-9cff4415878d" />
