# install_frape14
Install instructions to frappe/erpnext 14 on ubuntu 22.04

timedatectl set-timezone "America/Guatemala"

sudo apt-get update -y
sudo apt-get upgrade -y

sudo adduser frappe
sudo usermod -aG sudo frappe
su frappe
cd /home/frappe/


sudo apt-get install git

sudo apt-get install python3-dev python3.10-dev python3-setuptools python3-pip python3-distutils

sudo apt-get install python3.10-venv

sudo apt-get install software-properties-common

sudo apt install mariadb-server mariadb-client

mariadb --version

sudo apt-get install redis-server

sudo apt-get install xvfb libfontconfig wkhtmltopdf

sudo apt-get install libmysqlclient-dev

sudo service mysql restart
sudo mysql_secure_installation


#
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
# change line to utf8mb4_unicode_ci or fail next steps

#mariadb.conf.d
sudo nano /etc/mysql/my.cnf

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
#innodb_buffer_pool_size=6G
#innodb_log_file_size=2G
#query_cache_size=64M
#skip-name-resolve
#max_heap_table_size=64M
#tmp_table_size=64M

[mysql]
default-character-set = utf8mb4
sudo service mysql restart

sudo apt install curl
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile
nvm install 16.15.0
sudo apt-get install npm
sudo npm install -g yarn

sudo pip3 install frappe-bench
bench init --frappe-branch version-14 frappe-bench

cd frappe-bench/
chmod -R o+rx /home/frappe/

bench new-site misitio.gt

bench get-app --branch version-14 erpnext
bench get-app payments
bench get-app hrms
bench get-app ecommerce_integrations --branch main

bench --site misitio.gt install-app erpnext
bench --site misitio.gt install-app payments
bench --site misitio.gt install-app hrms
bench --site misitio.gt install-app ecommerce_integrations

bench --site  misitio.gt enable-scheduler
bench --site  misitio.gt set-maintenance-mode off

sudo bench setup production frappe

bench setup nginx

sudo supervisorctl restart all

sudo bench setup production frappe

# Optional POST install
# Change version pdf 

sudo apt-get remove wkhtmltopdf
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
sudo apt-get install -y xfonts-75dpi
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb
wkhtmltopdf -V

# Add Firewall support
ufw allow 22,25,143,80,443,3306,3022,8000/tcp
ufw enable


# Add multi-site DNS option

bench config dns_multitenant on

bench new-site misitio2.gt
bench setup nginx
sudo service nginx reload

bench --site misitio2.gt install-app erpnext
bench --site misitio2.gt install-app payments
bench --site misitio2.gt install-app hrms
bench --site misitio2.gt install-app ecommerce_integrations


# Add developer mode
bench --site misitio.gt set-config --global developer_mode 1

# Delete site
bench drop-site misitio.gt --no-backup

