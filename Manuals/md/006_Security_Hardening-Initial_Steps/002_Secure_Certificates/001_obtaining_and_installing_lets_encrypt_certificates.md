# Obtaining and Installing Let's Encrypt Certificates

## Contents

- [Obtaining and Installing Let's Encrypt Certificates](#obtaining-and-installing-lets-encrypt-certificates)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Let's Encrypt and Certbot](#understanding-lets-encrypt-and-certbot)
  - [Installing Certbot](#installing-certbot)
  - [Obtaining a Let's Encrypt Certificate](#obtaining-a-lets-encrypt-certificate)
    - [Certbot Challenges](#certbot-challenges)
      - [HTTP-01 Challenge](#http-01-challenge)
        - [Acme challenge](#acme-challenge)
        - [Standalone Mode](#standalone-mode)
      - [DNS-01 Challenge](#dns-01-challenge)
    - [Using the HTTP-01 Challenge](#using-the-http-01-challenge)
    - [Locating Your Obtained Certificates](#locating-your-obtained-certificates)
  - [Installing the Certificate on a Web Server](#installing-the-certificate-on-a-web-server)
    - [Apache HTTP Server](#apache-http-server)
    - [Nginx](#nginx)
  - [Auto-renewal of Certificates](#auto-renewal-of-certificates)

## Introduction

### Understanding Let's Encrypt and Certbot

Transport Layer Security (TLS) and its predecessor, Secure Sockets Layer (SSL), are cryptographic protocols that provide secure communication over a network. Digital certificates serve as electronic credentials that verify the identity of a website and establish an encrypted connection. These certificates are essential for securing web server communications, ensuring data privacy and integrity.

**Let's Encrypt** is a non-profit Certificate Authority (CA) that provides TLS/SSL certificates at no cost. Its mission is to make secure, encrypted connections the default standard across the internet.

**Certbot** is a command-line tool designed to automate the process of obtaining and installing Let's Encrypt certificates on web servers. It handles the necessary steps of certificate issuance and configuration, simplifying what can otherwise be a complex task, especially for new system administrators working in command-line environments.

## Installing Certbot

Debian-based Linux distributions commonly use a package management system called `apt`. This system allows you to easily install, upgrade, and remove software. Certbot is available as a package that you can install using `apt`.

Here's the command you'll need to run in your terminal:

```bash
sudo apt update
sudo apt install certbot
```

Let's break this down:

- `sudo`: This command allows you to run the following command with administrative privileges, which are necessary for installing software. You'll likely be prompted to enter your password.
- `apt update`: This command refreshes the package lists, ensuring you have the latest information about available software. It's always a good practice to run this before installing new packages.
- `apt install certbot`: This command tells `apt` to download and install the `certbot` package and any software it depends on.

Once you run these commands, `apt` will handle the installation process for you.

## Obtaining a Let's Encrypt Certificate

Before Let's Encrypt issues a certificate, it needs to verify that you actually own the domain for which you're requesting the certificate. This process is done through what are called "challenges." There are a couple of common types of challenges: HTTP-01 and DNS-01.

For this initial lesson, we'll focus on the **HTTP-01 acme challenge**.

Here are some common challenge types:

### Certbot Challenges

#### HTTP-01 Challenge

##### Acme challenge

1. When you request a certificate for your domain (let's say `yourdomain.com`), Let's Encrypt gives your Certbot software a unique token.
2. Your Certbot then creates a temporary file with this token (and a special "key" of its own) and places it in a specific location on your web server: `.well-known/acme-challenge/` within the webroot directory of yourdomain.com.
3. Let's Encrypt's servers then make a request to `http://yourdomain.com/.well-known/acme-challenge/<your_token>` to see if the file with the correct content is there.
4. If Let's Encrypt can successfully access this file and verify its contents, it confirms that you control the domain and issues the certificate.
5. Once the certificate is issued, Certbot cleans up this temporary challenge file from your web server.

**The key prerequisite for using the HTTP-01 acme challenge is that you must have a web server (like Nginx or Apache) already running and accessible on port 80 (the standard HTTP port) for the domain you want to secure.**

##### Standalone Mode

Certbot can handle the HTTP-01 challenge without relying on an already running web server. This is often called the **standalone mode**.

Here's how that process works:

1. When you run the Certbot command with the standalone option, Certbot itself temporarily spins up a minimal web server on your server. By default, it listens on port 80.
2. Just like the regular HTTP-01 challenge, Let's Encrypt provides Certbot with a unique token for your domain.
3. Certbot's temporary web server serves the required challenge file (with the token and key) at the expected location: `http://yourdomain.com/.well-known/acme-challenge/<your_token>`.
4. Let's Encrypt's servers then connect to this temporary web server on port 80 and request the challenge file to verify its content.
5. If the verification is successful, Let's Encrypt issues the certificate.
6. Once the certificate is obtained, Certbot shuts down its temporary web server.

The key advantage here is that **you don't need to have your primary web server (like Nginx or Apache) running on port 80 during the certificate issuance process**. This can be useful if your web server is not yet fully configured or if you need to obtain a certificate before deploying your main web server.

To use this method, you would typically include the `--standalone` flag in your Certbot command.

Keep in mind that while this is convenient, **it does mean that port 80 must be free and not in use by any other application** on your server when you run the Certbot command in standalone mode.

#### DNS-01 Challenge

The **DNS-01** challenge is another way Let's Encrypt can verify that you own the domain for which you're requesting a certificate. Instead of relying on a running web server, it uses the Domain Name System (DNS). Here's how it generally works:

1. When you request a certificate for your domain (again, let's say `yourdomain.com`), Let's Encrypt gives your Certbot software a unique token.
2. Your Certbot then instructs you to create a specific DNS TXT record under the subdomain `_acme-challenge.yourdomain.com` with the value of this token.
3. Let's Encrypt's servers then query the DNS records for `_acme-challenge.yourdomain.com`.
4. If they find the correct TXT record with the expected value, it confirms that you control the DNS for the domain and issues the certificate.
5. Once the certificate is issued, you can remove the DNS TXT record (though it doesn't hurt to leave it, it won't be checked again for the same issuance).

The main advantage of the DNS-01 challenge is that **it doesn't require a running web server on port 80**. This can be useful in situations where you might not have a web server running yet, or if you're obtaining a certificate for a subdomain that doesn't have its own web server.

However, the DNS-01 challenge typically requires you to have **programmatic access to your DNS records** (often through an API provided by your DNS registrar) so that Certbot can automatically create and then remove the TXT record. Manually creating and deleting DNS records for each certificate issuance and renewal can be cumbersome.

### Using the HTTP-01 Challenge

To obtain a Let's Encrypt certificate using the HTTP-01 challenge, you'll use the `certbot certonly` command. The `certonly` part tells Certbot that you only want to obtain the certificate and not have it automatically configure your web server (we'll do that manually in the next step for better understanding).

Here's the basic command structure you'll use:

```bash
sudo certbot certonly --webroot -w /var/www/yourdomain.com/html -d yourdomain.com
```

Let's break down this command:

- `sudo certbot`: This invokes the Certbot tool with administrative privileges.
- `certonly`: As mentioned, this tells Certbot to only obtain the certificate.
- `--webroot`: This specifies that you want to use the webroot plugin for the HTTP-01 challenge. The webroot plugin works by creating the temporary challenge file in a directory served by your web server.
- `-w /var/www/yourdomain.com/html`: This is the webroot path. You need to replace `/var/www/yourdomain.com/html` with the actual path to the root directory of your website on your server. This is where Certbot will create the `.well-known/acme-challenge/` directory and the challenge file.
- `-d yourdomain.com`: This specifies the domain name for which you want to obtain the certificate. If you have multiple domains or subdomains (e.g., `yourdomain.com` and `www.yourdomain.com`), you can include them with additional `-d` flags:

```bash
sudo certbot certonly --webroot -w /var/www/yourdomain.com/html -d yourdomain.com -d www.yourdomain.com
```

**Important Considerations**:

- **Replace the webroot path and domain name(s) with your actual values**. Incorrect paths will lead to the challenge failing.
- Ensure that your web server is configured to serve files from the specified webroot path for the domain(s) you are requesting the certificate for.
- Certbot will guide you through the process and might ask for an email address for renewal reminders and agreement to the Let's Encrypt terms of service.

### Locating Your Obtained Certificates

Once you successfully run the certbot certonly command, Let's Encrypt will issue the certificate for your domain. Certbot will then save these important certificate files on your server.

It's crucial to know where these files are located. By default, Certbot stores all the certificates and related keys within the `/etc/letsencrypt/` directory.

Here's a breakdown of the key files you'll find for your domain (e.g., `yourdomain.com`):

- `/etc/letsencrypt/live/yourdomain.com/privkey.pem`: This is the **private key** for your certificate. **Keep this file secure and do not share it with anyone**. If someone gains access to your private key, they can potentially impersonate your website.
- `/etc/letsencrypt/live/yourdomain.com/cert.pem`: This is the **actual TLS/SSL certificate** for your domain. This is the file your web server will use to identify itself.
- `/etc/letsencrypt/live/yourdomain.com/chain.pem`: This file contains the **Let's Encrypt certificate authority (CA) certificate(s)**, which help clients (like web browsers) verify the authenticity of your certificate.
- `/etc/letsencrypt/live/yourdomain.com/fullchain.pem`: This is a convenience file that combines `cert.pem` and `chain.pem`. Most web server configurations will ask you to point to this file for the certificate.

The `live` directory contains symbolic links to the actual certificate files in the `archive` directory. This allows Certbot to update certificates without changing the paths your web server configuration uses.

## Installing the Certificate on a Web Server

Now that you know where the certificate files are located, let's see how to configure your web server to use them and enable HTTPS for your website. Here are examples for **Apache HTTP Server** and **Nginx**:

### Apache HTTP Server

You'll typically need to edit the virtual host configuration file for your domain. These files are usually located in directories like `/etc/apache2/sites-available/` (and enabled in `/etc/apache2/sites-enabled/`). The filename might be something like `yourdomain.com.conf` or `000-default.conf`.

Here's a basic example of how you might configure your Apache virtual host for HTTPS:

First, ensure that the `ssl` module is enabled in Apache. You can usually do this with the following command:

```bash
sudo a2enmod ssl
```

Then, edit your virtual host configuration file. You might have a separate virtual host block for port 443 (HTTPS), or you might need to add the SSL configuration within your existing port 80 block. Here's an example of a dedicated HTTPS virtual host:

```Apache
<VirtualHost *:443>
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    DocumentRoot /var/www/yourdomain.com/html

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/yourdomain.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/yourdomain.com/privkey.pem

    # Optional: Intermediate Certificate Authority (CA) bundle
    SSLCACertificateFile /etc/letsencrypt/live/yourdomain.com/chain.pem

    <Directory /var/www/yourdomain.com/html>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# Optional: Redirect HTTP to HTTPS
<VirtualHost *:80>
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    Redirect permanent / https://yourdomain.com/
</VirtualHost>
```

Let's break down the key parts related to the certificate for Apache:

- `<VirtualHost *:443>`: This directive defines a virtual host that listens on port 443 for HTTPS connections.
- `SSLEngine on`: This enables the SSL/TLS engine for this virtual host.
- `SSLCertificateFile /etc/letsencrypt/live/yourdomain.com/fullchain.pem`: This line specifies the path to the `fullchain.pem` file (your certificate plus the intermediate certificates).
- `SSLCertificateKeyFile /etc/letsencrypt/live/yourdomain.com/privkey.pem`: This line specifies the path to your private key file.
- `SSLCACertificateFile /etc/letsencrypt/live/yourdomain.com/chain.pem` **(Optional but recommended)**: This line points to the chain certificate file, which can help older browsers properly verify your certificate.

After making these changes to your Apache virtual host configuration file, you'll need to **save the file** and then **restart or reload the Apache service** for the changes to take effect. You can usually do this with the following commands:

```bash
sudo apachectl configtest  # Test the configuration for any syntax errors
sudo systemctl restart apache2
```

Now your website should also be accessible via `https://yourdomain.com` if you are using Apache.

### Nginx

To enable HTTPS in your Nginx configuration, you'll need to edit the server block for your domain. This is typically found in a configuration file within the `/etc/nginx/sites-available/` directory (and often linked to `/etc/nginx/sites-enabled/`). The filename usually corresponds to your domain name (e.g., `yourdomain.com`).

Here's a basic example of how you might configure your Nginx server block for HTTPS:

```Code snippet
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # Additional SSL settings (recommended)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE+AESGCM:CHACHA20';
    ssl_ecdh_curve secp384r1;
    ssl_session_timeout 10m;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;

    root /var/www/yourdomain.com/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Let's break down the important parts related to the certificate:

- `listen 443 ssl;`: This line tells Nginx to listen for HTTPS connections on port 443 (the standard HTTPS port) and that SSL/TLS should be enabled for this server block.
- `ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;`: This line specifies the path to the `fullchain.pem` file, which contains both your certificate and the necessary intermediate certificates.
- `ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;`: This line specifies the path to your private key file.

After making these changes to your Nginx configuration file, you'll need to **save the file** and then **reload or restart the Nginx service** for the changes to take effect. You can usually do this with the following commands:

```bash
sudo nginx -t  # Test the configuration for any syntax errors
sudo systemctl reload nginx
```

Once Nginx reloads successfully, your website should be accessible via `https://yourdomain.com`.

## Auto-renewal of Certificates

Let's Encrypt certificates are only valid for **90 days**. This relatively short lifespan encourages good security practices by prompting you to regularly update your certificates. Thankfully, Certbot makes this renewal process very easy and can even automate it for you.

When you installed Certbot, it likely set up a systemd timer (on modern Ubuntu systems) or a cron job (on older systems) that automatically runs a renewal check twice a day. This process will attempt to renew any certificates that are within their expiry window (typically 30 days).

To check if the automatic renewal is configured, you can run the following command:

```bash
sudo systemctl list-timers | grep certbot
```

If you see an active timer related to `certbot.timer`, then automatic renewal is likely set up correctly.

Alternatively, you can check for a cron job by running:

```bash
crontab -l
```

and look for a line that includes certbot renew.

**To test the renewal process without actually renewing your certificates (since they are likely still valid), you can use the following command**:

```bash
sudo certbot renew --dry-run
```

This command simulates the renewal process and will show you if any errors occur. It's a good practice to run this periodically to ensure everything is working as expected.

If the dry run is successful, you can be confident that your certificates will be automatically renewed before they expire, keeping your website secure without any manual intervention.
