## Domino

Our goal is to chain vulnerabilities together in order to escalate access and obtain multiple different flags.

<sub>(Note that my IP addresses change multiple times throughout the writeup without warning, this is because the Attackbox and Target VM ran into a few issues on several occasions, requiring a restart. If there is ever any change in the IP addresses used for commands, this is why.)</sub>

We start with an nmap scan.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 125657" src="https://github.com/user-attachments/assets/890d8192-5327-4cfd-8859-75d8fcf79ed6" />

Visiting the webpage, we find a login page.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 125758" src="https://github.com/user-attachments/assets/245ef767-5396-4f65-8c02-bd80e4f3ae05" />


One thing to note right away is that usernames are in the format firstname.lastname, which is helpful information. There's also a link below that leads to a page showing the team, which gives us several potential usernames to use. We can put those users in a list, which I named users.txt.

Performing an enumeration on the target shows a directory of interest, in the form of /backup.
```
gobuster dir -u http://10.66.161.229 -w /usr/share/wordlists/dirb/common.txt
```
<img width="1920" height="1140" alt="Screenshot 2026-07-07 125821" src="https://github.com/user-attachments/assets/c728831a-1257-4fdf-a8d7-920340bf7f47" />

Searching for the backup directory does yield results, we find a document named README.txt.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 125939" src="https://github.com/user-attachments/assets/d4d7e8a3-e4ee-4070-a748-d194c1d502c6" />

The contents of the file tell us the encryption format of config.enc and where to find the decryption key.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 130103" src="https://github.com/user-attachments/assets/6e761570-4d4f-401f-aff1-f9060424e1cd" />

We can download config.enc to decrypt later. Going to /static/app.js, we find the key for decryption.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 130331" src="https://github.com/user-attachments/assets/7d45b5f5-a2e6-4e3f-98ef-6edca048f9d2" />

Decrypting the file doesn't actually give us anything of use though.

We can use the users.txt file from earlier to perform a dictionary attack. I managed to discover 3 valid sets of credentials.
```
hydra -L users.txt -P /usr/share/wordlists/SecLists/Passwords/xato-net-10-million-passwords-10000.txt 10.66.161.229 http-form-post '/index.php:username=^USER^&password=^PASS^:F=Invalid credentials'
```
<img width="1920" height="1140" alt="Screenshot 2026-07-07 133331 (Redacted)" src="https://github.com/user-attachments/assets/8b361d7f-f2a3-4536-9047-a8fad39ce7bb" />

Logging into sarah.johnson's account, we learn we can access a file viewer with a token.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 133742" src="https://github.com/user-attachments/assets/1982f6d6-d1e7-47f7-a87b-ac3931099716" />

The My Profile API link takes us to a page to view users' profile information. Access controls weren't properly set here, so changing the ID to 1 lets us view the information of laura.hayes, which contains the first flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 133855 (Redacted)" src="https://github.com/user-attachments/assets/0b11d8eb-c2f4-488b-8fc2-91c92f946b1a" />

We have the ability to open a ticket, and looking closer, it seems HttpOnly was set to false for the cookies, so an XSS attack is possible. We can craft a ticket with a payload designed to steal a cookie. Then we can start a listener and submit the payload.
```
nc -lvnp 8000
```
<img width="1920" height="1140" alt="Screenshot 2026-07-07 145202" src="https://github.com/user-attachments/assets/01d9f2ab-170e-43a9-9bc6-9ba057cea14a" />

The response is the cookie containing admin privileges.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 145246" src="https://github.com/user-attachments/assets/a6dac09f-20f6-4e96-a5d8-6f65d72ecbc3" />

Changing the value of our cookie to the new one and going back to the dashboard, we see we now have access as laura.hayes, who is an admin user.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 145627" src="https://github.com/user-attachments/assets/8f133186-6b3d-4489-875a-376d0522982b" />

With our admin access, we can view /admin, and we find the second flag there.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 160831 (Redacted)" src="https://github.com/user-attachments/assets/175060aa-a427-4f9b-bae5-409fa0ce94bc" />

We know we can access a file viewer, but we need a token to do so. We can get a token, but decoding it, we see it has the role set to user, so we change that to admin and use that encoded token. Using Burp Suite, we can edit the request to include that token.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 150644" src="https://github.com/user-attachments/assets/ba3825f7-c2e9-4472-b416-522aeee975f0" />

Sending a request for a file, we do see that it does accept the token. Now we craft the payload.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 151309" src="https://github.com/user-attachments/assets/3bed64af-21db-44be-b2f6-60634534f5d8" />

Start a web server and a listener, then send the request to upload the payload. Doing so gives us access as user www-data.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 162303" src="https://github.com/user-attachments/assets/f6eab966-f13a-4173-936b-a49c2ad842a7" />

We find the third flag in /opt/flag3.txt.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 162408 (Redacted)" src="https://github.com/user-attachments/assets/8a6f19f3-1885-454f-92d0-1ecde4954edd" />

Looking at /var/www/html/config.php, we find a set of credentials.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 165907 (Redacted)" src="https://github.com/user-attachments/assets/14388266-a694-41da-94f8-cee58d42495e" />

We now have another set of credentials we can use. We can also look at /etc/passwd, and we find devops is a user. Trying to switch to devops using the password we found turns out to work. We can find the fourth flag on the desktop of devops.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 170534 (Redacted)" src="https://github.com/user-attachments/assets/fc764e09-9461-45d6-a16f-9844b8a11d59" />

It turns out the target machine has pspy64 in the /opt/tools directory. We can use that to identify the running processes on the system and see if we can take advantage of any to gain root access.
```
./pspy64
```
<img width="1920" height="1140" alt="Screenshot 2026-07-07 170831" src="https://github.com/user-attachments/assets/91e957b3-72ee-4564-9055-2fa3c957b439" />

It turns out health_report.sh is run with root permissions, and we are able to edit it.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 171152" src="https://github.com/user-attachments/assets/97c3d106-dd66-48a1-bc2c-bff2a26dfcbc" />

Change the file contents to start a reverse shell. The target machine has busybox, so we can use that in our command.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 171647" src="https://github.com/user-attachments/assets/2dd7756d-e4e4-44d3-9580-b2d051bcd099" />

We wait for the script to be automatically run by root, and we gain root privileges. We can find the fifth and final flag in the root directory.
<img width="1920" height="1140" alt="Screenshot 2026-07-07 174041 (Redacted)" src="https://github.com/user-attachments/assets/e20641ab-ab33-4080-b685-d359cc3e8f64" />
