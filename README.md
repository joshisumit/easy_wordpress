# Easy Wordpress

**Easy Wordpress** easily creates wordpress hosting environment.

**Easy Wordpress** is a shell script that rapidly installs wordpress on Ubuntu 12.04, by eliminating a lot of the up front setup.

- You don't need to install/configure LEMP Stack(Linux,nginx,MySQL,PHP). :)
- If you have already installed LEMP Stack, then it directly installs Wordpress on it.
- In a few minutes you'll be set up with a minimal, wordpress environment like the one below 
(giving you more time to spend on writing epic blog posts!)
 


##Pre-requisites

1. This script requires sudo permission for its execution or it should be executed with root user.
2. This script requires `sample_nginx_config` file for its execution.
3. This script installs mysql with username=`root` and password=`sumit`. You should change your mysql password once it is installed.


##How to Set it up ?

    git clone https://github.com/joshisumit/easy_wordpress.git
    
Execute script with:

    ./easy_wordpress.sh
    
    
## Verify your wordpress installation

Once script has completed successfully,just open example.com in your browser, wordpress installation wizard will greet you (e.g. `example.com/wp-admin/install.php` )

SCript makes changes in following directories:

1. Check your example.com configuration (nginx server block) - `/etc/nginx/sites-available/example.com`
2. Check your example.com contents - `/usr/share/nginx/www/example.com/htdocs`
3. Login to mysql database :




    mysql -u root -p sumit
    
    show databases;
    
    use example.com_db;
    
    show tables;
    
    SELECT User FROM mysql.user;

