# FSND-12-Linux-Server-Configuration
Linux Server Configuration

## How will I complete this project?

### Get your server.
##### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.

For this project, I have installed a Ubuntu as a virtual machine on my local computer.

##### 2. Follow the instructions provided to SSH into your server.
Install ssh client and server software
- `sudo apt install openssh-client`
- `sudo apt install openssh-server` 

##### 3. Update all currently installed packages

- `sudo apt-get update`
- `sudo apt-get upgrade`

##### 4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
- `sudo nano /etc/ssh/sshd_config`
Change from `Port 22` to `Port 2200`
- `sudo services sshd restart`

##### *For Internet access I have enable the port forward of 2200 from the Internet to 2200 internal port.*

<br>
##### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
<br>
`sudo ufw status`
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow 2200`
`sudo ufw allow www`
`sudo ufw allow 123`
`sudo ufw enable`
----
#### Give grader access.

##### 6. Create a new user account named `grader`.

`sudo adduser grader`
`123456` for (password)
	
##### 7. Give grader the permission to sudo

`sudo usermod –aG sudo grader`
	
	Source: https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart 

##### 8. Create an SSH key pair for grader using the ssh-keygen tool.

On local machine run:
- `sudo ssh-keygen`
provide the name of the file as `udacity_key`
- `sudo nano udacity_key.pub` and copy the content of this file

On the prospective server run:
	
- `sudo mkdir .ssh`
- `sudo touch .ssh/authorized_keys`
- `sudo nano .ssh/authorized_keys`
and past the content of previously copied from `udacity_key.pub` into `authorized_keys` file and save
- `sudo chmod 700 .ssh`
- `sudo chmod 644 .ssh/authorized_keys`

So now the Ubuntu computer will be able to login by using the ssh key by the following command:

`ssh grader@loynin.changeip.net -p 2200 -i udacity_key`

###### Disable ssh login for root user

- `sudo nano /etc/ssh/sshd_config`
Change `PermitRootLogin without_password` line to `PermitRootLogin no`
- Restart ssh by `sudo service ssh restart`

----
#### Prepare to deploy your project

##### 9. Configure the local timezone to UTC.
- `sudo dpkg-reconfigure tzdata`
and select UTC
 
##### 10. Install and configure Apache to serve a Python mod_wsgi application
- `sudo apt-get install apache2`

##### 11. Install mod_wsgi

-	Run `sudo apt-get install libapache2-mod-wsgi`
-	Enable mod_wsgi by `sudo a2enmod wsgi`
-	Start the web server by `sudo service apache2 start`

##### 12. Clone the Catalog app from Github

-	Install git by `sudo apt-get install git`
-	Change directory to `cd /var/www`
-	Make new directory `sudo mkdir catalog`
-	Change ownership of the created directory by `sudo chown –R grader:grader catalog`
-	Change to directory `cd catalog`
-	Clone project from Github by `git clone https://github.com/loynin/FSND-05-Catalog-Item.git catalog`
-	Create a `catalog.wsgi` file with the following content: 
```
import sys
import os
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
os.environ['PYTHON_EGG_CACHE'] = '/var/www/catalog/catalog/venv/temp'
from catalog import app as application
application.secret_key = 'supersecretkey'
```
-	Change folder to `cd catalog`
-	Rename project.py to init.py by `sudo mv project.py __init__.py`

##### 13. Install virtual environment

- Install virtual invironment by `sudo pip install virtualenv`
- Create a new virtual environment with `sudo virtualenv venv`
- Activate the virtual environment by `source venv/bin/activate`
- Change permissions `sudo chomd –R 777 venv`
##### 14. Install Flask and other dependencies
- Install pip with `sudo apt-get install python-pip`
- Install Flask by `pip install Flask`
- Install other dependencies `pip install httplib2 oauth2client sqlalchemy psycopg2`

##### 15. Update path of client_secrets.json and fb_client_secrets.json in __init__.py file
- `sudo nano __init__.py`
- Change `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json`
- Change `fb_client_secrets.json` to `/var/www/catalog/catalog/fb_client_secrets_json`

##### 16. Configure and enable a new virtual host for apache2
- Create a new file `sudo nano /etc/apache2/sites-available/catalog.conf` with the content as following codes: 
```
<VirtualHost *:80>
    ServerName 155.186.92.206
    ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
WSGIDaemonProcess catalog python-path=/usr/lib/python2.7:/usr/local/lib/python2.7/dist-packages
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

##### 17. Install and configure PostgreSQL
- Install PostgreSQL `sudo apt install postgresql`
- Check if no remote connections are allow `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
- Login as user ‘postgres’ `sudo su – postgres`
- Get into postgreSQL shell `psql`
- Create a new database named catalog and create a new user named catalog in postgreSQL shell:
    - `CREATE DATABASE catalog;`
    - `CREATE USER catalog;`
- Set a password for user catalog
`ALTER ROLE catalog WITH PASSWORD 'password';`
- Give user ‘catalog’ permission to catalog application database
`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
- Quit postgreSQL `postgres=# \q`
- `exit`

    ** Incase you don't have it:** The Item Catalog project used SQLAlchemy data server to store the data. Below are the process how to install SQLAlchemy in the Ubuntu system:

    - `sudo apt-get install mysql-server`
    - `sudo apt-get install mysql-client`
    - `sudo apt-get install libmysqlclient-dev`
    - `sudo apt-get install python-mysqldb`
    - `sudo apt-get install python-setuptools python-dev build-essential`
    - `sudo easy_install MySQL-Python`
    - `sudo easy_install SQLAlchemy`
- Change create_engine in `__init__.py` and `database_setup.py` to 
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`
- `sudo python /var/www/catalog/catalog/database_setup.py`
- `sudo python /var/www/catalog/catalog/lotofitems.py`

##### 18. Restart apache
- `sudo service apache2 restart

##### 19 Visit site at http://loynin.changeip.net:222

** Note: the public port is 222 and internal port is 80** 


