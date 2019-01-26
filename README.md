# Linux Server Configuration
In this project, I'm going to show you how to prepare a baseline linux server in
order to host a flask application. I'll use Amazon Web Service to create a
remote virtual machine (Ubuntu 18.04 server). Then I'll secure, configure it and
deploy my Item Catalogue application: [Movie Actors App](https://github.com/mdifils/CatalogApp).
Here is a summary of information about my server for the reviewer:

* **The IP address and SSH port**,  IP: **35.177.223.249** and Port: **2200**
* **The complete URL to my hosted web application**: [https://movie.actors.mdifils.com](https://movie.actors.mdifils.com)
* **A list of some third-party resources I used to complete this project**: <br>
<ins>Amazon Web Service</ins>: To create a remote ubuntu server and to host my web application.
<ins>GoDaddy</ins>: To purchase and register the domain name of my web application. <br>
<ins>Letsencrypt</ins>: To secure my web application with a free signed SSL certificate.<br>
* **A summary of software I installed**: <br>
<ins>Finger</ins>: <br>
<ins>New user added</ins>: <br>
<ins>Public ssh key</ins>: <br>
<ins>UTC time</ins>: <br>
<ins>Apache2</ins>: <br>
<ins>Libapache2-mod-wsgi-py3</ins>: <br>
<ins>/home/new-user/.local/bin added to $PATH</ins>: <br>
<ins>pip</ins>: <br>
<ins>Virtualenv</ins>: <br>
<ins>Flask</ins>: <br>
<ins>Flask-Login</ins>: <br>
<ins>Flask-SQLAlchemy</ins>: <br>
<ins>Flask-Migrate</ins>: <br>
<ins>Oauth2client</ins>: <br>
<ins>requests</ins>: <br>
<ins>certbot</ins>: <br>
* **SSH key location**: <br>
<ins>Public key</ins>: /home/grader/.ssh/authorized_keys. <br>
<ins>Private key</ins>: Put the content provided during the submission in either
suggested folder <br>
_Windows_: /c/Users/owner/.ssh/privatekey.pem <br>
_MAC_: /Users/owner/.ssh/privatekey.pem <br>
_Linux_: /home/owner/.ssh/privatekey.pem <br>

### Remote Virtual Machine Creation
There are many ways to do that, you can create lightsail or EC2 instances with [Amazon Web Service](https://aws.amazon.com/console/). But you can also create droplet instances with [Digital Ocean](https://www.digitalocean.com/). In this project, I'll launch a Linux Virtual Machine with Amazon EC2.
* In the AWS Management console, choose first your region in the up right corner (mine is London). Then click on EC2 under "Compute" section: All services->Compute->EC2.
![AWS console](img/AmazonEC2_console.png)
* Launch EC2 instance and choose an Amazon Machine Image
![Launch_Instance](img/Launch_Instance.png)
![AMI](img/Choose_AMI.png)
* Choose an Instance Type (_Free tier eligible means that you can use free of charge for the first year_) and go to next step (Instance configuration: leave default)
![Instance_Type](img/Instance_Type.png)
![Instance_config](img/Configure_Instance.png)
* Add storage and go to next step (Add tag: give a name to your server)
![Storage](img/Add_Storage.png)
![Tag](img/Add_Tag.png)
* Configure the security group (create a new one, give it a name, a description allow incoming ports: 22, 2200, 80, 123)
![SG](img/Security_Group.png)
* Check that everything is okay and Launch
![Review](img/Instance_Review.png)
* Create a new key pair and download it in order to log into your instance via ssh.
![Key_Pair](img/Key_pair.png)
![status](img/Launch_status.png)
![Instance_running](img/Instance_running.png)

### Ubuntu Server Configuration
* Connect to your instance using the key you've just downloaded and your instance public IP (look at the last picture) <br>
`$ ssh -i /path/to/privatekey.pem ubuntu@Public_IP`
![Connecting](img/Connecting.png)
* After updating all current packages, install Finger package and add a new user grader (User: grader, password: grader, Full name: Udacity Grader) <br>
`$ sudo apt install finger` <br>
`$ sudo adduser grader` <br>
`$ finger grader`
* Give grader sudo access <br>
`$ sudo nano /etc/sudoers.d/grader` write and save this file.
![sudo_grader](img/Sudo_grader.png)
* Create a new key pair <br>
On LOCAL computer generate key pair using ssh-keygen <br>
![ssh-key](img/ssh-keygen.png)
On server side, use (**su grader** followed by password: grader). Switch to grader user to load in public key file. Make sure you are in grader home directory (/home/grader) by checking with this command: `$ pwd` <br>
`$ sudo touch .ssh/authorized_keys` <br>
Copy public key in the local machine <br>
`$ cat /c/Users/miche/.ssh/ubuntuserverconfig.pem.pub` <br>
Then paste it into the server. <br>
`$ nano .ssh/authorized_keys `
* Change SSH Port to 2200 <br>
`$ sudo nano /etc/ssh/sshd_config` modify and save
![ssh_port](img/ssh_port.png)
`$ sudo service ssh restart` to restart ssh.
* UFW firewall configuration
![Firewall](img/Firewall.png)

You can now use this new key pair and this new port to connect to server.
![ssh windows](img/ssh_win.png)
![ssh linux](img/ssh_linux.png)

* Change Time Zone to UTC (**sudo dpkg-reconfigure tzdata**). Scroll down to none of them  then select UTC.
* install Apache2 `$ sudo apt-get install apache2` and test it
![Apache](img/ApacheCheck.png)
`$ sudo apt-get install libapache2-mod-wsgi-py3` <br>
`$ sudo service apache2 restart` to restart apache2
* Installing pip: <br>
It is advised to never use pip with sudo. Instead you should always install into your user directory (via pip install --user) or within virtualenvs
```
$sudo nano ~/.profile
# add these two lines at the end of `~/.profile` file:
export PY_USER_BIN=$(python3 -c 'import site; print(site.USER_BASE + "/bin")')
export PATH=$PY_USER_BIN:$PATH
$ source ~/.profile
$ echo $PATH
/home/grader/.local/bin:/home/grader/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
$ sudo apt-get install -y python3-dev
$ curl -LO https://bootstrap.pypa.io/get-pip.py
$ python3 get-pip.py --user
$ pip -V
pip 18.1 from /home/grader/.local/lib/python3.6/site-packages/pip (python 3.6)
# Installing virtualenv
$ pip install virtualenv --user
```
* Deploying flask application
```
$ sudo mkdir -p /var/www/flaskApp/catalogApp
# Giving ownership
$ sudo chown $USER:$USER -R /var/www/flaskApp
$ cd /var/www/flaskApp/catalogApp
$ virtualenv venv
$ source venv/bin/activate
# Installing flask application requirements in the virtualenv
$ pip install flask flask-login flask-sqlalchemy flask-migrate
$ pip install requests oauth2client
$ pip freeze
$ deactivate
$ git clone https://github.com/mdifils/CatalogApp.git
# Visit my github repo to better understand my flask application
$ sudo nano /etc/apache2/sites-available/catalogapp.conf
```
![flaskAppConf](img/flaskApp_conf.png)
```
# Disabling the default virtualHost and enabling the new one.
$ sudo a2dissite 000-default.conf
$ sudo a2ensite catalogapp.conf
$ sudo systemctl restart apache2
$ nano /var/www/flaskApp/catalogApp/catalogapp.wsgi
$ sudo service apache2 restart
```
![flaskAppWsgi](img/flaskApp_wsgi.png)

![flaskAppWsgi](img/flaskApp_folder.png)

```
# Setting up SSL
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt install python3-certbot-apache
# Allowing HTTPS through the Firewall
$ sudo ufw allow 443/tcp
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)

# Obtaining an SSL Certificate
# Make sure to comment first in the virtualHost the following line:
# WSGIDaemonProcess catalogapp user=grader group=grader threads=5

$ sudo certbot --apache -d mdifils.com -d movie.actors.mdifils.com
```

If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let's Encrypt server, then run a challenge to verify that you control the domain you're requesting a certificate for.
If that's successful, certbot will ask how you'd like to configure your HTTPS settings:

![certbot](img/certbot_success.png)
You can uncomment the same line: "WSGIDaemonProcess catalogapp user=grader group=grader threads=5" in the virtualHost and notice the three last lines added by certbot.
![virtualHost](img/virtualhost_edited.png)
Notice the new virtualHost created by certbot : `catalogapp-le-ssl.conf` in order to redirect HTTP to HTTPS <br>
![New VirtualHost](img/new_virtualhost1.png)
![New VirtualHost](img/new_virtualhost2.png)
Use links provided in certbot report to test your configuration.
![SSLTest](img/SSLcertificate_test.png)
Now Test your application in the Browser.
![Browser](img/flaskApp.png)
