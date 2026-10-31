# EC2-Based Web Server: Nginx on Ubuntu (AWS)

> Deploying a web server on Amazon EC2 using Ubuntu and Nginx, built from scratch with the AWS Management Console and the Linux command line.

![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu_Server-E95420?logo=ubuntu&logoColor=white)
![Nginx](https://img.shields.io/badge/Web_Server-Nginx-009639?logo=nginx&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Core Concepts](#3-core-concepts)
4. [Tech Stack](#4-tech-stack)
5. [Project Output](#5-project-output)
6. [Step-by-Step Implementation](#6-step-by-step-implementation)
7. [Verification and Testing](#7-verification-and-testing)
8. [Useful Commands](#8-useful-commands)
9. [Troubleshooting](#9-troubleshooting)
10. [Security Considerations](#10-security-considerations)
11. [Cost and Cleanup](#11-cost-and-cleanup)
12. [Future Improvements](#12-future-improvements)
13. [Author](#13-author)

---

## 1. Project Overview

This project shows how to host a website on a cloud virtual machine instead of a managed hosting service. I launched an **Amazon EC2** instance running **Ubuntu**, installed **Nginx**, opened the required ports with a **Security Group**, and served a custom HTML page to the internet.

**What this project covers**

- Provisioning a virtual server in the AWS cloud
- Securing access with a key pair and security group rules
- Connecting to a remote Linux server over SSH
- Installing and managing a service with `apt` and `systemd`
- Serving static content with Nginx
- Making the service start automatically after a reboot
- Testing, troubleshooting, and cleaning up cloud resources

**Skills demonstrated**

| Area | Skill |
|---|---|
| Cloud | Launching and managing EC2 instances, AMIs, instance types |
| Networking | Security groups, ports 22 and 80, public IPs, inbound rules |
| Linux | SSH, file permissions, package management, `systemctl`, logs |
| Web servers | Nginx installation, document root, serving static files |
| Security | Key-based authentication, restricting SSH access |

---

## 2. Architecture

```mermaid
flowchart LR
    U[User Browser] -- HTTP :80 --> IGW[Internet Gateway]
    IGW --> SG{Security Group<br/>Allow 22, 80}
    SG --> EC2[EC2 Instance<br/>Ubuntu + Nginx]
    EC2 --> WEB[/var/www/html/index.html/]
    D[Developer Laptop] -- SSH :22 + key pair --> SG
```

**Request flow**

1. The user opens `http://<public-ip>` in a browser.
2. The request enters AWS through the VPC's Internet Gateway.
3. The Security Group checks the rules. Port 80 is allowed, so the request passes.
4. Nginx, listening on port 80, receives the request.
5. Nginx reads `index.html` from the document root and returns it to the browser.

---

## 3. Core Concepts

**Amazon EC2 (Elastic Compute Cloud):** A virtual server rented from AWS. You choose the operating system, CPU, memory, and storage, and pay for what you use.

**AMI (Amazon Machine Image):** A template that contains the operating system and base software. This project uses the official Ubuntu Server AMI.

**Instance type (`t2.micro`):** Defines the hardware: 1 vCPU and 1 GiB RAM, which is enough for a static website. Free Tier eligibility depends on your account and region, so check your AWS Billing page.

**Key pair:** A public/private key pair used for SSH login instead of a password. AWS stores the public key on the server, and you keep the private `.pem` file. If you lose it, you lose SSH access.

**Security Group:** A virtual, stateful firewall attached to the instance. By default it blocks all inbound traffic, so each needed port must be allowed explicitly.

| Port | Protocol | Purpose |
|---|---|---|
| 22 | SSH | Remote administration |
| 80 | HTTP | Public web traffic |

**Nginx:** A high-performance web server and reverse proxy, known for handling many concurrent connections with low memory use.

**systemd / systemctl:** The Linux service manager that starts, stops, and enables services at boot.

**Document root:** The folder from which the web server serves files. For Nginx installed with `apt` on Ubuntu, it is `/var/www/html`.

---

## 4. Tech Stack

| Component | Choice |
|---|---|
| Cloud provider | Amazon Web Services (AWS) |
| Compute | EC2, `t2.micro` |
| Operating system | Ubuntu Server |
| Web server | Nginx |
| Access method | SSH with key pair |
| Content | Static HTML |

---

## 5. Project Output

### Website Served by Nginx

![EC2 Instances List](screenshots/Screenshot%202026-10-29%20224003.png)

### EC2 Instance Running

![Website Interface](screenshots/Screenshot%202026-10-29%20224206.png)

---

## 6. Step-by-Step Implementation

### Step 1: Open EC2 in the AWS Console

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Select your preferred region (top-right corner).
3. Search for **EC2** and open the dashboard.

### Step 2: Launch the instance

Click **Launch instance** and configure:

| Setting | Value |
|---|---|
| Name | `MyWebServer` |
| AMI | Ubuntu Server (latest LTS) |
| Instance type | `t2.micro` |
| Key pair | Create new, RSA, `.pem` format |
| Public IP | Auto-assign enabled |
| Storage | 8 GiB (default) |

> Download the `.pem` file immediately. AWS shows it only once.

### Step 3: Configure the Security Group

Create a security group with these **inbound rules**:

| Type | Protocol | Port | Source | Reason |
|---|---|---|---|---|
| SSH | TCP | 22 | My IP | Only I can administer the server |
| HTTP | TCP | 80 | 0.0.0.0/0 | Anyone can view the website |

Click **Launch instance**, wait for the state to show **Running** with **2/2 status checks passed**, then copy the **Public IPv4 address**.

### Step 4: Connect via SSH

Protect the key file (SSH rejects keys with open permissions):

```bash
chmod 400 your-key.pem
```

Connect (the default Ubuntu user is `ubuntu`, not `ec2-user`):

```bash
ssh -i your-key.pem ubuntu@<your-instance-public-ip>
```

Type `yes` when asked to trust the host.

### Step 5: Update the system

```bash
sudo apt update -y
sudo apt upgrade -y
```

### Step 6: Install Nginx

```bash
sudo apt install nginx -y
```

### Step 7: Start and enable Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

- `start` runs the service now.
- `enable` makes it start automatically on every boot.

Verify:

```bash
sudo systemctl status nginx
```

The output should show `active (running)`. Press `q` to exit.

### Step 8: Firewall check

Ubuntu includes `ufw`, which is inactive by default on EC2. Traffic is controlled by the AWS Security Group, so no extra firewall configuration is needed.

```bash
sudo ufw status
```

Expected: `Status: inactive`

> The `firewall-cmd` commands seen in many tutorials are for Amazon Linux / RHEL, not Ubuntu.

### Step 9: Test the default page

Open `http://<your-instance-public-ip>` in a browser. You should see **"Welcome to nginx!"**. Use `http://`, not `https://`, because HTTPS is not configured.

### Step 10: Deploy a custom web page

```bash
echo "<h1>Welcome to My Web Server</h1>" | sudo tee /var/www/html/index.html
```

> Do not use `sudo echo "..." > file`. The `sudo` applies only to `echo`, while the `>` redirect runs as your normal user, which fails with *Permission denied*. Piping into `sudo tee` avoids this.

For a fuller page:

```bash
sudo nano /var/www/html/index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My EC2 Web Server</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; margin-top: 15%; background: #0f172a; color: #e2e8f0; }
    h1 { color: #38bdf8; }
  </style>
</head>
<body>
  <h1>Welcome to My Web Server</h1>
  <p>Hosted on AWS EC2 with Ubuntu and Nginx</p>
</body>
</html>
```

Save with `Ctrl + O`, `Enter`, then exit with `Ctrl + X`.

> On Ubuntu, the document root is `/var/www/html`. The path `/usr/share/nginx/html` applies to Amazon Linux / RHEL.

### Step 11: Access the website

Refresh `http://<your-instance-public-ip>`. Your custom page should appear.

### Step 12: Confirm auto-start after reboot

```bash
sudo systemctl is-enabled nginx
sudo reboot
```

Wait about a minute, reconnect over SSH, and run `sudo systemctl status nginx`. If Nginx is `active (running)` without being started manually, the setup is reboot-safe.

> A reboot keeps the public IP, but stopping and starting the instance changes it. Use an Elastic IP for a permanent address.

---

## 7. Verification and Testing

| Test | Command / Action | Expected result |
|---|---|---|
| Service running | `sudo systemctl status nginx` | `active (running)` |
| Port listening | `sudo ss -tulnp \| grep :80` | Nginx on port 80 |
| Local response | `curl http://localhost` | Returns the HTML |
| Config syntax | `sudo nginx -t` | `syntax is ok` |
| Public access | Browser: `http://<public-ip>` | Custom page displays |
| HTTP headers | `curl -I http://<public-ip>` | `HTTP/1.1 200 OK` |
| Reboot survival | `sudo reboot`, then check status | Nginx runs automatically |

---

## 8. Useful Commands

```bash
# Service control
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx        # apply config changes without downtime
sudo systemctl status nginx

# Configuration
sudo nginx -t                      # test configuration
sudo nginx -v                      # show version

# Logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

**Key locations**

| Path | Purpose |
|---|---|
| `/var/www/html/` | Website files (document root) |
| `/etc/nginx/nginx.conf` | Main configuration |
| `/etc/nginx/sites-available/` | Site configuration files |
| `/etc/nginx/sites-enabled/` | Active sites (symlinks) |
| `/var/log/nginx/` | Access and error logs |

---

## 9. Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| Browser: connection timed out | Port 80 not open in Security Group | Add inbound HTTP rule (TCP 80) |
| Browser: connection refused | Nginx not running | `sudo systemctl start nginx` |
| Page fails on `https://` | HTTPS not configured | Use `http://` or set up SSL |
| SSH: Permission denied (publickey) | Wrong user or key | Use `ubuntu@` and the correct `.pem` |
| SSH: UNPROTECTED PRIVATE KEY FILE | Key permissions too open | `chmod 400 your-key.pem` |
| SSH: connection timed out | Port 22 blocked or your IP changed | Update the SSH rule to your current IP |
| Permission denied writing `index.html` | `sudo echo > file` mistake | `echo "..." \| sudo tee file` |
| Old page still showing | Browser cache | Hard refresh: `Ctrl + Shift + R` |
| Default page shows instead of mine | Wrong folder or file name | Confirm `/var/www/html/index.html` exists |
| IP changed after stopping instance | Public IPs are dynamic | Allocate an Elastic IP |

---

## 10. Security Considerations

- **SSH is restricted to my IP**, not `0.0.0.0/0`. Open SSH to the world and bots begin brute-forcing it within minutes.
- **Key-based login only.** Password authentication is disabled by default on Ubuntu EC2 images.
- **The `.pem` file stays private.** Never commit it to GitHub.
- **Only necessary ports are open** (22 and 80). Everything else is blocked by default.
- **Keep the system patched:** `sudo apt update && sudo apt upgrade -y`.
- **HTTP is unencrypted.** This is acceptable for a static demo, but not for logins or personal data.
- **Billing alerts** in AWS Budgets prevent surprise charges.

---

## 11. Cost and Cleanup

A `t2.micro` instance may be covered by the AWS Free Tier, but eligibility depends on your account and region. Verify on your AWS Billing page.

**To avoid charges, delete resources when finished:**

1. EC2 → Instances → select the instance → **Instance state → Terminate instance**
2. EC2 → Elastic IPs → **Release** any allocated address
3. EC2 → Volumes → confirm no leftover volumes
4. Optionally delete the security group and key pair

*Stopping* an instance pauses compute charges but storage is still billed. *Terminating* deletes it.

---

## 12. Future Improvements

- [ ] Attach an **Elastic IP** for a permanent address
- [ ] Register a **domain** and point it with Route 53
- [ ] Enable **HTTPS** with Let's Encrypt:
  ```bash
  sudo apt install certbot python3-certbot-nginx -y
  sudo certbot --nginx -d yourdomain.com
  ```
- [ ] Configure Nginx **server blocks** to host multiple sites
- [ ] Use Nginx as a **reverse proxy** for a Node.js, Flask, or Django app
- [ ] Add a **Load Balancer** and **Auto Scaling Group**
- [ ] Automate provisioning with **Terraform** or **CloudFormation**
- [ ] Add **CloudWatch** monitoring and alarms
- [ ] Set up **CI/CD** with GitHub Actions

**Bonus: bootstrap the server with EC2 User Data**

Paste this under *Advanced details → User data* when launching, and the server configures itself on first boot:

```bash
#!/bin/bash
apt update -y
apt install nginx -y
systemctl enable nginx
systemctl start nginx
echo "<h1>Welcome to My Web Server</h1>" > /var/www/html/index.html
```

---

## 13. Author

**Vinay**
Cloud and DevOps enthusiast | Bengaluru, India

GitHub: [@Vinay-PR-Gowda](https://github.com/Vinay-PR-Gowda)

---

## Conclusion

I deployed a working web server on AWS EC2 with Ubuntu and Nginx, controlled access with a key pair and security group, served a custom page publicly, and verified that the service survives a reboot. This setup is the base for hosting websites, web applications, and reverse-proxy architectures.
