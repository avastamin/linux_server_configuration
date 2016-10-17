# Linux-Server-Configuration

A baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system  and installing/configuring web and database servers(postresql).


**Note to be considered:**
 Below, I described step by step solution and also I ponited out what kind of problem I faced and it's solution. I configured my existing
 [catalog application][1]
 which can be accessed here:  [Public link to the site][23]

## Step by Step guidelines
In the detailed guide below, I tried to briefly document the steps I used to run my application. I also tried to write all in order so that,
it's become more easier for you. Also I meantions, the problem I faced and it's solutions

### 1. Instructions for SSH access to the instance
Source: [Udacity][2]

1. Create new development environment (if it's not created already).
2. Download private keys and write down your public IP address.
3. Move the private key file into the folder ~/.ssh:
  `$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
4. Set file rights (only owner can write and read.):
  `$ chmod 600 ~/.ssh/udacity_key.rsa`
5. SSH into the instance:
  `<pre>$ ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS`

### 2. User Management: Create a new user(catalog) and give user the permission to sudo
Source: [DigitalOcean][3]

1. Create a new user:
  `$ adduser catalog`
2. Give new user the permission to sudo
  1. Open the sudo configuration:
    `$ visudo`
  2. Add the following line below `root ALL...`:
    `catalog ALL=(ALL:ALL) ALL`
 3. To details about the use I sued `finger grader`
 Source: [askubuntu][4]
   1. To install finger `sudo apt-get install finger`
   2. To see details about user `grader` use `finger grader`

### 3.1 - Update and upgrade all currently installed packages
Source: [Ask Ubuntu][5]

   1. Update the list of available packages and their versions:
    `$ sudo apt-get update`
   2. Install newer vesions of packages you have:
   `$ sudo sudo apt-get upgrade`

### 3.2 - Change the SSH port from 22 to 2200 and configure SSH access
Source: [Ask Ubuntu][6]
Source: [Digitalocean][22]

1. Change ssh config file:
  1. Open the config file:
    `$ sudo nano /etc/ssh/sshd_config`
  2. Change to Port 2200.
  3. To see by Rule: `sudo ufw status numbered` to delete port 22 `sudo ufw delete 1`(for me 1 was port 22)
  4. Change `PermitRootLogin` from `without-password` to `no`.
  5. Temporalily change `PasswordAuthentication` from `no` to `yes`.
  6. Append `UseDNS no`.
  7. Append `AllowUsers catalog`.
2. Restart SSH Service:
  `$ /etc/init.d/ssh restart` or `# service sshd restart`
3. Create SSH Keys:
  Source: [DigitalOcean][7]

  1. Generate a SSH key pair on the local machine(from your computer):
    `$ ssh-keygen`
  2. Copy the public id to the server:
    `$ ssh-copy-id username@remote_host -p**_PORTNUMBER_**`
  3. Login with the new user:
    `$ ssh -v grader@PUBLIC-IP-ADDRESS -p2200`
  4. Open SSHD config:
    `$ sudo nano /etc/ssh/sshd_config`
  5. Change `PasswordAuthentication` back from `yes` to `no`.
4. *Get rid of the warning message `sudo: unable to resolve host ...` when sudo is executed:
Source: [Ask Ubuntu][8]

  1. Open `$ sudo nano /etc/hostname`.
  2. Copy the hostname.
  3. Append the hostname to the first line:
    `$ sudo nano /etc/hosts`
     it should looks like
     127.0.0.1 localhost
     127.0.0.1 ip-10-20-31-29
     # The following lines are desirable for IPv6 capable hosts
     ::1 ip6-localhost ip6-loopback
     fe00::0 ip6-localnet
     ff00::0 ip6-mcastprefix
     ff02::1 ip6-allnodes
     ff02::2 ip6-allrouters
     ff02::3 ip6-allhosts`


### 4 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port   123)
Source: [Ubuntu documentation][9]

1. Turn UFW on with the default set of rules:
  `$ sudo ufw enable`
2. *Check the status of UFW:
  `$ sudo ufw status verbose`
3. Allow incoming TCP packets on port 2200 (SSH):
  `$ sudo ufw allow 2200/tcp`
4. Allow incoming TCP packets on port 80 (HTTP):
  `$ sudo ufw allow 80/tcp`
5. Allow incoming UDP packets on port 123 (NTP):
  `$ sudo ufw allow 123/udp`

### 5 - Configure the local timezone to UTC
Source: [Ubuntu documentation][10]

1. Open the timezone selection dialog:
  `$ sudo dpkg-reconfigure tzdata`
2. Then chose 'None of the above', then UTC.

### 6 - Install and configure Apache to serve a Python mod_wsgi application
Source: [Udacity][11]

1. Install Apache web server:
  `$ sudo apt-get install apache2`
2. Open a browser and open your public ip address, e.g. http://54.190.18.187/ - It should  say 'It works!' on the top of the page.
3. Install **mod_wsgi** for serving Python apps from Apache and the helper package **python-setuptools**:
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
4. Restart the Apache server for mod_wsgi to load:
  `$ sudo service apache2 restart`

### 7 - Install git, clone and setup your Catalog App project
As this is by far the biggest project task, it is split in several parts.
#### 7.1 - Install and configure git
Source: [GitHub][12]

1. Install Git:
  `$ sudo apt-get install git`
2. Set your name, e.g. for the commits:
  `$ git config --global user.name "YOUR NAME"`
3. Set up your email address to connect your commits to your account:
  `$ git config --global user.email "YOUR EMAIL ADDRESS"`

#### 7.2 - Setup for deploying a Flask Application on Ubuntu VPS
Source: [DigitalOcean][13]

1. Extend Python with additional packages that enable Apache to serve Flask applications:
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi (if not already enabled):
  `$ sudo a2enmod wsgi`
3. Create a Flask app:
  1. Move to the www directory:
    `$ cd /var/www`
  2. Setup a directory for the app, e.g. catalog:
    1. `$ sudo mkdir catalog`
    2. `$ cd catalog` and `$ sudo mkdir catalog`
    3. `$ cd catalog` and `$ sudo mkdir static templates`
    4. Create the file that will contain the flask application logic:
      `$ sudo nano __init__.py`
    5. Paste in the following code:
    ```python
      from flask import Flask
      app = Flask(__name__)
      @app.route("/")
      def hello():
        return "Here is your first application!!"
      if __name__ == "__main__":
        app.run()
    ```
4. Install Flask
  1. Install pip installer:
    `$ sudo apt-get install python-pip`
  2. Install virtualenv:
    `$ sudo pip install virtualenv`
  3. Set virtual environment to name 'venv':
    `$ sudo virtualenv venv`
  4. Enable all permissions for the new virtual environment (no sudo should be used within):
    Source: [Stackoverflow][14]
    `$ sudo chmod -R 777 venv`
  5. Activate the virtual environment:
    `$ source venv/bin/activate`
  6. Install Flask inside the virtual environment:
    `$ pip install Flask`
  7. Run the app:
    `$ python __init__.py`
  8. Deactivate the environment:
    `$ deactivate`
5. Configure and Enable a New Virtual Host#
  1. Create a virtual host config file
    `$ sudo nano /etc/apache2/sites-available/catalog.conf`
  2. Paste in the following lines of code and change names and addresses regarding your application:
  ```
    <VirtualHost *:80>
        ServerName PUBLIC-IP-ADDRESS
        ServerAdmin admin@PUBLIC-IP-ADDRESS
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
  3. Enable the virtual host:
    `$ sudo a2ensite catalog`
6. Create the .wsgi File and Restart Apache
  1. Create wsgi file:
    `$ cd /var/www/catalog` and `$ sudo nano catalog.wsgi`
  2. Paste in the following lines of code:
  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'Add your secret key'
  ```
  7. Restart Apache:
    `$ sudo service apache2 restart`

#### *** Problem I faced:
Source: [stackoverflow][15]

I got an error regarding, PYTHON_EGG_CACHE environment. To solve that problem, I had to add following code into catalog.wsgi
 1. `import os`
 2. `os.environ['PYTHON_EGG_CACHE'] = '/tmp'`


#### 7.3 - Clone GitHub repository and make it web inaccessible
1. Clone project 3 solution repository on GitHub:
  `$ git clone https://github.com/avastamin/python_catalogApp.git`
2. Move all content of created python_catalogApp directory to `/var/www/catalog/catalog/`-directory and delete the empty directory.
3. Make the GitHub repository inaccessible:
  Source: [Stackoverflow][16]
  1. Create and open .htaccess file:
    `$ cd /var/www/catalog/` and `$ sudo nano .htaccess`
  2. Paste in the following:
    `RedirectMatch 404 /\.git`

#### 7.4 - Install needed modules & packages
1. Activate virtual environment:
  `$ source venv/bin/activate`
2. Install httplib2 module in venv:
  `$ pip install httplib2`
3. Install requests module in venv:
  `$ pip install requests`
4. Install oauth2client.client:
  `$ sudo pip install --upgrade oauth2client`
5. Install SQLAlchemy:
  `$ sudo pip install sqlalchemy`
6. Install the Python PostgreSQL adapter psycopg:
  `$ sudo apt-get install python-psycopg2`

### 8.1 - Install and configure PostgreSQL
Source: [DigitalOcean][17]

1. Install PostgreSQL:
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Check that no remote connections are allowed (default):
  `$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Open the database setup file:
  `$ sudo vim database_setup.py`
4. Change the line starting with "engine" to (fill in a password):
  ```python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')```
5. Change the same line in application.py respectively
6. Rename application.py:
  `$ mv application.py __init__.py`
7. Create needed linux user for psql:
  `$ sudo adduser catalog` (choose a password)
8. Change to default user postgres:
  `$ sudo su - postgres`
9. Connect to the system:
  `$ psql`
10. Add postgres user with password:
  1. Create user with LOGIN role and set a password:
    `# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB';` (# stands for the command prompt in psql)
  2. Allow the user to create database tables:
    `# ALTER USER catalog CREATEDB;`
  3. *List current roles and their attributes:
    `# \du`
11. Create database:
  `# CREATE DATABASE catalog WITH OWNER catalog;`
12. Connect to the database catalog
  `# \c catalog`
13. Revoke all rights:
  `# REVOKE ALL ON SCHEMA public FROM public;`
14. Grant only access to the catalog role:
  `# GRANT ALL ON SCHEMA public TO catalog;`
15. Exit out of PostgreSQl and the postgres user:
  `# \q`, then `$ exit`
16. Create postgreSQL database schema:
  $ python database_setup.py

#### 8.2 - Run application
 1. Restart Apache:
  `$ sudo service apache2 restart`
 2. Open a browser and put in your public ip-address as url, e.g. 54.190.18.187 - if everything works, the application should come up
 3. *If getting an internal server error, check the Apache error files:
  Source: [A2 Hosting][18]
  1. View the last 20 lines in the error log:
    `$ sudo tail -20 /var/log/apache2/error.log`
### Problem I faced:
  1. If a file like 'g_client_secrets.json' couldn't been found:
    Source: [Stackoverflow][19]
    it means, you path problem. This is how I solved it.
    `APP_PATH = '/var/www/catalog/catalog/'
    CLIENT_ID = json.loads(
    open(APP_PATH +'client_secrets.json', 'r').read())['web']['client_id']
    APPLICATION_NAME = "Catalog Application"`
   2. I was getting loging error using gplus. then I need to change
   from 'login_session["credentials"] = credentials'
   to
   'login_session["credentials"] = credentials.access_token'

#### 8.3 - Get OAuth-Logins Working
  Source: [Udacity][20] and [Apache][21]

1. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 54.190.18.187, its ec2-54-190-18-187.us-west-2.compute.amazonaws.com
2. Open the Apache configuration files for the web app:
  `$ sudo nano /etc/apache2/sites-available/catalog.conf`
3. Paste in the following line below ServerAdmin:
    `ServerAlias HOSTNAME`, e.g. ec2-54-190-18-187.us-west-2.compute.amazonaws.com
4. Enable the virtual host:
  `$ sudo a2ensite catalog`
5. To get the Google+ authorization working:
  1. Go to the project on the Developer Console: https://console.developers.google.com/project
  2. Navigate to APIs & auth > Credentials > Edit Settings
  3. add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-54-190-18-187.us-west-2.compute.amazonaws.com/
6. To get the Facebook authorization working:
  1. Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
  2. Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
  3. To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

[1]: https://github.com/avastamin/python_catalogApp "GitHub repository of an item catalog web app using Python programming"
[2]: https://www.udacity.com/account#!/development_environment "Instructions for SSH access to the instance"
[3]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[4]: http://askubuntu.com/questions/359821/installing-finger-on-ubuntu "Installing finger on ubuntu"
[5]: http://askubuntu.com/questions/414098/will-apt-get-upgrade-upgrade-all-my-installed-software "update installed packages on ubuntu"
[6]: http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server"
[7]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server "How To Configure SSH Key-Based Authentication on a Linux Server"
[8]: http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none "Error message when I run sudo: unable to resolve host (none)"
[9]: https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall"
[10]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management"
[11]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html "A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu"
[12]: https://help.github.com/articles/set-up-git/#platform-linux "Set Up Git for Linux"
[13]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How To Deploy a Flask Application on an Ubuntu VPS"
[14]: http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip "python packages not installing in virtualenv using pip"
[15]: http://stackoverflow.com/questions/24230950/cant-extract-files-to-egg-cache-error-in-gae "to solve particular problem on PYTHON_EGG_CACHE"
[16]: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible "Make .git directory web inaccessible"
[17]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps "How To Secure PostgreSQL on an Ubuntu VPS"
[18]: https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files "How to view Apache log files"
[19]: http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory "Python: No such file or directory"
[20]: http://discussions.udacity.com/t/oauth-provider-callback-uris/20460 "OAuth Provider callback uris"
[21]: http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html "Name-based Virtual Host Support"
[22]: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04 "How To Set Up a Firewall with UFW on Ubuntu 14.04"
[23]: http://ec2-54-190-18-187.us-west-2.compute.amazonaws.com/  "Public link of this project" 
