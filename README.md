# Linux Server Configuration

The instructions are written specifically for hosting an app called [Item-catalog-Application](https://github.com/GhadaMagdy/Item-catalog-Application) on an Amazon EC2 instance but can easily be adapted to work for an alternative application and/or server provider.


## 1. Details specific to the server I set up
The IP address is 35.180.174.9.

The SSH port used is `2200`.

The URL to the hosted webpage is: http://35.180.174.9/ or http://ec2-35-180-174-9.eu-west-3.compute.amazonaws.com/.


## 2. Software to install during the configuration
- Apache2
- mod_wsgi
- PostgreSQL
- git
- pip
- virtualenv
- httplib2
- Python Requests
- oauth2client
- SQLAlchemy
- Flask
- libpq-dev
- Psycopg2


## 3. Configuration steps
### Create an instance with Amazon Lightsail
1. Sign in to [Amazon EC2](https://eu-west-3.console.aws.amazon.com/ec2/v2/home?region=eu-west-3#Home:) using an Amazon Web Services account

1. Follow the 'Launch instance' button

1. Choose the Select Ubuntu Server 16.04 LTS (HVM), SSD Volume Type

1. follow launch steps

1. in launch last step You have to download the private key file (*.pem file) before you can continue. Store it in a secure and accessible location. You will not be able to download the file again after it's created

1. Wait for the instance to start up

### Connect to the instance on a local machine
Note: The following steps outline how to connect to the instance via your Terminal program 

1. Copy you private key and put it the local ~/.ssh/ directory you can find it in "C:\Users\user_name\.ssh"

1. Run `chmod 600 ~/.ssh/lightrail_key.rsa`

1. Log in with the following command: `ssh -i ~/.ssh/private_key.pem ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance 

### Upgrade currently installed packages
1. Notify the system of what package updates are available by running `sudo apt-get update`

1. Download available package updates by running `sudo apt-get upgrade`

### Create a new user named `grader`
1. Run `sudo adduser grader`

1. Enter in a new UNIX password (twice) when prompted

1. Fill out information for the new `grader` user

1. To switch to the `grader` user, run `su - grader`, and enter the password

### Give `grader` user sudo permissions
1. Run `sudo visudo`

1. Search for a line that looks like this:

	`root    ALL=(ALL:ALL) ALL`

1. Add the following line below this one:

	`grader	   ALL=(ALL:ALL) ALL`

1. Save and close the visudo file

### Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine( it will create two files of key one of them .pub in the the directory ~/.ssh)

1. Choose a file name for the key pair (such as grader_key)

1. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

1. Log in to the virtual machine

1. switch to the `grader` user, run `su - grader`, and enter the password, and create a new directory called `.ssh` (run `mkdir .ssh`)

1. Run `touch .ssh/authorized_keys`

1. On the local machine, open file.pub (which is saved in ~/.ssh directory) and copy its content

1. Run `nano authorized_keys`

1. paste the contents of the file.pub in authorized_keys file on the virtual machine

1. Run `sudo chmod 700 /home/grader/.ssh` on the virtual machine

1. Run `sudo chmod 644 /home/grader/.ssh/authorized_keys` on the virtual machine

1. Finally change the owner from root to grader `sudo chown -R grader:grader /home/grader/.ssh`

1. open ne terminal and login with grader by `ssh -i ~/.ssh/grader_key grader@XX.XX.XX.XX` (grader_key is the second file generated from run command ssh-keygen)

1. Run `sudo nano /etc/ssh/sshd_config` (you may asked to do that using root you can switch to root by `sudo su -`)

1. Force key-based authentication by change the line in sshd_config 'PasswordAuthentication yes' from yes to no

1. Prevent root from login by change the line 'PermitRootLogin' to no

1. ctrl + x -> y -> enter to save and exit sshd_config file

1. Restart ssh service `sudo service ssh restart`

### Change ssh port to 2200
1. Run `sudo nano /etc/ssh/sshd_config` (you may asked to do that using root you can switch to root by `sudo su -`)

1. find line Port 22

1. change to Port 2200

1. ctrl + x -> y -> enter to save and exit sshd_config file

1. Restart ssh service `sudo service ssh restart`

### Configure the firewall
1. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running `sudo ufw status`

1. Run `sudo ufw default deny incoming` to set the ufw firewall to block everything coming in

1. Run `sudo ufw default allow outgoing` to set the ufw firewall to allow everything outgoing

1. Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port `2200` so that SSH will work

1. Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server

1. Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP

1. Run `sudo ufw deny 22` to deny the default port of ssh

1. Run `sudo ufw enable` to enable the ufw firewall

1. Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

	```
	To                         Action      From
	--                         ------      ----
	22                         DENY        Anywhere
	2200/tcp                   ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	123/udp                    ALLOW       Anywhere
	22 (v6)                    DENY        Anywhere (v6)
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	123/udp (v6)               ALLOW       Anywhere (v6)
	```

### Configure the local timezone to UTC
1. Run `sudo dpkg-reconfigure tzdata`, and follow the instructions (UTC is under the 'None of the above' category)

1. Test to make sure the timezone is configured correctly by running`date`

### Install and configure Apache
1. Run `sudo apt-get install apache2` to install Apache

1. Start the web server with `sudo service apache2 start`

1. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load


### Install mod_wsgi
1. Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

1. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`

### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. Install PostgreSQL by running `sudo apt-get install postgresql`

### Intsall python 
1. `sudo apt-get install python-pip`

### Create a new PostgreSQL user named `catalog` with limited permissions and database named `catalog`
1. switch to postgres user `sudo su - postgres`

1. Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

1. Create database catalog   postgres=# `CREATE DATABASE catalog;`

1. Create the `catalog` user by running `CREATE ROLE catalog;`

1. Give user password `ALTER ROLE catalog WITH PASSWORD 'password';`
    
1. Give catalog user permission to catalog database `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

1. Exit psql by running `\q`

1. Switch back to the `ubuntu` user by running `exit`

### Install git and clone the catalog project
1. Run `sudo apt-get install git`

1. move to `cd /var/www`

1. Run `sudo mkdir catalog`

1. Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`

1. Move to catalog `cd /catalog`

1. Clone your project from github `git clone https://github.com/GhadaMagdy/Item-catalog-Application`

1. Create catalog.wsgi `touch catalog.wsgi` in directory /var/www/catalog

1. Add the following in catalog.wsgi file
    activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
    execfile(activate_this, dict(__file__=activate_this))

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application

1. then ctrl + x --> y --> enter to save

1. Move to you cloned project `Item-catalog-Application`

1. change your application.py file name to __init__.py using `mv application.py __init__.py`

1. Edit database_setup.py, __init__.py and database_init.py and change engine = create_engine('sqlite:///toyshop.db') to engine = create_engine    ('postgresql://catalog:password@localhost/catalog')

1. Back to catalog `cd ..` and change folder name Item-catalog-Application to catalog `mv Item-catalog-Application catalog`

### Set up a vitual environment and install dependencies
1. Start by installing pip (if it isn't installed already) with the following command:

	`sudo apt-get install python-pip`

1. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

1. Change to the /var/www/catalog/catalog/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running `virtualenv venv` (make sure to _not_ use `sudo` here as it can cause problems later on)

1. Activate the new environment, `venv`, by running `. venv/bin/activate`

1. With the virtual environment active, install the following dependenies:
    `pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev` (Note: this will install to the global evironment) or you can use `sudo apt-get install libpq-dev`

	`pip install psycopg2` (you may need to install it by `sudo apt-get -qqy install postgresql python-psycopg2`)

1. Now you can create your database schema by `python database_setup.py`

1. Deactivate the virtual environment by running `deactivate`

### Set up and enable a virtual host
1. Create a file in /etc/apache2/sites-available/ called catalog.conf
     `sudo nano /etc/apache2/sites-available/catalog.conf`
1. Add the following into the file:

    ```
    <VirtualHost *:80>
        ServerName 35.180.174.9
        ServerAlias ec2-35-180-174-9.eu-west-3.compute.amazonaws.com        
        ServerAdmin admin@35.180.174.9
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
    Note:  ServerName YOUR_PUBLIC_IP
           ServerAlias YOUR_Public DNS (IPv4)

1. Run `sudo a2ensite catalog` to enable the virtual host
    The following prompt will be returned:

	```
	Enabling site nuevoMexico.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```
1. Run `sudo service apache2 reload`

### Disable the default Apache site
1. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`

	The following prompt will be returned:

	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

1. Run `sudo service apache2 reload` 

1. Now open up a browser and check to make sure the app is working by going to http://XX.XX.XX.XX or http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com

### 5. A few helpful command to know

`sudo tail /var/log/apache2/error.log`<br>
View Apache error logs, and open the file starting with the last line (note: vi can be replaced here with nano or another text editor)

##Fixing Error opening file 'client_secrets.json': No such file or directory
1. Put static variable contains the path of that file (appPath='/var/www/catalog/catalog')
    for example in __init__.py it will be 
    ```
    appPath = '/var/www/catalog/catalog/'
    CLIENT_ID = json.loads(
        open(appPath + 'client_secrets.json', 'r').read())['web']['client_id']
    APPLICATION_NAME = "Item-catalog-Application"

    ```
###Fixng the session is unavailable because no secret key was set. Set the secret_key on the application to something unique and secret.
1. You should put unique secret key to you application as the following in __init__.py

    ```
    app = Flask(__name__)
    app.secret_key = os.urandom(24)

    ```
    and 
    ```
    if __name__ == '__main__':
    app.secret_key =  os.urandom(24)

    ```
###Fixing can't read module 'module_name'
1. when run `sudo tail /var/log/apache2/error.log` you may find error no module named 'flask' or 'sqlalchemy' ..etc
         this mean we need to install these module grobally 
         Run `sudo -H pip install package_name`
