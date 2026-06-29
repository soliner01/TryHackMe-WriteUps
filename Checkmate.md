Our goal is to crack 5 different passwords by using any information we find.

For Level 1, we are informed that a firewall is deployed at firewall.thm:5001, but the credentials were kept as the default.
<img width="1920" height="1140" alt="Screenshot 2026-06-18 135310" src="https://github.com/user-attachments/assets/020d76c8-d3ab-47bb-aa39-4d0001b26859" />

From the hint we were given, we know the password is likely something generic, so we try generic passwords. We also know the username is likely admin, as that is the default value on the login page. Through trial and error (or using hydra with a custom-made wordlist fitting the criteria), we find that the username is admin, and the password is [REDACTED].
<img width="1920" height="1140" alt="Screenshot 2026-06-18 140004" src="https://github.com/user-attachments/assets/fd84f4b6-1764-430f-a2ae-2078ba0f582c" />

For level 2, we are informed that there is an internal Employee Login panel on port 5002, and it used common company keywords as passwords. This once again gives us the knowledge that we need to construct a wordlist to use. Looking at the top of the page, we see a bunch of keywords, such as innovation, excellence, security, etc. We can craft a wordlist with those words.
<img width="1920" height="1140" alt="Screenshot 2026-06-18 140950" src="https://github.com/user-attachments/assets/f6185de1-e5cd-4fc7-a9c0-0d6c15dba444" />

We know the username is likely marco because of the username defaulting to that value. Trying the wordlist we crafted, we find the username is indeed marco, and the password is [REDACTED].
<img width="1920" height="1140" alt="Screenshot 2026-06-18 141144 (Redacted)" src="https://github.com/user-attachments/assets/8d15e2aa-409f-4677-97ab-6c0d246e42c8" />

Trying that combination gives us access to the portal.
<img width="1920" height="1140" alt="Screenshot 2026-06-18 141218" src="https://github.com/user-attachments/assets/b6fa092e-51a8-4b35-a872-fa6a0afbd1b1" />

For level 3, we are told to navigate to port 5003, and derive Marco's password from personal info. From logging in on level 2, we got access to that personal info, so we can try using that. However, upon trying the info we got, it seems that's not the password on its own, so we need to go a bit further. Using the CUPP (Common User Passwords Profile) tool, we can generate many combinations using the personal info we found. Note that CUPP does not come pre-installed on the VMs TryHackMe provides, however you can install it yourself. Using this, we find that the username is marco and the password is [REDACTED].
<img width="1920" height="1140" alt="Screenshot 2026-06-18 142147 (Redacted)" src="https://github.com/user-attachments/assets/22b8e251-0666-4dcc-878e-f2ae6e58cc08" />

For level 4, we just need to get the hash of the filename for the profile picture Marco uploaded, and crack it. Using Developer Tools, we can see the filename of the picture.
<img width="1920" height="1140" alt="Screenshot 2026-06-18 142523" src="https://github.com/user-attachments/assets/6774fd48-0620-4a9e-a640-03d8a5c29b3e" />

We simply need to identify the type of hash, and run it through a tool to crack it, such as John the Ripper and Hashcat. We find that the hash is in SHA256 format, so now we can use a tool to crack it. I used John the Ripper with the wordlist file rockyou.txt, and found the plaintext to be [REDACTED].
<img width="1920" height="1140" alt="Screenshot 2026-06-18 143212 (Redacted)" src="https://github.com/user-attachments/assets/c4a36278-9cc8-4961-8608-49c933e5b2fe" />

For level 5, we need to brute-force the SSH service. We are told the username is marco, and the password is predictable based on a post made on his social media, which we have access to from completing level 3. He said he takes a company keyword, capitalizes it, and appends a year & an exclamation mark. We have 5 keywords to work with here, being security, excellence, innovation, digital, and cloud. We can use crunch to generate a wordlist fitting the criteria mentioned. Due to how crunch works, I will generate a wordlist using one keyword at a time, starting with security. Using crunch 13 13 -t Security20%%! -o level5-1.txt, we get a list containing Security2000! through Security2099!, and if this list fails, we just make another one doing the same thing but with the next keyword. Through this, we find the password is [REDACTED].
<img width="1920" height="1140" alt="Screenshot 2026-06-18 144540 (Redacted)" src="https://github.com/user-attachments/assets/25aeb54d-757e-4038-a113-116260ecad8f" />
