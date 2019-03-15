# Magento Developer Box

The purpose of this Magento Developer Box is to allow for testing and developing locally with the latest Magento release.

I have taken inspiration for the box from [the documentation](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/build/optimal-dev-environment.html) but have chosen MariaDB over Percona for the MySQL drop-in replacement and Nginx as webserver.

In this guide you can found:
- Magento 2.3 Developer Documentation
- Provisioning
	- Shared folders
	- Virtual hosts
	- Mysql
- Fresh Installation
- Add Existing Magento


### Magento 2.3 Documentation
Developer documentation
https://devdocs.magento.com/#/individual-contributors

Dev-Environments
https://devdocs.magento.com/guides/v2.3/extension-dev-guide/build/optimal-dev-environment.html

Original Box
https://github.com/Repox/magrant


## Provisioning

### Shared foldes

There are two shared folders

1. The **root** folder of this project which is shared in the /vagrant folder into the the machine
2. The **$HOME/projects/www** folder which is shared the web projects. Please note that folder is configurable. You should change this folder for your own personal projects folder.
 
### Virtual Hosts

 VirtualHosts are inside **/ansible/roles/nginx/files/**. 
 You can add a new virtualhost in this folder. After that, add the creation task in the main.yml (**/ansible/roles/nginx/tasks/main.yml**)

### Mysql

You can add a new database in the mysql role (**/ansible/roles/mysql/vars/main.yml**). 
When you run *vagrant up* or *vagrant reload* (if the machine is running) the new database will be created.

## FRESH INSTALLATION

### Authentication keys

You need authentiation keys for installing Magento via Composer.

Documentation: https://devdocs.magento.com/guides/v2.3/install-gde/prereq/connect-auth.html

Quicklink (requires account): https://marketplace.magento.com/customer/accessKeys/ 

### Installation via Composer (requires authentication keys)

Documentation: https://devdocs.magento.com/guides/v2.3/install-gde/install/cli/install-cli-sample-data-composer.html

    $ cd /vagrant
    $ composer create-project --repository=https://repo.magento.com/ magento/project-community-edition application

### Setup up Magento via CLI:

 The following installation instruction is for local development based on the Vagrant setup.

 Please review the options to ensure that settings are as you need.

     $ cd /vagrant/application
     $ bin/magento setup:install \
	--base-url=http://192.168.22.65 \
	--db-host=localhost \
	--db-name=vagrant \
	--db-user=vagrant \
	--db-password=vagrant \
	--backend-frontname=admin \
	--admin-firstname=admin \
	--admin-lastname=admin \
	--admin-email=admin@admin.com \
	--admin-user=admin \
	--admin-password=admin123 \
	--language=en_US \
	--currency=USD \
	--timezone=Etc/UTC \
	--use-rewrites=1

### Use Redis for cache

Documentation: https://devdocs.magento.com/guides/v2.3/config-guide/redis/redis-pg-cache.html

    $ cd /vagrant/application
    $ bin/magento setup:config:set --cache-backend=redis

### Setup cron for Magento

Documentation: https://devdocs.magento.com/guides/v2.3/config-guide/cli/config-cli-subcommands-cron.html

You can install crontabs and not worry about them:

    $ cd /vagrant/application
    $ php bin/magento cron:install


Or you can run them when needed:

    $ cd /vagrant/application
    $ php bin/magento cron:run


### Magento mode

Documentation: https://devdocs.magento.com/guides/v2.3/config-guide/cli/config-cli-subcommands-mode.html

Default mode for running Magento is `default`.

You can change this mode to be either `developer` or `production`:

    $ cd /vagrant
    $ bin/magento deploy:mode:set {mode}

### Use Elasticsearch for Catalog Search

Elasticsearch 6.x is installed. Configuration is handled in the administration panel.

Nginx is configured with localhost proxy.

Documentation: https://devdocs.magento.com/guides/v2.3/config-guide/elasticsearch/configure-magento.html

## ADD EXISTING MAGENTO
Clone the gitlab repository in your project folder

    cd $HOME/projects/www
    git clone <git-url> <name-project>
    
If the machine is not running, run it.

    vagrant up
    
Access to the machine with SSH

    vagrant ssh
    
Test the virtualhost

    cd /etc/nginx/sites-available/<virtualhost>
    cd ..
    cd /etc/nginx/sites-enable/<virtualhost>

If the symbolic link doesn't exist, create it.

    cd /etc/nginx/sites-enable/
    sudo ln -s /etc/nginx/sites-available/<virtualhost> /etc/nginx/sites-enabled/
    sudo servic nginx restart

Dump the orignal database. Next, a command line example:
    
    ## Login into server
    ssh -i "<cert>.pem" <usuario>@<ip>
    ## Export database
    mysqldump -u<user> -p <database> > <name>_backup.sql
    ## Logout server
    exit
    ## Get dump through rsync
    rsync -av --progress -e "ssh -i <cert>.pem" <usuario>@<ip>:/<path>/<server>/<name>_backup.sql /<path>/<local>/<name>_backup.sql
    
If the database doesn't exist, create it in the vagrant machine:
    
    ## Login in mysql
    mysql -uvagrant -pvagrant
    ## Show  databases
    show databases;
    ## Create database
    create database <database_name>;
    ## Exit
    exit

Import the database

    mysql -uvagrant -pvagrant <database_name> < <dump>_back.sql
    
Test the import

    ## Login in mysql
    mysql -uvagrant -pvagrant
    ## Select database
    use <database_name>;
    ## Show tables
    show tables;
     ## Exit
    exit

Surely the development urls are different. You must change it. Command line example:

    mysql -uvagrant -pvagrant -e "USE <database_name>;
    UPDATE core_config_data set value = '0' where path = 'web/secure/use_in_frontend';
    UPDATE core_config_data set value = '0' where path = 'web/secure/use_in_adminhtml';
    UPDATE core_config_data set value = 'http://<url>' where path = 'web/unsecure/base_url';
    UPDATE core_config_data set value = 'https://<url>' where path = 'web/secure/base_url';
    UPDATE core_config_data set value = 'https://<url>' where path = 'web/secure/base_link_url';
    UPDATE core_config_data set value = 'mysql' where path = 'catalog/search/engine';"

Go to the project folder and install the project dependencies.   

    cd /var/www/<name-project>
    composer install
    
Clean cache and upload project

    bin/magento deploy:mode:set developer
    bin/magento setup:upgrade
    bin/magento cache:clean
    bin/magento cache:flush
    rm -rf var/cache/*
    rm -rf var/page-cache/*
    rm -rf generated/code/*
    
Go to browser and  test the installation

**HAPPY CODDING!!**