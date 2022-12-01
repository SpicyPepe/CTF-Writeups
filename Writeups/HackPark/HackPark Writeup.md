#Writeups #Hydra 


**TASK 1**

Deploy the machine and access its web server.

  
Q: Whats the name of the clown displayed on the homepage?

Navigated to the IP address: 10.10.44.152
![../Assets/Images/[Pasted image 20221201113634.png]]
A: `Pennywise`

**Recon**

Started Nmap scan "nmap -v -Pn -oN nmap1 10.10.44.152" to find open ports.
Ran follow up scan using service detection scripts "nmap -sV -v -Pn -p 80,3389 -oN nmap2 10.10.44.152"

Found two open ports:
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
3389/tcp open  ssl/ms-wbt-server?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

![[Pasted image 20221201113122.png]]

Ran Gobuster to locate additional directories stored on the web server using the following command: gobuster -u http://10.10.44.152/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -o Gobuster_results

![[Pasted image 20221201113430.png]]

/admin looks interesting.

Navigating to this page we can see a login form.

![[Pasted image 20221201113955.png]]

Googled "blog engine default login" to see if there are any easy default login's we could use. 

![[Pasted image 20221201114053.png]]

Attempted admin:admin however this failed.

**TASK 2**

Q: What request type is the Windows website login form using?

Intercepted login with Burpsuite

![[Pasted image 20221201114321.png]]
![[Pasted image 20221201115259.png]]

A: POST

"Now we know the request type and have a URL for the login form, we can get started brute-forcing an account. Run the following command but fill in the blanks:""

hydra -l (username) -P /usr/share/wordlists/(wordlist) (ip) http-post-form

Q: Guess a username, choose a password wordlist and gain credentials to a user account!

Pulled URL and cookie from Burp "/Account/login.aspx:"

Added ' :Login Failed ' to ensure Hydra moves on from each failed attempt.
Added -vv to turn on verboise mode.

In the cookie, I've set the "Name" field to ^USER^ and "Password" field to ^PASS^ 

hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.44.152 http-post-form "/Account/login.aspx:__VIEWSTATE=54As4GFCqjGQGRbh9x7p0JNYvP%2B%2FWYn8j0cgScQS%2FAETVoSDvDYb2jlruIqzrWu%2BFKXaRDQHiCzou6Bwih4wO%2Flpe%2B7gsWB%2FMi5Y8eoy4DpuwR2bHgGVFlsHNNvGI8Dplh3kdQhPd%2FvhfA3jGzbrRpq71XPpAN52VHLRfmGNGrXgcS0f&__EVENTVALIDATION=pTDpOJM1eosWaKomMwNkAu0p7SKMFK6s9h9gOoJgd5JW94cM%2FvcianYE9uu7SmmJHRfy2akgVkIJNAB0Zdu%2BHwcRsIZWkOdIlyKPF7kZacVMKvMwlNVtHDKB3ZaejVQHJM12Aa3W1RLRe7mtING9aEYCmdSJu8uYEV1PUbBGByebLRt1&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed" -vv

![[Pasted image 20221201122947.png]]

A: `1qaz2wsx`

**TASK 3**

Q:  Now you have logged into the website, are you able to identify the version of the BlogEngine?

Inspecting the source code we can easily find the version of BlogEngine being run:
![[Pasted image 20221201123720.png]]

A: `3.3.6.0`

  
Use the [exploit database archive](http://www.exploit-db.com/) to find an exploit to gain a reverse shell on this system.

What is the CVE?

![[Pasted image 20221201124723.png]]

A: `CVE-2019-6714`

  
Using the public exploit, gain initial access to the server.

Q: Who is the webserver running as?

As per the details listed on CVE-2019-6714

![[Pasted image 20221201124953.png]]

Started netcat listener on port 4445

![[Pasted image 20221201125709.png]]

Navigated to: http://10.10.44.152/admin/app/editor/editpost.cshtml
Uploaded the payload using the file manager button on the tool bar.

![[Pasted image 20221201130317.png]]

Navigate to: http://10.10.44.152/?theme=../../App_Data/files which activated the connection to the netcat listener

![[Pasted image 20221201130611.png]]

A: `iis apppool\blog`

Our netcat session is a little unstable, so lets generate another reverse shell using msfvenom.

![[Pasted image 20221201133004.png]]

Started up a web server so I can pull the payload over to the taget machine and execute the file manually.

![[Pasted image 20221201133124.png]]

Copied over using Powershell

![[Pasted image 20221201134429.png]]

Startup msfconsole & setup a multi handler 

![[Pasted image 20221201134910.png]]

PAYLOAD => windows/meterpreter/reverse_tcp

Ran 'Exploit -j' to run the job in the background. Moved over to the netcat shell and executed the payload.

![[Pasted image 20221201135351.png]]

  
You can run metasploit commands such as sysinfo to get detailed information about the Windows system. Then feed this information into the [windows-exploit-suggester](https://github.com/GDSSecurity/Windows-Exploit-Suggester) script and quickly identify any obvious vulnerabilities.

Q: What is the OS version of this windows machine?

We can see by running the meterpreter command `sysinfo` we get certain information.

![[Pasted image 20221201140046.png]]

A: `Windows 2012 R2 (6.3 Build 9600)`

Q: Further enumerate the machine.
What is the name of the abnormal _service_ running?

running `ps` to see what processes are running.
![[Pasted image 20221201140118.png]]

Enumerated the machine further by running WinPEAS. Copied over WinPEAS using the same method as earlier.

![[Pasted image 20221201145831.png]]

A: `WindowsScheduler`

Q: What is the name of the binary you're supposed to exploit?

Checking the events directory for the WindowsScheduler service for what what ever is being run.

![[Pasted image 20221201150823.png]]

Checking the INI_LOG.txt file there looks to be an (administrator) process which is repeatedly starting and stopping. This will be the binary to exploit here.

![[Pasted image 20221201150900.png]]

A: `Message.exe`

  
Using this abnormal service, escalate your privileges!

Q: What is the user flag (on Jeffs Desktop)?

Moving back to the WindowsScheduler directory (where the Message.exe file is located), copied accross the same shell used earlier and re-named it "Message.exe" while also changing the original Message.exe file to "Message.bak".

![[Pasted image 20221201152939.png]]

Returned to the msfconsole and ran the multi handler again to catch the incoming connection which will be triggered by the service we're exploiting. 

![[Pasted image 20221201153709.png]]

![[Pasted image 20221201153728.png]]

Located the flag on Jeff's Desktop and viewed its contents to find the next answer.

![[Pasted image 20221201153912.png]]

A: `759bd8af507517bcfaede78a21a73e39`

Q: What is the root flag?

Repeated the above process except in the Administrator directory.

![[Pasted image 20221201154107.png]]

A: `7e13d97f05f7ceb9881a3eb3d78d3e72`

**TASK 5**

Q: Using winPeas, what was the Original Install time? (This is date and time)

Already ran winPeas from earlier

![[Pasted image 20221201155216.png]]

A: `8/3/2019, 10:43:23 AM`
