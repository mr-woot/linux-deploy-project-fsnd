# Linux project deploying Full Stack Nanodegree.

35.166.221.186 is my [PUBLIC\_IP] or [HOST\_IP]<br>
http://ec2-35-161-139-158.us-west-2.compute.amazonaws.com is my [HOSTED\_URL]<br>

## Instructions for SSH access to the instance

* Download Private Key from [Udacity Developer Environment](https://www.udacity.com/account#!/development_environment)
* Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. ``mv ~/Downloads/udacity_key.rsa ~/.ssh/``. **Note:** In my case environment's home directory is ``tushar@dell $`` (not the virtual environment like root@[PUBLIC_IP]).
* Open your terminal and type in ``chmod 600 ~/.ssh/udacity_key.rsa``
* In your terminal, type in ``ssh -i ~/.ssh/udacity_key.rsa root@35.166.221.186``

## Add grader user (from root's VM environment)
* ``sudo adduser grader``
* ``sudo touch /etc/sudoers.d/grader``
* ``sudo nano /etc/sudoers.d/grader``
* Add: ``grader ALL=(ALL:ALL) ALL``, ``Ctrl+X, Y, Enter`` to save in nano editor.
* **Note:** In terminal you will see something like ``root@ip-10-20-17-19:~$`` where ``ip-10-20-17-19`` is your local machine IP (different for different machines), add this IP to hosts file to avoid conflict of hosts. Edit ``sudo nano /etc/hosts``, append below ``127.0.0.1 localhost`` line ``127.0.0.1 localhost ip-10-20-17-19``. Save it.
* **Note:** When you first login using ssh, we used something like ``ssh -i ~/.ssh/udacity_key.rsa root@35.166.221.186``, where ``35.166.221.186`` is your [PUBLIC\_IP] but when you are logged in to the VM, you will see ``root@ip-10-20-17-19:~$`` where ``ip-10-20-17-19`` is your [LOCAL\_IP].

## Update and Upgrade the packages
* Run ``sudo apt-get update``, followed by ``sudo apt-get upgrade``.
* **Note:** You may want this to automate, ``sudo apt-get install unattended-upgrades`` followed by ``sudo dpkg-reconfigure -plow unattended-upgrades``.

## SSH Configurations (from grader's VM environment)
* Change SSH Port from SSH (port 22 to 2200).
 * ``sudo nano /etc/ssh/sshd_config`` change port 22 to port 2200.
 * Change ``PermitRootLogin without-password`` to ``PermitRootLogin no`` to disallow root login.
 * Change PasswordAuthentication from no to yes. Change back after finishing SSH setup.
 * Append AllowUsers grader inside file to allow grader to login through SSH.
 * Save file and restart ssh service ``sudo service ssh reload``.
* Create SSH keys and copy to server manually.
 * On your local machine (in my case ``tushar@dell:$``), run ``ssh-keygen``, change file name if you want, default is ``id_rsa``, ``id_rsa.pub``, keys will be generated in local pc's ``~/.ssh`` directory.
 * Now, in grader's VM environment, create ``.ssh`` directory as ``sudo mkdir .ssh`` followed by ``sudo touch authorized_keys``. Copy the content of id_rsa.pub (in my case, linuxSetup.pub) from local machine into ``authorized_keys`` uisng ``sudo nano authorized_keys``.
 * Give permissions 644 to authorized_keys and 700 to .ssh using ``sudo chmod 700 ~/.ssh && sudo chmod 644 ~/.ssh/authorized_keys`` in grader's VM environment.
 * To login remotely into grader's account, ``ssh -i ~/.ssh/id_rsa grader@35.166.221.186`` (in my case, ``ssh -i ~/.ssh/linuxSetup grader@35.166.221.186``).

## Firewall Configurations (Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)).
* ``sudo ufw status``: UFW status to make sure its inactive.
* ``sudo ufw default deny incoming``: deny incoming by default.
* ``sudo ufw default allow outgoing``: allow outgoing by default.
* ``sudo ufw allow 2200/tcp``: allow SSH(port 2200).
* ``sudo ufw allow 80/tcp``: allow HTTP(port 80).
* ``sudo ufw allow 123/udp``: allow NTP(port 123).
* ``sudo ufw enable``: turn on firewall.

## Configure the local timezone to UTC
* ``sudo dpkg-reconfigure tzdata``, select None of the Above and then UTC.

## Install and configure Apache to serve a Python mod_wsgi application
* Setting up Apache.
 * ``sudo apt-get install apache2 libapache2-mod-wsgi python-dev git``.
 * Add ``WSGIScriptAlias / /var/www/html/catalog.wsgi`` at end, before </ VirtualHost> in ``sudo nano /etc/apache2/sites- enabled/000-default.conf``.
 * Save and restart Apache using ``sudo service apache2 restart``.
 * To enable mod_wsgi, run the following command: ``sudo a2enmod wsgi``.
* Setting Catalog Project.
 * We will place our app in the /var/www directory: ``cd /var/www ``
 * ``sudo mkdir catalog``, ``cd catalog``. Repeat again ``sudo mkdir catalog``, ``cd catalog``.
 * Install python-pip and configure virtualenv: ``sudo apt-get install python-pip``<br>
   ``sudo pip install virtualenv``<br>``sudo virtualenv venv``<br>``sudo chmod -R 777 venv``<br>``source venv/bin/activate``<br>
   ``pip install Flask``<br>``python __init__.py``<br>``deactivate``.
* Configure Virtual Host
 * ``sudo nano /etc/apache2/sites-available/catalog``
 * ``sudo nano /etc/apache2/sites-available/catalog.conf``<br>**Note:** Newer versions of Ubuntu (13.10+) require a ".conf" extension for VirtualHost files.
 * Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your PULIC\_IP:<br>
  ```
  <VirtualHost *:80>
		  ServerName ec2-35-161-139-158.us-west-2.compute.amazonaws.com
		  ServerAdmin admin@ec2-35-161-139-158.us-west-2.compute.amazonaws.com
		  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		  <Directory /var/www/FlaskApp/FlaskApp/>
			  Order allow,deny
			  Allow from all
		  </Directory>
		  Alias /static /var/www/catalog/catalog/static
		  <Directory /var/www/FlaskApp/FlaskApp/static/>
			  Order allow,deny
			  Allow from all
		  </Directory>
		  ErrorLog ${APACHE_LOG_DIR}/error.log
		  LogLevel warn
		  CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  Save and Close.
  * Enable the virtual host ``sudo a2ensite catalog``.
* Catalog Project Modifications 
 * Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/catalog directory and create a file named catalog.wsgi with following commands:
  ```
   cd /var/www/catalog
   sudo nano catalog.wsgi 
   Add the following lines of code to the catalog.wsgi file:
  ```
  
  ```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/FlaskApp/") 
   
   from FlaskApp import app as application
   application.secret_key = 'Add your secret key'
  ```
 * Clone catalog project repository into ``/var/www/catalog/catalog``.
 * Dirctory structure should looks like this:<br>
   (Current Working Directory is ``/var/www``)<br>
   ├── catalog<br>
   │   ├── catalog (cloned repo from github)<br>
   │   │   ├── client_secrets.json (To be modified for JavaScript Origins)<br>
   │   │   ├── database_setup.py (To be modified for db connectons)<br>
   │   │   ├── fb_client_secrets.json<br>
   │   │   ├── \_\_init\_\_.py (To be modified for db connectons)<br>
   │   │   ├── static<br>
   │   │   ├── templates<br>
   │   │   ├── tempmusic.py (To be modified for db connectons)<br>
   │   │   └── venv<br>
   │   └── catalog.wsgi<br>
   └── html (ignore)<br>
  * **Note:** Here ``\_\_init\_\_.py`` was previously named ``project.py``, I moved contents of ``project.py`` to ``\_\_init\_\_.py`` using, ``sudo mv project.py \_\_init\_\_.py``.
  * Restart Apache: ``sudo service apache2 restart``
    **Note:** You may see a message similar to the following:<br>
    Could not reliably determine the VPS's fully qualified domain name, using 127.0.0.1 for ServerName<br>
    
    This message is just a warning, and you will be able to access your virtual host without any further issues. To view    
    your application, open your browser and navigate to the domain name or IP address that you entered in your virtual host     configuration.

## Installing python dependencies and configuring PostgreSql
* Install python dependencies
 * ``source venv/bin/activate``.
 * ``sudo pip install httplib2 requests oauth2client sqlalchemy Flask-SQLAlchemy``.
 * ``sudo apt-get install python-psycopg2``.
 * **Note:** Install other 3rd party dependencies used in project.
* Install PostgreSql
 * ``sudo apt-get install postgresql postgresql-contrib``.
 * Make the changes in ``\_\_init\_\_.py, database_setup.py and tempmusic.py`` to configure PostgreSql connections.<br>
   ``engine = create_engine('postgresql://[database_name]:[database_password]@localhost/catalog')``.
 * Add catalog user: ``sudo adduser catalog``.
 * Login as postgres super user: ``sudo su - postgres``.
* Postgres Configuration
 * Enter postgrespsql using: ``psql``.
 * Create user catalog: ``CREATE USER catalog WITH PASSWORD 'db-password';``
 * Change role of user catalog: ``ALTER USER catalog CREATEDB;``
 * List all users and roles to verify: ``\du``
 * Create new DB "catalog" with owner catalog: ``CREATE DATABASE catalog WITH OWNER catalog;``
 * Connect to database: ``\c catalog``
 * Revoke all rights: ``REVOKE ALL ON SCHEMA public FROM public;``
 * Give access only to catalog: ``GRANT ALL ON SCHEMA public TO catalog;``
 * Quit postgres: ``\q``
 * Logout from postgres super user: ``exit``
 * Setup your database schema ``python database_setup.py``
 * Setup init values to database using: ``python tempmusic.py``
 * Restart apache ``sudo service apache2 restart``
* **Note:** I got INTERNAL SERVER ERROR when opened on public ip of aws.<br>Check logs of Apache using: ``sudo cat /var/log/apache2/error.log``<br>
 ```
  [Mon Jan 30 17:45:06.474646 2017] [:error] [pid 16874:tid 140519943501568] [client 103.211.15.229:29408]     CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web', referer: http://ec2-35-161-139-158.us-west-2.compute.amazonaws.com/
 ```<br>
 I added static path (/var/www/catalog/catalog/[json_file]) to the files where ``client_secrets.json`` and ``fb_client_secrets.json`` are located (in my case ``\_\_init\_\_.py``).
* **Note:** Change JavaScript Origins in both Google and Facebook Oauth.

## Sources if any:
* Digital Ocean VPS Flask Hosting [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](Tutorial)
* Udacity Discussion Forums.










