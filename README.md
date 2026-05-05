# HTTPS + Self-Signed SSL on Apache — Live Lab Commands
**Run these IN ORDER on Kali Linux**

---

## STEP 1 — Install Required Packages
```bash
sudo apt update
sudo apt install openssl apache2 -y
```

## STEP 2 — Start Apache & Verify
```bash
sudo systemctl start apache2
sudo systemctl status apache2
```
Open browser → `http://localhost` → must show Apache default page.

---

## STEP 3 — Generate Private Key (2048-bit RSA)
```bash
sudo openssl genrsa -out /etc/ssl/private/server.key 2048
```

## STEP 4 — Generate CSR (Certificate Signing Request)
```bash
sudo openssl req -new -key /etc/ssl/private/server.key -out /etc/ssl/certs/server.csr
```

When prompted, type:
```
Country Name (2 letter code) [AU]: IN
State or Province Name: Uttar Pradesh
Locality Name: Mathura
Organization Name: GLA University
Organizational Unit Name: CSE
Common Name (server FQDN or YOUR name): gla.com
Email Address: dev@gmail.com
A challenge password: (press Enter)
An optional company name: Dev
```

> **Critical:** Common Name MUST be `gla.com` — same as what you'll type in browser.

---

## STEP 5 — Generate Self-Signed Certificate (Valid 365 Days)
```bash
sudo openssl x509 -req -days 365 \
  -in /etc/ssl/certs/server.csr \
  -signkey /etc/ssl/private/server.key \
  -out /etc/ssl/certs/server.crt
```

Verify the certificate:
```bash
openssl x509 -in /etc/ssl/certs/server.crt -text -noout
```

---

## STEP 6 — Enable Apache SSL Module
```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

---

## STEP 7 — Edit SSL Virtual Host Config
```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Find these lines and update them (inside `<VirtualHost _default_:443>`):
```apache
ServerName gla.com
DocumentRoot /var/www/html

SSLEngine on
SSLCertificateFile      /etc/ssl/certs/server.crt
SSLCertificateKeyFile   /etc/ssl/private/server.key
```

Save: `Ctrl+O` → Enter → `Ctrl+X`

---

## STEP 8 — Enable SSL Site & Restart Apache
```bash
sudo a2ensite default-ssl
sudo apache2ctl configtest
sudo systemctl restart apache2
```

`configtest` should output: `Syntax OK`

---

## STEP 9 — Map Domain to Localhost
```bash
sudo nano /etc/hosts
```

Add this line:
```
127.0.0.1   gla.com
```

Save & exit.

---

## STEP 10 — Create Test Page (Looks Good in Practical File)
```bash
sudo nano /var/www/html/index.html
```

Paste:
```html
<html>
<head><title>Kali Apache HTTPS Test</title></head>
<body style="background:#0a0a0a;color:#00FFB2;font-family:monospace;text-align:center;padding-top:80px;">
<h1>Kali Apache Test - HTTPS Secure!</h1>
<p>SSL Configured by Dev Singh Chauhan (2415000502)</p>
</body>
</html>
```

---

## STEP 11 — TEST IT (Screenshot These!)

### A) Browser Test
```
https://gla.com
```
- Browser shows warning ("Connection not secure") because cert is self-signed
- Click **Advanced → Accept Risk and Continue**
- Page loads with 🔒 lock icon
- Click lock icon → **View Certificate** → confirm:
  - CN = `gla.com`
  - Issuer = `gla.com` (self-signed)
  - Valid 365 days

### B) Terminal Test (curl)
```bash
curl -vk https://gla.com
```

Look for these lines in output:
```
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* Server certificate:
*  subject: C=IN; ST=Uttar Pradesh; L=Mathura; O=GLA University; OU=CSE; CN=gla.com
```

This **TLS handshake** = proof HTTPS is working.

---

## ✅ Done!
You've successfully configured HTTPS on Apache with a self-signed certificate. Take screenshots of:
1. Browser showing 🔒 + page loading on `https://gla.com`
2. Certificate details (CN, validity)
3. `curl -vk` output showing TLS handshake

---

## 🔧 If Something Breaks

| Error | Fix |
|---|---|
| `apache2: SSL Library Error` | `sudo a2enmod ssl && sudo systemctl restart apache2` |
| Browser: "connection refused" on 443 | `sudo ufw allow 443` |
| `gla.com` not resolving | Check `/etc/hosts` line is correct |
| `configtest` fails | `sudo apache2ctl configtest` and read the error |
| Cert paths wrong | Verify with: `ls /etc/ssl/private/server.key /etc/ssl/certs/server.crt` |
