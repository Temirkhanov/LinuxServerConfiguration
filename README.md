# Linux Server Configuration

This are the instructions to configure Linux(ubuntu) server to deploy flask app (Item Catalog) using apache.

## Notes:

    Login system and Google sign-in are not set up and not configured.

## Tasks

1. Start a new Ubuntu Linux server instance on Amazon Lightsail
2. Follow the instructions provided to SSH into your server.
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
11. Install and configure PostgreSQL:
    - Do not allow remote connections
    - Create a new database user named catalog that has limited permissions to your catalog application database.
12. Install git
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!

## Amazon Lightsail

1. Log in!
2. Create an instance.
3. Choose an instance image: Ubuntu
4. Choose your instance plan.
5. Give your instance a hostname.
6. Download private key

## SSH, User and Security

1. From terminal on your machine login remotely to your server '''\$ ssh -i ~/.ssh/LightsailDefaultKeyPair.pem ubuntu@YourServerIP'''
2. Create new user _grader_ '''\$ sudo adduser grader'''
3. Add _grader_ to sudoers '''\$ sudo nano /etc/sudoers.d/grader'''. Add '''grader ALL=(ALL:ALL) ALL'''. Save and exit.
4. Create new key pair in the terminal. '''\$ ssh-keygen grader'''.
5. Print, copy and save the public key '''\$ cat ~/.ssh/grader_key.pub'''
6. Put the public key _public_key.pub_ to _authorized_keys_ and add the permission: '''$ sudo chmod 700 /home/grader/.ssh''' and '''$ sudo chmod 644 /home/grader/.ssh/authorized_keys'''
7. Change key-based authentication and change SSH port to 2200 and disab;e remote root login:
   - '''\$ sudo nano /etc/ssh/sshd_config'''
   - 'PasswordAuthentication', 'PermitRootLogin' -> 'no'
   - 'Port' -> '2200'
   - Restart ssh service '''\$ sudo service ssh restart'''
8. Config the timezone: '''\$ sudo dpkg-reconfigure tzdata'''

## Installation and updating

1. '''\$ sudo apt-get update'''
2. '''\$ sudo apt-get upgrade'''

## Config Firewall

1. '''\$ sudo ufw default deny incoming'''
2. '''\$ sudo ufw default allow outgoing'''
3. '''\$ sudo ufw allow 2200/tcp'''
4. '''\$ sudo ufw allow 80/tcp'''
5. '''\$ sudo ufw allow 123/udp'''
6. '''\$ sudo ufw enable'''

## Apache, Git, mod_wsgi

1. '''\$ sudo apt-get install apache2'''
2. Install mod_wsgi '''\$ sudo apt-get install libapache2-mod-wsgi python-dev'''
3. Enable mod_wsgi '''\$ sudo a2enmod wsgi'''
4. '''\$ sudo service apache2 start'''
5. '''\$ sudo apt-get install git'''

## Config Apache

1.  Clone project from Github and change the owner: '''$ cd /var/www $ sudo mkdir catalog $ sudo chown -R grader:grader catalog $ cd catalog \$ git clone https://github.com/GithubRepoLink catalog'''
2.  Create catalog.wsgi file and edit it:
    ''' import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

        from catalog import app as application'''

3.  Rename application.py to init.py
4.  Virtual Environment:
    - Install and create '''sudo pip install virtualenv''' '''sudo virtualenv venv'''
    - Activate and change permission '''source venv/bin/activate''' '''sudo chmod -R 777 venv'''
5.  Install Flask and all dependecies:
    - pip '''\$ sudo apt-get install python-pip'''
    - Flask '''\$ pip install Flask'''
    - '''sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils'''

## Config virtual host

1. Edit/create catalog configuration file '''sudo nano /etc/apache2/sites-available/catalog.conf'''
2. Paste/edit for your server:
   '''<VirtualHost \*:80>
   ServerName YOURSERVNAME
   ServerAlias YOURSERVALIAS
   ServerAdmin YOUREMAIL
   WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
   WSGIProcessGroup catalog
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
   </VirtualHost>'''
3. Enable the virtual host: '''sudo a2ensite catalog'''

## Install and config PostrgreSQL

1. '''\$ sudo apt-get install libpq-dev python-dev'''
2. '''\$ sudo apt-get install postgresql postgresql-contrib'''
3. Connect '''\$ sudo -u postgres psql'''
4. Create new user '''CREATE USER catalog WITH PASSWORD 'catalog';'''
5. Grant permission to create DB '''ALTER USER catalog CREATEDB;'''
6. Create DB '''REATE DATABASE catalog WITH OWNER catalog;'''
7. Disconnect '''\q'''
8. Edit db_setup.py and other neccissary py files :
   '''create_engine('sqlite:///category.db')''' -> '''create_engine('postgresql://catalog:catalog@localhost/catalog')'''
