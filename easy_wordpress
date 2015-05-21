#!/bin/bash
##################################   Easy Wordpress ##############################
###
#Author: Sumit Joshi
#Description: This shell script rapidly creates wordpress hosting environment on Ubuntu 12.04
#Date: May 20 2015
#
#PreRequisites
#
#
#
#

function install_nginx
{
	#echo "This function will install nginx"
	#sudo apt-get update

	sudo apt-get -y install nginx || (echo "Nginx installation Failed" && exit)

	#Start nginx
	sudo /etc/init.d/nginx start
	sudo /etc/init.d/nginx status

	#start nginx in all runlevels
	sudo update-rc.d nginx defaults
	return 0
}

function install_php5-fpm
{
	#echo "This function will install php"
	#Install php5-fpm
	apt-get update || (echo "Update Failed" && exit)
	apt-get -y install php5-fpm || (echo "php5-fpm Installation failed" && exit )

	#configure php
	#uncomment line in php.ini to make it work
        if [ -s /etc/php5/fpm/php.ini ];then
		sed -i "/=msql.so/ s/;/ /" /etc/php5/fpm/php.ini
	fi

	#sudo sed -i "/cgi.fix_pathinfo=/ s/1/0/" /etc/php5/fpm/php.ini
	#sudo sed -i "/listen = / s/127\.0\.0\.1\:9000/\/var\/run\/php5-fpm\.sock/" /etc/php5/fpm/pool.d/www.conf

	#restart php
	sudo service php5-fpm restart || (echo "PHP restart Failed" && exit)
	return 0
}

function install_mysql
{
	#echo "This function will install mysql"
	sudo apt-get update || (echo "Update Failed" && exit)
	
	#Set MySQL password in advance
	echo "mysql-server mysql-server/root_password password sumit" | debconf-set-selections
	echo "mysql-server mysql-server/root_password_again password sumit" | debconf-set-selections
	
	#Install MySQL
	apt-get -y install mysql-server php5-mysql || (echo "MySQL installation Failed" && exit)
	
	#configure MySQL
	sudo mysql_install_db
	sudo /usr/bin/mysql_secure_installation

	return 0
}

function is_installed 
{
	pkg_name=$1
	hash $pkg_name &> /dev/null
	if [ $? -eq 1 ];
	then
		echo "$pkg_name is not installed"
		echo "##"
		echo "# $pkg_name will be installed."
		echo "##"
		install_$pkg_name
		if [ $? -ne 0 ];then
			echo "Failed to install $pkg_name "
			exit 1
		fi
	else
		echo "$pkg_name is already installed"
		echo 
		echo 
	fi
	return 0
}

function ask_domain_add_hosts
{
	#ask for domian name
	echo "Please provide domain name like example.com, example.ac.in, example.in....."
	read dom

	if [ "$dom" == "" ];then
        	echo "please provide valid domain name"
        	exit 1
	fi
	
	#add entry of $dom in /etc/hosts
	sudo echo "127.0.0.1 $dom" >> /etc/hosts
	return 0
}

function set_nginx_config
{
	echo "Setting up nginx......."
	nginx_home=$1
	site_config=$2

	#Create root document and logs for your website
	sudo mkdir -p $nginx_home/$dom/htdocs/  $nginx_home/$dom/logs/

	#Create nginx server block for your domain
	#Here separate file is used for defining nginx server block (e.g sample_nginx_config file)
	
	if [ -s sample_nginx_config ];then
		sudo cp sample_nginx_config $site_config/$dom
	else
		echo -e "Failed to copy default nginx file\nFile does not exist"
		exit 1
	fi

	#Another approach: for defining nginx server block
	#cat <<EOF > $site_config/$dom
#server {
#        server_name example.com www.example.com;
#
#        access_log /var/log/nginx/example.com.access.log;
#        error_log /var/log/nginx/example.com.error.log;
#
#        root /usr/share/nginx/www/example.com/htdocs;
#        index index.php index.html index.htm;
#
#        location / {
#               try_files $uri $uri/ /index.php?q=$uri&$args;
#        }
#
#	error_page 404 /404.html;
#
#        #passes this to php-fpm socket
#       location ~ \.php$ {
#        	try_files $uri =404;
#        	fastcgi_pass 127.0.0.1:9000;
#		#fastcgi_pass unix:/var/run/php5-fpm.sock
#		fastcgi_index index.php;
#        	include fastcgi_params;
#        }
#}
#EOF
#
	#Make changes in nginx server block defined in /etc/nginx/sites-available/example.com
	sudo sed -i "s/example\.com/$dom/g" $site_config/$dom
	#Make symlink for sites-enabled
	sudo ln -s $site_config/$dom /etc/nginx/sites-enabled/
	nginx -t
	
	#Make symlink for log files
	#Now log is visble in two locations 1)/var/log/nginx/$dom.access.log   2)/usr/share/nginx/www/$dom/access.log
	sudo ln -s /var/log/nginx/$dom.access.log $nginx_home/$dom/logs/access.log
	sudo ln -s /var/log/nginx/$dom.error.log $nginx_home/$dom/logs/error.log

	#restart nginx
        sudo service nginx restart
	return 0
}


function download_wordpress
{
	home=$1
	
	echo "Downloading Wordpress........."
	#download wordpress 
	#tar has better option of no-stripping
	sudo wget -P $home/$dom/htdocs/ http://wordpress.org/latest.zip || (echo "Failed to download Wordpress..." && exit 1)
	sudo unzip $home/$dom/htdocs/latest.zip -d $home/$dom/htdocs/ || (echo "Unzipping opertaion failed..." && exit 1)
	
	#Doing similar to --strip-components=1 used in tar....Removing wordpress directory
	sudo mv $home/$dom/htdocs/wordpress/* $home/$dom/htdocs/
	sudo rm -rf $home/$dom/htdocs/wordpress/
	sudo rm $home/$dom/htdocs/latest.zip
	return 0
}

function create_db_wordpress
{
	quote="\`"
	last="_db"
	
	wp_db=""
	wp_db+=$quote
	wp_db+=$dom
	wp_db+=$last
	wp_db+=$quote
	
	echo "DB $wp_db will be created"

	mysql -u root -psumit -e "create database $wp_db;"
	if [ $? -ne 0 ];then
		echo "Failed to create Wordpress database"
		exit 1
	fi
	return 0
}

function configure_wordpress
{
	nginx_home=$1
	wp_home=$2
	wp_user=$3
	wp_passwd=$4

	echo "You are just few steps away from your Wordpress Site...."
	echo "Configuring Wordpress......"

	#Change in MYSQL DB
	#Create new wordpress user with password and give him privilege to access $dom_db database
	q1="GRANT USAGE ON *.* TO $wp_user@localhost IDENTIFIED BY '$wp_passwd';"
	q2="GRANT ALL PRIVILEGES ON $wp_db.* TO $wp_user@localhost;"
	q3="FLUSH PRIVILEGES;"
	SQL="${q1}${q2}${q3}"

	mysql -u root -psumit -e "$SQL"
	if [ $? -ne 0 ];then
		echo "Changes failed in MYSQL DB"
	fi

	#Do changes in wp-config.php
	if [ -s "$nginx_home/$dom/htdocs/wp-config-sample.php" ];then
	        sudo cp "$nginx_home/$dom/htdocs/wp-config-sample.php" "$nginx_home/$dom/htdocs/wp-config.php"

		#Make changes in wp-config.php
		#Change DBname,WordpressUser,Password
		sed -i "/DB_NAME/ s/database_name_here/$dom$last/" $nginx_home/$dom/htdocs/wp-config.php
		sed -i "/DB_USER/ s/username_here/$wp_user/" $nginx_home/$dom/htdocs/wp-config.php
		sed -i "/DB_PASSWORD/ s/password_here/$wp_passwd/" $nginx_home/$dom/htdocs/wp-config.php
	else
		echo "wp-config-sample.php file does not exist"
		exit 1
	fi
	
	#set permissions
	sudo chown -R www-data:www-data $nginx_home/$dom

	sudo /etc/init.d/php5-fpm restart
	sudo /etc/init.d/nginx restart
	
	#Openup example.com in browser
	echo "***************"
	echo "***Congo!! Your wordpress is successfully installed..."
	echo "***Open $dom in your browser"
	echo "***************"
	return 0
}

#Read from here

packages="php5-fpm mysql nginx"
nginx_home="/usr/share/nginx/www"
site_config="/etc/nginx/sites-available"
#mysql root username password db_name
#uname=root
#passwd=sumit
#db_name=$dom_db
wordpress_loc="$nginx_home/$dom/htdocs/wordpress"
wordpress_user="wordpuser"
wordpress_password="password"


#1) Check necessary packages and install them
for pkg_name in $packages; do
	is_installed $pkg_name
done
if [ $? -ne 0 ];then
        echo "Failed in Package Installation mode..."
        exit 1
fi
echo

#2) Ask user for domain name
ask_domain_add_hosts

#3) Set nginx config
set_nginx_config $nginx_home $site_config
if [ $? -ne 0 ];then
        echo "Failed in nginx configuration"
	exit 1
fi
echo 

#4) Download wordpress
download_wordpress $nginx_home $site_config
if [ $? -ne 0 ];then
       echo "Failed to download Wordpress"
        exit 1
fi

#5)Create Wordpress Database
create_db_wordpress
if [ $? -ne 0 ];then
        echo "Failed to create Wordpress DB"
        exit 1
fi

#6) Setup wordpress
configure_wordpress $nginx_home $wordpress_loc $wordpress_user $wordpress_password $dom
if [ $? -ne 0 ];then
	echo "Failed to configure Wordpress"
	exit 1
fi
