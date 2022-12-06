#Linux #SQL #PHP


![](THM/Writeups/Daily%20Bugle/Assets/Images/Daily_Bugle.png)

![](THM/Writeups/Daily%20Bugle/Assets/Images/fREnB0x.png)

Hack into the machine and obtain the root user's credentials.

## Reconnaissance 

Pinging the server we can see that its responding to ICMP pings.
Started up Nmap scan, Gobuster scan in the background while checking out the webpage & source code for hints.

![](Assets/Images/Pasted%20image%2020221206103244.png)

Looking at the home page, there is a picture of spiderman robbing a bank and an article written by a 'Super User'. The bottom of the page shows the current web directory and on the right hand side of the page there is a login form. Forgot username and forgot password button and a cog wheel option in the top right to have the article emailed or printed.
The source code hasn't got anything obvious we can use.

##### Nmap scan

![](Assets/Images/Pasted%20image%2020221206105225.png)

We've discovered 3 open ports including a mysql server on port 3306.
Apache webserver running on port 80 is using version 2.4.6 and PHP version 5.6.40.

##### Gobuster Scan

![](Assets/Images/Pasted%20image%2020221206105504.png)

In the /administrator directory, a login form can be found with the option to select forgot user name and password.

![](Assets/Images/Pasted%20image%2020221206111244.png)

When clicking on the forgot password link a new page is opened up to enter the associated email address:

![](Assets/Images/Pasted%20image%2020221206112921.png)

## Question 1:
#### Who robbed the bank?

##### Answer: `Spiderman`

## Question 2: 
#### What is the Joomla version?

As mentioned earlier there was no useful information found in the source code of the site. Usual tools like Wappalizer weren't able to pick up the version of Joomla the webserver was using. 

## Services

After a bit of research I came across an article on how to easily find the version of Joomla running for a server not owned by us.
Reference: 
https://www.itoctopus.com/how-to-quickly-know-the-version-of-any-joomla-website

We effectively need to navigate to `_/administrator/manifests/files/joomla.xml_` which contains an xml file with the current version, creation date and license info.

![](Assets/Images/Pasted%20image%2020221206115650.png)

##### Answer: `3.7.0`

## Question 3:
#### *Instead of using SQLMap, why not use a python script!*  
#### What is Jonah's cracked password?

We now know the version of Joomla being used so perhaps there is a vulnerability we can exploit to gain an initial foothold on the server.
Using Searchsploit we can see a vulnerabilities listed for Joomla 3.7.0:

![](Assets/Images/Pasted%20image%2020221206120055.png)

## Gaining Access
##### Joomla! 3.7.0 - 'com_fields' SQL Injection

This is the exploit for Joomla 3.7.0 (CVE-2017-8917)
The way to make use of this exploit as listed in on exploit-DB is to use sqlmap and manually enter in the required information:
`sqlmap -u "http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]`

Question 3 includes a statement to try using a python script instead of using sqlmap however there were no python scripts included on exploit-DB. 
Its pretty easy to find one simply searching for python scripts using google. I had a couple of results show up for github accounts sharing the relevant python script which can be used.
Example: https://github.com/XiphosResearch/exploits/tree/master/Joomblah

The Python script can be activated using the following command and filling in the IP address:
`python joomblah.py http://127.0.0.1:8080`

![](Assets/Images/Pasted%20image%2020221206124432.png)

The exploit paid off and provided us with valuable information such as the super user name, email and password hash!
`Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm`

We can now attempt to crack this users password using John the Ripper and a wordlist such as rockyou.txt

![](Assets/Images/Pasted%20image%2020221206130337.png)

After waiting 7 minutes for John to do his thing, we've managed to crack Jonah's password!

##### Answer: `spiderman123`

Using these new found credentials we're now able to login to the Joomla admin portal.

![](Assets/Images/Pasted%20image%2020221206131341.png)



## Question 4:
#### What is the user flag?

Looking through the admin portal there are lots of interesting areas to check out, some of which included file upload buttons which could potentially used to upload a payload however a lot of these didn't work for me.
After a bit I came across the 'templates' section where you can choose the design of a web page. 
Within this section, the site allows you to edit existing PHP files which we can use to create our payload!
![](Assets/Images/Pasted%20image%2020221206144615.png)

Once you assign the template to the home page and refresh the home page in a new tab, this activates our payload. From here we can catch our shell using a netcat listener.

![](Assets/Images/Pasted%20image%2020221206144912.png)

##### Netcat

![](Assets/Images/Pasted%20image%2020221206144942.png)

Under the users directory we can see that there is a user named `jjameson`
We don't have access to their home directory as we're logged in as `apache` so we'll need to elevate our privileges to a normal user on the system and/or find a way to gain a root shell.

## Lateral Movement

Now that we have a foothold, the /tmp directory is a writable directory for us so it can be used to download tools such as LinPeas on in order to enumerate the system for escalation vectors.

![](Assets/Images/Pasted%20image%2020221206150856.png)

After running Linpeas a 'public' password was discovered which I'm wondering if we can use to login to `jjameson`'s account with.

![](Assets/Images/Pasted%20image%2020221206163306.png)

Interestingly enough it is in fact their password and I was able to successfully switch users:

![](Assets/Images/Pasted%20image%2020221206163543.png)

Our first flag is located in `jjameson`'s home directory ``/home/jjameson/user.txt`

##### Answer: `27a260fe3cba712cfdedb1c86d80442e`

## Privilege Escalation

Now that we've got access to a normal user rather than the previous apache login, we can check to see which sudo privileges `jjameson` has using `sudo -l`

![](Assets/Images/Pasted%20image%2020221206164122.png)

Looks like we've got sudp privileges for `yum`.
According to gtfobins, if the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

We can spawn an interactive root shell by loading a custom plugin using the below commands:

```
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

Once you've created the above custom plugin and run `sudo tum-c $TF/x --enebleplugin=y` we now have out root shell and can capture our final flag!

##### Answer: eec3d53292b1821868266858d7fa6f79

Big thank you for reading through my writeup of the Daily Bugle room on TryHackMe.com. I really hope you've found the information here useful!