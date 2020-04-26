![picture](/public/banner.jpg)
# Install Free Let’s Encrypt SSL Cert with Nginx

![picture](/public/divider.png)

# Table of Contents

- [Introduction]
- [Create .well_known]
- [Setting up Let’s Encrypt SSL]
- [Setting up Nginx]
- [Setting up Auto - Renewal Cron Job]
- [Copyright]

##

## Introduction

Let’s Encrypt is a Certificate Authority (CA) that provides an easy way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers.
Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx web servers. In this article, I will show you how to set it up for Nginx.

##

![formbuilder](/public/secure-env.gif)

##

## Create .well_known directory

```
$ mkdir /usr/share/nginx/html/.well_known
```

## Setting up Let’s Encrypt SSL

##### Step 1: Downloading Let’s Encrypt

Download the letsencrypt client into the `/opt/letsencrypt` folder.

```
$ git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
```

##### Step 2: Fix Locale Error

While requesting the certificate, Sometimes I have come across some locale error. To fix that, update the following environment variables:

```
$ export LC_ALL="en_US.UTF-8"
$ export LC_CTYPE="en_US.UTF-8"
```

##### Step 3: Creating a Let's Encrypt SSL Cert

We can create a cert using the following command. Note that you'll need to change the domain name.

- `mysite.com` should be replaced with your domain

```
$ /opt/letsencrypt/letsencrypt-auto certonly --webroot -w /usr/share/nginx/html -d 'mysite.com,www.mysite.com'
```

## Setting Up Nginx

##### Step 1: Installing Nginx

```
$ sudo apt update
$ sudo apt install nginx
```

#### Step 2: Enhancing security

Once you have the certificate and chain saved on the server, you can check the Nginx configuration to further tune the HTTPS connection using the new certificates.

- Only use secure protocols

  The old insecure `SSLv2` and `SSLv3` should be disabled and never used on modern web servers. This is due to the inherent insecurities in these protocols which is why they have been replaced by the TLS protocols. The example below enables the two most secure TLS versions `1.1` and `1.2`

  ```
  ssl_protocols TLSv1.1 TLSv1.2;
  ```

- Enable HTTP Strict Transport Security (HSTS)

  The Strict Transport Security safeguards TLS by not allowing any insecure communication with the websites that use it. It was designed to ensure security even in the case of configuration problems and implementation errors. HSTS automatically converting all plaintext links to a secure version. It also disables click-through certificate warnings which are an indicator of an active MITM attack and prevents users from circumventing the warning.

  ```
  add_header Strict-Transport-Security "max-age=31536000; includeSubdomains";
  ```

Enable HSTS in by using the option as shown below

##

![picture](/public/encrypt.PNG)

# Setup Renewal Cron Job

To enable automatic renewal of your ssl certificate, follow the following steps to create a Cron task that would handle it automatically for you.
Run this command to open up the crontab:

```
$ sudo crontab -e
```

Paste following below two line in the end of cron tab.

```
30 4 * * 1 /opt/letsencrypt/letsencrypt-auto renew >> /var/log/le-renew.log
35 4 * * 1 /etc/init.d/nginx reload
```

We have registered a new cron job that will execute the auto renew command every Monday at `4:30 am`, and reload Nginx at `4:35am`.
The output produced by the command will be piped to a log file located at `/var/log/le-renew.log`.
Create this log file if it doesn’t already exist.

```
$ touch /var/log/le-renew.log
```

Your SSL certificate is now up and running and it gets renewed automatically. You don’t have to keep an eye on it.

##

![picture](/public/decrypt.PNG)

![picture](/public/divider.png)

### Copyright

Copyright (C) [Faizan AH][1] - All Rights Reserved

- Unauthorized copying of this file, via any medium is strictly prohibited
- Written by Faizan Ahmad <faizanahmad.herokuapp.com>, April 2020
  [![](https://www.aiondigital.com/wp-content/uploads/2018/07/Aion_white-1024x241.png)](https://aiondigital.com)
  [1]: https://faizanahmad.herokuapp.com
