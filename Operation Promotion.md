## Operation Promotion

Our goal is to exploit a public-facing portal to compromise the host, while capturing 2 flags along the way.

We start with an nmap scan.
```
nmap -sC -sV -p- 10.66.157.61 --min-rate 1000
```
<img width="1920" height="1140" alt="Screenshot 2026-07-10 123418" src="https://github.com/user-attachments/assets/2bbe0582-876d-4b7b-b729-865cb4cf5e5a" />

One thing to note is robots.txt doesn't allow /admin/.

We can enumerate the SMB share.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 124141" src="https://github.com/user-attachments/assets/d36e8e1c-69ba-4b5d-81ce-544a56494396" />

We can access the SMB share, and we find a file called README.txt.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 124359" src="https://github.com/user-attachments/assets/4de17101-4ea3-442b-b45a-6eadcd8ec7b6" />

Unfortunately, there is nothing of use in the document, so we'll need to switch gears. Visit the target's website.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 124859" src="https://github.com/user-attachments/assets/0966ef2c-fca6-483f-a054-4f1f4e622cdf" />

We don't really find anything on the default page, so our next step is to enumerate.
```
gobuster dir -u http://10.66.157.61 -w /usr/share/wordlists/dirb/common.txt
```
<img width="1920" height="1140" alt="Screenshot 2026-07-10 125028" src="https://github.com/user-attachments/assets/22aa7cb4-241a-4430-8bb9-ac2c608b9c45" />

Visiting /admin, we find a login page.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 125247" src="https://github.com/user-attachments/assets/ed950c83-95c6-41d6-b6f6-611bd5165871" />

We can try using SQL injection to get access. It turns out ' or 1 or ' works.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 125631" src="https://github.com/user-attachments/assets/173bbb82-3ff3-4ea5-a295-90af6fa81d3c" />

When we login, we find a dashboard.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 125714" src="https://github.com/user-attachments/assets/26aa7593-a031-4a7a-bb85-9872df03dbf9" />

We have the ability to look up user profiles. It turns out the user profile with ID 7 is of interest.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 125907" src="https://github.com/user-attachments/assets/db55205c-d586-4551-913a-36b77d5c21d8" />

We can go to that directory we just found, and we find a php that pings the host you give it as a query.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 130658" src="https://github.com/user-attachments/assets/d90f97f0-183c-4fd4-9bfb-d43886d47d31" />

We can try to abuse this with command injection. A ping command is required to be included in the query, so we can just work around that. Setting up a listener and putting in a busybox shell request will start a shell session as www-data.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 132103" src="https://github.com/user-attachments/assets/0040caeb-94ce-4ecf-8292-401629929948" />

As www-data, we can now access the config directory. There, we find a file named db.conf. The contents of the file give us a username and a password hash to work with.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 132420" src="https://github.com/user-attachments/assets/c6b12ee1-fdd7-4bf3-bbbb-513df403748e" />

Unfortunately, I was not able to crack the hash with tools, so I will try brute-forcing the password instead. Recall that on the default webpage, there was a mention of a "Spring 2026 Hiring Drive" going on. Perhaps Spring 2026 could be a clue to the password. Using hashcat with dive.rule, we can create a list of passwords to try using "spring2026" as a base. Then, run hydra to brute-force.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 135033 (Redacted)" src="https://github.com/user-attachments/assets/1adb667d-f989-4e6b-a1a1-97366dda642a" />

We found a set of working credentials. We can ssh into the target with them. On the user's desktop, we find the first flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 135143 (Redacted)" src="https://github.com/user-attachments/assets/cdadcdd5-e258-428d-9aaa-90a7a131df8f" />

We can execute a command to run a root shell using find.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 140116" src="https://github.com/user-attachments/assets/c174cbc9-0261-48aa-ace3-4f869d97c70b" />

We find the second and final flag in /root/flag.txt.
<img width="1920" height="1140" alt="Screenshot 2026-07-10 140352 (Redacted)" src="https://github.com/user-attachments/assets/e66ed9b4-4dad-4b20-bc00-6607db9421ff" />
