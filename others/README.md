#

![Altschool Africa](https://github.com/tuyojr/cloudTasks/blob/main/logos/AltSchool.svg)

![automation](https://github.com/tuyojr/cloudTasks/blob/main/logos/automation.gif)

> This project's instructions can be found [here](https://docs.google.com/document/d/1dhO--9fqxrvU5Y6xC52-0AIplQe23LyscqSp6F0_9SM/edit>#). The repository used for deployment is also [here](https://github.com/f1amy/laravel-realworld-example-app).

Below is my ansible playbook used in deploying this project, which is live at [www.tuyojr.me](https://www.tuyojr.me/). All other files and variables used in the playbook are present in the directory above.

`main.yml`

```YAML
# This playbook was written by Originally Adedolapo Olutuyo ALT/SOE/022/4858. ©October-November 2022
# To apply and solidify my knowledge on everything taught during the 
# 2nd Semester of the Cloud Engineering Course at Altschool Africa, in the real world.
# If you're reading this, MAY THE FORCE BE WITH YOU!
---
- name: Deploying A Laravel Application
  hosts: ubuntuServer
  remote_user: ubuntu
  become: true

  tasks:
  # Updating and Upgrading The Server's Apt Repo
  # sudo apt update
  - name: Update Apt Repository
    apt: update_cache=yes cache_valid_time=3600 
  # sudo apt upgrade -y
  - name: Upgrade The Apt Repository
    apt: upgrade=yes update_cache=yes cache_valid_time=3600
  # Set the timezone
  - name: set timezone
    timezone:
      name: Africa/Lagos
  
  # Install Some basic Applications that are very Important to ensure our deployment runs smooth
  # sudo apt install wget git curl apache2 -y
  - name: Install 'wget', 'git', 'curl', and 'apache2'
    apt:
     pkg:
     - wget
     - git
     - curl
     - apache2
  
  
  # These rules have already been set during instance launch.
  # Set Up Important Firewall Permissions and Enable UFW so TCP/IP Packets can be sent to and fro, seamlessly
  # sudo apt install ufw -y
  - name: Install Uncomplicated Firewall
    apt: name=ufw state=present
  # sudo ufw allow Apache && sudo ufw allow 'Apache Full' && sudo ufw allow OpenSSH
  # sudo ufw allow 22 && sudo ufw allow 80 && sudo ufw allow 443 && sudo ufw allow 853
  # sudo ufw allow 3306 && sudo ufw allow 5432 && sudo ufw enable
  - name: Allow Apache
    ufw: name=Apache rule=allow
  - name: Allow Apache Full
    ufw: name='Apache Full' rule=allow
  - name: Allow OpenSSH
    ufw: name=OpenSSH rule=allow
  - name: Allow access to tcp port 22 (SSH)
    ufw: rule=allow port='22' proto=tcp
  - name: Allow access to tcp port 80 (HTTP)
    ufw: rule=allow port='80' proto=tcp
  - name: Allow access to port 443 (HTTPS)
    ufw: rule=allow port='443'
  - name: Allow access to port 853 (DNS over TLS)
    ufw: rule=allow port='853'
  - name: Allow access to port 3306 (MySQL Server)
    ufw: rule=allow port='3306'
  - name: Allow access to port 5432 (PostgreSQL Server)
    ufw: rule=allow port='5432'
  - name: Enable Uncomplicated Firewall
    ufw: state=enabled policy=allow
  

  # Set Up PostgreSQL with a Bash Script
  # sudo cp testInstall.sh /home/ubuntu
  - name: Copy PostgreSQL Script to Install and configure it
    copy:
      src: testInstall.sh
      dest: /home/ubuntu
      mode: '0775'
      owner: ubuntu
      group: ubuntu
  # sh testInstall.sh
  - name: Run PostgreSQL Script to Install and configure it
    command: sh /home/ubuntu/testInstall.sh
  

  # Update and Upgrade the Apt Repo, Install Python, PIP, and Set Up MySQL
  # We're upgrading so that all dependencies work together with the latest available applications
  # sudo apt update
  - name: Update Apt Repository
    apt: update_cache=yes cache_valid_time=3600

  # sudo apt install python3 -y
  - name: Install Python3
    apt: name=python3 state=latest
  # sudo apt install pip -y
  - name: Install Pip
    apt: name=pip state=latest

  # Install MySQL Server and Other Important Dependencies
  # sudo apt install mysql-server -y
  - name: Install MySQL server
    apt:
      name: mysql-server
      state: latest

  # sudo apt install mysql-client -y
  - name: Install MySQL client
    apt:
      name: mysql-client
      state: latest

  # sudo pip install pymysql
  - name: Install PyMySQL Library
    pip: name=pymysql state=latest

  # sudo systemctl start mysql
  - name: Start the MySQL service
    service:
      name: mysql
      state: started
      enabled: true

  # Nex, we want to configure our database, so we set a default password for the root user; this allows us to now 
  # create our new password so we can log in to the database.
  # Next we create a database, and then a user, we grant the user all the necessary permissions on the database
  # Ensure the user can use the database, and then log in as root user to set a password for the created user
  # Also we want to remove anonymous users and default databases. 
  # sudo mysql -u root -e 'UPDATE mysql.user SET plugin="mysql_native_password" WHERE user="root" AND host="localhost"'
  - name: Change the authentication plugin of MySQL root user to mysql_native_password
    command: mysql -u root -e 'UPDATE mysql.user SET plugin="mysql_native_password" WHERE user="root" AND host="localhost"'

  # sudo mysql -u root -e 'FLUSH PRIVILEGES'
  - name: Reload all Privileges
    command: mysql -u root -e 'FLUSH PRIVILEGES'

  # sudo mysql -u root -e 'ALTER USER 'root'@'localhost' IDENTIFIED BY "New3p1a22w0rd"'
  - name: Set MySQL root password
    mysql_user:
      login_host: 'localhost'
      login_user: 'root'
      login_password: ''
      name: 'root'
      password: "{{ mysql_root_password }}"
      state: present

  # sudo mysql -u root -p
  # > DELETE FROM mysql.user WHERE user='';
  - name: Delete Anonymous MySQL server user for localhost
    mysql_user:
      login_user: 'root'
      login_password: "{{ mysql_root_password }}"
      name: ''
      host: localhost
      state: absent

  # sudo mysql -u root -p
  #> CREATE DATABASE {{ DB_NAME }};
  - name: Create a Database
    mysql_db:
      login_user: 'root'
      login_password: "{{ mysql_root_password }}"
      name: "{{ DB_NAME }}"
      state: present

  # sudo mysql -u root -p
  #> CREATE USER '{{ DB_USER }}'@'localhost';
  #> GRANT ALL ON {{ DB_NAME }}.* TO '{{ DB_USER }}'@'localhost';
  - name: Create a Database User
    mysql_user:
      login_user: 'root'
      login_password: "{{ mysql_root_password }}"
      name: "{{ DB_USER }}"
      password: ''
      host: localhost
      priv: '{{ DB_NAME }}.*:ALL,GRANT'

  # sudo mysql -u demoUser -e 'USE altschool'
  - name: Ensure the created User Uses the database
    command: mysql -u demoUser -e 'USE altschool'

  # sudo mysql -u demoUser -e 'ALTER USER 'demoUser'@'localhost' IDENTIFIED BY "vagrant";'
  # OR
  # sudo mysql -u root -p
  #> ALTER USER 'demoUser'@'localhost' IDENTIFIED BY "vagrant";
  - name: Set MySQL created user's password
    mysql_user:
      login_host: 'localhost'
      login_user: 'root'
      login_password: "{{ mysql_root_password }}"
      name: "{{ DB_USER }}"
      password: "{{ USER_PASSWORD }}"
      state: present

  # sudo mysql -u root -p
  #> DROP DATABASE test;
  - name: Remove the MySQL Test Database If exists
    mysql_db:
      login_user: 'root'
      login_password: "{{ mysql_root_password }}"
      name: test
      state: absent


  # Setting Up all the necessary packages to Install PHP.
  # This is a very important step that ensures our application runs smoothly.
  # sudo apt install -y software-properties-common
  # sudo add-apt-repository ppa:ondrej/php -y
  - name: Install Software properties
    command: apt install -y software-properties-common
  - name: Add Apt Repo for PHP
    command: add-apt-repository ppa:ondrej/php -y
  
  # sudo apt update
  - name: Update Apt Repository
    apt: update_cache=yes cache_valid_time=3600 
  # sudo apt upgrade -y
  - name: Upgrade The Apt Repository
    apt: upgrade=yes update_cache=yes cache_valid_time=3600
  
  # - name: Install PHP
  #   command: apt install php8.0 -y
  # - name: Install PHP dependency
  #   command: apt install libapache2-mod-php8.0 -y

  # sudo apt install php8.1 libapache2-mod-php8.1 php8.1-xmlrpc php8.1-cli -y
  # sudo apt install php8.1-common php8.1-pgsql php8.1-mysql php8.1-zip php8.1-mbstring php8.1-imap -y
  # sudo apt install php8.1-dev php8.1-imap php8.1-opcache php8.1-intl php8.1-bcmath -y
  - name: Install PHP and other PHP Modules
    apt:
     pkg:
     - php8.1
     - libapache2-mod-php8.1
     - php8.1-cli # command interpreter, useful for testing PHP scripts from a shell or performing general shell scripting tasks
     - php8.1-curl # lets you make HTTP requests in PHP
     - php8.1-xmlrpc
     - php8.1-mysql # for working with MySQL databases
     - php8.1-zip # for working with compressed files
     - php8.1-common  # documentation, examples, and common modules for PHP
     - php8.1-xml # for working with XML data
     - php8.1-pgsql # for working with PostgreSQL databases
     - php8.1-gd # for working with images
     - php8.1-mbstring # used to manage non-ASCII strings
     - php8.1-imagick
     - php8.1-dev
     - php8.1-imap 
     - php8.1-opcache 
     - php8.1-soap 
     - php8.1-intl
     - php8.1-bcmath # used when working with precision floats, Just in case lol.


  # Update and Upgrade the Apt Repo and Install Composer and it's Dependencies
  # sudo apt update
  - name: Update Apt Repository
    apt: update_cache=yes cache_valid_time=3600 
  # sudo apt upgrade -y
  - name: Upgrade The Apt Repository
    apt: upgrade=yes update_cache=yes cache_valid_time=3600

  # sudo apt install php-cli unzip
  - name: Install Required packages for composer again
    command: apt install php-cli unzip -y
  # sudo curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
  - name: Download Composer Installer
    command: curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
  # Export the HASH for verification
  - name: Programmatically obtain the latest hash from the Composer page and store it in a shell variable
    command: echo $HASH
    environment:
     HASH: '`curl -sS https://composer.github.io/installer.sig`'
  # sudo php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
  - name: Verify Installation Script is Safe to Run
    command: php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
  # sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
  - name: Install Composer Globally
    command: sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
  # sudo chmod u+x /usr/local/bin/composer
  - name: Give Execute Permission for Composer
    file:
      path: /usr/local/bin/composer
      mode: u+x
  

  # Clone the project repo
  # sudo git clone https://github.com/f1amy/laravel-realworld-example-app.git /var/www/demoApp
  - name: Clone The Laravel Repo
    command: git clone https://github.com/f1amy/laravel-realworld-example-app.git /var/www/demoApp
  # sudo chown -R ubuntu:ubuntu /var/www/demoApp
  - name: Give the project folder necessary permissions so composer install can run smoothly
    file:
      path: /var/www/demoApp
      state: directory
      recurse: yes
      owner: ubuntu
      group: ubuntu
  
  # Next we want to copy our app's environment variables our project requires and give the file the necessary permissions
  # Composer will help us generate an application key and input it into the .env file
  # sudo cp .env /var/www/demoApp
  # sudo chmod '0775' /var/www/demoApp/.env
  # sudo chown ubuntu:ubuntu /var/www/demoApp/.env
  - name: Copy The .env You Have Configured
    copy:
      src: .env
      dest: /var/www/demoApp
      mode: '0775'
      owner: ubuntu
      group: ubuntu

# This is very important so our play is not stuck in limbo.
# Stop Using Sudo So we can Install project dependencies with Composer
- name: Install Application Dependencies with Composer
  hosts: ubuntuServer
  remote_user: ubuntu

  tasks:
  # This is very necessary as all the plays run previously will update again
  # The curl module works with composer to access and transfer data require for the project to work seamlessly
  # Install php-curl
  - name: Install PHP curl again
    command: sudo apt install php8.1-curl -y

  # This updates composer to the latest version again
  # ..and composer install installs all the project's/app's dependencies
  # cd /var/www/demoApp
  # composer update
  - name: Update all libs and dependencies outlined in the /var/www/demoApp/composer.lock
    composer:
      command: update
      working_dir: /var/www/demoApp
  - name: Downloads and installs all libs and dependencies outlined in the /var/www/demoApp/composer.lock
    composer:
      command: install
      working_dir: /var/www/demoApp


# Now, we can run the remaining commands with Sudo
- name: Continue Application Deployment
  hosts: ubuntuServer
  remote_user: ubuntu
  become: true

  # The next series of tasks is just to configure our project to populate our application with our database,
  # give our application a speed boost also... To do that, we need the right permissions.
  tasks:

  # First, Let us set up the necessary file permissions so our app can be accessed by our web server
  # - name: Change Ownership of variables file To be Assigned to Web Servers
  #   file:
  #     path: /var/www/demoApp/.env
  #     state: file
  #     owner: www-data
  #     group: www-data

  # sudo chown -R www-data:www-data /var/www/demoApp
  - name: Change Ownership To be Assigned to Web Servers
    file:
      path: /var/www/demoApp
      state: directory
      recurse: yes
      owner: www-data
      group: www-data

  # The app key is already generated during `composer install` but I'd like to generate it again to be certain.
  # PHP artisan commands to set our app key, migrate our database and give our app a little speed boost
  - name: Set the App Key in your .env file # By default, this command is run following a composer create-project, but lets just confirm   
    command: php /var/www/demoApp/artisan key:generate
  - name: Give Application Speed Boost
    command: php /var/www/demoApp/artisan config:cache
  - name: Install Our MySQL Database
    command: php /var/www/demoApp/artisan migrate:fresh

  # The change modes and owners are duplicate tasks but helps get some commands working when I need them
  # to, without becoming an elevated user.

  # Next we want to configure our web.php file so loading our page redirects to the public folder's file
  # and display the default page for the application
  # Also, we want to ensure the app is accessible by our web servers
  # sudo cp web.php /var/www/demoApp/routes
  # sudo chmod '0775' /var/www/demoApp/routes
  # sudo chown www-data:www-data /var/www/demoApp/routes
  - name: Uncomment the route so the default page opens
    copy:
      src: web.php
      dest: /var/www/demoApp/routes
      mode: '0775'

 # This step ensures that all our modifications above, if contributes to any changes... We can
 # run `composer` commands again to be certain 
  - name: Change The permissions of the app folder back to the user so we can run composer once again
    file:
      path: /var/www/demoApp
      state: directory
      recurse: yes
      owner: ubuntu
      group: ubuntu

- name: Install Application Dependencies with Composer Once More
  hosts: ubuntuServer
  remote_user: ubuntu

  tasks:
  # Install php-curl for the last time just be sure it's updated.
  - name: Install PHP curl again
    command: sudo apt install php8.1-curl -y
  # cd /var/www/demoApp
  # composer update
  - name: Update all libs and dependencies outlined in the /var/www/demoApp/composer.lock
    composer:
      command: update
      working_dir: /var/www/demoApp
  - name: Downloads and installs all libs and dependencies outlined in the /var/www/demoApp/composer.lock
    composer:
      command: install
      working_dir: /var/www/demoApp
  
  # PHP artisan commands to set our app key, migrate our database and give our app a little speed boost AGAIN!
  # By default, this command is run following a composer create-project, but lets just confirm or run it one LAST it
  - name: Set the App Key in your .env file    
    command: php /var/www/demoApp/artisan key:generate
  - name: Give Application Speed Boost
    command: php /var/www/demoApp/artisan config:cache
  - name: Install Our MySQL Database
    command: php /var/www/demoApp/artisan migrate:fresh


  # Now, we can run the remaining commands with elevated privileges
  # First we want to hand the project/app folder over to the web server user and group
- name: Continue Application Deployment
  hosts: ubuntuServer
  remote_user: ubuntu
  become: true

  tasks:
  - name: Change The permissions of the app folder back to the web server so we can access it on the web.
    file:
      path: /var/www/demoApp
      state: directory
      recurse: yes
      owner: www-data
      group: www-data
  
  # Next we want to configure our default web server configurations so the default apache page doesn't show
  # sudo cp demoApp.conf /etc/apache2/sites-available/demoApp.conf
  # sudo chmod '0644' /etc/apache2/sites-available/demoApp.conf
  # sudo chown root:root /etc/apache2/sites-available/demoApp.conf
  - name: Create Our Application Configuration File That will display our page
    copy:
      src: demoApp.conf
      dest: /etc/apache2/sites-available
      owner: root
      group: root
      mode: '0644'

  # Next, we want to make sure our app redirects to the public folder were all the public files ca load correctly
  # sudo cp .htaccess /var/www/demoApp
  # sudo chmod '0775' /var/www/demoApp/.htaccess
  # sudo chown www-data:www-data /var/www/demoApp/.htaccess
  - name: Copy Our HyperText Access File to Redirect Our Page to Display the public files
    copy:
      src: .htaccess
      dest: /var/www/demoApp
  - name: Change the Mode of the .htaccess file so it can also be in the www-data
    file:
      path: /var/www/demoApp/.htaccess
      state: file
      owner: www-data
      group: www-data
      mode: '0775'

  # Name is descriptive
  # sudo a2dissite 000-default.conf
  - name: Disable the default apache web page
    command: a2dissite 000-default.conf
  
  # This ensures our changes take effect on the web server.
  # sudo systemctl restart apache2
  - name: Restart apache
    service:
       name: apache2
       state: restarted

  # sudo a2ensite demoApp.conf
  - name: Enable Laravel Application Configurations To Display as Web Page
    command: a2ensite demoApp.conf
  - name: Restart apache
    service:
       name: apache2
       state: restarted
  
  # # Apache’s mod_rewrite module lets you rewrite URLs more cleanly, translating 
  # human-readable paths into code-friendly query strings. It also enables you to rewrite
  # URLs based on conditions. As done in our .htaccess file
  # sudo a2enmod rewrite
  - name: Enable Apache Rewrite Rules
    command: a2enmod rewrite
  - name: Restart apache
    service:
       name: apache2
       state: restarted

  # We want to secure our site using SSL/TLS from Let's Encrypt
  - name: Install Certbot and Other Dependency
    apt:
     pkg:
     - certbot
     # plugin that integrates Certbot with Apache, making it possible 
     # to automate obtaining a certificate and configuring HTTPS within 
     # your web server with a single command
     - python3-certbot-apache 
     - openssl

  - name: Restart apache
    service:
       name: apache2
       state: restarted   

  - name: Install The SSL certificate on our domain
    command: sudo certbot --apache -w /var/www/demoApp -d tuyojr.me -d www.tuyojr.me --non-interactive --agree-tos -m olutuyod@outlook.com
  
# Next we want to change the ownership of the site from the web, back to the user so we can run composer install again
# and also we can seed our database so our endpoints don't come out empty
- name: Continue Application Deployment
  hosts: ubuntuServer
  remote_user: ubuntu
  become: true

  tasks:
  - name: Change The permissions of the app folder back to the web server so we can access it on the web.
    file:
      path: /var/www/demoApp
      state: directory
      recurse: yes
      owner: ubuntu
      group: ubuntu

- name: Install Application Dependencies with Composer One last time
  hosts: ubuntuServer
  remote_user: ubuntu

  tasks:
  # cd /var/www/demoApp
  # composer update
  - name: Update all libs and dependencies outlined in the /var/www/demoApp/composer.lock
    composer:
      command: update
      working_dir: /var/www/demoApp
  - name: Downloads and installs all libs and dependencies outlined in the /var/www/demoApp/composer.lock
    composer:
      command: install
      working_dir: /var/www/demoApp
  
  # PHP artisan commands to set our app key, migrate our database and give our app a little speed boost
  - name: Give Application Speed Boost
    command: php /var/www/demoApp/artisan config:cache
  - name: Install Our MySQL Database
    command: php /var/www/demoApp/artisan migrate:fresh

# For the last time, we set our app owner to the web server and reload the server
- name: Continue Application Deployment
  hosts: ubuntuServer
  remote_user: ubuntu
  become: true

  tasks:
  - name: Change The permissions of the app folder back to the web server so we can access it on the web.
    file:
      path: /var/www/demoApp
      state: directory
      recurse: yes
      owner: www-data
      group: www-data

  - name: Restart apache
    service:
       name: apache2
       state: restarted 

  
# My configurations are a bit crude, I will commit to working better. Thank you for making it to the end.
# Again, MAY THE FORCE BE WITH YOU!

```

![thankYou](https://github.com/tuyojr/cloudTasks/blob/main/logos/thankYou.gif)
