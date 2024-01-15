# Laravel on Ubuntu
This instructions are based on others knowledge and resources on the internet combined with my own experince.

This this instruction on your own risk.

This is my setup
* Ubuntu 22.04
* PHP 8.2
* Mysql-server
* Nginx
* Node

## Installation steps
1. Set-up server (I use AWS EC2)
2. SSH into server.
3. update and upgrade server.
4. Install PHP (per now, I prefer 8.2.). You can use the ondrej/php repository.
5. Install Mysql-server
6. Install npm and node.
7. Install other server requirements.

Now server is ready. You can proceed setting up your project.

## Set up laravel project
1. Create a project user and add user to www-data group
2. Generate SSH key for the user
3. Create a project directory inside `/var/www/html`
4. Give www-data ownership, and write access for the group to the new directory.
5. CD into the project dir and clone github repository to the project dir root. I.e. /var/www/html/project.
6. Give write access to group for /storage and /bootstrap/cache.
7. Create a database for the project and a database user.
8. Copy .env.example to .env.
9. Run `php artisan key:generate` with the project-user you have created.
10. Edit .env. Set environment to production and debug to false on a production (published website) and set the correct database values.
11. ```bash
    composer install --optimize-autoloader --no-dev
    ```
12. You can now migrate and seed database and start browsing your site.

## Frequently used paths
| Path                       | Description                                                                         |
|----------------------------|-------------------------------------------------------------------------------------|
| `/var/www/html`             | The common path to have your projects located. I.e. /var/www/html/laravel_project   |
| `/home/ubuntu/.ssh`          | SSH keys                                                                            |
| `/etc/nginx/sites-available` | NGINX config files. Name config file after domain. Filename example: sub.domain.com |
|                            |                                                                                     |
|                            |                                                                                     |
|                            |                                                                                     |

## Composer and artisan
I prefer using a project-user, as described below, when running composer and artisan.

## Server users
You will typically log in as ubuntu. I will create a new user for each project, and add the user to www-data group.
Add user:
```bash
sudo adduser tools
```

Add user to group: 
```bash
sudo usermod -aG www-data project_user
```

Switch user with 
```bash
sudo -u project_user bash
```

Use project_user for composer and php artisan.

## Project permissions
Set ownership in your project
```bash
sudo chown -R project_user:www-data /var/www/html/laravel_project
```
Set ownership of storage and bootstrap cache to www-data (common nginx server user).
```bash
sudo chown -R www-data storage
sudo chown -R www-data bootstrap/cache
```

Give www-data group access to write the directories above.
```bash
sudo chmod -R g+w /var/www/html/tools/storage
sudo chmod -R g+w /var/www/html/tools/bootstrap/cache
```

## NGINX config
```nginx
# Some lines from ChatGPT commented out.
server {
    listen 80;
    listen [::]:80;
    server_name sub.domain.com;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    root /var/www/html/tools/laravel_project;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    error_page 404 /index.php;

    location ~ \.php$ {
#        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
#        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

#    location ~ /\.ht {
#        deny all;
#    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Install PHP
I will go with PHP 8.2.
1. ```bash
   sudo add-apt-repository ppa:ondrej/php
   ```
2. ```bash
   sudo apt-get update
   ```
3. ```bash
   sudo apt-get install php8.2 php8.2-fpm php8.2-mysql php8.2-ctype php8.2-curl php8.2-dom php8.2-fileinfo php8.2-filter php8.2-hash php8.2-mbstring php8.2-openssl php8.2-pcre php8.2-pdo php8.2-session php8.2-tokenizer php8.2-xml
   ```

## Generate SSH key to access Github
Generate a new SSH key for the user (on the Ubuntu server) who are going to clone repository. It shall be stored in /home/username/.ssh.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
Add SSH Key to SSH Agent:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Copy the public key and paste into Github SSH settings:
```bash
cat ~/.ssh/id_rsa.pub
```
Copy the output.
