# Linux-Server-Configuration

## Description

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: 34.219.232.61
- Accessible SSH port: 2200
- Application URL: http://34.219.232.61/
## Step::
1. Create new user named grader and give it the permission to sudo
  - SSH into the server through `ssh -i ~/.ssh/udacity_key.rsa root@34.219.232.61`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
  - Run `sudo nano /etc/hosts`
  - Prevent the error `sudo: unable to resolve host` by adding this line `127.0.1.1 ip-10-20-52-12`
  
2. Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200

4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
5. Configure the key-based authentication for *grader* user

  - Generate an encryption key **on your local machine** with: `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`.
  - Log into the remote VM as *root* user through ssh and create the following file: `$ touch /home/grader/.ssh/authorized_keys`.
  - Copy the content of the *udacity_key.pub* file from your local machine to the */home/grader/.ssh/authorized_keys* file you just created on the remote VM. Then change some permissions:
	- `$ sudo chmod 700 /home/grader/.ssh`.
	- `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
	- Finally change the owner from *root* to *grader*: `$ sudo chown -R grader:grader /home/grader/.ssh`.
  
6. Enforce key-based authentication
  - `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and edit it to *no*.
  - `$ sudo service ssh restart`.

7. Install Apache
  - `sudo apt-get install apache2`

8. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

9. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/rrjoson/udacity-item-catalog.git catalog`
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  
10. Install virtual environment
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

11. Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

12. Update path of client_secrets.json file

13. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
      ServerName 35.167.27.204
      ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
      ServerAdmin admin@35.167.27.204
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
  </VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`

14. Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - Change create engine line in your `__init__.py` and `database_setup.py` to: 
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_setup.py`
  - Make sure no remote connections to the database are allowed. Check if the contents of this file `sudo nano /etc/postgresql/9.3/main/pg_hba.conf` looks like this:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
  
15. Restart Apache 
  - `sudo service apache2 restart`
  
 ## Special Thanks to *[Ali Mahmoud](https://github.com/AliMahmoud7)* for a very helpful
