#Writeups #Hydra 

**TASK 1**

Deploy the machine and access its web server.
  
## Question 1: Whats the name of the clown displayed on the homepage?

Navigated to the IP address: 10.10.44.152
![](Assets/Images/Pasted%20image%2020221201113634.png)
##### Answer: <details> <summary>Show Answer</summary> `Pennywise` </details>


**Recon**

Started Nmap scan "nmap -v -Pn -oN nmap1 10.10.44.152" to find open ports.
Ran follow up scan using service detection scripts "nmap -sV -v -Pn -p 80,3389 -oN nmap2 10.10.44.152"

Found two open ports:
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
3389/tcp open  ssl/ms-wbt-server?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

![](Assets/Images/Pasted%20image%2020221201113122.png)

Ran Gobuster to locate additional directories stored on the web server using the following command: gobuster -u http://10.10.44.152/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -o Gobuster_results

![](Assets/Images/Pasted%20image%2020221201113430.png)

/admin looks interesting.

Navigating to this page we can see a login form.

![](Assets/Images/Pasted%20image%2020221201113955.png)

Googled "blog engine default login" to see if there are any easy default login's we could use. 

![](Assets/Images/Pasted%20image%2020221201114053.png)

Attempted admin:admin however this failed.

**TASK 2**

## Question 1: What request type is the Windows website login form using?

Intercepted login with Burpsuite
![](Assets/Images/Pasted%20image%2020221201114321.png)
![[Assets/Images/Pasted image 20221201115259.png]]

##### Answer: <details> <summary>Show Answer</summary> `POST` </details>

"Now we know the request type and have a URL for the login form, we can get started brute-forcing an account. Run the following command but fill in the blanks:""

hydra -l (username) -P /usr/share/wordlists/(wordlist) (ip) http-post-form

## Question 2: Guess a username, choose a password wordlist and gain credentials to a user account!

Pulled URL and cookie from Burp "/Account/login.aspx:"

Added ' :Login Failed ' to ensure Hydra moves on from each failed attempt.
Added -vv to turn on verboise mode.

In the cookie, I've set the "Name" field to ^USER^ and "Password" field to ^PASS^ 

hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.44.152 http-post-form "/Account/login.aspx:__VIEWSTATE=54As4GFCqjGQGRbh9x7p0JNYvP%2B%2FWYn8j0cgScQS%2FAETVoSDvDYb2jlruIqzrWu%2BFKXaRDQHiCzou6Bwih4wO%2Flpe%2B7gsWB%2FMi5Y8eoy4DpuwR2bHgGVFlsHNNvGI8Dplh3kdQhPd%2FvhfA3jGzbrRpq71XPpAN52VHLRfmGNGrXgcS0f&__EVENTVALIDATION=pTDpOJM1eosWaKomMwNkAu0p7SKMFK6s9h9gOoJgd5JW94cM%2FvcianYE9uu7SmmJHRfy2akgVkIJNAB0Zdu%2BHwcRsIZWkOdIlyKPF7kZacVMKvMwlNVtHDKB3ZaejVQHJM12Aa3W1RLRe7mtING9aEYCmdSJu8uYEV1PUbBGByebLRt1&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed" -vv

![](Assets/Images/Pasted%20image%2020221201122947.png)

##### Answer: <details> <summary>Show Answer</summary> `1qaz2wsx` </details>

**TASK 3**

## Question 1:  Now you have logged into the website, are you able to identify the version of the BlogEngine?

Inspecting the source code we can easily find the version of BlogEngine being run:
![](Assets/Images/Pasted%20image%2020221201123720.png)

##### Answer: <details> <summary>Show Answer</summary> `3.3.6.0` </details>

  
Use the [exploit database archive](http://www.exploit-db.com/) to find an exploit to gain a reverse shell on this system.

## Question 2: What is the CVE?

![](Assets/Images/Pasted%20image%2020221201124723.png)

##### Answer: <details> <summary>Show Answer</summary> `CVE-2019-6714` </details>

  
Using the public exploit, gain initial access to the server.

## Question 3: Who is the webserver running as?

As per the details listed on CVE-2019-6714

![](Assets/Images/Pasted%20image%2020221201124953.png)

Started netcat listener on port 4445

![](Assets/Images/Pasted%20image%2020221201125709.png)

Navigated to: http://10.10.44.152/admin/app/editor/editpost.cshtml
Uploaded the payload using the file manager button on the tool bar.

![](Assets/Images/Pasted%20image%2020221201130317.png)

Navigate to: http://10.10.44.152/?theme=../../App_Data/files which activated the connection to the netcat listener

![](Assets/Images/Pasted%20image%2020221201130611.png)

##### Answer: <details> <summary>Show Answer</summary> `iis apppool\blog` </details>

Our netcat session is a little unstable, so lets generate another reverse shell using msfvenom.

![](Assets/Images/Pasted%20image%2020221201133004.png)

Started up a web server so I can pull the payload over to the taget machine and execute the file manually.

![](Assets/Images/Pasted%20image%2020221201133124.png)

Copied over using Powershell

![](Assets/Images/Pasted%20image%2020221201134429.png)

Startup msfconsole & setup a multi handler 

![](Assets/Images/Pasted%20image%2020221201134910.png)

PAYLOAD => windows/meterpreter/reverse_tcp

Ran 'Exploit -j' to run the job in the background. Moved over to the netcat shell and executed the payload.

![](Assets/Images/Pasted%20image%2020221201135351.png)

  
You can run metasploit commands such as sysinfo to get detailed information about the Windows system. Then feed this information into the [windows-exploit-suggester](https://github.com/GDSSecurity/Windows-Exploit-Suggester) script and quickly identify any obvious vulnerabilities.

## Question 4: What is the OS version of this windows machine?

We can see by running the meterpreter command `sysinfo` we get certain information.

![](Assets/Images/Pasted%20image%2020221201140046.png)

##### Answer: <details> <summary>Show Answer</summary> `Windows 2012 R2 (6.3 Build 9600)` </details>

## Question 5: Further enumerate the machine.
What is the name of the abnormal _service_ running?

running `ps` to see what processes are running.

![](Assets/Images/Pasted%20image%2020221201140118.png)
Enumerated the machine further by running WinPEAS. Copied over WinPEAS using the same method as earlier.

![](Assets/Images/Pasted%20image%2020221201145831.png)

##### Answer: <details> <summary>Show Answer</summary> `WindowsScheduler` </details>

## Question 6: What is the name of the binary you're supposed to exploit?

Checking the events directory for the WindowsScheduler service for what what ever is being run.

![](Assets/Images/Pasted%20image%2020221201150823.png)

Checking the INI_LOG.txt file there looks to be an (administrator) process which is repeatedly starting and stopping. This will be the binary to exploit here.

![](Assets/Images/Pasted%20image%2020221201150900.png)

##### Answer: <details> <summary>Show Answer</summary> `Message.exe` </details>

  
Using this abnormal service, escalate your privileges!

## Question 7: What is the user flag (on Jeffs Desktop)?

Moving back to the WindowsScheduler directory (where the Message.exe file is located), copied accross the same shell used earlier and re-named it "Message.exe" while also changing the original Message.exe file to "Message.bak".

![](Assets/Images/Pasted%20image%2020221201152939.png)

Returned to the msfconsole and ran the multi handler again to catch the incoming connection which will be triggered by the service we're exploiting. 

![](Assets/Images/Pasted%20image%2020221201153709.png)

![](Assets/Images/Pasted%20image%2020221201153728.png)

Located the flag on Jeff's Desktop and viewed its contents to find the next answer.

![](Assets/Images/Pasted%20image%2020221201153912.png)

##### Answer: <details> <summary>Show Answer</summary> `759bd8af507517bcfaede78a21a73e39` </details>

## Question 8: What is the root flag?

Repeated the above process except in the Administrator directory.

![](Assets/Images/Pasted%20image%2020221201154107.png)

##### Answer: <details> <summary>Show Answer</summary> `7e13d97f05f7ceb9881a3eb3d78d3e72` </details>

**TASK 5**

## Question 1: Using winPeas, what was the Original Install time? (This is date and time)

Already ran winPeas from earlier

![](Assets/Images/Pasted%20image%2020221201155216.png)

##### Answer: <details> <summary>Show Answer</summary> `8/3/2019, 10:43:23 AM` </details>

## Conclusion

Thank you for reading through my write up. This is the first of many writeups I will be doing for TryHackMe rooms and I hope you found the information here helpful!
