## Silent Monitor

Our goal is to exploit a vulnerable web application to get into an internal service, and gain root access while collecting 2 flags along the way.

We start with an nmap scan.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 125415" src="https://github.com/user-attachments/assets/6b266274-7bde-42a5-bbb5-43ff3e82caa1" />

Going to the website run on port 5050, we see a dashboard.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 125514" src="https://github.com/user-attachments/assets/47fc6a86-80db-4caf-9c17-5ff4d681b07d" />

Doing a gobuster scan to enumerate only reveals one directory, being /internal. Going to that directory gives us a login page.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 125838" src="https://github.com/user-attachments/assets/26f5b137-d2b2-4953-b314-18272fe209f2" />

We can try SQL injection on the login page. I tried a list of items that I put into a txt file and used Burp Suite to try them all. It turns out that ' or 1 or ' works.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 130742" src="https://github.com/user-attachments/assets/58dc63ea-30e4-4622-9241-a17d1e28874a" />

We successfully login using this SQL injection.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 130850" src="https://github.com/user-attachments/assets/5df491e1-7eda-4ccd-b772-a47420307cae" />

One thing to note is towards the top of the Audit Log, we do see some attempts at SQL injection were attempted previously by some other person. Host Health may be vulnerable to SQL injection as well.

Using Burp Suite, it turns out we can do SQL injection by putting our query on a new line, similar to the attacker before us.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 131548" src="https://github.com/user-attachments/assets/e588bccc-b575-48c0-8022-ee8db7a42e31" />

We can try to use this to set up a reverse shell. It turns out the target machine has busybox set up, so we can use this to get a reverse shell.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 131721" src="https://github.com/user-attachments/assets/57d14f45-2fab-4fb6-be37-983ea0154e8c" />

Set up a listener, and send the query to start a shell. A session should open up.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 132833" src="https://github.com/user-attachments/assets/6f68ec9c-34b9-4f5f-b43a-2939b2bdd73e" />

We find a file called secret.config. Looking at it, we find credentials for sysadmin.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 133001 (Redacted)" src="https://github.com/user-attachments/assets/a7327c11-269f-4c6c-bce4-1f80ce7968e2" />

We can SSH into the target with these credentials.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 133055" src="https://github.com/user-attachments/assets/c34b19a6-4163-4d3e-91a3-fe7cd68ad862" />

We can view the contents of user.txt to obtain the first flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 133130 (Redacted)" src="https://github.com/user-attachments/assets/7a907515-0fb1-4778-9980-aefb47fd3b67" />

In the backups directory, we find a README.txt file and a .kdbx file. This is a KeePass credential database, so we need to crack it. We can start a http server on the target machine, and grab the file onto our local machine.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 133528" src="https://github.com/user-attachments/assets/0f24e93c-2f92-44cf-b557-c3830a96e47d" />

We can convert the file with keepass2john, and then crack it with John The Ripper.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 133803 (Redacted)" src="https://github.com/user-attachments/assets/5bdc1778-df08-493d-9b11-ea7e8fbc0ba0" />

We can use that password to view the contents of the .kdbx file. We find the password for the root user here.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 134352" src="https://github.com/user-attachments/assets/13a229d1-20fc-44ed-b345-6b6222c73d92" />

Using the password, we can gain root access, and get the second and final flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-08 134525 (Redacted)" src="https://github.com/user-attachments/assets/3cf33781-43f2-4eb1-bd53-2dcbb5d95216" />
