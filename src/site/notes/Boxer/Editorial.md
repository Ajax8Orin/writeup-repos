---
{"dg-publish":true,"permalink":"/boxer/editorial/","tags":["gardenEntry"]}
---

## Writeup
1. Initial Recon

``` 
sudo nmap -sT -vvv -n -Pn -p- -oG allPorts 10.10.10.3
```

![Pasted image 20240629093014.png](/img/user/Sources/Pasted%20image%2020240629093014.png)

2. Enumeration
```
sudo nmap -sT -A -Pn -sV 10.10.10.3 
```

![Pasted image 20240629093048.png](/img/user/Sources/Pasted%20image%2020240629093048.png)

- Since the host was initially unreachable we include in local hosts list to try and access it.
```
sudo nano /etc/hosts
```

![Pasted image 20240629093306.png](/img/user/Sources/Pasted%20image%2020240629093306.png)

- As we have enumerated two services of ssh and http lets try and access the web app and find ways to exploit it

3. Gaining Access and Exploitation

- As we navigate through the web application we find a page with form and upload fields
![Pasted image 20240629093543.png](/img/user/Sources/Pasted%20image%2020240629093543.png)
- We now check the http history to find some useful directories. We explore the outputs for some of these entries
![Pasted image 20240629093719.png](/img/user/Sources/Pasted%20image%2020240629093719.png)
- It seems like a classic SSRF attack can be probed so let's try and alter the request to test its response
![Pasted image 20240629093923.png](/img/user/Sources/Pasted%20image%2020240629093923.png)
- We try enumerating internal ports responses using localhost to validate if we get an unique response by setting up the payload as > number --> 1 to 65535 --> Step 1
![Pasted image 20240629094118.png](/img/user/Sources/Pasted%20image%2020240629094118.png)
- We execute the attack and filter the results wherein we don't have a response bearing .jpeg endpoint output, we discover at port 5000 we have a different response
![Pasted image 20240629094318.png](/img/user/Sources/Pasted%20image%2020240629094318.png)
- Upon testing the endpoint in browser apparently a file is downloaded named: 72c3ef9e-d3fa-4c51-a22b-f92337c3da95.txt which showcased the following results:
![Pasted image 20240629094625.png](/img/user/Sources/Pasted%20image%2020240629094625.png)
- We manually test all the endpoints provided in this list and on a particular endpoint of  "/api/latest/metadata/messages/authors" we got another download file end point
![Pasted image 20240629094913.png](/img/user/Sources/Pasted%20image%2020240629094913.png)
- The test of this endpoint lead us to a response credentials through which as we remember SSH port was also opened and through which we were able to gain access!
![Pasted image 20240629095116.png](/img/user/Sources/Pasted%20image%2020240629095116.png)

4. Maintaining access and Extracting outputs

- Now we have gained access so let's extract the details using the shell and as we go through it we have our user flag

```
-bash-5.1$ whoami
dev
-bash-5.1$ ls
apps  user.txt
-bash-5.1$ 
-bash-5.1$ cat user.txt 

```

```
Flag user: b7b8bf8451abcad2c285c807fcc644d6
```

- Now we need to attempt a possible privilege escalation in order to obtain root flag. Let's explore this shell further, maybe there are hidden directories so we can try listing using ls -la command
![Pasted image 20240629095738.png](/img/user/Sources/Pasted%20image%2020240629095738.png)

- We found a git directory it is time to go git busting let's see if we can extract some details. We found an interesting branch while going through the logs

![Pasted image 20240629095954.png](/img/user/Sources/Pasted%20image%2020240629095954.png)

- Now let's switch to this branch to see if there is something interesting in it and we have obtained production credentials
![Pasted image 20240629100236.png](/img/user/Sources/Pasted%20image%2020240629100236.png)

- Let's try gaining access to this shell using SSH and we successfully do so as shown below
![Pasted image 20240629100331.png](/img/user/Sources/Pasted%20image%2020240629100331.png)

- We tried exploring it and we obtained info that a python3 can be executed using root for this user
![Pasted image 20240629100506.png](/img/user/Sources/Pasted%20image%2020240629100506.png)

- Since python is something we can use so we try listing installed libraries using pip and explore whether there is a suspicious or vulnerable library
- We find GitPython library version is vulnerable 
![Pasted image 20240629100713.png](/img/user/Sources/Pasted%20image%2020240629100713.png)

- So we go ahead and search for a potential POC payload to exploit it and do find one which we use to extract the root flag as follows:
![Pasted image 20240629100829.png](/img/user/Sources/Pasted%20image%2020240629100829.png)

- We tried obtaining shell and writing output to /tmp/root dir to extract root flag
- WE DO OBTAIN THE ROOT FLAG!

```
Flag root: d664c14f6a526507b9bd56135de3332f
```

5. We have officially pwned the box!

![Pasted image 20240629101042.png](/img/user/Sources/Pasted%20image%2020240629101042.png)