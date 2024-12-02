1. Instalación del Servidor Web Apache

sudo apt update
sudo apt install apache2

2. Configurar el Archivo /etc/hosts

Edita el archivo /etc/hosts para que incluya los dominios locales:

sudo nano /etc/hosts

Añade las siguientes líneas al final del archivo:

127.0.0.1   centro.intranet
127.0.0.1   departamentos.centro.intranet
127.0.0.1   servidor2.centro.intranet

3. Activar Módulos Necesarios para PHP y MySQL

Instala PHP, MySQL y los módulos de Apache necesarios para ejecutar PHP y aplicaciones Python con WSGI.

sudo apt install php libapache2-mod-php php-mysql
sudo a2enmod php7.4

A continuación, activa el módulo wsgi para aplicaciones Python:

sudo apt install libapache2-mod-wsgi-py3
sudo a2enmod wsgi

4. Instalar y Configurar WordPress

Primero, instala las dependencias necesarias:

sudo apt install mysql-server
sudo mysql_secure_installation








Luego crea una base de datos y un usuario para WordPress en MySQL:

sudo mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

Descarga WordPress y muévelo al directorio de Apache:

wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
sudo mv wordpress /var/www/centro.intranet
sudo chown -R www-data:www-data /var/www/centro.intranet

Crea el archivo de configuración para el sitio:

sudo nano /etc/apache2/sites-available/centro.intranet.conf

Configura el archivo para que apunte al directorio de WordPress:

<VirtualHost *:80>
	ServerName centro.intranet
	DocumentRoot /var/www/centro.intranet
	<Directory /var/www/centro.intranet>
    	AllowOverride All
	</Directory>
</VirtualHost>

Activa el sitio y recarga Apache:

sudo a2ensite centro.intranet.conf
sudo systemctl reload apache2

5. Despliegue de una Aplicación Python bajo departamentos.centro.intranet

Crea el archivo de configuración para el sitio:

sudo nano /etc/apache2/sites-available/departamentos.centro.intranet.conf

Configura el archivo para utilizar WSGI:

<VirtualHost *:80>
	ServerName departamentos.centro.intranet
	WSGIScriptAlias / /var/www/departamentos.centro.intranet/app.wsgi
	<Directory /var/www/departamentos.centro.intranet>
    	Require all granted
	</Directory>
</VirtualHost>

Crea el directorio para la aplicación:

sudo mkdir /var/www/departamentos.centro.intranet

Dentro del directorio de la aplicación, crea un archivo app.wsgi y un simple script en Python:

sudo nano /var/www/departamentos.centro.intranet/app.wsgi

Contenido del archivo app.wsgi:

def application(environ, start_response):
	status = '200 OK'
	output = b'Hello, Python Application Works!'

	response_headers = [('Content-type', 'text/plain'),
                    	('Content-Length', str(len(output)))]
	start_response(status, response_headers)

	return [output]

Otorga los permisos adecuados:

sudo chown -R www-data:www-data /var/www/departamentos.centro.intranet

6. Protección del Acceso a la Aplicación con Autenticación

Crea un archivo .htpasswd para almacenar las credenciales:

sudo apt install apache2-utils
sudo htpasswd -c /etc/apache2/.htpasswd usuario

Edita el archivo de configuración para aplicar la autenticación:

<Directory /var/www/departamentos.centro.intranet>
	AuthType Basic
	AuthName "Restricted Content"
	AuthUserFile /etc/apache2/.htpasswd
	Require valid-user
</Directory>

Recarga Apache para aplicar los cambios:

sudo systemctl reload apache2

7. Instalación y Configuración de Awstats

Instala awstats:

sudo apt install awstats

Configura awstats:

sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.centro.intranet.conf
sudo nano /etc/awstats/awstats.centro.intranet.conf

Modifica el archivo de configuración con el nombre de dominio adecuado:

SiteDomain="centro.intranet"
HostAliases="localhost 127.0.0.1 centro.intranet"

Actualiza las estadísticas manualmente:

sudo /usr/lib/cgi-bin/awstats.pl -config=centro.intranet -update

Para acceder a las estadísticas en el navegador, configura un alias en Apache:

Alias /awstats /usr/lib/cgi-bin
<Directory /usr/lib/cgi-bin>
	Options ExecCGI
	AllowOverride None
	Require all granted
</Directory>

Recarga Apache:

sudo systemctl reload apache2

8. Instalación y Configuración de un Segundo Servidor (Nginx) en el Puerto 8080

Instala Nginx y PHP:

sudo apt install nginx php-fpm

Configura Nginx para que sirva en el puerto 8080 bajo servidor2.centro.intranet:

sudo nano /etc/nginx/sites-available/servidor2.centro.intranet

Añade la configuración para el sitio:

server {
	listen 8080;
	server_name servidor2.centro.intranet;

	root /var/www/servidor2.centro.intranet;
	index index.php index.html;

	location / {
    	try_files $uri $uri/ =404;
	}

	location ~ \.php$ {
    	include snippets/fastcgi-php.conf;
    	fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
	}
}

Activa el sitio y recarga Nginx:

sudo ln -s /etc/nginx/sites-available/servidor2.centro.intranet /etc/nginx/sites-enabled/
sudo systemctl reload nginx

9. Instalación de phpMyAdmin

sudo apt install phpmyadmin

En Nginx, añade una localización para phpMyAdmin:

location /phpmyadmin {
	root /usr/share;
	index index.php index.html index.htm;
}

Por último, recarga Nginx:

sudo systemctl reload nginx
