# Setting Up a Secure Local Server with DuckDNS and Nginx Proxy Manager

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setting Up DuckDNS](#setting-up-duckdns)
- [Installing Nginx Proxy Manager](#installing-nginx-proxy-manager)
- [Configuring Nginx Proxy Manager](#configuring-nginx-proxy-manager)
- [Setting Up SSL Certificates](#setting-up-ssl-certificates)
- [Installing CasaOS](#installing-casaos)
- [Configuring CasaOS](#configuring-casaos)
- [Connecting Everything Together](#connecting-everything-together)
- [Troubleshooting](#troubleshooting)
- [Conclusion](#conclusion)

## Introduction
[⬆️ Back to Top](#table-of-contents)

This guide will walk you through setting up a secure local server using DuckDNS for free domain registration, Nginx Proxy Manager for handling connections, and CasaOS as your server operating system. By the end of this tutorial, you'll have a fully functional secure server with SSL that you can access remotely.

## Prerequisites
[⬆️ Back to Top](#table-of-contents)

Before starting, ensure you have:
- A computer or Raspberry Pi to act as your server
- A stable internet connection
- Basic understanding of networking concepts
- Router access to configure port forwarding

## Setting Up DuckDNS
[⬆️ Back to Top](#table-of-contents)

DuckDNS is a free service that allows you to create a custom domain name that points to your home IP address.

1. **Create a DuckDNS account**
   - Go to [DuckDNS website](https://www.duckdns.org/)
   - Sign in using your preferred method (GitHub, Twitter, etc.)

2. **Create a domain**
   - Enter your desired subdomain name (e.g., myserver)
   - Click "Add domain"
   - Your domain will be: `myserver.duckdns.org`

3. **Save your token**
   - After creating your domain, you'll see a token
   - Save this token somewhere secure as you'll need it later

4. **Update your IP address**
   - To ensure DuckDNS stays updated with your current IP, you'll need to set up a periodic update
   - You can do this by creating a cron job or using a Docker container

Example cron job command:
```bash
echo "*/5 * * * * echo url=\"https://www.duckdns.org/update?domains=YOURDOMAIN&token=YOURTOKEN&ip=\" | curl -k -o ~/duckdns/duck.log -K -" | crontab -
```

## Installing Nginx Proxy Manager
[⬆️ Back to Top](#table-of-contents)

Nginx Proxy Manager provides a simple way to manage Nginx as a reverse proxy with SSL termination.

1. **Install Docker and Docker Compose**
   ```bash
   sudo apt update
   sudo apt install docker.io docker-compose -y
   ```

2. **Create a directory for Nginx Proxy Manager**
   ```bash
   mkdir -p ~/nginx-proxy-manager
   cd ~/nginx-proxy-manager
   ```

3. **Create a Docker Compose file**
   ```bash
   nano docker-compose.yml
   ```

4. **Add the following content to the file**
   ```yaml
   version: '3'
   services:
     app:
       image: 'jc21/nginx-proxy-manager:latest'
       restart: unless-stopped
       ports:
         - '80:80'
         - '81:81'
         - '443:443'
       volumes:
         - ./data:/data
         - ./letsencrypt:/etc/letsencrypt
   ```

5. **Start Nginx Proxy Manager**
   ```bash
   docker-compose up -d
   ```

6. **Access the web interface**
   - Open your browser and navigate to `http://YOUR_SERVER_IP:81`
   - The default login credentials are:
     - Email: admin@example.com
     - Password: changeme
   - You'll be prompted to change these credentials on first login

## Configuring Nginx Proxy Manager
[⬆️ Back to Top](#table-of-contents)

1. **Add a new proxy host**
   - Click on "Hosts" > "Proxy Hosts" > "Add Proxy Host"
   - Enter your DuckDNS domain (e.g., myserver.duckdns.org)
   - Set the scheme to http
   - Enter your local server IP and port (e.g., 192.168.1.100 and port 80)
   - Click "Save"

2. **Set up port forwarding on your router**
   - Log into your router's admin interface
   - Find the port forwarding section
   - Forward ports 80 and 443 to your server's IP address

## Setting Up SSL Certificates
[⬆️ Back to Top](#table-of-contents)

1. **Edit your proxy host**
   - Click on the three dots next to your proxy host
   - Click "Edit"
   - Go to the "SSL" tab

2. **Request a Let's Encrypt certificate**
   - Select "Request a new SSL Certificate"
   - Choose "Let's Encrypt"
   - Enter your email address
   - Check both "I Agree" boxes
   - Enable "Force SSL" if you want to redirect all HTTP traffic to HTTPS
   - Click "Save"

3. **Verify certificate**
   - Your certificate should now be issued
   - You can verify by visiting your domain with https:// prefix

## Installing CasaOS
[⬆️ Back to Top](#table-of-contents)

CasaOS is a simple, easy-to-use home cloud system that turns your hardware into a home server.

1. **Install CasaOS**
   ```bash
   curl -fsSL https://get.casaos.io | sudo bash
   ```

2. **Access CasaOS**
   - Open your browser and navigate to `http://YOUR_SERVER_IP:80`
   - Complete the initial setup wizard
   - Create your admin account

## Configuring CasaOS
[⬆️ Back to Top](#table-of-contents)

1. **Configure network settings**
   - Go to "System" > "Network"
   - Make sure your server has a static IP address

2. **Install applications**
   - Go to "App Store"
   - Browse and install the applications you need

3. **Set up users**
   - Go to "System" > "Users"
   - Create accounts for anyone who will be accessing your server

## Connecting Everything Together
[⬆️ Back to Top](#table-of-contents)

1. **Create a proxy host for CasaOS**
   - Go back to Nginx Proxy Manager
   - Add a new proxy host
   - Domain: `casaos.yourdomain.duckdns.org`
   - Scheme: http
   - Forward Hostname/IP: YOUR_SERVER_IP
   - Forward Port: 80 (or the port CasaOS is running on)
   - Request a new SSL certificate as done previously

2. **Test the connection**
   - Open your browser and navigate to `https://casaos.yourdomain.duckdns.org`
   - You should be redirected to the CasaOS login page
   - Log in with your credentials

## Troubleshooting
[⬆️ Back to Top](#table-of-contents)

### Common issues and solutions:

1. **Cannot access server remotely**
   - Verify your port forwarding settings
   - Check if your ISP blocks ports 80/443
   - Ensure your DuckDNS domain is correctly pointing to your IP

2. **SSL certificate issues**
   - Make sure ports 80 and 443 are open and forwarded
   - Check if your domain is correctly pointing to your IP
   - Try manually renewing the certificate

3. **Nginx Proxy Manager not working**
   - Check Docker logs: `docker logs nginx-proxy-manager`
   - Verify the container is running: `docker ps`
   - Restart the container: `docker-compose restart`

4. **CasaOS not accessible**
   - Check if CasaOS is running: `systemctl status casaos`
   - Restart CasaOS: `systemctl restart casaos`
   - Check firewall settings: `ufw status`

## Conclusion
[⬆️ Back to Top](#table-of-contents)

Congratulations! You now have a secure local server with:
- A free domain name from DuckDNS
- Nginx Proxy Manager handling connections and SSL
- CasaOS providing a user-friendly interface for managing your server

This setup allows you to access your local services securely from anywhere in the world. You can expand on this by adding more services through CasaOS or by configuring additional proxy hosts in Nginx Proxy Manager.

Remember to keep your system updated and regularly back up your important data!
