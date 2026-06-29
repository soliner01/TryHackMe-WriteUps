## Support

Our goal is to gain admin access on the portal, and grab the contents of a particular file.

We start with an Nmap port scan, and found that the httponly flag is not set, giving us an attack vector.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 155612" src="https://github.com/user-attachments/assets/474fcefe-e7d2-4591-8291-2ebec48aee23" />

Next, by enumerating the directories using gobuster dir -u http://10.64.183.254 -w /usr/share/wordlists/dirb/common.txt, we can see several directories of note (although not shown, it is more effective to add the parameter -x php,html as it reveals more things of note, including config.php).
<img width="1920" height="1140" alt="Screenshot 2026-06-14 160358" src="https://github.com/user-attachments/assets/ad253ae9-2c35-4377-b5a7-681fecc02fa1" />

Knowing that the form defaults to help@support.thm, it's very likely an account with that email exists. Using hydra to brute-force the account, we successfully found a working password.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 163851" src="https://github.com/user-attachments/assets/e3b2509b-152a-4c43-94fd-cf0274ed1fa2" />

Checking the cookie, we can see that the value for isITUser appears as if it was encrypted. It turns out that it is encrypted as an MD5 hash. Decrypting it, we find it says "false" in plaintext. By changing the value of that cookie to an MD5 encrypted "true" and refreshing the page, I gain access to the IT Admin Panel.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 164540" src="https://github.com/user-attachments/assets/5980441b-dc86-49ce-9585-913aefc08029" />

Opening the API, we can see that we can make GET requests for our own profile. It turns out there is an IDOR vulnerability here, and by viewing the profile of the user with ID 1, we see the details of the admin account.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 164821" src="https://github.com/user-attachments/assets/a6877fc0-dfe0-4d7e-9b83-4e94a79eca52" />

There is also a button to select theme which affects the dashboard. It does appear to be vulnerable to File Inclusion. By executing File Inclusion on config.php, we find it contains a password that appears to belong to the admin account.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 165938 (Redacted)" src="https://github.com/user-attachments/assets/71b1682c-1c79-48e8-9bb8-a96071f9f6a8" />

Using the obtained username and password, we find the credentials DO NOT give us access. However, by slightly changing the password, we do successfully gain access to the admin account.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 170532 (Redacted)" src="https://github.com/user-attachments/assets/94c38921-bb36-4faf-a39f-c606b7f64154" />

On this dashboard, there appears to be a new dropdown, in the form of date/time. It seems to be a POST request, so editing and resending that post request with the addition of cat /home/ubuntu/user.txt gives us the second flag.
<img width="1920" height="1140" alt="Screenshot 2026-06-14 171751 (Redacted)" src="https://github.com/user-attachments/assets/11e3f6c5-3b25-48c4-aed7-f57d0496cf7c" />
