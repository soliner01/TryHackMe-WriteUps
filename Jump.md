## Jump

Our goal is to get from anonymous access all the way to root access by exploiting misconfigurations on different users along the way.

We can start with an nmap scan on the target.
```
nmap -sC -sV -p- 10.67.168.102
```
<img width="1920" height="1140" alt="Screenshot 2026-06-25 122715" src="https://github.com/user-attachments/assets/8b89fd1c-ff94-4484-889a-458443d51e3d" />

We can see that there are 2 ports open, one is ftp, which we can use to gain anonymous access. Looking in the pub directory, we find a file called README.txt.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 123235" src="https://github.com/user-attachments/assets/2067f9d9-c1f5-4cc3-927d-66d9831a6f24" />

Grabbing that file, we see that recon jobs must be placed in the incoming/ directory, and files are processed automatically upon arrival.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 123248" src="https://github.com/user-attachments/assets/54959572-9913-4cf5-b175-0a47d9005d86" />

Create a shell file called shell.sh, and include a script that starts a reverse shell.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 123718" src="https://github.com/user-attachments/assets/a10e6e28-4097-4fe0-851a-c1ee6774f18f" />

Start a listener to the port mentioned in the code, and upload the shell script to the FTP server. A shell session will open up, and you can get the flag from recon_user's desktop.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 124127 (Redacted)" src="https://github.com/user-attachments/assets/828c7c52-fa05-4482-b564-8688de7ef218" />

Next, we need to get the flag found in the dev_user's home directory. Surprisingly enough, merely going back to the /home directory and then going into the dev_user directory works. It seems permissions weren't set to prevent that directory switching for that account specifically.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 124532 (Redacted)" src="https://github.com/user-attachments/assets/10decd3f-e483-4d5e-aad2-1291a5586f4f" />

From here, just print the flag from the dev_user's desktop.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 124644 (Redacted)" src="https://github.com/user-attachments/assets/4ac3f3b0-2406-4849-b044-c18fb80d9161" />

Looking at /opt/dev/backup.sh, we see the file does a bash backup, which looks like a good attack vector. We can change the contents of the file to set up a reverse shell with the Attackbox VM.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 131429" src="https://github.com/user-attachments/assets/e024f808-9f12-4c98-b51d-33099696a5a4" />

It seems that this session granted us access as dev_user, so we can't get the next flag yet, but our permissions have just increased. Whilst poking around earlier, I stumbled upon another file in the /opt/dev directory of interest. Looking at /opt/dev/bin/ps, we see the file is set up to establish a connection with a different IP address (the image below shows my Attackbox IP address instead, as I already changed the script). By changing the script so it connects to our Attackbox IP address instead, we can establish another session.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 132230" src="https://github.com/user-attachments/assets/4ff80b35-c13b-48c7-a4c4-d15d8f5f25f5" />

With a listener running, we gain access to monitor_user, meaning we can also get the flag from that desktop.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 132505 (Redacted)" src="https://github.com/user-attachments/assets/f36abd72-643a-4902-ab14-ae7b9cae36d9" />

As monitor_user, we have the ability to run sudo commands. Running sudo -l tells us we can run /usr/local/bin/deploy.sh without needing to input a password.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 132756 (Redacted)" src="https://github.com/user-attachments/assets/ed91a65c-db3d-4068-a618-11dd00f80044" />

Looking at the contents of this deploy.sh script, we see it is another bash script, and it runs a script called deploy_helper.sh in the /opt/app directory, which is yet another bash script. We can replace the contents of that script with a script that will start another shell session for us.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 133420" src="https://github.com/user-attachments/assets/dce114bf-ab04-4638-8deb-3e760484d4be" />

Run that script with a listener running, and you will gain access as ops_user.
```
sudo -u ops_user /usr/local/bin/deploy.sh
```

From here, we can go to the desktop of ops_user and get the flag.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 134601 (Redacted)" src="https://github.com/user-attachments/assets/730313f4-7c33-4f06-9a18-b9049b45027b" />

Running sudo -l as ops_user, we see that this user can run /usr/bin/less without needing to input a password.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 134737 (Redacted)" src="https://github.com/user-attachments/assets/5f26696b-4220-4cfc-9485-57bbb41a812e" />

Upon running less /usr/bin/less, we are prompted with a warning message that /usr/bin/less may be a binary file, and asks if we want to see it anyway. Upon entering yes, we see the contents of said binary file. This means less reads the given file. Since we can run this with sudo, we can just use this to have it read for us the contents of flag.txt on the root_user desktop.
<img width="1920" height="1140" alt="Screenshot 2026-06-25 144247 (Redacted)" src="https://github.com/user-attachments/assets/fcef02ee-a564-4c37-8825-91399273307e" />
<sub>(Note that the IP addresses are different in this picture than the previous ones, as my TryHackMe VMs expired)</sub>
