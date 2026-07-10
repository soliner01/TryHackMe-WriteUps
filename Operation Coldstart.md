## Operation Coldstart

Our goal is to take advantage of an old staging server to gain access and obtain 2 flags.

We start with an nmap scan.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 151600" src="https://github.com/user-attachments/assets/04834c77-e8ac-4589-84cc-0b8411f18b58" />

One thing to note right away is anonymous access to the FTP server is allowed. We can get into the FTP server. Looking around a bit, we find a file called backup.tar.gz. Download it to the AttackBox.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 152033" src="https://github.com/user-attachments/assets/18b0d2f4-26c9-478a-bec9-22ba9bc0d99a" />

Extract the files.
```
tar xfz backup.tar.gz
```

In app.py, we find a piece comments near the top that inform us that kestrel.thm resolves to 127.0.0.1 in the internal /etc/hosts file. We also learn that the admin endpoint is only accessible internally.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 153400" src="https://github.com/user-attachments/assets/93eced71-0117-44c0-b156-c9df212268e6" />

There's nothing else of interest found there, so we move on to the next location, being the website. Navigating to the website, we find a URL Preview Service.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 154748" src="https://github.com/user-attachments/assets/85435974-0844-4860-93e1-bfea8ed29aa6" />

However, upon trying 127.0.0.1 as a test, I was told that was not in the approved internal allow-list. It seems like it will only take websites on the allow list. app.py told us that kestrel.thm resolves to 127.0.0.1 in the internal /etc/hosts file. It turns out that kestrel.thm is on the allow list.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 154859" src="https://github.com/user-attachments/assets/7893c8ff-0073-4b0b-8396-2a82bd7b3d1f" />

Towards the bottom of app.py, we see a bit of code mentioning /admin/, and it also says if the path is /admin/notes, it will open a file called admin_notes.txt. This sounds of interest, so we check this path out. Doing so, we find a set of credentials for SSH.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 155325 (Redacted)" src="https://github.com/user-attachments/assets/d0cc3125-a9d2-40fa-800f-ac0a432f61cb" />

We ca SSH in as the user webdev, and find the first flag on their desktop in user.txt.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 155439 (Redacted)" src="https://github.com/user-attachments/assets/51b82df1-92cf-416a-aa25-4483f39cffde" />

I tried sudo -l, but webdev does not appear to have the ability to use sudo commands. So the next place to check would be cron jobs. Looking around, we find one of interest, being voltlabs-backup.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 155720" src="https://github.com/user-attachments/assets/fadc446d-07d6-4181-8248-3e66eb5c2d63" />

The cron job seems to have a vulnerability. Because of the wildcard usage, if we were to upload a file with a name that is a command, it will likely run said command. We can take advantage of that to run a shell as a root user.
```
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' > shell.sh

touch -- '--checkpoint=1'

touch -- '--checkpoint-action=exec=sh shell.sh'
```

Wait until a file named bash appears in /tmp/.Once the file is there, run it. You will get a shell with root permissions.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 165304" src="https://github.com/user-attachments/assets/afd89d87-f188-49b6-8fb9-e3ace9cd254d" />

The second and final flag is located at /root/flag.txt.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 165321 (Redacted)" src="https://github.com/user-attachments/assets/47486e4c-8f29-4cb2-bb0f-143ac67750c4" />
