# Run A Private, LAN Access Only, Instance Of Vaultwarden On A Synology NAS With Docker-Compose And Cloudflare DNS

## Description

This repository will provide you the details to run a private, LAN access only, instance of Vaultwarden, using docker-compose, on a Synology NAS.  The setup will be using the Caddy web server that will obtain a Let's Encrypt certificate for a subdomain that is setup through Cloudflare DNS.

Because Synology NASs use both ports 80 and 443 for DSM we'll need to use a custom port number when accessing Vaultwarden.  The url to access Vaultwarden would look something like https://vw220.digitalaloha.net:8443.  See step 8 in the Setup Steps below to set the custom ports.

I referenced the following links for much of the setup description below.

* [Running a private vaultwarden instance with Let’s Encrypt certs](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let%27s-Encrypt-certs)
* [Docker-compose Caddy with DNS challenge](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose#caddy-with-dns-challenge)

## Directions

### YouTube Video
If you would like directions in visual format then check out my YouTube video that runs through the setup.
* [Run Your Own Private Vaultwarden Instance With Docker-Compose, Git, And Cloudflare DNS On Your Synology NAS](Video to come)

### Prerequistes
On your Synology NAS you'll need to do the following:
1. Install the Docker package.
2. Install the Git Server package.
3. Enable SSH.

### Setup Steps 
1. SSH into your Synology NAS.
```
ssh <admin account>@<IP address of Synology NAS>
```
2. Change directory into /volume1/docker. 
```
cd /volume1/docker
```
4. Clone this repository.
```
sudo git clone https://github.com/digtalaloha/synology-private-vaultwarden-cloudflare.git
```
5. Change directory into the newly created synology-private-vaultwarden-cloudflare directory.
```
cd synology-private-vaultwarden-cloudflare
```
6. Create host volume mount points that will be used by the Vaultwarden and Caddy containers.
```
mkdir vw-data caddy-config caddy-data
```
7. Go to [cloudflare.com](https://www.cloudflare.com) and login to your account (Assumption - You already have a domain configured with the nameservers assigned by Cloudflare).
   * First we'll setup a subdomain that will be used to access Vaultwarden.
      1. Select an active domain you would like to use.
      2. Select the DNS listing then click Add record.
      3. Enter or select the following:
         * Type - A.
         * Name - Name you would like to use (example - enter vw220 if you would like to use vw220.digitalaloha.net).
         * IPv4 address - Enter in the private IP address of your Synology NAS (example 10.0.4.43).
         * Proxy status - Toggle off.
         * Click Save.
   * Next we'll need to create an API Token.
      1. Click on the person icon (upper right corner), select My Profile then select the API Tokens listing.
      2. Click Create Token.
      3. Click Use Template for the Edit zone DNS template and do the following:
         * Token name: Edit it as you wish (example ACME DNS challenge for vw220).
         * Under Permissions, leave the Zone/DNS/Edit listing as is.
         * Under Permissions, click + Add more and populate the boxes from left to right with Zone/Zone/Read.
         * Under Zone Resources, from left to right select Include/Specific zone/CLOUDFLARE_DOMAIN_YOU_ARE_USING.
         * Under TTL, set an End Date for a date far into the future (this is when the token that will be generated will expire).
         * Click Continue to summary.
         * Click Create Token then copy and save the API Token that was generated (it will be used later).
8. Edit the .env file with the specifics of your setup.  
   * Note - port 8080 and 8443 are the host (Synology NAS) ports that will be mapped to the Caddy ports 80 and 443 respectively.  These are the ports I used in my setup.
   * These entries will be used to auto populate the docker-compose.yaml file, which will in turn auto populate the Caddyfile.
```
CADDY_HTTP_PORT=8080
CADDY_HTTPS_PORT=8443
CLOUDFLARE_DOMAIN="https://YOUR_CLOUDFLARE_SUB_DOMAIN"
EMAIL="YOUR_EMAIL_ADDRESS"
CLOUDFLARE_TOKEN="YOUR_CLOUDFLARE_DNS_API_TOKEN"
```
13. Create and start up Vaultwarden with the Caddy web server.
```
sudo docker-compose up -d
```
14. You should now be able to access your private, LAN access only Vaultwarden docker container.
```
Go to https://YOUR_CLOUDFLARE.DOMAIN.NAME:CADDY_HTTPS_PORT (Example - https://vs220.digitalaloha.net:8443)
```
