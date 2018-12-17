# Drupal-nginx-php Docker with Redis
This is a Drupal Docker image with redis 

# Components
This docker image currently contains the following components:
1. Drupal (8.5.1)  
2. nginx (1.13.8)
3. PHP (7.0.27)
4. Redis
5. Drush 
6. Composer (1.6.1)
7. MariaDB ( 10.1.26/if using Local Database )
8. Phpmyadmin ( 4.7.7/if using Local Database )

How to build a custom container with Drupal & Redis


	1. Download the Azure App service default image to your local
	https://github.com/Azure-App-Service/drupal-nginx-fpm
	
	2. Edit the docker file and add the below lines to install redis
	
	RUN set -ex \
	    && apt-get update \
		&& apt-get install -y -V --no-install-recommends redis-server 
	
	RUN pecl install -o -f redis \
	    &&  rm -rf /tmp/pear

	3. Add the below command on entrypoint.sh to unzip redis and build the extension 


	setup_redis_extension(){
	    #unzip phpredis and build the extension
	
	   echo "extension=redis.so" > /etc/php/7.0/mods-available/redis.ini
	   ln -sf /etc/php/7.0/mods-available/redis.ini /etc/php/7.0/fpm/conf.d/20-redis.ini
	   ln -sf /etc/php/7.0/mods-available/redis.ini /etc/php/7.0/cli/conf.d/20-redis.ini
	
	    echo "\$conf['redis_cache_socket'] = '/tmp/redis.sock';" >> "$DRUPAL_HOME/sites/default/settings.php"
	    echo "\$settings['redis.connection']['base'] = 1;" >> "$DRUPAL_HOME/sites/default/settings.php"
	    echo "\$settings['cache']['default'] = 'cache.backend.redis';" >> "$DRUPAL_HOME/sites/default/settings.php"
	    echo "\$settings['redis.connection']['host'] = '127.0.0.1';" >> "$DRUPAL_HOME/sites/default/settings.php"
	    echo "\$settings['cache']['default'] = 'cache.backend.redis';" >> "$DRUPAL_HOME/sites/default/settings.php"
	}

	4. Now build the image with 
	docker build -t <docker hub ID>/drupal-redis:0.1 .
	
	5. Push the image to docker hub
	Docker push <docker hub ID>/drupal-redis:0.1>
	
	6. Create a new web app for container on Azure portal and select the configuration container with the docker hub
	7. Install the Drupal image with database configuration
	8. Add the below to settings.php under /home/site/wwwroot/sites/default/settings.php file 
	
	$conf['redis_client_interface'] = 'Predis';
	$conf['redis_client_host'] = 'hostname';
	$conf['redis_client_port'] = port;
	$conf['redis_client_password'] = 'password';
	$conf['lock_inc'] = 'sites/all/modules/contrib/redis/redis.lock.inc';
	$conf['cache_backends'][] = 'sites/all/modules/contrib/redis/redis.autoload.inc';
	$conf['cache_default_class'] = 'Redis_Cache';
	
	9. If you are getting an access denied error while accessing the  settings.php file SSH to the container and modify the file permissions
	chmod -R 777 settings.php
	
	10. Replace Hostname with your Database host name
	11. Replace port with your database port
	12. Replace password with your Database password
	13. Save the settings.php file
	14. To check if redis is working or not, SSH to the container and type redis-cli or redis-cli ping
