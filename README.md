## Pre-requisite

Before you try to set up [Cloud Docker for Commerce](https://developer.adobe.com/commerce/cloud-tools/docker/setup/#php-and-composer) on your local machine, make sure you have **PHP** installed **and** ensure that the following PHP extensions/modules are installed/enabled:  
```  
bcmath  bz2  calendar  exif  gd  gettext  intl  libxml  mysqli opcache  pcntl  pdo_mysql  Reflection  soap  sockets  SPL  standard  swoole  sysvmsg  sysvsem  sysvshm  zip  zlib  
```  
See a list of system requirements here, [https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirement](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirements)

Also, make sure you have composer installed on your system.

## Build a Project by Composer.

To start setting up Magento 2, the first step is to create a directory. After that, you can proceed with running the code below:

```  
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition <install-directory-name>
```


If asked for credentials for repo.magento.com, go to here https://commercemarketplace.adobe.com/customer/accessKeys/ and in the username use public key and in the password, use private key.

If any problem occurs, this is probably because of missing php extension.


The provided command will install the most recent release of Magento. If you wish to install a particular version of Magento, input the corresponding version number. For instance:

```  
magento/project-community-edition=2.4.2  
```

## Include the ece-tools and Cloud Docker in the setup

Cloud Docker allows easy deployment of Magento 2.4 in a Docker environment, serving purposes like development, testing, and automation. To include the `ece-tools` and `cloud docker` for community packages, execute the command

```  
composer require --no-update --dev magento/ece-tools magento/magento-cloud-docker  
```

Make sure you are in the project directory before running the command.

## Create a default configuration source file

We now generate the default configuration source file called `.magento.docker.yml`. This file is utilized to construct the Docker containers for the local environment.

```  
name: magento    
  
system:    
  mode: 'production'    
  
  
services:    
  php:    
    version: '8.4'  # use your php version here,   
    extensions:     # if you use 8.4 here and on your host machine  
      enabled:      # life would be easier for you.  
        - xsl    
        - json    
        - redis    
  mysql:    
    version: '10.4'    
    image: 'mariadb'    
  redis:    
    version: '6.0'    
    image: 'redis'    
  elasticsearch:    
    version: '7.11'    
    image: 'magento/magento-cloud-docker-elasticsearch'    
  
hooks:    
  build: |    
    set -e    
    php ./vendor/bin/ece-tools run scenario/build/generate.xml    
    php ./vendor/bin/ece-tools run scenario/build/transfer.xml    
  deploy: 'php ./vendor/bin/ece-tools run scenario/deploy.xml'    
  post_deploy: 'php ./vendor/bin/ece-tools run scenario/post-deploy.xml'    
  
mounts:    
  var:    
    path: 'var'    
  app-etc:    
    path: 'app/etc'    
  pub-media:    
    path: 'pub/media'    
  pub-static:    
    path: 'pub/static'  
```

## Use the Magento Installation Script

Download the script from [https://github.com/magento/magento-cloud-docker/releases/](https://github.com/magento/magento-cloud-docker/releases/) Move `bin/init-docker.sh` into your project and execute it.

As i have PHP 8.4 installed I ran the script with flag   
```  
./init-docker.sh --php 8.4 --image 1.4.6  
```

After this, run `composer update` on project root.


### Generate the Docker Compose Configuration file

```
./vendor/bin/ece-docker build:compose --mode="developer"
```

## Build files to the container

To build files into the container and run them in the background, use the command below: `docker-compose up -d`

## Deploy Magento as a Docker container

```  
docker compose run --rm deploy cloud-deploy  
docker compose run --rm deploy magento-command deploy:mode:set developer  
docker compose run --rm deploy cloud-post-deploy  
  
```

## Configure and Connect Varnish

Next, set up varnish and connect it with Magento.

```  
docker compose run --rm deploy magento-command config:set system/full_page_cache/caching_application 2 --lock-env  
  
docker compose run --rm deploy magento-command setup:config:set --http-cache-hosts=varnish  
```

## Clear the Cache

To complete the process, clear the cache and verify the results. Use the command below to clear the cache:

```  
docker compose run --rm deploy magento-command cache:clean  
```

After this, navigate to https://localhost


## Disable 2FA Module

```
bin/magento module:disable Magento_TwoFactorAuth
bin/magento cache:flush
```

## Create an Admin user

```
bin/magento admin:user:create \
--admin-user='admin' \
--admin-password='admin123' \
--admin-email='admin@example.com' \
--admin-firstname='Admin' \
--admin-lastname='User'
```


## References

1. [https://developer.adobe.com/commerce/cloud-tools/docker/setup/](https://developer.adobe.com/commerce/cloud-tools/docker/setup/)  
2. [https://developer.adobe.com/commerce/php/development/build/development-environment/](https://developer.adobe.com/commerce/php/development/build/development-environment/ )
3. [https://www.mgt-commerce.com/tutorial/magento-2-docker/](https://www.mgt-commerce.com/tutorial/magento-2-docker/)  
4. [https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/overview](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/overview)
