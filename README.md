# Instruções de Instalação do FreeRADIUS e daloRADIUS

Este guia fornece instruções passo a passo para a instalação do FreeRADIUS e daloRADIUS em um ambiente Linux. Esses passos foram testados em um ambiente Ubuntu e podem variar dependendo da distribuição do Linux.

Pré-requisitos:

-Ambiente Linux (este guia usa Ubuntu);

-Acesso de administrador (ou sudo) no terminal.

Passos de Instalação:

Instalação de Pré-Requisitos:

Execute os seguintes comandos no terminal para atualizar o sistema e instalar os pacotes necessários:

sudo apt-get update -y
sudo apt-get upgrade -y

# Instalar Apache
sudo apt-get install apache2

# Instalar PHP e dependências
sudo apt-get install php libapache2-mod-php php-gd php-common php-mail php-mail-mime php-mysql php-pear php-db php-mbstring php-xml php-curl

# Instalar MariaDB
sudo apt -y install mariadb-server mariadb-client

Configuração do Banco de Dados

Configurar o banco de dados para o FreeRADIUS:

# Configurar banco de dados para FreeRADIUS
sudo mysql_secure_installation

# Siga as instruções interativas para configurar a segurança do MySQL

# Criar banco de dados e usuário para o FreeRADIUS
sudo mysql -u root -p

CREATE DATABASE radius CHARACTER SET UTF8 COLLATE UTF8_BIN;

CREATE USER 'radius'@'%' IDENTIFIED BY 'radius';

GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'%';

QUIT;

Instalação do FreeRADIUS

Instalar e configurar o FreeRADIUS:

sudo apt policy freeradius

sudo apt -y install freeradius freeradius-mysql freeradius-utils

# Importar esquema do banco de dados MySQL do FreeRADIUS
sudo su -

sudo mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql

sudo mysql -u root -p -e "use radius;show tables;"

# Criar link virtual para o módulo sql

sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/

# Configurar arquivo sql no FreeRADIUS

sudo nano /etc/freeradius/3.0/mods-enabled/sql

# Faça as alterações correspondentes ao seu banco de dados e salve o arquivo

# Definir permissões adequadas

sudo chgrp -h freerad /etc/freeradius/3.0/mods-available/sql

sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql

# Reiniciar o FreeRADIUS

sudo systemctl restart freeradius

sudo systemctl status freeradius

Instalação do daloRADIUS

Instalar e configurar o daloRADIUS:

sudo apt -y install wget unzip

wget https://github.com/lirantal/daloradius/archive/refs/tags/1.3.zip

unzip 1.3.zip

mv daloradius-1.3 daloradius

cd daloradius

# Configurar o daloradius no MySQL

sudo mysql -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql 

sudo mysql -u root -p radius < contrib/db/mysql-daloradius.sql

cd ..

sudo mv daloradius /var/www/html/

# Configurar o daloradius.conf.php

sudo cp /var/www/html/daloradius/library/daloradius.conf.php.sample /var/www/html/daloradius/library/daloradius.conf.php

sudo nano /var/www/html/daloradius/library/daloradius.conf.php

# Faça as alterações correspondentes ao seu banco de dados e salve o arquivo

# Importar tabelas do daloRAIUS MySQL para o banco de dados FreeRADIUS

cd /var/www/html/daloradius/

sudo mysql -u root -p radius < /var/www/html/daloradius/contrib/db/fr2-mysql-daloradius-and-freeradius.sql

sudo mysql -u root -p radius < /var/www/html/daloradius/contrib/db/mysql-daloradius.sql

# Configurar permissões adequadas
sudo chown -R www-data:www-data /var/www/html/daloradius/

# Reiniciar serviços
sudo systemctl restart freeradius.service apache2

Acesso ao daloRADIUS

Após concluir todos os passos acima, você pode acessar o daloRADIUS através do seguinte URL no seu navegador:

http://localhost/daloradius/login.php

Usuário: administrator

Senha: radius

