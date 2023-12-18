<h1>Penetration Testing </h1>

<h2>Description</h2>
Project consists of penetration testing; identified and exploited vulnerabilities using Kali Linux and Nmap. Conducted
password spraying and privilege escalation on Linux and Windows systems.
Established persistent network access and analyzed credentials. 
<br />


<h2> Utilities Used</h2>

- <b>nmap</b> 
- <b>Metasploit</b>

<h2>Environments Used </h2>

- <b>Kali Linux</b>

<h2>Program walk-through:</h2>
<p align="center">
Scan for Open Ports:  <br/>
Perform ip addr to identify IP address, then an Nmap scan on that subnet (/24) to identify all machines on the network <br/>
The machine with port '88' open will always be the Domain Controller due to how Kerberos operates. Port 445 and 135 are also required for active directory to work properly.  <br/> <br/>
  
<img src="https://imgur.com/a4nfEwm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<br/>


  Metasploit Commands: <br/>
  msfconsole which launches MSFconsole from the command line<br/>
  show + module type which displays all the available modules<br/>
  search + keyword which searches for modules based on keywords<br/>
  use + module name which selects a certain module<br/>
  info which displays the module details<br/>
  options which displays the options available for the module<br/>
  exploit or run which runs the module<br/>
  show payloads which will give you the payloads that are compatible with each exploit<br/>
  <br/>
  <p align="center">
  Password Spraying: <br/>
  <br/>
Password Spraying involves using a single password against a list of usernames. 
Load the auxiliary module.   
<img src="https://imgur.com/G7vpBY0.png" height="80%" width="80%" alt="Password Spray Steps"/>
<br />
<br />
Set SMBUser and SMBPass to a set of cracked credentials in /etc/shadow file:  <br/>
<img src="https://imgur.com/qsFdyHt.png" height="80%" width="80%" alt="Password Spray Steps"/>
<br />
<br />
Set RHOSTS to the subnet.  Passwords were removed from screenshot: <br/>
<img src="https://imgur.com/AurlyXm.png" height="80%" width="80%" alt="Password Spray Steps"/>
<br />
<br />
Scan for Open Ports:  <br/>
<img src="https://imgur.com/QDDtLf2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
<p align="center">
 Windows Privilege Escalation<br />

Using password spraying, you gained a foothold on a Windows machine in a previous activity. Now that we understand and recognize our privilege-escalation attack path, you are tasked to implement it with Metasploit. Specifically, you will escalate your privileges on the Windows machine from `tstark` to `SYSTEM` privileges, giving you full control of the entire machine. Work off of the tstark user's Meterpreter session.  With the active Meterpreter session, you will attempt to escalate your privileges by creating a service that will run a malicious payload.  Remember, when a service is run, it is done with SYSTEM privileges. <br />
<p align="center">
Background the Meterpreter session via the `background` command.
<br />
<br />
<img src="https://imgur.com/FHBT9j3.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<p align="center">
Use the `windows/local/persistence_service` module in Metasploit. <br />
<img src="https://imgur.com/kWEqEbd.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
<p align="center">
  View the OPTIONS and set the SESSION to your current Meterpreter session number ID. If you're unsure of the session number, type `sessions`.
<p align="center">
Once the parameters are set, run the module.

<img src="https://imgur.com/7UZRuxG.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
Once complete, view the user ID.<br />
<br />
<br />
Windows Persistence<br />
Now that we have SYSTEM access over the machine, we will establish persistence on it to ensure our SYSTEM access. This technique will utilize Task Scheduler. We will create a scheduled task that will execute a custom Meterpreter payload.

- You will establish persistence on the Window10 machine by abusing Task Scheduler. 

- This will allow your payload to be executed at a certain defined interval, ensuring you always have a reverse shell to the target.
<p align="center">
You will work off your Meterpreter session on the Window10 machine. If you do not have an active session on the WIN10 machine, refer to prior activities to obtain a Meterpreter shell.
<p align="center">
In your Meterpreter session, drop into a `shell` session.<br />
Create a scheduled task that will execute your payload every day at midnight.  <br />
 <img src="https://imgur.com/LvluVmr.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />

     `schtasks /create /f /tn Backdoor /SC ONCE /ST 00:00 /TR "C:\shell.exe"`

	
<p align="center">
Test your scheduled task. 

     - `schtasks /run /tn Backdoor`

<img src="https://imgur.com/tnNoXbr.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />

<p align="center">
To improve this technique to make it more stealthy:  Better task name, Execute a better-named payload,  Schedule the task only to run on certain events, such as logon
<br />
<br />
<p align="center">
 Credential Dumping

In this activity, use the Metasploit `kiwi` extension to dump the credentials that are cached on the WIN10 machine.<br />

This requires the SYSTEM Meterpreter session to complete. Refer to prior activities if you do not have the SYSTEM Meterpreter shell active.<br />
<p align="center">
 In your Meterpreter session, load the `kiwi` extension.<br />

<img src="https://imgur.com/hMQS3jq.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
<p align="center">
Once a new extension is loaded into Metasploit, it will update the help menu. View the `kiwi` command options by calling the help menu in Meterpreter.<br />
     - <img src="https://imgur.com/hO42niP.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
<p align="center">
Dump all of the cached credentials from LSASS using a `kiwi_cmd` command. Pay attention to the "lsadump" section.  <br />
If the `kiwi` command is not dumping credentials as expected, try migrating to another process using the `migrate` command. Keep in mind that you'll want to migrate to another SYSTEM x64 process. <br />
	 
<img src="https://imgur.com/tnNoXbr.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
<p align="center">
In the output, the hashes are displayed after the 'MsCacheV2 field'. MsCacheV2 is just the format of the hash. Save the hashes in the format `username:password`, as shown below.<br />

<img src="https://imgur.com/w36Oc9A.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
	<p align="center">
Using `john`, attempt to crack the password. Your `john` command should use the flag --format=mscash2, e.g. `john --format=mscash2 hashes.txt`
 <img src="https://imgur.com/51CFcUj.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />
<p align="center">

You should now have the plaintext password to the new account, 'bbanner'.  Password was removed from screenshot.<br />
<p align="center">
<img src="https://imgur.com/qOj44K0.png" height="80%" width="80%" alt="Privilege Execution Steps"/>
<br />
<br />


</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
