# Setting Up a Secure Local Server with DuckDNS and Nginx Proxy Manager

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setting Up DuckDNS](#setting-up-duckdns)
- [Understanding and Using the DuckDNS Token](#understanding-and-using-the-duckdns-token)
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
   - This token is very important - save it somewhere secure
   - You will need this token to update your IP address automatically

## Understanding and Using the DuckDNS Token
[⬆️ Back to Top](#table-of-contents)

The DuckDNS token is a crucial part of your setup but is often misunderstood.

1. **What is the DuckDNS token?**
   - It's an authentication key specific to your DuckDNS account
   - It allows you to update where your domain points to
   - Without it, anyone could hijack your domain

2. **What the token is used for:**
   - To authenticate your update requests to DuckDNS
   - To allow automatic updates of your home IP address
   - To keep your domain pointing to your server even when your IP changes

3. **Where to use the token:**
   - The token is NOT used in Nginx Proxy Manager
   - The token is used in an update script or cronjob that runs on your server
   - This script periodically updates DuckDNS with your current IP address

4. **Setting up the update script:**
   ```bash
   # Create a directory for DuckDNS
   mkdir -p ~/duckdns
   
   # Create the update script
   echo '#!/bin/bash' > ~/duckdns/duck.sh
   echo "echo url=\"https://www.duckdns.org/update?domains=YOURDOMAIN&token=YOURTOKEN&ip=\" | curl -k -o ~/duckdns/duck.log -K -" >> ~/duckdns/duck.sh
   
   # Replace YOURDOMAIN with your subdomain (e.g., myserver)
   # Replace YOURTOKEN with the token from DuckDNS
   # For example:
   # echo "echo url=\"https://www.duckdns.org/update?domains=myserver&token=a7c4d0b2-1234-5678-abcd-ef1234567890&ip=\" | curl -k -o ~/duckdns/duck.log -K -" >> ~/duckdns/duck.sh
   
   # Make the script executable
   chmod 700 ~/duckdns/duck.sh
   
   # Run the script to test
   ~/duckdns/duck.sh
   
   # Check if it worked
   cat ~/duckdns/duck.log
   # It should show "OK" if successful
   ```

5. **Setting up automatic updates:**
   ```bash
   # Add to crontab to run every 5 minutes
   (crontab -l 2>/dev/null; echo "*/5 * * * * ~/duckdns/duck.sh") | crontab -
   
   # Verify crontab entry
   crontab -l
   ```

6. **Why this matters:**
   - Most home internet connections have dynamic IP addresses that change
   - Without regular updates, your domain would stop pointing to your server
   - The token ensures these updates happen securely and automatically

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
   - For example: `http://192.168.1.100:81`
   - The default login credentials are:
     - Email: admin@example.com
     - Password: changeme
   - You'll be prompted to change these credentials on first login

## Configuring Nginx Proxy Manager
[⬆️ Back to Top](#table-of-contents)

1. **Add a new proxy host**
   - Click on "Hosts" > "Proxy Hosts" > "Add Proxy Host"
   - In the "Domain Names" field, enter your DuckDNS domain (e.g., myserver.duckdns.org)
   - Set the scheme to http
   - For "Forward Hostname/IP", enter your local server IP (e.g., 192.168.1.100)
   - For "Forward Port", enter the port of your local service (e.g., 80)
   - Click "Save"
   
   **Note:** You do NOT need to enter your DuckDNS token here. The token is only used in the update script.

2. **Set up port forwarding on your router**
   - Log into your router's admin interface (usually at 192.168.1.1 or 192.168.0.1)
   - Find the port forwarding section (might be under "Advanced" or "NAT")
   - Create two port forwarding rules:
     - Forward external port 80 to your server's IP address, port 80
     - Forward external port 443 to your server's IP address, port 443
   - Save the settings

## Setting Up SSL Certificates
[⬆️ Back to Top](#table-of-contents)

1. **Edit your proxy host**
   - In Nginx Proxy Manager, click on the three dots next to your proxy host
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
   - Example: `https://myserver.duckdns.org`
   - You should see a secure padlock in your browser

## Installing CasaOS
[⬆️ Back to Top](#table-of-contents)

CasaOS is a simple, easy-to-use home cloud system that turns your hardware into a home server.

1. **Install CasaOS**
   ```bash
   curl -fsSL https://get.casaos.io | sudo bash
   ```

2. **Wait for installation to complete**
   - The installation process may take several minutes
   - You'll see a message when it's done

3. **Access CasaOS**
   - Open your browser and navigate to `http://YOUR_SERVER_IP:80`
   - Example: `http://192.168.1.100:80`
   - You may also be able to access it at `http://casa.local`
   - Complete the initial setup wizard
   - Create your admin account with a strong password

## Configuring CasaOS
[⬆️ Back to Top](#table-of-contents)

1. **Configure network settings**
   - Go to "System" > "Network"
   - Make sure your server has a static IP address
   - This ensures your server's IP doesn't change on your local network

2. **Install applications**
   - Go to "App Store"
   - Browse and install the applications you need
   - Each app will be installed as a Docker container

3. **Set up users**
   - Go to "System" > "Users"
   - Create accounts for anyone who will be accessing your server
   - Assign appropriate permissions to each user

## Connecting Everything Together
[⬆️ Back to Top](#table-of-contents)

1. **Create a proxy host for CasaOS**
   - Go back to Nginx Proxy Manager
   - Add a new proxy host
   - Domain: `casaos.yourdomain.duckdns.org` (replace with your actual domain)
   - Scheme: http
   - Forward Hostname/IP: YOUR_SERVER_IP (e.g., 192.168.1.100)
   - Forward Port: 80 (or the port CasaOS is running on)
   - Request a new SSL certificate as done previously

2. **Test the connection**
   - Open your browser and navigate to `https://casaos.yourdomain.duckdns.org`
   - You should be redirected to the CasaOS login page
   - Log in with your credentials
   - You now have a secure connection to your server from anywhere in the world!

3. **How it all works together:**
   - DuckDNS keeps your domain pointing to your home IP (using the token in the update script)
   - Your router forwards incoming traffic on ports 80/443 to your server
   - Nginx Proxy Manager receives this traffic and routes it to CasaOS
   - The SSL certificate ensures all traffic is encrypted
   - CasaOS provides the user interface and application management

## Troubleshooting
[⬆️ Back to Top](#table-of-contents)

### Common issues and solutions:

1. **Cannot access server remotely**
   - Verify your DuckDNS update script is working: `cat ~/duckdns/duck.log` should show "OK"
   - Check your router's port forwarding settings
   - Test if ports are open: use an online port checker tool
   - Check if your ISP blocks ports 80/443 (some do)
   - Try accessing with your phone's cellular data (not WiFi) to test external access

2. **SSL certificate issues**
   - Make sure ports 80 and 443 are open and forwarded
   - Check if your domain is correctly pointing to your IP
   - Try manually renewing the certificate in Nginx Proxy Manager
   - Check Nginx Proxy Manager logs for error messages

3. **Nginx Proxy Manager not working**
   - Check Docker logs: `docker logs nginx-proxy-manager`
   - Verify the container is running: `docker ps`
   - Restart the container: `docker-compose restart`
   - Check if something else is using ports 80, 81, or 443

4. **CasaOS not accessible**
   - Check if CasaOS is running: `systemctl status casaos`
   - Restart CasaOS: `systemctl restart casaos`
   - Check firewall settings: `ufw status`
   - Verify you can access it locally before trying remote access

5. **DuckDNS not updating**
   - Check the update script logs: `cat ~/duckdns/duck.log`
   - Verify the token is correct in your script
   - Make sure the script is executable: `chmod 700 ~/duckdns/duck.sh`
   - Check if the cron job is running: `grep duckdns /var/log/syslog`

## Conclusion
[⬆️ Back to Top](#table-of-contents)

Congratulations! You now have a secure local server with:
- A free domain name from DuckDNS that automatically updates
- Nginx Proxy Manager handling connections and SSL
- CasaOS providing a user-friendly interface for managing your server

This setup allows you to access your local services securely from anywhere in the world. You can expand on this by adding more services through CasaOS or by configuring additional proxy hosts in Nginx Proxy Manager.

Remember to keep your system updated and regularly back up your important data!

Key takeaways:
- The DuckDNS token is used ONLY in the update script, not in Nginx Proxy Manager
- Regular IP updates are crucial for remote access to work reliably
- SSL certificates provide secure encrypted connections to your server
- CasaOS makes managing your server and applications easy
