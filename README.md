# Frappe-ERPNext-Version-15-in-Ubuntu-24.04-LTS
A complete Guide to Install Frappe/ERPNext version 15  in Ubuntu 24.04 LTS


### Pre-requisites 

      Python 3.11+                                  (python 3.12 is inbuilt in 24.04 LTS)
      Node.js 18+
      
      Redis 5                                       (caching and real time updates)
      MariaDB 10.3.x / Postgres 9.5.x               (to run database driven apps)
      yarn 1.12+                                    (js dependency manager)
      pip 20+                                       (py dependency manager)
      wkhtmltopdf (version 0.12.5 with patched qt)  (for pdf generation)
      cron                                          (bench's scheduled jobs: automated certificate renewal, scheduled backups)
      NGINX                                         (proxying multitenant sites in production)


> ## Note:
> ubuntu 24.04 default python version is python3.12
> 
> ubuntu 24.04 default mariadb version is 10.11

### STEP 1 Install git
    sudo apt-get install git

### STEP 2 install python-dev

    sudo apt-get install python3-dev

### STEP 3 Install setuptools and pip (Python's Package Manager).

    sudo apt-get install python3-setuptools python3-pip

### STEP 4 Install virtualenv
    
    sudo apt install python3.12-venv
    

### STEP 5 Install MariaDB

    sudo apt-get install software-properties-common
    sudo apt install mariadb-server
    sudo systemctl status mariadb
    sudo mysql_secure_installation
    
    
      In order to log into MariaDB to secure it, we'll need the current
      password for the root user. If you've just installed MariaDB, and
      haven't set the root password yet, you should just press enter here.

      Enter current password for root (enter for none): # PRESS ENTER
      OK, successfully used password, moving on...
      
      
      Switch to unix_socket authentication [Y/n] Y
      Enabled successfully!
      Reloading privilege tables..
       ... Success!
 
      Change the root password? [Y/n] Y
      New password: 
      Re-enter new password: 
      Password updated successfully!
      Reloading privilege tables..
       ... Success!

      Remove anonymous users? [Y/n] Y
       ... Success!
 
       Disallow root login remotely? [Y/n] Y
       ... Success!

       Remove test database and access to it? [Y/n] Y
       - Dropping test database...
       ... Success!
       - Removing privileges on test database...
       ... Success!
 
       Reload privilege tables now? [Y/n] Y
       ... Success!

 
    
    
    
### STEP 6  MySQL database development files

    sudo apt-get install libmysqlclient-dev

### STEP 7 Edit the mariadb configuration ( unicode character encoding )

    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

add this to the 50-server.cnf file

    
    [server]
    user = mysql
    pid-file = /run/mysqld/mysqld.pid
    socket = /run/mysqld/mysqld.sock
    basedir = /usr
    datadir = /var/lib/mysql
    tmpdir = /tmp
    lc-messages-dir = /usr/share/mysql
    bind-address = 127.0.0.1
    query_cache_size = 16M
    log_error = /var/log/mysql/error.log
    
    [mysqld]
    innodb-file-format=barracuda
    innodb-file-per-table=1
    innodb-large-prefix=1
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci      
     
    [mysql]
    default-character-set = utf8mb4

Now press (Ctrl-X) to exit

    sudo service mysql restart

### STEP 8 install Redis
    
    sudo apt-get install redis-server

### STEP 9 install Node.js 18.X package

    sudo apt install curl 
    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
    source ~/.profile
    nvm install 18

### STEP 10  install Yarn

    sudo apt-get install npm

    sudo npm install -g yarn

### STEP 11 install wkhtmltopdf

    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    

### STEP 12 install frappe-bench

    sudo -H pip3 install frappe-bench --break-system-packages
    
    bench --version
    
### STEP 13 initilise the frappe bench & install frappe latest version 

    bench init frappe-bench --frappe-branch version-15
    
    cd frappe-bench/
    bench start
    
### STEP 14 create a site in frappe bench 

>### Note 
>Warning: MariaDB version ['10.11', '7'] is more than 10.8 which is not yet tested with Frappe Framework.
    
    bench new-site {site-name}
    
    bench --site {site-name} add-to-hosts

    bench use {site-name}


### STEP 15 install ERPNext latest version in bench & site

    
    bench get-app erpnext --branch version-15

   In case you have custome app

    bench get-app {custom-app (Clone using the web URL.)}

    bench --site {site-name} install-app erpnext
    
    bench start
### Setting ERPNext for Production

### STEP 16 Enable Scheduler
      bench --site [site-name] enable-scheduler

### STEP 17 Disable maintenance mode
      bench --site [site-name] set-maintenance-mode off
      
### STEP 18 Setup production config
      sudo bench setup production [frappe-user]

### STEP 19 Setup NGINX to apply the changes
      bench setup nginx

### STEP 20 Restart Supervisor and Launch Production Mode
      sudo supervisorctl restart all sudo bench setup production [frappe-user]

### Setting Site SSL

### STEP 21 Set SSL for Site
      ***********Execute the following to add your URL to your ERPNext Server installation:
      export LC_ALL=C.UTF-8
      cd frappe-bench
      bench config dns_multitenant on
      bench setup add-domain --site {site-name} {site-URL}
      bench setup nginx (answer Y to the question about replacing the config file)
      sudo service nginx restart
      cd ~
      
      Now it is time to work on the SSL certificate for your server. First we need to make sure that certbot is not already installed:
      sudo apt-get remove certbot
      
      Now we are going to install the package bundle provider and make sure it is all up to date with the latest release of ‘snapd’ and let it handle the certbot for us (much simpler this way):
      sudo apt install snapd
      sudo snap install core
      sudo snap refresh core
      
      Now we use the ‘snap’ service to automatically install and configure certbot to work properly with your new ERPNext server:
      sudo snap install --classic certbot
      sudo ln -s /snap/bin/certbot /usr/bin/certbot
      
      Now we can safely use the pre-configured certbot to install your SSL certificate:
      sudo certbot --nginx
      (The system will pause at some point and ask you to select the site name you wish to install the certificate from a list of names it found. Be sure to select the URL you used and not ‘site1.local’ at this point)
      

### Restoring Backup
      sudo bench --force --site {site_name} restore {database_path} --with-public-files {path} --with-private-files {patch}

      

      

