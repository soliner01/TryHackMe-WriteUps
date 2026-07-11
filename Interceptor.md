## Interceptor

Our goal is to use a proxy to understand traffic behaviour, then use that information to manipulate requests and gain access to the system.

We start with an nmap scan.
```
nmap -sC -sV -p- 10.64.165.184 --min-rate 1000
```
<img width="1920" height="1140" alt="Screenshot 2026-07-11 130515" src="https://github.com/user-attachments/assets/49bba911-f05c-4dee-8ae3-eb2e31a2cab7" />

Visiting the webpage, we find an introductory page, and a link to a login page.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 130853" src="https://github.com/user-attachments/assets/cc6045bf-e413-4b4b-b90f-678a103d0832" />

The login page doesn't seem to have any information that could be used to help us get in.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 131001" src="https://github.com/user-attachments/assets/74aa2790-6fa5-420f-90bc-c5e7af00ea7e" />

I tried using SQL injection to get in, but it didn't seem to work. I also noticed it said "Too many login attempts" so brute-force doesn't seem like an option. Enumerating the directory has an issue as well because it returns a 200 ok response to pages that don't exist, but since those have a fixed length as a response, we can just filter out any results with that length. This lets us successfully enumerate the directories.
```
gobuster dir -u http://10.64.165.184 -w /usr/share/wordlists/dirb/common.txt --exclude-length 1491
```
<img width="1920" height="1140" alt="Screenshot 2026-07-11 134153" src="https://github.com/user-attachments/assets/e6da1a8f-17ab-4ac2-8d2a-bbfec11f2a57" />

I kept digging around with the enumeration a bit more, and came across a file called login.php.bak.
```
gobuster dir -u http://10.64.165.184 -w /usr/share/wordlists/dirb/common.txt -x bak,tmp,zip,sql,backup,php.bak --exclude-length 1491
```
<img width="1920" height="1140" alt="Screenshot 2026-07-11 134656" src="https://github.com/user-attachments/assets/70d814a0-2be4-4532-9525-ba81c5039993" />

Looking at the contents of the file, we find information of use.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 134927" src="https://github.com/user-attachments/assets/7d4abe1d-1d14-4f70-8f6c-3172994f76c8" />

We now know an email account we can use, and the format for the password. I managed to figure out the password, which was [REDACTED]. However, despite having valid credentials, I didn't get in, as there appears to be MFA.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 135220" src="https://github.com/user-attachments/assets/7f198127-59d6-4a03-8619-c1f9a6465b69" />

Obviously, we can't actually get the OTP, so we need to try to bypass it instead. Intercepting the request, we see the response has a field named is_verified.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 135447" src="https://github.com/user-attachments/assets/28ea7eed-2896-42d0-8697-c98edf386281" />

I added is_verified:true to my POST request, and it appears as though it accepted that.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 135604" src="https://github.com/user-attachments/assets/4be57520-22c7-4a7a-bb8c-3a65c562bf71" />

Switching back to the browser, we now have access to the dashboard, which contains the first flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 140619 (Redacted)" src="https://github.com/user-attachments/assets/2d04c579-b410-4161-869d-68e29d2daa0e" />

The Import Feed function looks to be a potential vector to get a shell session. I tried http://127.0.0.1 as a test, but it didn't allow that. However, doing a nonsensical ip address such as http://127.0.0 worked, so we can phrase our exploit that way. I tried to force the target to use busybox to start a shell session, and it worked.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 141034" src="https://github.com/user-attachments/assets/df8d27aa-4d7f-4751-9933-eaa660ff3d3f" />

With this shell session, we can view the contents of /var/www/user.txt to get the second and final flag.
<img width="1920" height="1140" alt="Screenshot 2026-07-11 141223 (Redacted)" src="https://github.com/user-attachments/assets/f5b26189-f511-49e5-aea6-aacd88fdd4ae" />
