# Load Balancer Solution with NGINX and SSL/TLS

<details>
<summary>📑 Table of Contents</summary>

- [Load Balancer Solution with NGINX and SSL/TLS](#load-balancer-solution-with-nginx-and-ssltls)
  - [📌 Introduction](#-introduction)
  - [🔒 Why Secure Connections Matter](#-why-secure-connections-matter)
  - [👾 Man-in-the-Middle (MITM) Attack](#-man-in-the-middle-mitm-attack)
  - [🔐 Enter SSL and TLS](#-enter-ssl-and-tls)
  - [SSL vs. TLS](#ssl-vs-tls)
  - [📄 Role of Digital Certificates](#-role-of-digital-certificates)
  - [🤝 Automating Certificates with Let’s Encrypt](#-automating-certificates-with-lets-encrypt)
  - [🔄 Deep Dive: How SSL/TLS Handshakes Work](#-deep-dive-how-ssltls-handshakes-work)
  - [⚖️ From Apache to NGINX: Why the Change?](#️-from-apache-to-nginx-why-the-change)
    - [1. Performance \& Concurrency](#1-performance--concurrency)
    - [2. Load Balancing Features](#2-load-balancing-features)
    - [3. Resource Efficiency](#3-resource-efficiency)
    - [4. Reverse Proxy Capabilities](#4-reverse-proxy-capabilities)
    - [5. Modern Web Use Cases](#5-modern-web-use-cases)
    - [📌 Summary Table](#-summary-table)
- [🛠️ Project Overview](#️-project-overview)
- [⚙️ Part 1 – Configure NGINX as a Load Balancer](#️-part-1--configure-nginx-as-a-load-balancer)
  - [🖥️ Step 1: Provision a New EC2 Instance](#️-step-1-provision-a-new-ec2-instance)
  - [📌 Step 2: Install NGINX](#-step-2-install-nginx)
  - [📌 Step 3: Configure Local DNS Resolution](#-step-3-configure-local-dns-resolution)
  - [📌 Step 4: Configure NGINX as a Load Balancer](#-step-4-configure-nginx-as-a-load-balancer)
  - [📌 Step 5: Validate Configuration](#-step-5-validate-configuration)
  - [📌 Step 6: Verify Load Balancing](#-step-6-verify-load-balancing)
  - [✅ Step 7: Restart and Confirm](#-step-7-restart-and-confirm)
  - [🎯 Result](#-result)
- [🔐 Part 2 – Register a Domain Name and Configure SSL/TLS Certificates](#-part-2--register-a-domain-name-and-configure-ssltls-certificates)
  - [🏷️ Step 1: Register a Domain Name](#️-step-1-register-a-domain-name)
  - [🌍 Step 2: Assign an Elastic IP](#-step-2-assign-an-elastic-ip)
  - [📌 Step 3: Update DNS Records](#-step-3-update-dns-records)
  - [⚙️ Step 4: Update NGINX to Recognize Your Domain](#️-step-4-update-nginx-to-recognize-your-domain)
  - [🔑 Step 5: Install Certbot and Request SSL/TLS Certificate](#-step-5-install-certbot-and-request-ssltls-certificate)
  - [✅ Step 6: Test Secure HTTPS Access](#-step-6-test-secure-https-access)
  - [🔄 Step 7: Automate Certificate Renewal](#-step-7-automate-certificate-renewal)
  - [🎯 Result](#-result-1)
- [✅ Conclusion](#-conclusion)
</details>


## 📌 Introduction

A DevOps engineer must be versatile and always aware of multiple alternative solutions for the same problem. Why? Because in real-world systems, there is no “one size fits all.” What works for one organization might not work for another.

In this project, we will build and configure an NGINX Load Balancer solution. But that’s not all—modern applications don’t just need to be scalable; they also need to be secure. Ensuring that connections to your web applications are protected is just as important as serving traffic reliably.

This is why we’ll also set up secure connections over HTTPS by implementing SSL/TLS certificates. Let’s break this down from both a holistic and technical angle.

## 🔒 Why Secure Connections Matter

Imagine you’re writing a confidential letter to your bank. If you put it in a transparent envelope, anyone who handles it along the way—postal workers, neighbors, or hackers—could read its contents. That’s essentially what happens when you send data over plain HTTP: it travels as clear text and can be intercepted.

This brings us to a critical security threat:

## 👾 Man-in-the-Middle (MITM) Attack

A Man-in-the-Middle attack occurs when an attacker secretly intercepts and possibly alters the communication between two parties (e.g., your browser and a web server).
- **Analogy:** Imagine you’re talking to a friend over the phone, but someone silently taps the line, listens to your conversation, and occasionally changes your words before passing them along. You and your friend wouldn’t even notice the interference.
- **In Web Traffic:** Without encryption, attackers can eavesdrop on sensitive information such as login credentials, banking details, or personal data. Worse still, they can inject malicious content into the data stream.

This is why encryption in transit is not optional—it’s essential.

## 🔐 Enter SSL and TLS

To protect against these threats, we use SSL (Secure Sockets Layer) and its newer, more secure replacement TLS (Transport Layer Security).

Think of SSL/TLS as a sealed and tamper-proof envelope for your digital communication:

1. **Encryption** – The message is scrambled, so only the intended recipient can read it.
2. **Authentication** – The recipient (a website, for instance) must prove it’s really who it claims to be.
3. **Integrity** – Ensures that the message isn’t altered in transit.

## SSL vs. TLS

- **SSL** was the original protocol.
- **TLS** is its improved, more secure successor (just like how smartphones replaced pagers).
- Today, when people say “SSL certificates,” they usually mean **TLS certificates**—it’s just that the older term stuck around.

## 📄 Role of Digital Certificates

SSL/TLS relies on digital certificates to establish trust.

- When you visit a website, your browser looks at its certificate.

- This certificate is issued by a Certificate Authority (CA)—a trusted third party that vouches for the website’s identity.

- If the certificate checks out, the browser creates a secure, encrypted channel with the server.

**Analogy:** It’s like boarding a plane—your passport is the certificate, and the immigration officer is the CA. Without a verified passport, you’re not trusted to enter.

## 🤝 Automating Certificates with Let’s Encrypt

Manually buying, installing, and renewing certificates is tedious. This is where Let’s Encrypt comes in:

- It’s a free, automated, and open certificate authority.

- It provides SSL/TLS certificates that are recognized by all major browsers.

- Certificates typically expire every 90 days (for security reasons), but the renewal process can be automated.

We’ll use Certbot, the recommended shell client from Let’s Encrypt, to handle certificate issuance and renewal seamlessly.

## 🔄 Deep Dive: How SSL/TLS Handshakes Work

Before encrypted communication begins, your browser and the web server must **agree on how to talk securely**. This process is called the **SSL/TLS Handshake**.

Think of it like two strangers meeting for the first time and deciding how to communicate safely:

- **Hello Exchange**

    - **Browser (Client Hello):** “Hello, I’d like to connect securely. Here are the encryption methods I support.”
    - **Server (Server Hello):** “Great. I choose this method, and here’s my digital certificate proving who I am.”

- **Certificate Verification**

    - The browser checks the server’s certificate against trusted **Certificate Authorities (CAs)**.

    - If valid, the browser trusts it. If not, the browser shows a warning (“This connection is not secure”).

- **Key Exchange**

     - Both sides exchange or generate a **shared secret key**.

    - This key will be used to encrypt all communication.

    - Methods vary (RSA, Diffie-Hellman, ECDHE), but the goal is the same: ensure only the client and server can derive the key.

- **Handshake Completion**

    - Both sides confirm they are ready to use encryption.

    - From now on, every request (like fetching an HTML page) and every response (like sending back CSS or API data) is encrypted.

**🔑 Analogy:**
Imagine two people speaking in a crowded café. First, they agree on a secret language (encryption algorithm). Then, they privately exchange a dictionary (the session key). After that, even though the café is full of eavesdroppers, only the two can understand each other.

The beauty of TLS is that this all happens in milliseconds—invisible to the end user, but critical to web security.

## ⚖️ From Apache to NGINX: Why the Change?

Earlier in this project series, we configured an Apache HTTP Server (httpd) load balancer to distribute traffic across our 3 web servers, which shared an NFS and connected to a database server. Apache worked, but in this phase, we are switching to NGINX as the load balancer.

So, why NGINX?

### 1. Performance & Concurrency

- **Apache:** Uses a process/thread-per-connection model. Each new client connection requires spawning a process or thread, which consumes more CPU and memory. Under high traffic, this can become inefficient.
- **NGINX:** Uses an event-driven, asynchronous architecture. It can handle tens of thousands of concurrent connections using minimal resources, making it a better fit for high-traffic, modern web applications.

**👉 Real-life analogy:**
Apache is like hiring one waiter per customer in a restaurant. If 500 people walk in, you need 500 waiters—expensive and chaotic.
NGINX is like having a few highly efficient waiters who can serve multiple tables at once using smart scheduling.

### 2. Load Balancing Features

- NGINX offers built-in load balancing algorithms (Round Robin, Least Connections, IP Hash, etc.) with very low overhead.
- Advanced features like health checks, session persistence, and caching are easier to configure and more efficient compared to Apache’s `mod_proxy_balancer`.

### 3. Resource Efficiency

- NGINX is known for its low memory footprint, even under heavy load.
- Apache, while powerful, tends to consume more resources under scale.

### 4. Reverse Proxy Capabilities

NGINX was originally designed as a reverse proxy and load balancer. While Apache can do this via modules, NGINX was purpose-built, making it faster and more reliable in this role.

### 5. Modern Web Use Cases

- NGINX excels at handling static files, SSL termination, and HTTP/2 connections.

- It’s widely used in cloud-native and containerized environments (e.g., Kubernetes Ingress controllers).

### 📌 Summary Table
| Feature                 | Apache HTTP Server                      | NGINX                                 |
| ----------------------- | --------------------------------------- | ------------------------------------- |
| Connection Model        | Process/thread per connection           | Event-driven, async                   |
| Scalability             | Moderate                                | Very High                             |
| Resource Usage          | Higher                                  | Lower                                 |
| Load Balancing          | Available via `mod_proxy_balancer`      | Built-in, highly efficient            |
| Static Content Handling | Slower                                  | Very fast                             |
| Reverse Proxy Features  | Add-ons required                        | Native                                |
| Best Use Case           | Dynamic content (mod\_php, legacy apps) | Load balancing, high concurrency apps |

While Apache is still a great choice for serving dynamic content (like PHP apps), NGINX is the superior choice as a load balancer especially when performance, scalability, and efficiency matter. That’s why in this project, we’re upgrading our architecture to use NGINX for load balancing and SSL/TLS for secure communication.

---

# 🛠️ Project Overview

This project has two main tasks:

1. **Configure NGINX as a Load Balancer**
    - Distribute incoming traffic across multiple backend servers.
    - Improve performance, scalability, and reliability.

2. **Register a New Domain Name and Configure SSL/TLS**
    - Secure communication between users and your application.
    - Use Let’s Encrypt + Certbot for automated certificate management.

By the end, you’ll have a scalable, secure, and production-ready web solution.

![alt text](/images/z.png)

---

# ⚙️ Part 1 – Configure NGINX as a Load Balancer

In the previous section, we discussed why NGINX is more efficient than Apache for load balancing. Now, let’s set up NGINX as the traffic director for our web solution.

To reiterate Think of a **load balancer** like an **air traffic controller** at an airport:
- Planes (users’ requests) arrive from all directions.
- The controller (NGINX) decides which runway (web server) each plane should land on.
- This prevents congestion on a single runway, keeps traffic moving smoothly, and ensures no server gets overwhelmed.

## 🖥️ Step 1: Provision a New EC2 Instance

We can either:
- Uninstall Apache from the existing Load Balancer server, or
- Provision a new Linux VM dedicated for NGINX.

Here, we’ll create a fresh EC2 instance:
- AMI: Ubuntu Server 20.04 LTS
- Name: Nginx-LB
- Security Group: Allow inbound traffic on
    - TCP Port 80 (HTTP)
    - TCP Port 443 (HTTPS)

![alt text](/images/1.png)
- SSH into the instance
![alt text](/images/2.png)

## 📌 Step 2: Install NGINX

Update system packages and install NGINX:
```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install nginx -y
```
![alt text](/images/3.png)
![alt text](/images/4.png)
Verify installation:
```bash
sudo systemctl status nginx
```
![alt text](/images/5.png)
If active, you should see:
```arduino
Active: active (running)
```

## 📌 Step 3: Configure Local DNS Resolution

To simplify NGINX’s configuration, we’ll map our web servers’ private IPs to hostnames (Web1, Web2, Web3) inside /etc/hosts.
```bash
sudo nano /etc/hosts
```
![alt text](/images/6a.png)
Example entries:
```
10.0.1.101   Web1
10.0.1.102   Web2
10.0.1.103   Web3
```
![alt text](/images/6b.png)

This allows us to reference servers by name (Web1, Web2, Web3) instead of hardcoding IP addresses—making the setup more flexible and maintainable.

> 👉 Analogy: Like saving your friends’ numbers under names in your phonebook. Instead of remembering “+44 7000…”, you just call Alice.


## 📌 Step 4: Configure NGINX as a Load Balancer

Open the main configuration file:
```bash
sudo nano /etc/nginx/nginx.conf
```
![alt text](/images/7a.png)
Inside the http { } block, add an upstream block and configure the server section:
```nginx
http {
    upstream myproject {
        server Web1 weight=5;
        server Web2 weight=5;
        server Web3 weight=5;
    }

    server {
        listen 80;
        server_name www.domain.com;

        location / {
            proxy_pass http://myproject;
        }
    }

    # Comment out the line below
    # include /etc/nginx/sites-enabled/*;
}
```
![alt text](/images/7b.png)

**Explanation:**
- upstream myproject {} – Defines a pool of backend servers (Web1, Web2).
- weight=5 – Assigns equal weight to both servers (requests will be balanced evenly).
- proxy_pass http://myproject; – Routes incoming traffic to the backend pool.

>👉 Analogy: Imagine a call center. Incoming calls (requests) go to whichever available agent (web server) is ready. The weight parameter controls how many calls each agent gets.

## 📌 Step 5: Validate Configuration

Test the syntax of your configuration:
```bash
sudo nginx -t
```
![alt text](/images/8.png)
If successful, reload NGINX to apply changes:
```bash
sudo systemctl reload nginx
```
Check service status:
```bash
sudo systemctl status nginx
```
![alt text](/images/9.png)

## 📌 Step 6: Verify Load Balancing

Run the following commands to ensure requests are being distributed across the web servers:
```bash
curl http://Web1
curl http://Web2
curl http://Web3
curl -I http://<Public-IP-of-Nginx-LB>
```
![alt text](/images/10.png)
You should see different responses based on which backend serves the request.

👉 Tip: If you refresh multiple times, you’ll notice NGINX alternating between Web1,Web2 and Web3 responses.

## ✅ Step 7: Restart and Confirm

Finally, restart the NGINX service to ensure all settings persist:
```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

## 🎯 Result

At this stage, we now have:
- An NGINX Load Balancer distributing requests evenly between multiple web servers.
- Improved scalability and redundancy (if one web server fails, the other continues serving traffic).
- A cleaner and more efficient setup compared to our earlier Apache-based load balancer.

Next, we will move to Part 2: Registering a Domain Name and Configuring SSL/TLS to secure our connections with HTTPS.

----

# 🔐 Part 2 – Register a Domain Name and Configure SSL/TLS Certificates

In Part 1, we configured an NGINX Load Balancer to distribute traffic across our web servers. However, scalability alone is not enough, security is equally important.

Modern users expect that any website they visit is secure (HTTPS), not insecure (HTTP). To achieve this, we need to configure SSL/TLS certificates for encrypted communication.

> 👉 Remember: SSL/TLS prevents man-in-the-middle attacks by ensuring that data exchanged between your browser and the server is private and trustworthy.

## 🏷️ Step 1: Register a Domain Name

You must first purchase a domain name. Some popular registrars include:
- GoDaddy
- Domain.com
- Bluehost

Here we used Duck DNS subdomain

📌 Example: You register mytoolingsolution.com
![alt text](/images/11.png)

## 🌍 Step 2: Assign an Elastic IP

- Allocate an Elastic IP (EIP) in AWS.
- Associate this Elastic IP with your NGINX Load Balancer EC2 instance.

![alt text](/images/12.png)
![alt text](/images/13.png)
![alt text](/images/14.png)

This ensures your domain always points to the same static IP, even if the instance restarts.

## 📌 Step 3: Update DNS Records

In your domain registrar’s dashboard:

- Create an A record pointing your domain name (e.g., mytoolingsolution.com) to the Elastic IP.
![alt text](/images/15.png)
- After propagation (may take a few minutes to hours), verify by visiting:
```text
http://mytoolingsolution.com
```
![alt text](/images/16.png)
If configured correctly, you should see traffic being served via NGINX Load Balancer over HTTP.

## ⚙️ Step 4: Update NGINX to Recognize Your Domain

Edit the NGINX configuration:
```bash
sudo nano /etc/nginx/nginx.conf
```
![alt text](/images/17a.png)
Update your server_name directive:
```nginx
server {
    listen 80;
    server_name www.mytoolingsolution.com mytoolingsolution.com;

    location / {
        proxy_pass http://myproject;
    }
}
```
![alt text](/images/17b.png)
Reload NGINX to apply changes:
```bash
sudo systemctl reload nginx
```
![alt text](/images/18.png)

##  🔑 Step 5: Install Certbot and Request SSL/TLS Certificate

Let’s use Let’s Encrypt, a free and trusted Certificate Authority (CA), to issue our SSL/TLS certificate.

1. Ensure **snapd** is active:
    ```bash
    sudo systemctl status snapd
    ```
    ![alt text](/images/19.png)
2. Install Certbot:
    ```bash
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```
    ![alt text](/images/20.png)
3. Run Certbot with NGINX integration:
    ```bash
    sudo certbot --nginx
    ```
    ![alt text](/images/21.png)
👉 Certbot will automatically:
- Verify your domain with Let’s Encrypt.
- Install the certificate into NGINX.
- Configure automatic HTTPS redirection.

## ✅ Step 6: Test Secure HTTPS Access

Now, visit:
```text
https://mytoolingsolution.com
```
![alt text](/images/22.png)
![alt text](/images/23.png)
- You should see a padlock icon in your browser’s address bar.
- Click the padlock → view certificate details → confirm that Let’s Encrypt issued it.
    ![alt text](/images/26.png)

## 🔄 Step 7: Automate Certificate Renewal

Let’s Encrypt certificates are valid for 90 days. Best practice is to renew every 60 days.

Test renewal:
```bash
sudo certbot renew --dry-run
```
![alt text](/images/24.png)
Set up an automated cronjob to renew certificates twice daily:
```bash
sudo crontab -e
```
![alt text](/images/25a.png)
Add this line:
```bash
* */12 * * * root /usr/bin/certbot renew > /dev/null 2>&1
```
![alt text](/images/25b.png)

## 🎯 Result

At this point, we now have:

- A registered domain name pointing to our NGINX Load Balancer.
- A valid SSL/TLS certificate automatically issued and installed via Let’s Encrypt.
- Automatic renewal configured to maintain security without manual intervention.

This ensures that all user traffic is served securely via HTTPS, building trust and protecting data in transit.

# ✅ Conclusion

By registering a domain name, mapping it to your Elastic IP, and securing it with SSL/TLS certificates from Let’s Encrypt, we have successfully transitioned our Tooling Web Solution from plain HTTP to fully encrypted HTTPS. This ensures that:

- All communication between clients and the web servers is encrypted and protected against eavesdropping or tampering.
- Our Nginx Load Balancer is not only distributing traffic efficiently but also enforcing modern web security standards.
- With automated renewal via certbot and cron, our certificates stay valid without manual intervention, ensuring long-term reliability.

In summary, we now have a scalable, secure, and production-ready infrastructure where Nginx serves as both the load balancer and the TLS termination point, providing a solid foundation for our web applications.