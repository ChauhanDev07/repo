# Experiment 5: Configuring and Testing HTTPS on a Web Server
**Dev Singh Chauhan | Roll: 2415000502 | Section: 2_SA**

---

## Objective
Configure HTTPS using a self-signed SSL certificate on an Apache web server in Kali Linux and verify secure communication.

---

## Theory
HTTPS (HyperText Transfer Protocol Secure) is the secure version of HTTP that uses SSL/TLS encryption to protect data exchanged between a client and a server. It provides three key security services:

- **Confidentiality** — data is encrypted, no one can read it in transit.
- **Integrity** — data cannot be modified during transmission.
- **Authentication** — the server proves its identity using a digital certificate.

A **self-signed SSL certificate** is created using OpenSSL (no external CA). It works fine for testing/lab purposes but browsers show a warning since it isn't issued by a trusted authority. Apache uses the `mod_ssl` module and listens on port **443** for HTTPS connections.

---

## Tools Used
- **Kali Linux** (or Ubuntu)
- **OpenSSL** — for generating private key, CSR, and certificate
- **Apache2** — web server
- **Browser + curl** — to test HTTPS

---

## Procedure (Lab Steps)

### Step 1: Update System and Install Required Packages
```
sudo apt update
sudo apt install openssl -y
sudo apt install apache2 -y
```

### Step 2: Verify Apache is Running
```
sudo systemctl start apache2
sudo systemctl status apache2
```
Open browser → `http://localhost` → "Apache2 Default Page" should appear.

---

### Step 3: Generate the Private Key
```
sudo openssl genrsa -out /etc/ssl/private/server.key 2048
```
This creates a 2048-bit RSA private key for the server.

---

### Step 4: Generate Certificate Signing Request (CSR)
```
sudo openssl req -new -key /etc/ssl/private/server.key -out /etc/ssl/certs/server.csr
```

When prompted, fill in:
```
Country Name (2 letter code) [AU]: IN
State or Province Name: Uttar Pradesh
Locality Name: Mathura
Organization Name: GLA University
Organizational Unit Name: CSE
Common Name (server FQDN or YOUR name): gla.com
Email Address: dev@gmail.com
A challenge password: (leave blank or 12345678)
An optional company name: Dev
```

> **Important:** Common Name (CN) must match the domain you'll access in the browser (e.g., `gla.com`).

---

### Step 5: Generate the Self-Signed Certificate
```
sudo openssl x509 -req -days 365 -in /etc/ssl/certs/server.csr \
  -signkey /etc/ssl/private/server.key -out /etc/ssl/certs/server.crt
```
This creates `server.crt` valid for 365 days.

You can verify it:
```
openssl x509 -in /etc/ssl/certs/server.crt -text -noout
```

---

### Step 6: Enable Apache SSL Module
```
sudo a2enmod ssl
sudo systemctl restart apache2
```

---

### Step 7: Configure SSL Virtual Host
```
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Modify these lines inside `<VirtualHost *:443>`:
```
ServerAdmin webmaster@localhost
ServerName gla.com
DocumentRoot /var/www/html

SSLEngine on
SSLCertificateFile /etc/ssl/certs/server.crt
SSLCertificateKeyFile /etc/ssl/private/server.key
```

Save and exit (Ctrl+O, Enter, Ctrl+X).

---

### Step 8: Enable SSL Site and Restart Apache
```
sudo a2ensite default-ssl
sudo systemctl reload apache2
sudo systemctl restart apache2
```

---

### Step 9: Map Domain Name in /etc/hosts
```
sudo nano /etc/hosts
```
Add this line:
```
127.0.0.1   gla.com
```

---

### Step 10: Create Test Web Page (Optional but Looks Good in File)
```
sudo nano /var/www/html/index.html
```
Paste:
```html
<html>
<head><title>Kali Apache HTTPS Test</title></head>
<body style="background:#0a0a0a;color:#00FFB2;font-family:monospace;">
<h1>Kali Apache Test - HTTPS Secure!</h1>
<p>SSL Configured by Dev Singh Chauhan (2415000502)</p>
</body>
</html>
```

---

### Step 11: Test HTTPS Configuration

**A) From Browser:**
- Open `https://gla.com`
- A warning appears (because it is self-signed) → click *Advanced → Accept the risk and continue*.
- The page loads with a 🔒 lock icon in the address bar.
- Click on the lock → view certificate → confirm CN = `gla.com`, valid for 365 days.

**B) From Terminal (curl):**
```
curl -vk https://gla.com
```

Expected output (key parts):
```
* TLSv1.3 (OUT), TLS handshake, Client hello
* TLSv1.3 (IN), TLS handshake, Server hello
* TLSv1.3 (IN), TLS handshake, Certificate
* TLSv1.3 (IN), TLS handshake, Finished
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* Server certificate:
    subject: CN=gla.com
    issuer: CN=gla.com
```

The TLS handshake confirms HTTPS is working.

---

## Verification Checklist
- [x] Apache running on port 443.
- [x] `https://gla.com` loads with lock icon.
- [x] Certificate details show CN=gla.com, validity = 365 days.
- [x] `curl -v` shows successful TLS handshake.

---

## Result
HTTPS was successfully configured on the Apache web server using a self-signed SSL certificate generated with OpenSSL. The communication between client and server is now encrypted using TLS, ensuring confidentiality, integrity, and authentication.

---

## Quick Troubleshooting (If Something Fails)

| Problem | Fix |
|---|---|
| `apache2: SSL Library Error` | Run `sudo a2enmod ssl` and restart apache2 |
| Browser cannot connect on 443 | Check firewall: `sudo ufw allow 443` |
| Certificate not loading | Verify paths in `default-ssl.conf` match where files are saved |
| `gla.com` not resolving | Check `/etc/hosts` entry `127.0.0.1 gla.com` |
| Apache won't restart | Run `sudo apache2ctl configtest` to find syntax errors |

---

## One-Line Command Summary (For Quick Revision)
```
sudo apt install openssl apache2 -y
sudo openssl genrsa -out /etc/ssl/private/server.key 2048
sudo openssl req -new -key /etc/ssl/private/server.key -out /etc/ssl/certs/server.csr
sudo openssl x509 -req -days 365 -in /etc/ssl/certs/server.csr -signkey /etc/ssl/private/server.key -out /etc/ssl/certs/server.crt
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl restart apache2
echo "127.0.0.1 gla.com" | sudo tee -a /etc/hosts
curl -vk https://gla.com
```
