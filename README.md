# Linux_server_configuration
A baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

# user_info
IP address: 13.126.24.32 <br>
SSH port: 2200 <br>
URL of Hosted WEB APPLICATION: http://ec2-13-126-24-32.ap-south-1.compute.amazonaws.com <br>

# steps to configure linux based server for this project
## 1. Amazon lightsail server setup(Get Your server)
* First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
* Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
* For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
* The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine. And as long as you complete the project within a month and shut your instance down, the price will be zero.
* Give a instance hostname.
* create an instance.
* wait for it to startup.
* connect to ssh. you'll be logged as the ubuntu user. When you want to execute commands as root, you'll need to use the sudo command to do it. 

## 2. Create user grader and Give grader Access
* connect to ssh.
* Add a new user called grader. ```$ sudo add user grader```.
* give grader permission to sudo: <br>
a) Create a new file under the suoders directory: ```$ sudo touch /etc/sudoers.d/grader```. <br>
b) edit the file with ```$sudo nano /etc/sudoers.d/grader``` put the following text ```grader ALL=(ALL:ALL) ALL```, then save it.
* In order to prevent the "sudo: unable to resolve host" error, edit the hosts: <br>
a) ```$ sudo nano /etc/hosts```. <br>
b) Add the host under : ```127.0.1.1 ip-10-20-37-65``` under ```127.0.1.1:localhost```.

## 3. To secure server <br>
3.1) Configure the key-based authentication for grader user
* Generate an encryption key on your local machine with: ```$ ssh-keygen -f ~/.ssh/finalproject```. 
* connect to ssh and create the following file: ```$ sudo touch /home/grader/.ssh/authorized_keys```.
* Copy the content of the finalproject.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you just created on the remote VM. Then change some permissions: <br>
a) ```$ sudo chmod 700 /home/grader/.ssh```. <br>
b) ```$ sudo chmod 644 /home/grader/.ssh/authorized_keys```. <br>
* Finally change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh.
* Now you are able to log into the remote VM through ssh with the following command: ```$ ssh -i ~/.ssh/finalproject grader@13.126.24.32```.

3.2) Enforcekey based Authentication: <br>
a) ```$ sudo nano /etc/ssh/sshd_config```. Find the PasswordAuthentication line and edit it to no. <br>
b) ```$ sudo service ssh restart```.
<br><br>
3.3) Disable ssh login for root user, as required by Udacity: ```$ sudo nano /etc/ssh/sshd_config```. Find the PermitRootLogin line and edit to no. Restart ssh ```$ sudo service ssh restart``` <br>

3.4) Change the SSH port from 22 to 2200: <br>
a) ```$ sudo nano /etc/ssh/sshd_config```. Find the Port line and edit it to 2200. <br>
b) ```$ sudo service ssh restart```. Now you are able to log into the remote VM through ssh with the following command: ```$ ssh -i ~/.ssh/finalproject -p 2200 grader@13.126.24.32```.<br>
Source: i) [Ubuntu forums](https://ubuntuforums.org/showthread.php?t=1739013) <br>
ii) [Udacity classroom](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175461/lessons/4331066009/concepts/48010894990923)
<br>
<br>
3.5) Configure the Uncomplicated Firewall (UFW):
a) check status of ufw and make sure it is inactive. ```$ sudo ufw status ```.
<br> then run command <br>
b) ```$ sudo ufw defualt deny incoming```. <br>
c) ```$ sudo ufw default allow outgoing```. <br>
d) ```$ sudo ufw allow 2200/tcp```. <br>
e) ```$ sudo ufw allow 80/tcp```. <br>
f) ```$ sudo ufw allow 123/udp```. <br>
h) ```$ sudo ufw enable```.

## 4. Update all currently installed packages:
a) ```$ sudo apt-get update```. <br>
b) ```$ sudo apt-get upgrade```. <br>
c) Install finger, a utility software to check users' status: ```$ apt-get install finger```.

## 5. Configure the local timezone to UTC
a.) Open time configuration dialog and set it to UTC with: ```$ sudo dpkg-reconfigure tzdata```. <br>
b.) Install ntp daemon ntpd for a better synchronization of the server's time over the network connection: ```$ sudo apt-get install ntp```.
<br> Source: [Ubuntu Time](https://help.ubuntu.com/community/UbuntuTime)

## 6. Install and mod_wsgi
a.) ```$ sudo apt-get install apache2```. <br>
b.) Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install mod_wsgi with the following command: ```$ sudo apt-get install libapache2-mod-wsgi python-dev```. <br>
c.) Enable mod_wsgi: ```$ sudo a2enmod wsgi```. <br> 
d.) ```$ sudo service apache2 start```. <br>

## 7. Install Git 
1. ```$ sudo apt-get install git```. <br>
2. Configure your username: ```$ git config --global user.name <username>```. <br>
3. Configure your email: ```$ git config --global user.email <email>``` .

## 8. Clone the Catalog app from Github
1. ```$ cd /var/www```. Then: ```$ sudo mkdir catalog```. <br>
2. Change owner for the catalog folder: ```$ sudo chown -R grader:grader catalog```. <br>
3. Move inside that newly created folder: ```$ cd /catalog``` and clone the catalog repository from Github: ```$ git clone https://github.com/Ayush97Didwaniya/catalogshow.git catalog```. <br>
4. Make a /var/www/catalog/catalog.wsgi file to serve the application over the mod_wsgi. That file should look like this:
```
import sys 
import logging 
logging.basicConfig(stream=sys.stderr) 
sys.path.insert(0, "/var/www/catalog/") 

from catalog import app as application 
application.secret_key = 'supersecretkey' 
```

## 9. Install virtual environment, Flask and the project's dependencies
1. Install pip, the tool for installing Python packages: ```$ sudo apt-get install python-pip```. <br>
2. If virtualenv is not installed, use pip to install it using the following command: ```$ sudo pip install virtualenv```. <br>
3. Move to the catalog folder: ```$ cd /var/www/catalog```. Then create a new virtual environment with the following command: ```$ sudo virtualenv venv```. <br>
4. Activate the virtual environment: ```$ source venv/bin/activate```. <br>
5.Change permissions to the virtual environment folder: ```$ sudo chmod -R 777 venv```. <br>
6. Install Flask: ```$ sudo pip install Flask```. <br>
7. Install all the other project's dependencies: ```$ sudo pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2```.<br>
Source : [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps). 

## 10 - update in project
1. change the  ```/var/www/catalog/catalog/catalog.py``` file to ```/var/www/catalog/catalog/__init__.py``` <br>
use command 
```$ ca /var/www/catalog/catalog``` and then ```$ sudo mv catalog.py __init__.py```.
2. Use the ```nano __init__.py``` command to change: <br>
a) change ```CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']``` to ```CLIENT_ID = json.loads(
    open('var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']```. <br>
b) change the host to your Amazon Lightsail public IP address and port to 80. <br>
c) change this line ```oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')``` to 
```oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')```. <br>
d) change all /var/var/catalog/catalog/ all file permission to 777. using this command ```$ sudo chmod 777 file name```. <br>

## 11 - Configure and enable a new virtual host :
1. Create a virtual host conifg file: ```$ sudo nano /etc/apache2/sites-available/catalog.conf```. <br>
2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 13.126.24.32
    ServerAlias http://ec2-13-126-24-32.ap-south-1.compute.amazonaws.com 
    ServerAdmin admin@13.126.24.32
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
3. The WSGIDaemonProcess line specifies what Python to use and can save you from a big headache. In this case we are explicitly saying to use the virtual environment and its packages to run the application. <br>
4. Enable the new virtual host: ```$ sudo a2ensite catalog```. <br>
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps). 

## 12 - Install and configure PostgreSql
1. Install some necessary Python packages for working with PostgreSQL: ```sudo apt-get install libpq-dev python-dev```. <br>
2. Install PostgreSQL: ```$ sudo apt-get install postgresql postgresql-contrib```. <br>
3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a trusted user who can access the database software. So let's change the user with: ```$ sudo su - postgres```, then connect to the database system with $ psql. <br>
4. Create a new user called 'catalog' with his password: ```# CREATE USER catalog WITH PASSWORD 'catalog123';```. <br>
5. Give catalog user the CREATEDB capability: ```# ALTER USER catalog CREATEDB;```. <br>
6. Create the 'catalog1' database owned by catalog user: ```# CREATE DATABASE catalog1 WITH OWNER catalog```;. <br>
7. Connect to the database: ```# \c catalog1```. <br>
8. Revoke all rights: # REVOKE ALL ON SCHEMA public FROM public;. <br>
9. Lock down the permissions to only let catalog1 role create tables: ```# GRANT ALL ON SCHEMA public TO catalog1;```. <br>
10. Log out from PostgreSQL: # \q. Then return to the grader user: $ exit. <br>
11. Inside the Flask application, the database connection is now performed with: <br>
12. engine = create_engine('postgresql://catalog:catalog123@localhost/catalog1') <br>
13. Setup the database with: ```$ python /var/www/catalog/catalog/database_with_user.py```. <br>
output in cmd: "Restaurant menu item created" <br>
14. To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file: ```$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf``` and edit it, if necessary, to make it look like this:
```
local   all          postgres                                   peer 
local   all             all                                     peer 
host    all             all             127.0.0.1/32            md5 
host    all             all             ::1/128                 md5
```
source: [DigitalOceab](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).

## 13 - Update OAuth authorized JavaScript origins
To let users correctly log-in change the authorized URI to http://ec2-13-126-24-32.ap-south-1.compute.amazonaws.com  on Google developer dashboards. and also change on facebook developer dashboard if you have used that.

## 14 - Restart Apache server 
```$ sudo service apache2 restart``` <br>
application is online now!

## Reference 
Thanks to callforsky who wrote really helpfull README in his [repository](https://github.com/callforsky/udacity-linux-configuration).



