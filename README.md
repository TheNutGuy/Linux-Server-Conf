### Linux Server Configuration Project
A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

## Tasks
1. Launch your Virtual Machine with your Udacity account
2. Follow the instructions provided to SSH into your server
3. Create a new user named grader
4. Give the grader the permission to sudo
5. Update all currently installed packages
6. Change the SSH port from 22 to 2200
7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
8. Configure the local timezone to UTC
9. Install and configure Apache to serve a Python mod_wsgi application
10. Install and configure PostgreSQL:
	- Do not allow remote connections
	- Create a new user named catalog that has limited permissions to your catalog application database
11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

## Instructions for SSH access to the instance
1. Download Private Key
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory).
3. Open your terminal and type in
	```chmod 600 ~/.ssh/<PRIVATE_KEY_FILE_NAME>```
4. In your terminal, type in
	```ssh -i ~/.ssh/<PRIVATE_KEY_FILE_NAME> ubuntu@35.176.130.86```

## Update all packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
Enable automatic security updates
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

## Create User
Next we need to create a new user, `grader` to give the Udacity grader access to check our server.
```
sudo adduser grader
```
Next we give grader acces to sudo.
```
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
```
Next thing is to generate an SSH key pair for the user `grader`; I did that locally using the following commands:
```
ssh-keygen
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): grader

A private key, grader and public grader.pub are created in my directory ~/.ssh/ after which I ran: ```less grader.pub``` and copied it's contents.

Next I did the following.
```
sudo nano ../grader/.ssh/authorized_keys
```
I then pasted the entire public key, saved, exited and completed with following commands:
```
sudo chmod 700 ../grader/.ssh/
sudo chmod 644 ../grader/.ssh/authorized_keys
sudo chown grader ../grader/.ssh/
sudo chgrp grader ../grader/.ssh/
sudo chown grader ../grader/.ssh/authorized_keys
sudo chgrp grader ../grader/.ssh/authorized_keys
```
Now you can log in with grader.
```
ssh -i ~/.ssh/<PRIVATE_KEY_FILE_NAME> grader@35.176.130.86
```

## Safety
The next step which is good to do early on to secure and change your ports if necessary.
1. Edit your SSH Config.
  *Type following command in terminal```sudo nano /etc/ssh/sshd_config```
  * Look for "Port 22" and change to "Port 2200"
  * look for PasswordAuthentication yes and change to no
  * look for PermitRootLogin and change it to no.

2. Check your UFW status: ```sudo ufw status``` (it should be 'inactive') and then run the following commands:
  ```
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2200/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 123/udp
  sudo ufw allow www
  sudo ufw deny 22
  sudo ufw enable
  ```

3. Configure the Lightsail firewall in your control panel to match the ufw ports you've allowed.

4. Exit and restart your SSH session, this time using port 2200.
  ```
  ssh -i ~/.ssh/<PRIVATE_KEY_FILE_NAME> -p 2200 ubuntu@35.176.130.86
  ```

## *Special* Configure fail2ban to monitor unsuccessful login attempts
Type the following in your terminal.
```
sudo apt-get install fail2ban sendmail
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```
Then find and update the following:
```
destemail = [my email address]
action = %(action_mwl)s
[ssh]
banaction = ufw-ssh
port = 2200
```
Save and exit.
Create the ufw-ssh action referenced above:
```
sudo nano /etc/fail2ban/action.d/ufw-ssh.conf
```
Add the following:
```
[Definition]
actionstart =
actionstop =
actioncheck =
actionban = ufw insert 1 deny from 35.176.130.86 to any app OpenSSH
actionunban = ufw delete deny from 35.176.130.86 to any app OpenSSH
```
Finally, restart fail2ban:
```
sudo service fail2ban restart
```

## Install python
python3 comes default with the application to install python 2 run the following command:
```
sudo apt install python
```

## Install apache2
```
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
sudo service apache2 restart
```

## PostgreSQL
Next install PostgreSQL:
```
sudo apt-get install postgresql
```
I then ran the following commands to log into the postgres user,I then created the database and user to which I assigned privileges to peform operations on the database:
```
sudo su - postgres
psql
```
postgres=# ```create database catalog;```
postgres=# ```create user catalog with encrypted password 'password';```
postgres=# ```grant all privileges on database catalog to catalog;```
postgres=# ```\q```

## Install and using clone
1. Install Git using ```sudo apt-get install git```
2. Use ```cd /var/www``` to change to the correct directory
3. remove the html file ```sudo rm html```
4. Clone the catalog project to the virtual machine ```git clone https://github.com/TheNutGuy/item-catalog.git```
5. Rename the file by using ```sudo mv /var/www/<filename> /var/www/html```
   then type ```sudo chown -R grader:grader html/```
6. Clone requirements.txt to the html file ```git clone https://github.com/TheNutGuy/requirements.git```
7. Install pip3 `sudo apt-get install python3-pip`
8. Use pip to install the requirements file `sudo pip3 install -r requirements.txt`
9. Install the following
    ```
    sudo apt-get -qqy install postgresql python-psycopg2
    sudo pip3 install sqlalchemy
    sudo pip3 install httplib2
    sudo apt-get install python-psycopg2
    sudo apt-get install libpq-dev
    sudo pip3 install flask
    sudo pip3 install flask-httpauth
    sudo pip3 install flask-sqlalchemy
    sudo pip3 install requests
    sudo pip3 install --upgrade oauth2client
    sudo pip3 install psycopg2
    ```
## Configure and Enable a New Virtual Host
1. Create catalog.conf ```sudo nano /etc/apache2/sites-available/catalog.conf```
2. Add the following lines of code to the file to configure the virtual host. 
	<VirtualHost *:80>
		ServerName 35.176.130.86
		ServerAdmin tristannutman@gmail.com
		WSGIScriptAlias / /var/www/html/myapp.wsgi
		<Directory /var/www/html>
			Order allow,deny
			Allow from all
		</Directory>
		<Directory /var/www/html/static>
			Order allow,deny
			Allow from all
		</Directory>
		<Directory /var/www/html/templates>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
save and close
Type the following to activate the virtual host
    ```
    sudo a2dissite /etc/apache2/sites-available/000-default.conf
    sudo a2ensite /etc/apache2/sites-available/catalog.conf
    ```

## Create WSGI file
1. Type ```cd /var/www/html```
2. Create file with ```sudo nano /var/www/html/myapp.wsgi```
3. Type the following
    #!/usr/bin/env python
    import sys
    sys.path.insert(0, "/var/www/html")
    from app import app as application
    application.secret_key = 'super_secret_key'
save and close

## *Special* Configure mod_wsgi to work with python 3.6
Type in terminal
```
sudo apt install build-essential
sudo apt install libssl-dev zlib1g-dev libncurses5-dev libncursesw5-dev libreadline-dev libsqlite3-dev
sudo apt install libgdbm-dev libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev tk-dev
wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tar.xz
tar xf Python-3.6.2.tar.xz
cd Python-3.6.2
./configure --enable-shared --enable-optimizations
```
Install the mod_wsgi 4.5.18 or latest version
```
wget "https://github.com/GrahamDumpleton/mod_wsgi/archive/4.5.18.tar.gz"
tar xf 4.5.18.tar.gz
cd mod_wsgi-4.5.18
./configure --with-python=/usr/local/bin/python3.6
```

## Edit the project files to be able to communicate to the web server
# Database_setup.py
	- find and replace "engine = create_engine('sqlite:///restaurantmenu.db')" with "engine = create_engine('postgresql://catalog:password@localhost/catalog')"
	- CTRL+X and Y to save and exit 
# lotsofmenus.py
	- find and replace "engine = create_engine('sqlite:///restaurantmenu.db')" with "engine = create_engine('postgresql://catalog:password@localhost/catalog')"
	- CTRL+X and Y to save and exit 
# project.py
	- find and replace "engine = create_engine('sqlite:///restaurantmenu.db')" with "engine = create_engine('postgresql://catalog:password@localhost/catalog')"
	- find and replace "open('client_secret.json', 'r').read())['web']['client_id']" with "open('/var/www/html/client_secrets.json', 'r').read())['web']['client_id']"
	- find in gconnect and replace "oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')" with "oauth_flow = flow_from_clientsecrets('/var/www/html/client_secrets.json', scope='')"
    - find in gconnect "result = json.loads(h.request(url, 'GET')[1])" Change to
    "result = json.loads((h.request(url, 'GET')[1]).decode())"
	- Delete the following lines:
    app.secret_key = 'supper secret key'
    app.run(debug=True, host='0.0.0.0', port=5000)
	- in if __name__ == '__main__': add app.run()
	- CTRL+X and Y to save and exit 

## Setup the restaurant database
Run the following commands:
```
cd /var/www/html
sudo python3 database_setup.py
```
Adjust the id column to the serial type, type the following commands:
```
sudo -u postgres psql
```
    \c restaurantdb
    alter table Item drop column id cascade;
    alter table Category drop column id cascade;
    alter table Category add column id serial primary key;
    alter table Item add column id serial primary key;
    \q
Fill the database with
```
sudo python3 DataAdd.py
```

## Reload & Restart Apache Server
```
sudo service apache2 reload
sudo service apache2 restart
sudo service ssh restart
```

## Finished!
You can now connect by using the ip "35.176.130.86.xip.io" in your browser!
