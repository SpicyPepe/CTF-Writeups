![](Assets/Images/Pasted%20image%2020221208105815.png)


```
You have been assigned to a client that wants a penetration test conducted on an environment due to be released to production in seven days. 

**Scope of Work**

The client requests that an engineer conducts an assessment of the provided virtual environment. The client has asked that minimal information be provided about the assessment, wanting the engagement conducted from the eyes of a malicious actor (black box penetration test).  The client has asked that you secure two flags (no location provided) as proof of exploitation:

-   User.txt
-   Root.txt  
    

Additionally, the client has provided the following scope allowances:

-   Any tools or techniques are permitted in this engagement, however we ask that you attempt manual exploitation first  
    
-   Locate and note all vulnerabilities found
-   Submit the flags discovered to the dashboard
-   Only the IP address assigned to your machine is in scope
-   Find and report ALL vulnerabilities (yes, there is more than one path to root)

(Roleplay off)

I encourage you to approach this challenge as an actual penetration test. Consider writing a report, to include an executive summary, vulnerability and exploitation assessment, and remediation suggestions, as this will benefit you in preparation for the eLearnSecurity Certified Professional Penetration Tester or career as a penetration tester in the field.

Note - Nothing in this room requires Metasploit
```

## Question 1: User Flag? 
##### Answer format: `***{******************************}`

## Question 2: Root Flag?
##### Answer format: `***{******************************}`


# Intro

This report is a 'mock report' of the room named 'Relevant' on TryHackMe.com.
Due to the fact that this is a learning platform, this report contains uncensored information of a non-production system for the purpose of simulating and showcasing a real pen-testing report.
As this is my first ever report, the format and *some* of the wording which I have used has been inspired by a few examples I've found online and adapted to my own liking.

[Reference 1](https://pentestreports.com/reports/)

[Reference 2](https://library.mosse-institute.com/articles/2022/02/example-of-a-penetration-testing-report-executive-summary/example-of-a-penetration-testing-report-executive-summary.html#:~:text=A%20penetration%20test%20report%20executive,not%20to%20pursue%20further%20action.)


--------------------------------------------------------------------------

# Penetration Test Report





##### Client: *MyClientPtyLtd*


##### Date of test: *09/12/2022*


## Contents

1. [Some Definitions](#Some-Definitions)
2. [Information Gathering](#Information-Gathering)
3. [Vulnerability Scan](#Vulnerability-Scan)
4. [Technical Write-up](#Technical-Write-up)
5. [Conclusions and recommendations](#Conclusions-and-recommendations)


## Chapter 1 

# Introduction to the penetration test

The aim of this penetration test is to assist the company with securing the network.
This report has been written in a way which a non-initiated reader with basic knowledge of computing can understand, it does however contain some technical terms.
References to more technical content are given along with this test report and can be found in the appendices for the administrator and security consultant of *MyClientPtyLtd*  to review them and possibly reproduce the test.
Should the reader encounter any difficulties with understanding this penetration test report, please refer to the "Conclusions and Recommendations" section for an executive summary. For further assistance understanding this report, I remain open to answer any questions you may have. In order to increase the understanding of the reader, I have included some definitions and clarifications in the following sections. 


## Some Definitions

• **Hacker**: `a person who uses computers to gain unauthorized access to data.`
This is word given by the mass media to define what we will more accurately call attacker or intruder in this report. 

• **Vulnerability**: `the quality or state of being exposed to the possibility of being attacked`
This is also known as a bug in computer program that may be abused to gain privileges on a computer. 

• **Exploit**: `a software tool designed to take advantage of a flaw in a computer system, typically for malicious purposes such as installing malware.`
An exploit can be a program or strategy to exploit a vulnerability. Depending on the vulnerability, an exploit may be either local, in which a previous “local” access to the target computer is required prior gain higher privileges, or remote where the exploit can be run without this prerequisite. 

• **Rootkit**: `a set of software tools that enable an unauthorized user to gain control of a computer system without being detected`
This is a set of programs replacing the tools, that an administrator would generally use to detect the presence of an intruder, by modified versions detecting everything but the presence and activities of the intruder, thus making the administrator confident that the system is free of any intrusions.



## Motivation of an attacker

There are many reasons why someone might want to penetrate your network and some of the more popular reasons have been outlined below:

• **Information theft**: to steal valuable information of your business such as contracts, documents or e-mail communication. In other words, information that, for example, competitors may like to know or to hold the information/systems for ransom.

• **Identity theft**: by using your network as relay to attack other networks, an attacker can mask his identity. 

• **Political Espionage**: the act of obtaining, delivering, transmitting, communicating, or receiving information about a target with the intent to cause damage to reputation and gain a political advantage.

• **Challenge to overcome**: to most attackers, your network represents a challenge that must be conquered or a way to prove their superior intelligence and technical skills.

It's important to understand the psychology of an attacker. Whatever the attackers final motivation may be, it always remains a challenge to carry out a successful attack and often brings a high level of frustration along with it. 
An attacker, usually, doesn’t give up easily and will try, again and again, by any means, to get all kinds of information that might be useful to detect weaknesses and mount attacks. Therefore, while performing the penetration test, its best to go through the same stages as an attacker would have, even though the strategy or tools may slightly differ.


## Chapter 2

# The Security Audit

The penetration test was carried out from one machine utilizing a virtual machine running a Kali Linux distribution. 



## Information Gathering

The first part of this test was targeted at the webserver with the address specified in the engagement agreement `http://10.10.132.76/` 
Prior to any penetration attempts, the very first thing that an attacker needs to do is gather as much information about a target as possible. 

![](Assets/Images/Pasted%20image%2020221208135415.png)

As this is not a real engagement scenario, there are no ways to utilize external services to perform passive recon such as making use of google filters to enumerate the site like an attacker could on an external facing website accessible to the public. 


### Port Scanning

The second process for me was to further enumerate the machine using `Nmap` which is a network scanning tool to determine open ports and services. Using the switch `-sV` & `-sC` nmap probes the list of open ports to establish the service running and what version it is.

```
PORT      STATE    SERVICE       VERSION
80/tcp    open     http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open     ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2022-12-12T04:02:28+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2022-12-11T03:35:05
|_Not valid after:  2023-06-12T03:35:05
|_ssl-date: 2022-12-12T04:03:07+00:00; +2s from scanner time.
49663/tcp open     http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
49666/tcp filtered unknown
49668/tcp filtered unknown
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h36m02s, deviation: 3h34m40s, median: 1s
| smb2-time: 
|   date: 2022-12-12T04:02:28
|_  start_date: 2022-12-12T03:35:10
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-12-11T20:02:29-08:00

```

As we can see, there were a few ports open on this machine and the first thing which caught my attention was the outdated Microsoft Servers running on TCP ports `445` & `49663`.

I was able to enumerate the machine for SMB shares and successfully accessed the `nt4wrksv` share as an anonymous user with no password required.
Within this share, there were login credentials stored in encoded text however after testing these credentials it appears as though they're invalid.

Upon further enumeration of the machine, I was able to discover additional open ports running on TCP ports `49663`, `49666` & `49668`.

### Site Discovery

What I've done in this scenario is skip straight to active recon by manually looking around the website running on port 80 where in this case there was not much to look at other than the  default IIS page.
This was followed by running a `gobuster` scan which is a tool which uses a specified word list containing a catalogue of common directories to enumerate the website for accessible and/or hidden directories however nothing turned up.

The exact same process was followed up on the webserver running on TCP port `49663`. The same default IIS page was displayed and this time the `Gobuster` scan picked up an additional accessible directory `/nt4wrksv` 
I found that this directory was hosing the same `password.txt` file as the one found on the SMB share mentioned earlier.

![](Assets/Images/Pasted%20image%2020221212125010.png)


## Vulnerability Scan

Before diving any deeper, I ran the discovered services through a database of known vulnerabilities which unfortunately validated my concerns over the outdated Windows Server.

### EternalBlue
*Port* - 445 
*cvss* base - 9.3 
*cvss* base risk factor - HIGH 
*cve* - CVE-2017-0144

#### Summary
The Microsoft Windows Server 2008 R2 - 2012 allows remote attackers to execute arbitrary code via crafted packets, aka "Windows SMB Remote Code Execution Vulnerability." There is a total compromise of system integrity and a complete loss of system protection, resulting in the entire system being compromised.

An attacker may use this exploit to gain total information disclosure, resulting in all system files being revealed. The attacker may also completely shut down the resource rendering the service completely unavailable. 
Some preconditions must be satisfied in order to successfully exploit this.


##### Solution: MS17-010: Security update for Windows SMB Server: March 14, 2017
##### References: 
* https://support.microsoft.com/en-us/topic/ms17-010-security-update-for-windows-smb-server-march-14-2017-435c22fb-5f9b-f0b3-3c4b-b605f4e6a655
* https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/
* https://www.exploit-db.com/exploits/42031
* https://learn.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010



## Chapter 3

# Technical Write-up

In this section I will explain the process I've undertaken to successfully exploit the target system and extract the User.txt & Root.txt flag.

Firstly after failing to login using the credentials in the password.txt found earlier, I was able to test for write permission on the same SMB share `nt4wrksv`. This was done by placing some plain text into a document for POC, followed up by uploading the file to the share logged in as a guest user. 

![](Assets/Images/Pasted%20image%2020221212130237.png)

Checking back on the web directory linked to the SMB share, I could see the text file successfully saved to the share thus confirming write access.

![](Assets/Images/Pasted%20image%2020221212130313.png)

## Exploitation

Having determined that I have read and write permissions to the web directory linked through the SMB share, I then crafted a reverse shell payload to connect to the machine. This was done using `msfvenom` with the `aspx` file type as required with IIS servers. Seeing that the machine is running Server 2016, x64 was the correct architecture to use in the payload. 
This payload was then uploaded to the SMB share and executed using the same process as shown in the POC above. 
The incoming shell was captured by a netcat listener as seen below:

![](Assets/Images/Pasted%20image%2020221212132209.png)

As seen in the screenshot above I was successfully able to gain a foothold on the host machine as user `defaultapppool`. The User.txt flag was located in `c:\Users\Bob\Desktop\user.txt`

**User.txt:**<details> <summary>Show Answer</summary> `THM{fdk4ka34vk346ksxfr21tg789ktf45}` </details>

## Privilege Escalation

Since gaining a foothold on the machine, the next step was to look for escalation vectors manually.
The first command I ran was `whoami /priv` to check which privileges could be potentially exploited for a root shell.

![](Assets/Images/Pasted%20image%2020221212142143.png)

The SeImpersonatePrivilege is enabled for the `iis apppool\defaultapppool` user.
A PrintSpoofer exploit exists that can be used to escalate service user permissions on Windows Server 2016, Server 2019, and Windows 10. 
The exploit can be found at: https://github.com/dievus/printspoofer

In this case, I uploaded the executable to the server using the same method as earlier in the smbclient and executed the program with the following command `PrintSpoofer.exe -i -c cmd`


```
c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
PrintSpoofer.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

From here I was able to extract the contents of `root.txt` in the following location `C:\Users\Administrator\Desktop`

**root.txt**
<details> <summary>Show Answer</summary> `THM{1fk5kf469devly1gl320zafgl345pv}` </details>

## Chapter 4

# Conclusions and recommendations

## Key Findings:

I have discovered various flaws that may be exploited by a malicious actor. The most important findings were:

-   A vulnerability in the web server that could allow an attacker to gain access to sensitive data.
    
-   A vulnerability in the Windows Version which allows an attacker to elevate privileges to an administrator level with the ability to have full control over the system.
    

These flaws could lead to serious breaches of confidentiality and integrity if they are exploited. Unauthorized access to extract sensitive information can be a gained by adversaries.

The consequences of a major cyber breach caused by these flaws could include lawsuits and financial losses. In this scenario I was able to obtain both the `User.txt` & `Root.txt` flags as requested in the original agreement without prior knowledge of the host system. The effects of a real-world attack could put your systems and information in great danger.

## Recommendations:

I recommend that MyClientPtyLtd:

-   Performs automated vulnerability scans of all its Internet-facing servers and patches all critical vulnerabilities including the write permissions provided to unauthenticated clients.
    
-   Perform a review of your server’s security settings and harden the server in accordance with industry best practices.
    

Within 15 days, these activities listed above should be completed, and any high or critical risk findings should be resolved within 45 days.


--------------------------------------------------------------------------


# Final Thoughts

This room was especially difficult as it was designed to lead you down a rabbit hole. The creator purposefully created a 'honey pot' to teach an important lesson behind the toxic 'Try Harder' mindset by leaving a blatantly obvious finding which ultimately lead us down the wrong track. The idea of this room was to teach us that the most seemingly obvious finding we see cannot always be exploited, and that we have to know when to quit and try something else.
It was really a test of a users ability to fully enumerate a machine before exploiting.
I admittedly spent half a day going down this rabbit hole growing increasingly frustrated and confused as to why all my attempts were failing before having a peek at the official writeup. 

After all the built up frustration, I was relieved to see that it was always the creators intention to have users eventually bring up his official writeup in order to learn the true lesson behind this room. 

The official writeup can be found [here](https://medium.themayor.tech/relevant-walk-through-on-tryhackme-f7dedfcb00dc).

Thank you for checking out my first ever penetration test report. I hope to improve on both my technical skills and my ability to writeup a thorough and easy to understand reports as I move forward along my learning journey. 
