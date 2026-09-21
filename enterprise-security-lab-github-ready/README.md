# Enterprise Security Lab 3 – Apache, DNS, SMTP & Snort IDS

> University of Technology, Jamaica  
> **Course:** CNS3004 – Enterprise Security

This repository documents the configuration of core enterprise network services on Ubuntu Server and the deployment of Snort as an Intrusion Detection System (IDS) to detect common network attacks.

---

## Technologies Used

- Ubuntu Server
- Apache2 Web Server
- BIND9 DNS Server
- Postfix SMTP Server
- Maildir + mutt
- Snort IDS
- Nmap
- SQLMap
- Hydra

---

## Lab Objectives

- Configure and secure an Apache web server.
- Host a custom HTML website using a Virtual Host.
- Configure a DNS server using BIND9.
- Configure SMTP services with Postfix.
- Test network services locally.
- Install Snort IDS and detect reconnaissance, SQL injection, brute-force, and DDoS attacks.

---

## Repository Structure

```text
enterprise-security-lab/
├── README.md
├── images/
├── configs/
│   ├── apache/
│   ├── bind9/
│   ├── postfix/
│   └── snort/
└── website/
```

---

# Part 1 – Enterprise Server Configuration

## 1. Apache Web Server Installation

### Install Apache

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
sudo ufw allow 'Apache'
```

Apache was installed, enabled at boot, and allowed through the Ubuntu firewall.

### Verification

Browse to the server IP:

```text
http://10.0.2.15
```

The default Apache page confirms the service is running.

![Apache Screenshot](images/lab3-step-1.png)


---

## 2. Hosting a Custom HTML Website

### Create Website Directory

```bash
sudo mkdir -p /var/www/lab3.com/html
sudo chown -R $USER:$USER /var/www/lab3.com/html
```

### Create Homepage

```html
<!DOCTYPE html>
<html>
<head>
    <title>Lab 3 Website</title>
</head>
<body>
    <h1>Enterprise Security Lab 3</h1>
</body>
</html>
```

### Configure Virtual Host

```apache
<VirtualHost *:80>
    ServerName lab3.com
    ServerAlias www.lab3.com
    DocumentRoot /var/www/lab3.com/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the site:

```bash
sudo a2ensite lab3.com.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## 3. Configure DNS with BIND9

### Install DNS Server

```bash
sudo apt install bind9 bind9utils bind9-doc -y
```

### Create Zone Files

```bash
sudo mkdir /etc/bind/zones
sudo cp /etc/bind/db.local /etc/bind/zones/db.lab3.com
```

### Example Zone File

```dns
$TTL 604800
@   IN SOA ns.lab3.com. admin.lab3.com. (
        2
        604800
        86400
        2419200
        604800 )

@       IN NS    ns.lab3.com.
ns      IN A     10.0.2.15
www     IN A     10.0.2.15
mail    IN A     10.0.2.15
```

### Test DNS

```bash
dig @127.0.0.1 lab3.com
```

Successful DNS resolution confirms the server is functioning.

---

## 4. Configure SMTP (Postfix + Maildir)

### Install Postfix

```bash
sudo apt install postfix mailutils mutt -y
```

### Important Postfix Settings

Edit `/etc/postfix/main.cf`:

```ini
myhostname = mail.lab3.com
mydomain = lab3.com
myorigin = /etc/mailname
home_mailbox = Maildir/
inet_interfaces = all
```

Restart Postfix.

```bash
sudo systemctl restart postfix
```

### Configure Maildir

```bash
mkdir -p ~/Maildir
maildirmake ~/Maildir
```

### Configure mutt

```bash
set mbox_type=Maildir
set folder="~/Maildir"
```

### Test Email

```bash
echo "Testing Postfix" | mail -s "Lab 3 Test" user@lab3.com
```

---

## 5. Verify Services

| Service | Command |
|--------|---------|
| Apache | `curl http://localhost` |
| DNS | `dig @127.0.0.1 lab3.com` |
| SMTP | `mail` / `mutt` |

Each service returned successful responses during testing.

---

# Part 2 – Network Defense with Snort IDS

## Install Snort

```bash
sudo apt install snort -y
```

Configure Snort for the correct network interface and HOME_NET.

---

## Reconnaissance Detection (Nmap)

### Attack

```bash
nmap -sS 10.0.2.15
```

### Snort Rule Purpose

Detect TCP SYN scans commonly used during reconnaissance.

### Expected Result

Snort generates alerts identifying port scan activity.

---

## DDoS Simulation

Generate repeated traffic toward the target server.

Snort monitors abnormal traffic volume and logs alerts for suspicious activity.

---

## SQL Injection Detection

### Attack Tool

```bash
sqlmap -u http://lab3.com/login.php --dbs
```

### Custom Rule Goal

Detect SQL keywords and injection attempts in HTTP requests.

### Expected Result

Snort logs SQL injection attempts in alert logs.

---

## SSH Brute Force Detection

### Attack Tool

```bash
hydra -l user -P passwords.txt ssh://10.0.2.15
```

### Detection Rule

Use Snort threshold rules to detect repeated SSH login attempts from a single source.

### Expected Result

Snort generates brute-force alerts after the configured threshold.

---

# Skills Demonstrated

| Area | Skills |
|------|--------|
| Linux Administration | Ubuntu Server, systemctl, firewall management |
| Web Services | Apache2 Virtual Hosts, HTML hosting |
| DNS | BIND9 zone configuration and testing |
| Email Services | Postfix, Maildir, mutt |
| Network Security | Snort IDS rule creation |
| Offensive Security Testing | Nmap, SQLMap, Hydra |

---

# Screenshots

The original screenshots from the lab report have been extracted into the **images/** folder. Insert or rearrange them throughout this README as needed.

Total screenshots extracted: **36**.

---

## Author

**Mekhi Bentley**  
Student ID: **2304855**  
University of Technology, Jamaica
