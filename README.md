# Linux Server Configuration
###### Udacity's Full Stack Web Developer Nanodegree
----

## Project Description

This project describes a baseline installation of a Linux server and prepares it to host web applications. It aims to secure the server from a number of attack vectors, install and configure a database server, and deploy one of the existing web applications onto it. This project is guided by [Udacity’s Full Stack Developer Nanodegree Program](https://sa.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

* The virtual private server from [Amazon Lighsail](https://lightsail.aws.amazon.com/).
* The Linux server distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.
* The database server is [PostgreSQL](https://www.postgresql.org/).
* The local machine is a MacBook Pro (Mac OS X 10.14.2).
* The web application is [Item Catalog project](https://github.com/zabdulmanea/items_catalog.git) created earlier in this Nanodegree program.

## Basic Information
- **Visit the web application:** [MOOC Providers Catalog](http://3.89.164.105.xip.io)
- **Public IP Address:** 3.89.164.105
- **Accessible SSH Port:** 2200

## Get a Virtual Private Server
### Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail
- Log in to [Lightsail](https://lightsail.aws.amazon.com/) or create new account.
- Click `Create an instance`.
- Choose an instance image: `OS Only` then `Ubuntu 16.04 LTS`.
- Choose the instance plan with the lowest tier of instance.
- Give the instance a hostname or keep the default name.
- Click `Create`.
- The public IP address: `3.89.164.105` of the instance is displayed along with its name.

### Step 2: SSH into the server from local machine
- From AWS Account Page click `SSH Keys` and download your preferred key pair file.
- Rename the key pair file to `key.pem` then move it into the local folder `/~.ssh`.
- In the terminal: type `chmod 600 ~/.ssh/key.pem`.
- Now Login into the server as `ubuntu`: `ssh -i ~/.ssh/key.pem -p 2200 ubuntu@3.89.164.105`.

## Secure the Server
### Step 3: Update all currently installed packages
```
sudo apt-get update
sudo apt-get upgrade
```
#### Step 3.1: Use unattended-upgrades to automatically install updates
- Install the `unattended-upgrades` package: `sudo apt install unattended-upgrades`
- Edit the configuration file: `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades`
- Uncomment the `updates` line: `${distro_id}:${distro_codename}-updates`. save then exit (`ctrl+x`, `y` then press `Enter`).
- Enable automatic updates by running: `sudo nano /etc/apt/apt.conf.d/20auto-upgrades`
- Copy and paste the following lines:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```
- Check if it works by running: `sudo unattended-upgrades --dry-run --debug`, the output should be something like:
```
Initial blacklisted packages: 
Initial whitelisted packages: 
Starting unattended upgrades script
Allowed origins are: ['o=Ubuntu,a=xenial', 'o=Ubuntu,a=xenial-security', 'o=UbuntuESM,a=xenial', 'o=Ubuntu,a=xenial-updates']
pkgs that look like they should be upgraded: 
Fetched 0 B in 0s (0 B/s)                                                                                                                                                         
fetch.run() result: 0
blacklist: []
whitelist: []
No packages found that can be upgraded unattended and no pending auto-removals
```
- Restart Apache: `sudo service apache2 restart`

### Step 4: Change the SSH port from 22 to 2200
- While logged in as `ubuntu`, edit `sshd_config` file: `sudo nano /etc/ssh/sshd_config`
- Change `Port 22` to `Port 2200`, save then exit (`ctrl+x`, `y` then press `Enter`).
- Restart SSH: `sudo service ssh restart`

### Step 5: Configure the Uncomplicated Firewall (UFW)
- Only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw status                     # The UFW status should be inactive
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow http
sudo ufw allow ntp
sudo ufw enable                     # Enable UFW to be active
```

- Check UFW status `sudo ufw status`, the output should be similar to this:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6) 
```

- Go back to AWS Lightsail instance page, click on the `instance` name that have been created earlier.
- Click `Networking` tab.
- Change the firewall configuration to match the pervious internal firewall settings. Allow only ports 80(TCP), 123(UDP), and 2200(TCP).
- Click `Save`

#### Step 5.1: Use Fail2Ban to monitor unsuccessful login attempts
Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks.
- Install Fail2Ban: `sudo apt-get install fail2ban`
- Rename a copy `fail2ban.conf` to `fail2ban.local`: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
- Open `jail.local`: `sudo nano /etc/fail2ban/jail.local`
- Under `[sshd]` change `port = ssh` by `port = 2200`
- Restart fail2ban service: `sudo service fail2ban restart`

## Give `grader` Access
### Step 6: Create a new user account named `grader`
- While logged in as `ubuntu`, add user: `sudo adduser grader`.
- Set password `123` (required) and fill out any additional information (optional).

### Step 7: Give `grader` the permission to `sudo`
- Create `/etc/sudoers.d/grader` file to give sudo access for the `grader`: `sudo nano /etc/sudoers.d/grader`.
- Add this line `grader ALL=(ALL:ALL) ALL`. save then exit (`ctrl+x`, `y` then press `Enter`).

### Step 8: Create an SSH key pair for `grader` using the `ssh-keygen` tool
- Exit `ubuntu` server machine.
- On the local machine, Run `ssh-keygen`
- Enter file in which to save the key (`grader_key`) in the local directory `~/.ssh`
- Enter `passphrase` (Optional)
- Copy the content of `grader_key.pub`: `cat ~/.ssh/grader_key.pub`
- Login into `ubuntu` server machine: `ssh -i ~/.ssh/key.pem -p 2200 ubuntu@3.89.164.105`
- Switch to `grader` server machine: `sudo su - grader`
- Create new directory `~/.ssh`: `mkdir .ssh`
- Create new file `authorized_keys`: `sudo nano ~/.ssh/authorized_keys`
- Paste `grader_key.pub` content. save then exit (`ctrl+x`, `y` then press `Enter`)
- Give the permissions:
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
- Read `sudo nano /etc/ssh/sshd_config`, check if `PasswordAuthentication` and `PermitRootLogin` are set to `no` to improve the security of the server.
- Restart SSH: `sudo service ssh restart`
- Exit SSH connection: `exit`
- From local machine login to the grader server machine:
```
ssh grader@3.89.164.105 -p 2200 -i ~/.ssh/grader_key
```


## Prepare to Deploy the Project
### Step 9: Configure the local timezone to UTC
- While logged in as `grader`, type: `sudo dpkg-reconfigure tzdata`. Choose `None of the above`, then `UTC`

### Step 10: Install and configure Apache to serve a Python mod_wsgi application
- While logged in as `grader`:
```
sudo apt-get install apache2                # Install Apache2
sudo apt-get install libapache2-mod-wsgi    # Install mod-wsgi 
sudo a2enmod wsgi                           # Enable mod_wsgi
```

### Step 11: Install and configure PostgreSQL
- While logged in as `grader`:
```
sudo apt-get install postgresql
```
- Switch to `postgres` server machine: `sudo su - postgres`
- Open PostgreSQL terminal-based front-end: `psql`
- Create `catalog database` with a `catalog user`. Then give this user a password and permission to `catalog database`.

```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
postgres=# \q
```
- Exit `postgres`: `exit`


## Deploy the Item Catalog Project
### Step 12: Clone and setup your Item Catalog
- While logged in as `grader`, install `git`: `sudo apt-get install git`
- Create directory `mkdir /var/www/catalog`, then change to that directory `cd /var/www/catalog`
- Clone Item catalog repository: `git clone https://github.com/zabdulmanea/items_catalog.git catalog`
- Change directory to `cd /var/www` then change the ownership of the catalog directory to `grader`: `sudo chown -R grader:grader catalog/`
- Move to Item catalog directory: `cd /var/www/catalog/catalog`
- Rename the `application.py` file: `mv application.py __init__.py`
- Replace some lines to setup the Item Catalog with the server

| File | Before change | After change |
|------|---------------|--------------|
| **`__init__.py`** | engine = create_engine('sqlite:///provider_courses.db?check_same_thread=False') | engine = create_engine(**'postgresql://catalog:catalog@localhost/catalog'**) |
| **`database_setup.py`** | engine = create_engine('sqlite:///provider_courses.db') | engine = create_engine(**'postgresql://catalog:catalog@localhost/catalog'**) |
| **`__init__.py`** | CLIENT_ID = json.loads(open('client_secrets.json', | CLIENT_ID = json.loads(open(**'/var/www/catalog/catalog/client_secrets.json'**, |
| **`__init__.py`** | oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='') | oauth_flow = flow_from_clientsecrets(**'/var/www/catalog/catalog/client_secrets.json'**, scope='') |
| **`__init__.py`** | app.run(host="0.0.0.0", port=8000) | **app.run()** |

- Remove unwanted files:
```
rm database_setup.pyc
rm provider_courses.db
rm fb_client_secrets.json

```
- Run the database `python database_setup.py`


### Step 13: Update Google OAuth Credentials
- Go to [Google Cloud Plateform](https://console.cloud.google.com/home/dashboard?project=mooc-providers-catalog)
- Under Authorized domains add `xip.io` 
- Under Authorized JavaScript origins add `http://3.89.164.105.xip.io` 
- Under Authorized redirect URIs add `http://3.89.164.105.xip.io/`, `	http://3.89.164.105.xip.io/login`, `http://3.89.164.105.xip.io/oauth2callback`.
- Download `JSON` file and copy its content.
- Go back to terminal as `grader` user.
- Edit `client_secrets` file: `sudo nano /var/www/catalog/catalog/client_secrets.json` and paste JSON file content.

### Step 14: Install the Web Application dependencies
While logged in as grader, install
```
sudo apt-get install python-pip
sudo pip install Flask
sudo pip install requests
sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
```

### Step 15: Set up and enable a virtual host
- Create `catalog.conf` file : `sudo nano /etc/apache2/sites-available/catalog.conf`
- Copy then paste the following lines to configure the virtual host:
```
<VirtualHost *:80>
        ServerName 3.89.164.105
        ServerAdmin zabdulmanea@gmail.com
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Enable virtual host: `sudo a2ensite catalog`
- Reload Apache: `sudo service apache2 reload`

### Step 16: Disable the default Apache site
- Disable the default Apache site: `sudo a2dissite 000-default.conf`
- Reload Apache: `sudo service apache2 reload`

### Step 17: Set up the Flask application
- Create `catalog.wsgi` file : `sudo nano /var/www/catalog/catalog.wsgi`
- Copy then paste the following lines to configure the application:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
- Restart Apache: `sudo service apache2 restart`

## Run the application
- Browse the Item Catalog Site through http://3.89.164.105.xip.io

## Helpful Resources and Commands
- Apache's error log: `sudo tail /var/log/apache2/error.log`
- Restart Apache: `sudo service apache2 restart`
- How to set up [automatic updates on Ubuntu Server](https://libre-software.net/ubuntu-automatic-updates/)
- Linux Server Configraution [Session](https://www.youtube.com/watch?v=qDnuf0zfkYE&feature=youtu.be) 
- Use [Fail2ban to Secure Your Server](https://www.linode.com/docs/security/using-fail2ban-for-security/)
- PostgreSQL [describe table](https://stackoverflow.com/questions/109325/postgresql-describe-table)
- Reinstall [python-pip package](https://stackoverflow.com/questions/16237490/i-screwed-up-the-system-version-of-python-pip-on-ubuntu-12-10)
- Fix [Broken Python pip](https://github.com/pypa/pipenv/issues/2122)
- GitHub Repositories
    - https://github.com/boisalai/udacity-linux-server-configuration
    - https://github.com/sagarchoudhary96/P8-Linux-Server-Configuration
    - https://github.com/kongling893/Linux-Server-Configuration-UDACITY
    - https://github.com/rrjoson/udacity-linux-server-configuration
    - https://github.com/kotamichael/amazon-lightsail-server-configuration.git