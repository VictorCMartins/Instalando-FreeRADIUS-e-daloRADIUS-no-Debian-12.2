# Instalando-FreeRADIUS-e-daloRADIUS-no-Debian-12.2

#!/bin/sh

sudo apt-get update -y
sudo apt-get upgrade -y

#Instalar Apache
sudo apt-get install apache2

#Instalar PHP
sudo apt-get install php libapache2-mod-php php-gd php-common php-mail php-mail-mime php-mysql php-pear php-db php-mbstring php-xml php-curl

#Instalar MariaDB
sudo apt -y install mariadb-server mariadb-client


#Configurando banco de dados para FreeRADIUS
sudo mysql_secure_installation

Enter current password for root (enter for none): Just press the Enter
Set root password? [Y/n]: Y
New password: Enter password
Re-enter new password: Repeat password
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y

sudo mysql -u root -p
CREATE DATABASE radius CHARACTER SET UTF8 COLLATE UTF8_BIN;
CREATE USER 'radius'@'%' IDENTIFIED BY 'radius';
GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'%';
QUIT;


#Instalar FreeRADIUS
sudo apt policy freeradius
sudo apt -y install freeradius freeradius-mysql freeradius-utils

#Depois de instalado, importando o esquema do banco de dados MySQL do freeradius com o seguinte comando:
sudo su -
sudo mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
sudo  mysql -u root -p -e "use radius;show tables;"

#Criando um link virtual para o módulo sql em:
sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/


#Fazendo as seguintes alterações de acordo com seu banco de dados:
sudo nano /etc/freeradius/3.0/mods-enabled/sql

sql {
driver = "rlm_sql_mysql"
dialect = "mysql"

        mysql {
                # If any of the files below are set, TLS encryption is enabled
#                tls {
#                       ca_file = "/etc/ssl/certs/ca-cert.pem"
#                        ca_path = "/etc/ssl/certs/"
#                       #certificate_file = "/etc/ssl/certs/private/client.crt"
                        #private_key_file = "/etc/ssl/certs/private/client.key"
#                        cipher = "DHE-RSA-AES256-SHA:AES128-SHA"
#                        tls_required = no
#                        tls_check_cert = no
#                        tls_check_cert_cn = no
#                }

# Connection info:
server = "localhost"
port = 3306
login = "radius"
password = "radius"

# Database table configuration for everything except Oracle
radius_db = "radius"
}

read_clients = yes
client_table = "nas"

#Salve e feche

#Definindo as permissês adequadas
sudo chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql

sudo systemctl restart freeradius
sudo systemctl status freeradius

#Instalar daloRADIUS
sudo apt -y install wget unzip
wget https://github.com/lirantal/daloradius/archive/refs/tags/1.3.zip
unzip 1.3.zip
mv daloradius-1.3 daloradius
cd daloradius


#Configurando o daloradius
sudo mysql -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql 
sudo mysql -u root -p radius < contrib/db/mysql-daloradius.sql

cd ..
sudo mv daloradius /var/www/html/

sudo chown -R www-data:www-data /var/www/html/daloradius/
sudo cp /var/www/html/daloradius/library/daloradius.conf.php.sample /var/www/html/daloradius/library/daloradius.conf.php
sudo chmod 664 /var/www/html/daloradius/library/daloradius.conf.php

sudo nano /var/www/html/daloradius/library/daloradius.conf.php

#Fazendo as seguintes alterações que correspondam ao seu banco de dados:
sudo nano /var/www/html/daloradius/library/daloradius.conf.php

$configValues['CONFIG_DB_HOST'] = 'localhost';
$configValues['CONFIG_DB_PORT'] = '3306';
$configValues['CONFIG_DB_USER'] = 'radius';
$configValues['CONFIG_DB_PASS'] = 'radius';
$configValues['CONFIG_DB_NAME'] = 'radius';

#Importando as tabelas daloRAIUS MySQL para o banco de dados FreeRADIUS
cd /var/www/html/daloradius/
sudo mysql -u root -p radius < /var/www/html/daloradius/contrib/db/fr2-mysql-daloradius-and-freeradius.sql
sudo mysql -u root -p radius < /var/www/html/daloradius/contrib/db/mysql-daloradius.sql

sudo chown -R www-data:www-data /var/www/html/daloradius/

sudo systemctl restart freeradius.service apache2

#Visite: 
http://localhost/daloradius/login.php

Username: administrator
Password: radius
