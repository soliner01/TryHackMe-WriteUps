## Windows Jump

Our goal is to get access to various accounts, escalating our privileges all the way to SYSTEM, and collecting flags along the way.

We can start with an nmap scan on the target.
```
nmap -sC -sV -p- -Pn 10.67.148.146 --min-rate 1000
```
<img width="1920" height="1140" alt="Screenshot 2026-06-26 130304" src="https://github.com/user-attachments/assets/021fb496-1efc-40da-ac04-ac39352f5134" />

We found a potential opening with smb services. Trying smbclient //10.66.148.146/public gives us access to the public smb share. In that directory, we find a file called welcome.txt. Grabbing that file gives us default credentials.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 130517" src="https://github.com/user-attachments/assets/0465d0f3-259e-46ec-99c5-16be31f7dd13" />

The Attackbox has access to xfreerdp, we can use that to log into the target VM using the credentials we obtained.
```
xfreerdp /v:10.66.148.146 /u:thmuser /p:Password1!
```

We find a flag on the user's desktop.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 130843 (Redacted)" src="https://github.com/user-attachments/assets/bdefa2a9-b770-4f06-a161-724388808851" />

By querying the Windows registration, we end up finding another set of credentials that we can use.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 132214" src="https://github.com/user-attachments/assets/f0bd4626-71d8-40c0-bd33-3450764f393e" />

Using those credentials to login to the notadmin account, we can gain access to the second flag, located on that user's desktop.
```
runas /user:notadmin cmd.exe
```
<img width="1920" height="1140" alt="Screenshot 2026-06-26 132825 (Redacted)" src="https://github.com/user-attachments/assets/878181e6-6265-452f-979e-39a644b77d36" />

Using sc query state= all, we can enumerate the services running on the system. One of the services we find is THMSvc. Looking closer at it, we find it is running with the svcadmin user.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 133621" src="https://github.com/user-attachments/assets/cd4f2ed9-c9ed-4499-92a7-060087526aa0" />

Checking the permissions, it appears that we have full access over the binary as notadmin, so this is the perfect attack vector to use.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 133840" src="https://github.com/user-attachments/assets/898703cd-9df9-4280-943d-69fc661a154b" />

We can generate a payload with msfvenom crafted specifically for this service.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 142538" src="https://github.com/user-attachments/assets/3794d061-1ad6-4f11-9536-62f7b0642964" />

Transfer the file over, and move it to the correct location, as seen in BINARY_PATH_NAME. Then, grant full access to the executable for everyone.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 142630" src="https://github.com/user-attachments/assets/8dfaa1bb-699d-43f6-8620-5fff130fe7fe" />

Restart the service with a listener running on the Attackbox VM, and you will gain access to svcadmin. The third flag is located on the desktop for the svcadmin user.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 143029 (Redacted)" src="https://github.com/user-attachments/assets/1bdc2e1d-a20a-43a2-adb0-c712ddf38798" />

Looking at C:\Windows\Tasks, we find a file called cleanup.bat, which we are allowed to modify as svcadmin.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 150713" src="https://github.com/user-attachments/assets/36c67622-e30e-4d89-9e67-67beba121294" />

One thing we can do is make cleanup.bat run a custom executable, and start a shell. Upload that shell executable to C:\Windows\Tasks, and change the contents of cleanup.bat to run that file.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 151601" src="https://github.com/user-attachments/assets/d0ed82b6-c384-400a-affa-f8a3dbf2aa80" />

Start a listener, and you will gain SYSTEM access. From there, you can grab the fourth and final flag from the C:\ directory.
<img width="1920" height="1140" alt="Screenshot 2026-06-26 151941 (Redacted)" src="https://github.com/user-attachments/assets/38e537b5-f7d5-416b-8311-9945627a411a" />
