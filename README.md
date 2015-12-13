
Project 5 Linux Web Server configuration:

i. The IP address and SSH port so your server can be accessed by the reviewer.
    52.35.28.93    SSH Port 2200

ii. The complete URL to your hosted web application.
    http://52.35.28.93/catalog/

iii. A summary of software you installed and configuration changes made.
   Following was done as per project requirement:
   1. Launch your Virtual Machine with your Udacity account:
      Click Create Development in the Udacity My Account page which provides the public IP address and 
      the udacity_key.rsa. Download the key to ~/.ssh directory of local machine and use login to the IP address as root.
      $ mv ~/Downloads/udacity_key.rsa ~/.ssh/
     
      Note: If the server configuration locks u out.. click delete on the Udacity page and restart with a new IP address 
            and a new udacity_key.rsa file

   2. Use ssh to login to the IP address as root.
      $ ssh -i ~/.ssh/udacity_key.rsa root@public_ip_address 

   3. Create a new user named grader
       $ adduser grader
         Optional: Install finger to check user .. Optional
         sudo apt-get install finger 
         
   4. Give the grader the permission to sudo
       $visudo   or sudo vim /etc/sudoers.d
         Add the following line below root ALL
         grader ALL=(ALL:ALL) ALL
       $ cat /etc/passwd 
         This should list the user 'grader'
       
   5. Update all currently installed packages
      $sudo apt-get update
      $sudo sudo apt-get upgrade

   6. Change the SSH port from 22 to 2200
      $ vim /etc/ssh/sshd_config
         Change to Port 2200 from Port 22 
         Change PermitRootLogin from without-password to no  ===> This disables root login and 
         advisable to do in the very end after project work is done. 
         Temporalily change PasswordAuthentication from no to yes
         Append UseDNS no
         Append AllowUsers grader 
      $ /etc/init.d/ssh restart    ===> restart ssh. This will change the port to 2200 and so from now on use that port to login
                                        It is safer to do this after the firewall has been setup and ssh key gen has been done.

   7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and 
       NTP (port 123)
       $ sudo ufw enable
       $ sudo ufw status verbose  ===> Check the status to say active
       $ sudo ufw allow 2200/tcp
       $ sudo ufw allow 80/tcp 
       $ sudo ufw allow 123/udp
       $ sudo ufw status verbose  ====> Check all allow and deny show correctly..

   8. Configure the local timezone to UTC
       $sudo dpkg-reconfigure tzdata
              chose 'None of the above', then UTC
       $ sudo apt-get install ntp
       $ sudo vim /etc/ntp.conf  ==> Choose North America from pool of servers
          Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace 
          the given servers with the new server list

   9. Install and configure Apache to serve a Python mod_wsgi application
       $ sudo apt-get install apache2   ==> install apache2
         Check if http://52.35.28.93  opens in browser and say 'It works!' on the top of the page
       $ sudo apt-get install python-setuptools libapache2-mod-wsgi
       $ sudo service apache2 restart  ==> This loads the mod_wsgi package
         * To Get rid of the message "Could not reliably determine the servers's fully qualified domain name" after restart 
            Source 
          $ echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf
          $ sudo a2enconf fqdn
       $ sudo apt-get install libapache2-mod-wsgi python-dev   ===> extend python with additional packages for catalag app
       $ sudo a2enmod wsgi

         Create a Flask app:
         $cd /var/www    ===> App should be installed in this dir
         Setup a directory for the app, e.g. catalog:
         $ sudo mkdir catalog
         $ cd catalog
	 $ sudo mkdir catalog
         $cd catalog
         $ sudo mkdir static templates images and other directories for catalog app
         $sudo vim __init__.py
            Add a simple hello world and run the app.. Later this file will have real catalog app code.
	     from flask import Flask  
             app = Flask(__name__)  
             @app.route("/")  
             def hello():  
                return "Hello World!"  
             if __name__ == "__main__":  
             app.run() 
     
         Install Flask
         $ sudo apt-get install python-pip   ===> Installs pip to install other s/w packages
	 Create virtual env to install and use specific versions of s/w
         $ sudo pip install virtualenv  
	 $ sudo virtualenv venv 
         $ sudo chmod -R 777 venv  ===> enable all permissions for virtual env
         $ source venv/bin/activate   ===> Activate virtual env
         $ pip install Flask    ===> Install flask within venv
         $ python __init__.py   ===> Run Hello App
	 $ deactivate     ===> deactivate venv 
        
         Configure and Enable a New Virtual Host:
           Create a virtual host config file
             $sudo vim /etc/apache2/sites-available/catalog.conf
               Add following 
		<VirtualHost *:80>
      			ServerName 52.35.28.93
      			ServerAdmin admin@52.35.28.93
			ServerAlias ===> amazon hostname will go here
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
		
	  Enable the virtual host by command $sudo a2ensite catalog

          Create the .wsgi File and Restart Apache
		$ cd /var/www/catalog and $ sudo vim catalog.wsgi
	  Paste in the following lines of code:
		#!/usr/bin/python
  		import sys
  		import logging
  		logging.basicConfig(stream=sys.stderr)
  		sys.path.insert(0,"/var/www/catalog/")

  		from catalog import app as application
  		application.secret_key = 'Add your secret key from the project 3 appication'

	  Restart Apache:
		$ sudo service apache2 restart

     10. Install and configure PostgreSQL:  And other modules for catalog application
      Do not allow remote connections
      Create a new user named catalog that has limited permissions to your catalog application database

      $sudo apt-get install postgresql postgresql-contrib  ===> installs postgresql
      $sudo vim /etc/postgresql/9.3/main/pg_hba.conf   => default setting disables remote connection
      $ sudo adduser catalog (choose a password)
      $ sudo su - postgres login as default user in postgres
      $ psql   ==>shell to configure database 
                Add postgre user with password:
		# CREATE USER catalog WITH PASSWORD 'Passwordsetupfordb'
                # ALTER USER catalog CREATEDB;
	 	# \du     ===> should list the user catalog
	        # CREATE DATABASE catalog WITH OWNER catalog;
		# \c catalog    ====> This connects to database catalog
	        # REVOKE ALL ON SCHEMA public FROM public; 
		# GRANT ALL ON SCHEMA public TO catalog;    ==> give permission to catalog
	        # \q,
                $ exit    

     Install Other packages required for project3
	$sudo pip install httplib2
        $sudo pip install flask-seasurf
	$sudo pip install --upgrade oauth2client
        $ sudo apt-get install python-psycopg2
	
     11     Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that 
            it functions correctly when visiting your server’s IP address in a browser. 
            Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

            $ git clone https::/github.com/nambw/Catalog.git
             Move all content of created Catalog directory to /var/www/catalog/catalog directory and delete the empty Catalog git directory
	  
            Make the GitHub repository inaccessible:
	    $cd /var/www/catalog/ 
	    $ sudo vim .htaccess  and add following lines
		RedirectMatch 404 /\.git

     12   Change the project3 code to use postgresql for database instead of sqllite
	    In database_setup.py  project.py replace create_engine line  with following
		engine = create_engine('postgresql://catalog:'Passwordsetupfordb@localhost/catalog')  ==> use passwd from setp 10
            rename project.py to __init__.py This will make the hello application that was run before stop and run the catalog application now
	    apache automatically detects change in the application code that is running.. but it is safer to restart apache2 when any change in config or code
	    is made.

	     
             $ sudo service apache2 restart
              
              http://52.35.28.93/catalog should show the catalog application now

    13       Changes made for google plus Oauth to work :
	     Change the Javascript Origin and redirect url in Credentials tab in Google developers console for the catalog project.
 	     https://console.developers.google.com/apis/credentials?project=catalog-964

	     This needs using the hostname for server in redirect url. The hostname is found from http://www.hcidata.info/host2ip.cgi   
	     Authorized JavaScript origins
	      Add http://52.35.28.93
		  http://ec2-52-35.28.93.us-west-2.compute.amazonaws.com

	     Authorized redirect URIs	
		http://ec2-52-35.28.93.us-west-2.compute.amazonaws.com/catalog/
           
	     Save changes. Download the client_secrets.json file and copy the contents to the /var/www/catalog/catalog/clients_secrets.json 
	     Check permission for the file and make it readable.
	
             Add the ec2...hostname in apache2 config
		 $ sudo vim /etc/apache2/sites-available/catalog.conf

	         $sudo service apache2 restart
		 $ sudo a2ensite catalog   to take changes for the hostname config

            In case things don;t work:
	    Check javascript console on browser for possible errors and apache2 error log
            $ sudo tail -30 /var/log/apache2/error.log
            Changes made in google console are not immediate. So Download client json file after 5-10 min of making changes.
	

          Change owndership of image upload directory to www-data so that the iage files uploaded by user of the catalog are uploaded
	  to /var/www/catalog/catalog/images correctly.
	  $su chown www-data /var/www/catalog/catalog/images
	  Also set the root apache directory to /var/www/catalog in __init__.py and use os.join calls to make full path name in project3 code.
          app.config['APP_DIR'] = os.path.abspath(os.path.dirname(__file__))
          app.config['UPLOAD_FOLDER'] = os.path.join( app.config['APP_DIR'], 'images')		
	  
         With this, the catalog application should work fully. 

         iv. A list of any third-party resources you made use of to complete this project.

          Resources used:
             Udacity Course: Configuring Linux based servers and all resources recommended there.
	     Udacity discussion forum:  resources suggested by many posts helped mainly: 
             https://wordpress.org/support/topic/right-permissions-for-uploads-and-all-the-images-inside
             http://askubuntu.com/questions/	      
	     https://www.digitalocean.com/community/tutorials




