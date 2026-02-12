# Certbot Previder DNS

This guide explains how to use **Certbot DNS validation** in combination with **Previder DNS**.

This guide assumes you are using Certbot installed via **snap** (Python installation is also possible but not covered here).

---

## Step 1 – Install snapd

Follow the official Snap installation instructions:

https://snapcraft.io/docs/installing-snapd/

---

## Step 2 – Remove certbot-auto and OS Certbot packages

If you have installed Certbot packages via your operating system’s package manager (such as `apt`, `dnf`, or `yum`), you must remove them before installing the Certbot snap version.

This ensures that when running the `certbot` command, the snap version is used instead of the OS package version.

Common examples:

```bash
sudo apt-get remove certbot
```

```bash
sudo dnf remove certbot
```

```bash
sudo yum remove certbot
```

---

## Step 3 – Install Certbot

Run the following commands on the machine to install Certbot and the DNS plugin:

```bash
sudo snap install --classic certbot
sudo snap install certbot-dns-multi
sudo snap set certbot trust-plugin-with-root=ok
sudo snap connect certbot:plugin certbot-dns-multi
```

---

## Step 4 – Generate an API key

A **Previder Portal [user](https://portal.previder.nl/#/user/current/tokens) or [application](https://portal.previder.nl/#/application) token** is required to modify DNS records and request certificates.

The token must have the following privileges:

- **Read** on Domains  
- **Read/Update** on DNS for the client  

This token will be visible to anyone who has access to the deployment.  
Create a dedicated role with only the required privileges for secure operation.

---

## Step 5 – Configure Previder DNS

Create the following file:

```
sudo vi /etc/letsencrypt/dns-multi.ini
```

Add the following configuration and insert the generated API key:

```ini
dns_multi_provider = pdns
PDNS_API_URL = https://portal.previder.com
PDNS_API_KEY = YOUR_API_KEY_HERE
PDNS_API_VERSION = 1
PDNS_SERVER_NAME = previder
PDNS_PROPAGATION_TIMEOUT = 800
PDNS_POLLING_INTERVAL = 5
PDNS_TTL = 300
```

Replace `YOUR_API_KEY_HERE` with your actual API key.

---

## Step 6 – Set correct file permissions

Secure the credentials file:

```bash
sudo chmod 600 /etc/letsencrypt/dns-multi.ini
```

---

## Step 7 – Test the configuration (dry run)

Run a dry run to test if everything works correctly:

```bash
certbot certonly -a dns-multi \
  --dns-multi-credentials=/etc/letsencrypt/dns-multi.ini \
  -d yourdomain.com \
  -d "*.yourdomain.com" \
  --dry-run
```

---

## Step 8 – Request the certificate

If the dry run is successful, request the actual certificate:

```bash
certbot certonly -a dns-multi \
  --dns-multi-credentials=/etc/letsencrypt/dns-multi.ini \
  -d yourdomain.com \
  -d "*.yourdomain.com"
```
