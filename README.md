# Linux Server Configuratin - A Udacity project

### Project Description ###
Use **Amazon Lightsail** as a Linux server instance and deploy the **Catalog app** to the web server.Secure the server from a number of attack vectors. Install and configure a database server.
* IP Address: 52.14.213.209
* Accessible SSH port: 2200
* Application URL: http://ec2-52-14-213-209.us-east-2.compute.amazonaws.com

### Walkthrough ###

1. Get a Amazon Lightsail instance and make it run
2. Update all currently installed packages.
    * In terminal, type`$ sudo apt-get update` to download package list.
    * Run`$ sudo apt-get upgrade` to get the newest version of packages.
    * Install `Finger`
3. Change the SSH port from 22 to 2200.
    * Run `$ sudo nano /etc/ssh/sshd_config` and change the port from 22 to 2200
4. Configure firewalls(UFW)to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
    * `$ sudo ufw allow 2200/tcp`
    * `$ sudo ufw allow 80/tcp`
    * `$ sudo ufw allow 123/udp`
    * `$ sudo ufw enable`
5. Create a new user **grader** and give it access
    * Create a new user `sudo adduser grader`
    * Give **grader** sudoer rights `$ sudo nano /etc/sudoers.d/grader` and add 'grader ALL=(ALL:ALL) ALL' to that file
6. Create an SSH key pair for **grader** using the **ssh-keygen** tool
    * Generate a SSH key on the local machine.`$ ssh-keygen -f ~/.ssh/udacity_key.rsa`
    * Create the file to store the key `$ touch /home/grader/.ssh/linuxGrader`
    * Copy the content of `udacity_key.rsa` to `linuxGrader`file
    * Change some permissions: `$ sudo chmod 700 /home/grader/.ssh` `$ sudo chmod 644 /home/grader/.ssh/linuxGrader`
    * Now you can login to the server with `ssh grader@52.14.213.209 -p 2200 -i ~/.ssh/linuxGrader` and a password
7. Enforce key-based authentication
    * `$ sudo nano /etc/ssh/sshd_config`
    * Find the line **PasswordAuthentication** and change it to no
8. Configure the local timezone to UTC
    * Run `sudo dpkg-reconfigure tzdata` and then choose UTC
9. Install **Apache** and **mod_wsgi**
    * Run `sudo apt-get install apache2` to install Apache2
    * Install **mod_wsgi** : `udo apt-get install libapache2-mod-wsgi`
    * Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
    * `$ sudo apache2 start`
10. Intall Git
    * `$ sudo apt-get install git`
11. Clone the Catalog app from Github
    * Create a new directory for the app: `$ sudo mkdir /var/www/catalog`
    * cd to the folder: `$ cd /var/www/catalog`
    * Clone the app from github: `$ git clone https://github.com/ruoranw/Catalog_app.git catalog` then change the **application.py** to **__init__.py**
    * Make a wsgi file : `$ touch catalog.wsgi` and then `$ sudo nano catalog.wsgi`. The file looks like:

```
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,'var/www/catalog')

from catalog import app as application
application.secret_key = 'super_secret_key'
```

    * The first line is for running python in virtual environment
12. Install virtual environment, Flask and other supporting packages
    * Install virtual environment with `$ sudo pip install virtualenv`
    * `cd /var/www/catalog/catalog`
    * Create a new virtual environment: `$ sudo virtualenv venv`
    * Activate the virtual environment:`source venv/bin/activate`
    * Change permissions to the virtual environment folder:`$ sudo chmod -R 777 venv`
    * Install Flask: `$ pip install Flask`
    * Install all the other project's dependencies: `$ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2` `sudo apt install libpq-dev python-dev`
13. Configure a new virtual host
    * Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`
    * The content of this file:

```
<VirtualHost *:80>
Servername 18.217.162.94
ServerAdmin admin@18.217.162.94
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

    * Enable the new virtual host: `$ sudo a2ensite catalog`
14.  Install and configure **PostgreSQL**
    * Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`
    * Postgres is automatically creating a new user named 'postgres' during its installation. Switch to that user: `$ sudo su - postgres` and then connect to the database: `psql`
    * Create a new user called 'catalog' with password: `# CREATE USER catalog WITH PASSWORD ''`
    * Create a new database 'cata_db': `# ALTER USER catalog CREATEDB` then `# CREATE DATABASE cata_db WITH OWNER catalog`
    * Connect to the database: `\c cata_db`
    * Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public`
    * Lock down the permissions to only let catalog role create tables: `# GRANT ALL ON SCHEMA public TO catalog`
    * Log out: `\q` and return to the previous user: exit
    * Change database setting in `__init__.py`:
    `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
    * Setup the database with: `$ python /var/www/catalog/catalog/setup_database.py`
    * Make sure no remote access to the database. Open the following file: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit it
15. Update OAuth authorized JavaScript origins
    * Change the url both in Facebook and Google developer tool to the app's url
16. Restart Apache
    * `sudo service apache2 restart`

