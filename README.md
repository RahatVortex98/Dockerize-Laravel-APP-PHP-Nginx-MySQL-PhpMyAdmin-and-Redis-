
<h3 align='center'>Create Docker File </h3>


command:

        touch Dockerfile

Inside Docker file:

        # Base image
        FROM php:8.3-fpm
        
        # Install system dependencies
        RUN apt-get update && apt-get install -y \
            git \
            curl \
            zip \
            unzip \
            libpng-dev \
            libonig-dev \
            libxml2-dev \
            libzip-dev \
            && docker-php-ext-install \
                pdo_mysql \
                mbstring \
                zip \
                exif \
                pcntl
        
        # Install Composer
        COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
        
        # Set working directory
        WORKDIR /var/www
        
        # Copy project files
        COPY . .
        
        # Install Laravel dependencies
        RUN composer install --no-interaction --prefer-dist --optimize-autoloader
        
        # Set permissions
        RUN chown -R www-data:www-data /var/www \
            && chmod -R 775 storage bootstrap/cache
        
        # Expose PHP-FPM port
        EXPOSE 9000
        
        # Start PHP-FPM
        CMD ["php-fpm"]

<h3 align='center'>Create Docker Compose File </h3>

command:

       touch docker-compose.yml

Inside the docker-compose.yml:


        version: "3.9"
    
        services:
          app:
            build:
              context: .
              dockerfile: Dockerfile
            container_name: laravel_app
            volumes:
              - .:/var/www
            depends_on:
              - db
        
          web:
            image: nginx:alpine
            container_name: laravel_nginx
            ports:
              - "8000:80"
            volumes:
              - .:/var/www
              - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
            depends_on:
              - app
        
          db:
            image: mysql:8.0
            container_name: laravel_db
            restart: always
            environment:
              MYSQL_DATABASE: laravel
              MYSQL_USER: laravel
              MYSQL_PASSWORD: secret
              MYSQL_ROOT_PASSWORD: root
            ports:
              - "3306:3306"
            volumes:
              - dbdata:/var/lib/mysql
        
        volumes:
          dbdata:

<h3 align='center'>Create default.conf (docker/nginx/default.conf)</h3>

command:

        mkdir docker

        Under that, make a folder named "nginx", and under that, make a file named "default.conf"
Inside the file:

            server {
        listen 80;
        index index.php index.html;
        root /var/www/public;
    
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location ~ \.php$ {
            fastcgi_pass app:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    
        location ~ /\.ht {
            deny all;
        }
        }


Note: make sure your docker-composer.yml and your Laravel's .env file's this part same  👉

        DB_CONNECTION=mysql
        DB_HOST=db
        DB_PORT=3306
        DB_DATABASE=laravel
        DB_USERNAME=laravel
        DB_PASSWORD=secret
 DB sessions:

        SESSION_DRIVER=database
        QUEUE_CONNECTION=database
        CACHE_STORE=database




👉 Open your Docker app on your desktop! 

And write this command in the terminal:

        docker-compose up

👉 Now hit in your browser:

       http://localhost:8000 #mentioned in docker-composer.yml
<p align='center'><img width="48%"  src="https://github.com/user-attachments/assets/22fb6769-cc1e-4566-a5f4-999d620af383" />

</p>
###Some commands:

   Clear Laravel config cache (MANDATORY) :
   
       docker compose exec app php artisan config:clear
       docker compose exec app php artisan cache:clear

   Make sure MySQL container is actually running:

       docker compose ps

   Restart everything cleanly:

       docker compose down
       docker compose up -d

  Run migrations:

       docker compose exec app php artisan migrate

 Run all Laravel commands inside the container:

       
       docker compose exec app php artisan tinker
       docker compose exec app php artisan db:seed
       docker compose exec app php artisan config:clear
          
⚠️If you really want to run PHP on host (not recommended)
    You would need to change .env temporarily:

        DB_HOST=127.0.0.1
        DB_PORT=3306

<h3 align='center'>Add phpMyAdmin to Docker Compose </h3>
Edit your docker-compose.yml and add this phpMyAdmin service:

        phpmyadmin:
          image: phpmyadmin/phpmyadmin
          container_name: laravel_phpmyadmin
          restart: always
          environment:
            PMA_HOST: db        # this must match your MySQL service name
            PMA_USER: laravel
            PMA_PASSWORD: secret
          ports:
            - "8080:80"         # access via http://localhost:8080
          depends_on:
            - db
<h3 align='center'>Add Redis to Docker Compose (optional) </h3>
If you want Redis for caching, queues, or sessions:

        redis:
          image: redis:alpine
          container_name: laravel_redis
          restart: always
          ports:
            - "6379:6379"       # optional for host access

Update your .env for Redis:

        REDIS_HOST=redis
        REDIS_PASSWORD=null
        REDIS_PORT=6379
        
Rebuild your Docker environment
After editing docker-compose.yml:

        docker compose down -v
        docker compose up -d --build

For phpMyAdmin access:

        http://localhost:8080

For Redis access  
Install any one:

        Redis Desktop Manager
        TablePlus
        Medis

For Nginx access :

        http://localhost:8000
Access summary

        | Service            | How to access                                  |
        | ------------------ | ---------------------------------------------- |
        | Laravel            | [http://localhost:8000](http://localhost:8000) |
        | phpMyAdmin         | [http://localhost:8080](http://localhost:8080) |
        | MySQL (Laravel)    | `DB_HOST=db`                                   |
        | MySQL (phpMyAdmin) | Server = `db`                                  |
        | Redis (CLI)        | `docker compose exec redis redis-cli`          |
        | Redis (GUI)        | localhost:6379                                 |
        | Nginx logs         | `docker compose logs web`                      |

        
<p align='center'><img width="48%"  src="https://github.com/user-attachments/assets/9533433d-7075-4007-92a9-beac06fb9247" />
</p>


