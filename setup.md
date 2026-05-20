# How to Set Up Metasploitable 2 in VirtualBox and Connect It to Kali Linux

### A step-by-step guide for ethical hackers and security students to build a safe, isolated lab environment

---

**Reading time:** ~8 minutes | **Level:** Beginner to Intermediate | **Tags:** *cybersecurity, ethical hacking, kali linux, virtualbox, penetration testing*

---

If you're learning penetration testing, you've probably heard the golden rule: **never practice on systems you don't own or have explicit permission to test.** That's where Metasploitable 2 comes in — a deliberately vulnerable Linux virtual machine designed specifically to be hacked. Paired with Kali Linux, it gives you a completely legal, self-contained lab to sharpen your offensive security skills.

In this guide, I'll walk you through:
- Downloading and importing Metasploitable 2 into VirtualBox
- Configuring the network so Kali and Metasploitable can talk to each other (but stay isolated from the internet)
- Verifying the connection and running your first scan

Let's get started.

---

## Prerequisites

Before we begin, make sure you have the following:

- **VirtualBox** installed (version 6.x or 7.x) — [download here](https://www.virtualbox.org/wiki/Downloads)
- **Kali Linux** already set up as a VM in VirtualBox (ISO or pre-built OVA)
- At least **20 GB of free disk space** and **4 GB of RAM** to run both VMs simultaneously
- A working internet connection for the initial downloads

---

## Step 1 — Download Metasploitable 2

Metasploitable 2 is maintained by Rapid7 and is freely available.

1. Head to [SourceForge — Metasploitable](https://sourceforge.net/projects/metasploitable/) or search "Metasploitable 2 download Rapid7."
2. Download the `.zip` file (it's around 800 MB).
3. Once downloaded, extract the `.zip`. You'll get a folder containing several files, including a `.vmdk` (virtual disk) file — this is the heart of the VM.

> **Note:** Metasploitable 2 was originally built for VMware, but we'll import it into VirtualBox using its `.vmdk` disk file. No conversion needed.

---

## Step 2 — Create a New VM in VirtualBox for Metasploitable 2

Open VirtualBox and follow these steps:

1. Click **New** to create a new virtual machine.
2. Set the following:
   - **Name:** `Metasploitable2` (or anything you like)
   - **Type:** `Linux`
   - **Version:** `Ubuntu (32-bit)`
3. Set **Memory (RAM)** to at least **512 MB** (1024 MB recommended).
4. On the **Hard Disk** screen, select **"Use an existing virtual hard disk file"**.
5. Click the folder icon, browse to the extracted Metasploitable folder, and select the `.vmdk` file.
6. Click **Create**.

Your Metasploitable 2 VM is now registered in VirtualBox.

---

## Step 3 — Configure the Network (The Most Important Step)

This is where most beginners get tripped up. We want Kali and Metasploitable to communicate with each other, but we **do not** want Metasploitable exposed to your home network or the internet — it's full of vulnerabilities by design.

The solution: **Host-Only Networking** or **Internal Networking**.

We'll use **Host-Only**, which allows:
- Kali ↔ Metasploitable communication ✅
- Host machine ↔ VMs communication ✅
- Metasploitable ↔ Internet ❌ (good — keeps it isolated)

### 3a — Create a Host-Only Network in VirtualBox

1. Go to **File → Host Network Manager** (or **Tools → Network** in newer VirtualBox versions).
2. Click **Create** to add a new host-only adapter (e.g., `vboxnet0`).
3. Set the **IPv4 Address** to `192.168.56.1` and **Subnet Mask** to `255.255.255.0`.
4. Under the **DHCP Server** tab, make sure DHCP is **enabled** with:
   - Server Address: `192.168.56.100`
   - Lower bound: `192.168.56.101`
   - Upper bound: `192.168.56.254`
5. Click **Apply** and close.

### 3b — Assign the Network to Metasploitable 2

1. Select your **Metasploitable2** VM in VirtualBox and click **Settings**.
2. Go to **Network → Adapter 1**.
3. Set **Attached to:** `Host-only Adapter`.
4. Set **Name:** `vboxnet0` (the adapter you just created).
5. Click **OK**.

### 3c — Assign the Network to Kali Linux

1. Select your **Kali Linux** VM and click **Settings**.
2. Go to **Network**.
3. **Adapter 1** — Keep this as `NAT` (so Kali can still reach the internet for updates).
4. **Adapter 2** — Enable it, set **Attached to:** `Host-only Adapter`, Name: `vboxnet0`.
5. Click **OK**.

> **Why two adapters on Kali?** Adapter 1 (NAT) gives Kali internet access for downloading tools. Adapter 2 (Host-only) lets it reach Metasploitable. Metasploitable only needs the Host-only adapter.

---

## Step 4 — Boot Both Virtual Machines

1. Start **Metasploitable 2** first. It will boot into a text login screen.
2. Log in with the default credentials:
   - **Username:** `msfadmin`
   - **Password:** `msfadmin`
3. Once logged in, run:
   ```bash
   ifconfig
   ```
   Note the IP address on the `eth0` interface — it should be something like `192.168.56.101`.

4. Now start your **Kali Linux** VM and log in normally.

---

## Step 5 — Verify Connectivity from Kali

Open a terminal in Kali and ping Metasploitable's IP:

```bash
ping 192.168.56.101
```

You should see replies like:

```
64 bytes from 192.168.56.101: icmp_seq=1 ttl=64 time=0.512 ms
64 bytes from 192.168.56.101: icmp_seq=2 ttl=64 time=0.431 ms
```

If you get responses, **congratulations — your lab is working!** 🎉

If you get timeouts, double-check:
- Both VMs are using `vboxnet0` as the Host-only adapter
- Metasploitable's `eth0` shows an IP in the `192.168.56.x` range
- The Host-only DHCP server is enabled in VirtualBox

---

## Step 6 — Run Your First Nmap Scan

Now that everything is connected, let's verify Metasploitable is reachable and see what services are running:

```bash
nmap -sV 192.168.56.101
```

The `-sV` flag detects service versions. You'll see output something like this:

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
80/tcp   open  http        Apache httpd 2.2.8
139/tcp  open  netbios-ssn Samba smbd 3.X
3306/tcp open  mysql       MySQL 5.0.51a
5432/tcp open  postgresql  PostgreSQL DB
8180/tcp open  http        Apache Tomcat/Coyote
...
```

That's a goldmine of vulnerable services — intentionally so. Each one of these is an opportunity to practice a different attack technique.

---

## Bonus: Taking a Snapshot Before You Start

Before you begin testing, take a VirtualBox snapshot of both VMs. This lets you roll back to a clean state anytime you break something (and you will break things — that's how you learn).

In VirtualBox:
1. With the VM selected, go to **Machine → Take Snapshot**.
2. Name it something like `Clean State — Pre-Testing`.

---

## What's Next?

Now that your lab is ready, here are some things you can try:

- **Exploit vsftpd 2.3.4** — Famous backdoor vulnerability, perfect for a first Metasploit exercise
- **Brute-force SSH** with Hydra using common wordlists
- **Enumerate Samba shares** using `enum4linux`
- **Practice web application attacks** on the DVWA (Damn Vulnerable Web App) served on port 80
- **Use Metasploit Framework** (`msfconsole`) to run modules against Metasploitable

---

## A Word on Ethics

Metasploitable is for **learning in a controlled, private lab only**. Never run it on a public network or expose it to the internet. Never use the techniques you learn here against systems without written permission. Ethical hacking starts with respect for boundaries.

---

## Summary

| Step | What You Did |
|------|-------------|
| 1 | Downloaded Metasploitable 2 (.vmdk) |
| 2 | Created a VirtualBox VM using the .vmdk |
| 3 | Configured Host-only networking for isolation |
| 4 | Booted both VMs and noted Metasploitable's IP |
| 5 | Verified connectivity with `ping` from Kali |
| 6 | Ran `nmap -sV` to enumerate services |

---

*Happy hacking — the legal kind. If you found this guide helpful, drop a clap and follow for more beginner-friendly security content.*

---

*Tags: #cybersecurity #ethicalhacking #kalilinux #virtualbox #penetrationtesting #metasploit #infosec*
