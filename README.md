<div align="center">

# ⚡ Hydra Brute-Force Demo ⚡
### A Hands-On HTTP Login Brute-Force Lab for Learning Hydra

[![Made for Education](https://img.shields.io/badge/Purpose-Educational-blueviolet?style=for-the-badge)](#-disclaimer)
[![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)](https://www.kali.org/)
[![Tool](https://img.shields.io/badge/Tool-Hydra-red?style=for-the-badge&logo=hackthebox&logoColor=white)](https://github.com/vanhauser-thc/thc-hydra)
[![PHP](https://img.shields.io/badge/Target-PHP%20Login%20Form-777BB4?style=for-the-badge&logo=php&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-Educational%20Use-lightgrey?style=for-the-badge)](#-disclaimer)

**Made by [Syed Muhammad Tayyab](https://github.com/Syed-Muhammad-Tayyab) — Happy Hacking! 🎉**

</div>

---

> ⚠️ **Educational use only.** Run this against your own local machine, VM, Docker container, or an authorized lab (TryHackMe/HTB). Never attack public or unauthorized systems.

---

## 📑 Table of Contents

- [🔥 What is Hydra?](#-what-is-hydra)
- [🚦 Before You Start](#-before-you-start-must-read)
- [🧱 Lab Architecture](#-lab-architecture)
- [🛠️ Step-by-Step Setup](#️-step-by-step-commands-copy-one-block-at-a-time)
- [🛡️ Detection & Defense Notes](#️-why-this-matters-detection--defense-notes)
- [⚠️ Disclaimer](#️-disclaimer)

---

## 🔥 What is Hydra?

**Hydra** is a fast and modular tool used to perform online password-guessing (brute-force) attacks against network services such as HTTP forms, SSH, FTP, SMTP and more.

In this repo, Hydra is pointed only at a tiny local PHP login page — the goal is to **teach** online brute-force concepts, how defenders detect them, and how to defend against them. 🧠🔐

---

## 🚦 Before You Start (Must Read)

| ✅ Do | ❌ Don't |
|---|---|
| Run on your own machine, VM, or Docker container | Attack public websites or services you don't own |
| Use an authorized lab (TryHackMe / HTB) | Test systems without written permission |
| Keep this contained to a local environment | Use the real `rockyou.txt` list carelessly on shared systems |

📜 *By using this repo you agree you are solely responsible for how you use it.*

---

## 🧱 Lab Architecture

```
┌─────────────────────┐        HTTP POST        ┌──────────────────────┐
│   Hydra (attacker)   │ ───────────────────────▶ │  PHP Login Server    │
│   rockyou-1000.txt   │                           │  index.php :8000     │
│   -l demo -P wordlist│ ◀─────────────────────── │  "Login failed" check│
└─────────────────────┘    "Login failed" / OK    └──────────────────────┘
        localhost                                        localhost
```

Both sides run **on the same machine** — this is a safe, self-contained lab with no external targets involved.

---

## 🛠️ Step-by-Step Commands (Copy One Block at a Time)

> ⚠️ Run the PHP server in one terminal so it stays visible (server logs show incoming POSTs).

### 1️⃣ System update & install PHP & Hydra

```bash
sudo apt update
```
*Update package lists.* 🔄

```bash
sudo apt install -y php-cli hydra
```
*Install PHP CLI (for the built-in server) and Hydra (the attacker tool).* 🧩

---

### 2️⃣ Create demo folder and the login page

```bash
mkdir -p ~/hydra-demo && cd ~/hydra-demo
```
*Create a demo directory and change into it.* 📁

```bash
nano index.php
```
*Open nano — paste the PHP code below, save (`Ctrl+O`) and exit (`Ctrl+X`).* ✍️

<details>
<summary><strong>📄 Click to expand: index.php</strong></summary>

```php
<?php
$correct_user = "demo";
$correct_pass = "password123";
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $user = $_POST['username'] ?? '';
    $pass = $_POST['password'] ?? '';
    if ($user === $correct_user && $pass === $correct_pass) {
        echo "Login successful. Welcome, " . htmlentities($user) . "!";
    } else {
        echo "Login failed: Invalid username or password.";
    }
    exit;
}
?>
<!doctype html>
<html><head><meta charset="utf-8"><title>Demo Login</title></head>
<body>
  <h2>Demo Login</h2>
  <form method="POST" action="index.php">
    <label>Username: <input type="text" name="username"></label><br><br>
    <label>Password: <input type="password" name="password"></label><br><br>
    <input type="submit" value="Login">
  </form>
</body></html>
```

</details>

*This minimal login page is the lab target. The failure message `Login failed` is what Hydra uses to detect unsuccessful attempts.* 🚨

---

### 3️⃣ Start the PHP server (keep this terminal open)

```bash
cd ~/hydra-demo
php -S 0.0.0.0:8000
```
*Start the built-in PHP server on port 8000.* ▶️

---

### 4️⃣ Build a small demo wordlist

```bash
cd /usr/share/wordlists
ls
sudo gunzip rockyou.txt.gz
ls
```
*Go to the wordlists folder and decompress `rockyou.txt.gz` (Kali default).* 🗂️

```bash
head -n 1000 rockyou.txt > ~/rockyou-1000.txt
```
*Create a small 1000-line subset for fast demos.* ⚡

```bash
wc -l ~/rockyou-1000.txt
```
*Sanity-check the line count.* 🔢

```bash
echo "password123" >> ~/rockyou-1000.txt
```
*Ensure the demo password is present in the list.* ✅

---

### 5️⃣ Confirm the server returns the fail string

```bash
curl -s -X POST http://127.0.0.1:8000/index.php -d "username=wrong&password=bad" | sed -n '1,4p'
```
*Confirm the output includes `Login failed` — Hydra will use this substring to detect failures.* 🔎

---

### 6️⃣ Run Hydra (from another terminal)

```bash
hydra -l demo -P ~/rockyou-1000.txt 127.0.0.1 http-post-form \
  "/index.php:username=^USER^&password=^PASS^:Login failed" \
  -s 8000 -t 4 -f -V
```
*Hydra brute-forces the `demo` user using the small list and stops on first success (`-f`).* ⚔️

---

## 🛡️ Why This Matters (Detection & Defense Notes)

Running this lab against your own server is a great way to see brute-force attacks from the **defender's** side:

- 🔍 Watch the PHP server logs — every failed attempt shows up as a POST request.
- 🚧 In a real app, this is where rate-limiting, account lockouts, CAPTCHAs, and WAF rules come in.
- ⏱️ Try adding a delay or an attempt counter to `index.php` and re-run Hydra to see the difference it makes.

---

## ⚠️ Disclaimer

This project is provided strictly for **educational and authorized security-testing purposes**. The author is not responsible for any misuse of the tools or techniques described in this repository. Always get explicit written permission before testing any system you do not own.

---

<div align="center">

Made with ❤️ by **[Syed Muhammad Tayyab](https://github.com/Syed-Muhammad-Tayyab)**
📂 [View this repo](https://github.com/Syed-Muhammad-Tayyab/hydra-brute-force-demo)

</div>
