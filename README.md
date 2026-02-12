Certbot DNS Validation with Previder DNS

This guide explains how to use Certbot DNS validation in combination with Previder DNS.

We assume you are using Certbot installed via snap.

Step 1 – Install snapd

Follow the official Snap installation instructions:

https://snapcraft.io/docs/installing-snapd/

Step 2 – Remove Existing Certbot Installations

If you previously installed Certbot using your operating system’s package manager (such as apt, dnf, or yum), you must remove those packages before installing the snap version.

This ensures that when you run the certbot command, the snap version is used instead of the OS package version.

Common examples:

sudo apt-get remove certbot

sudo dnf remove certbot

sudo yum remove certbot

Step 3 – Install Certbot and DNS Plugin

Run the following commands on your system:

sudo snap install --classic certbot
sudo snap install certbot-dns-multi
sudo snap set certbot trust-plugin-with-root=ok
sudo snap connect certbot:plugin certbot-dns-multi

Step 4 – Generate a Previder API Key

To modify DNS records and request certificates, you need a Previder Portal user token or application token.

Required Permissions

The token must have:

Read access on Domains

Read/Update access on DNS for the client

The token will be visible to anyone who has access to the deployment. Create a dedicated role with only the required permissions.

Step 5 – Configure Previder DNS

Create the following file:

/etc/letsencrypt/dns-multi.ini


Add the following configuration:

dns_multi_provider = pdns
PDNS_API_URL = https://portal.previder.com
PDNS_API_KEY = YOUR_API_KEY_HERE
PDNS_API_VERSION = 1
PDNS_SERVER_NAME = previder
PDNS_PROPAGATION_TIMEOUT = 800
PDNS_POLLING_INTERVAL = 5
PDNS_TTL = 300


Replace YOUR_API_KEY_HERE with the API key generated in Step 4.

Set correct file permissions:

sudo chmod 600 /etc/letsencrypt/dns-multi.ini

Requesting a Certificate (Example)
sudo certbot certonly \
  --dns-multi \
  --dns-multi-credentials /etc/letsencrypt/dns-multi.ini \
  -d example.com \
  -d '*.example.com'


Replace example.com with your actual domain.

Notes

DNS propagation timeout is set to 800 seconds.

Adjust PDNS_PROPAGATION_TIMEOUT if DNS validation fails due to slow propagation.

Wildcard certificates require DNS validation.
