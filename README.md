![](https://www.primechaintech.com/demo/img/github.jpg)

# Primechain

Installing, configuring, securing, troubleshooting, updating, and maintaining a blockchain ecosystem is a complex and time consuming task. Plus, there is a severe shortage of skilled blockchain developers. 

And that's why we have built Primechain.

**Primechain is a blockchain ecosystem that builds itself in 6 minutes (or less) with a functional web application, mobile Progressive Web App, and a REST API service.**

Table of Contents
-----------------
1. [Prerequisites](#1-prerequisites)
2. [Getting Started](#2-getting-started)
3. [Setting up ngnix and SSL](#3-setting-up-ngnix-and-ssl)
4. [Adding nodes](#4-adding-nodes)
5. [Updating Primechain](#5-updating-primechain)
6. [API Documentation](#6-api-documentation)
7. [Sandbox](#7-sandbox)
8. [Basic troubleshooting](#8-basic-troubleshooting)
9. [Obtaining third party API keys](#9-obtaining-third-party-api-keys)

## 1. Prerequisites
- To setup Primechain you need an 
  - Ubuntu 16.0.4 machine (1 GB RAM, 1 CPU) with CURL and git. 
  - The ports used are 22, 80, 443, 1410, 2512, 15590 and 61172.

**Notes:** 
- For full functionality of PWA, SSL enabed domain is needed. 
- For system generated emails (password reset etc), enter your sendgrid key.

## 2. Getting Started

Login to server / VM as a sudo or root user. Then run the following:
```
sudo git clone https://primechainuser@github.com/Primechain/primechain
<Enter-Password>
cd primechain/setup
sudo bash -e primechain_setup.sh <ip-address> <email-address>
```
***Note:***
1. Instead of the IP address you can enter the domain name above. Or after setup, go to the .env file and change the IP address or domain name.

2. The email address is the admin email address

**The setup should take about 6 minutes. Once done, you will see something like this:**
```
=============================================
ADMIN LOGIN CREDENTIALS FOR WEB APPLICATION
=============================================

#######################################################
#  Email address: info@primechain.in #
#  Password: 5Ofxy3bmMx0Z9xfelnDoHWbaGs5T2RyItZ1n4RYL #
#######################################################


===================================================
WEB APPLICATION UP AND RUNNING IN THE FOLLOWING URL
===================================================
http://example.com:1410


===================================================
API APPLICATION UP AND RUNNING IN THE FOLLOWING URL
===================================================
http://example.com:2512/api/v1/get_api_key

All your credentials are in root/primechain-api.out

```
To view mysql, mongo and multichain credentials
```
nano ~root/primechain-api.out
```

***The web application credentials can be obtained from:**
```
su primechain-user 
cd ~
cd primechain
sudo nano .env
```
***Update the below in .env, to use sendgrid for transactional emails.***
```
MAIL_SERVICE_NAME=SENDGRID
MAIL_USERNAME=<your-username>
MAIL_PASSWORD=<your-password>
```
Enter your **google** and **facebook** credentials if you want to use login through these services. If not, comment out the relevant code in `src/web/views/users/account/login.hbs`

Copy **Primechain-API** username and password if you will be using the API service.


To increase server timeout, login as root into your VM and then:
```
nano /etc/ssh/sshd_config

# Then add the following lines
ClientAliveInterval 120
ClientAliveCountMax 720
```

## 3. Setting up ngnix and SSL
Login to the VM as root and then
```
sudo ufw allow https
sudo apt install nginx
sudo nano /etc/nginx/sites-available/default

# Uncomment the following in SSL configuration

listen 443 ssl default_server;
listen [::]:443 ssl default_server;

# Add the following to the location part of the server block
# Add www.yourdomain.com only if you have made suitable A record entry in DNS

    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://<ip-address>:1410; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart

sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run

```
Now visit https://yourdomain.com and you should see your web app.

## 4. Adding nodes
You can connect multiple non-seed nodes to a running Primechain blockchain. Login to a newly created Ubuntu VM that you will use as a non-seed node.
```
sudo git clone https://primechainuser@github.com/Primechain/primechain_nodes
cd primechain_nodes
sudo bash -e primechain_nodes_setup.sh <primechain-seed-node-ip>
```
This does 
1. hardens the operating system, 
2. sets up the blockchain on the non-seed server

After some time, you will something like the following:

```
-------------------------------------------
INITIATING CONNECTION TO BLOCKCHAIN.....
-------------------------------------------

MultiChain 2.0.3 Daemon (latest protocol 20011)

Starting up node...

Retrieving blockchain parameters from the seed node 52.172.139.41:61172 ...
Blockchain successfully initialized.

Please ask blockchain admin or user having activate permission to let you connect and/or transact:
multichain-cli primechain grant 1aMiTQjLXABeoKtVgBcr5zzVXtT1eRvRTfY3RJ connect
multichain-cli primechain grant 1aMiTQjLXABeoKtVgBcr5zzVXtT1eRvRTfY3RJ connect,send,receive

GRANT PERMISSION FROM THE PRIMECHAIN MASTER NODE AND TYPE yes TO CONTINUE...
```

Login to the Primechain Seed Node and run the following:
```
su primechain-user
cd ~
multichain-cli primechain grant 1aMiTQjLXABeoKtVgBcr5zzVXtT1eRvRTfY3RJ connect
```
**Note:** Don't forget to use the correct address in "multichain-cli primechain grant" above.

After this, you will see a transaction ID like `fae186b309cf396f3a5e0f8dddbdb7d9ab4dd8bd08c69d5b044be19f030f4fc7`

Wait a few seconds and then go to the command line of the non-seed node and type ```yes``` and press Enter.
Few seconds later, you will see something like this:
```
----------------------------------------
BLOCKCHAIN SUCCESSFULLY SET UP!
----------------------------------------
--------------------------------------------
PRIMECHAIN NODE CREDENTIALS
--------------------------------------------
rpcuser=Tzt5COtBS9jeADGOlwMRj9akTyWlCgkZDcqgkQZc
rpcpassword=fzQQMIMJa1mKlDfghQPBQ0uYxgmqdLaYHqjYtTxe

========================================
SET UP COMPLETED SUCCESSFULLY!
========================================
```
Note down the rpcuser and rpcpassword. 

## 5. Updating Primechain 

Login to the server / VM as a sudo or root user.

```
sudo su primechain-user 
cd ~
cd primechain

# Then one of the following
sudo git pull
sudo git pull && pm2 restart 1
sudo git pull && npm i && pm2 restart 1
```

## 6. API Documentation
[https://www.primechaintech.com/documentation](https://www.primechaintech.com/documentation)

## 7. Sandbox
[https://primechainsandbox.com](https://primechainsandbox.com)

## 8. Basic troubleshooting
***Stop / start multichain***
Login to the server / VM as a sudo or root user.
For ***stopping*** multichain:
```
sudo su primechain-user 
cd ~
multichain-cli Primechain stop
```
For ***starting*** multichain:
```
sudo su primechain-user 
cd ~
multichaind Primechain --daemon
```

## 9. Obtaining third party API keys

### 9.1 Google

1. Visit <a href="https://cloud.google.com/console/project" target="_blank">Google Cloud Console</a>.

2. Click on the **Create Project** button.

3. Enter *Project Name*, then click on **Create** button.

4. Click on ***APIs & services*** in the top navigation bar.

5. Click on ***OAuth consent screen***. Choose user type as ***External*** and click on ***Create***. Fill the relevant details and click on ***Save***.

6. Click on ***Credentials***.

7. Click on ***Credentials --> Create credentials --> OAuth client ID***. 
- Choose ***Application type*** as ***Web Application***.
- Choose ***Authorized Javascript origins*** as something like https://test.primechainsandbox.com
- Choose ***Authorized redirect URI*** as http://test.primechainsandbox.com/auth/google/callback

8. Copy your client ID (in .env use it as ***GOOGLE_ID*** and client secret as ***GOOGLE_SECRET***

9. Login to the VM as root as restart pm2 `pm2 restart 1`
<hr>

### 9.2 Facebook

1. Visit <a href="https://developers.facebook.com/" target="_blank">Facebook Developers</a>

2. Click **My Apps**, then select ***Add a New App*** from the dropdown menu

3. Click on ***Setup*** in ***Facebook Login*** box.

4. Click on ***WWW***

5. Enter Site URL as https://test.primechainsandbox.com and click on Save

6. Click on ***Settings --> Basic*** and copy App ID (use in .env as ***FACEBOOK_ID***) and App Secret (use in .env as ***FACEBOOK_SECRET***). Add App domain as https://test.primechainsandbox.com. Click on Save changes

7. In Products -- Facebook Login --> Settings, in Valid OAuth Redirect URIs, enter https://test.primechainsandbox.com/auth/facebook/callback

**Note:** After a successful sign in with Facebook, a user will be
redirected back to the home page with appended hash `#_=_` in the URL.
It is *not* a bug. See this [Stack Overflow](https://stackoverflow.com/questions/7131909/facebook-callback-appends-to-return-url) discussion for ways to handle it.
