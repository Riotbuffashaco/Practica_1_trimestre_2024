## 1. Instalación del Servidor Web Apache

```
sudo apt update
sudo apt install apache2
```

![image](https://github.com/user-attachments/assets/634ca0f1-ff4a-4719-b872-2982f243cda1)

-----
## 2. Configurar el Archivo /etc/hosts

Edita el archivo /etc/hosts para que incluya los dominios locales:
```
sudo nano /etc/hosts
```
Añade las siguientes líneas al final del archivo:
```
127.0.0.1   centro.intranet
127.0.0.1   departamentos.centro.intranet
127.0.0.1   servidor2.centro.intranet
```
![image](https://github.com/user-attachments/assets/27ca5f1c-6001-4f3e-aec2-a6ed2a70e048)

-----
## 3. Activar Módulos Necesarios para PHP y MySQL

Instala PHP, MySQL y los módulos de Apache necesarios para ejecutar PHP y aplicaciones Python con WSGI.
```
sudo apt install php libapache2-mod-php php-mysql


sudo a2enmod php7.4
```
![image](https://github.com/user-attachments/assets/3948deb9-0fdf-4b52-8323-e95be5060b65)

A continuación, activa el módulo wsgi para aplicaciones Python:
```
sudo apt install libapache2-mod-wsgi-py3
sudo a2enmod wsgi
```

![image](https://github.com/user-attachments/assets/6b906be4-952a-44fe-92e8-aeb71d4fa19b)

-----
## 4. Instalar y Configurar WordPress

Primero, instala las dependencias necesarias:
```
sudo apt install mysql-server
sudo mysql_secure_installation
```

![image](https://github.com/user-attachments/assets/302b2421-4e05-4a93-9d8f-d6e98f5115e5)

Luego crea una base de datos y un usuario para WordPress en MySQL:
```
sudo mysql -u root -p
```
```
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

![image](https://github.com/user-attachments/assets/e9510b84-a020-430c-ae76-43cc6f0ba992)

Descarga WordPress y muévelo al directorio de Apache:
```
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
```
![image](https://github.com/user-attachments/assets/397ff608-39c7-4402-9968-1f79add464c3)
```
sudo mv wordpress /var/www/centro.intranet
sudo chown -R www-data:www-data /var/www/centro.intranet
```

![image](https://github.com/user-attachments/assets/5f969809-b0ac-49c4-81eb-6b441d05a6dc)

Crea el archivo de configuración para el sitio:
```
sudo nano /etc/apache2/sites-available/centro.intranet.conf
```
Configura el archivo para que apunte al directorio de WordPress:
```
<VirtualHost *:80>
	ServerName centro.intranet
	DocumentRoot /var/www/centro.intranet
	<Directory /var/www/centro.intranet>
    	AllowOverride All
	</Directory>
</VirtualHost>
```

![image](https://github.com/user-attachments/assets/ee586db4-eed8-44c7-b2dc-c6b18b9a1aab)

Activa el sitio y recarga Apache:
```
sudo a2ensite centro.intranet.conf
sudo systemctl reload apache2
```
![image](https://github.com/user-attachments/assets/9e6b4603-4112-418c-8b6f-872b89796e7b)

-----
## 5. Despliegue de una Aplicación Python bajo departamentos.centro.intranet

Crea el archivo de configuración para el sitio:
```
sudo nano /etc/apache2/sites-available/departamentos.centro.intranet.conf
```
Configura el archivo para utilizar WSGI:
```
<VirtualHost *:80>
	ServerName departamentos.centro.intranet
	WSGIScriptAlias / /var/www/departamentos.centro.intranet/app.wsgi
	<Directory /var/www/departamentos.centro.intranet>
    	Require all granted
	</Directory>
</VirtualHost>
```
![image](https://github.com/user-attachments/assets/3fd740a7-f349-42b2-ac86-991921d24dd6)

Crea el directorio para la aplicación:
```
sudo mkdir /var/www/departamentos.centro.intranet
```
Dentro del directorio de la aplicación, crea un archivo app.wsgi y un simple script en Python:
```
sudo nano /var/www/departamentos.centro.intranet/app.wsgi
```
Contenido del archivo app.wsgi:
```
def application(environ, start_response):
	status = '200 OK'
	output = b'Hello, Python Application Works!'
	response_headers = [('Content-type', 'text/plain'),
                    	('Content-Length', str(len(output)))]
	start_response(status, response_headers)
	return [output]
```
![image](https://github.com/user-attachments/assets/3e1e1502-0120-4b54-bc8f-e0a9b1790114)

Otorga los permisos adecuados:
```
sudo chown -R www-data:www-data /var/www/departamentos.centro.intranet
```
![image](https://github.com/user-attachments/assets/9f6b83a0-b590-4744-89a2-9c24abdbbee4)

-----
## 6. Protección del Acceso a la Aplicación con Autenticación

Crea un archivo .htpasswd para almacenar las credenciales:
```
sudo apt install apache2-utils
sudo htpasswd -c /etc/apache2/.htpasswd usuario
```
![image](https://github.com/user-attachments/assets/179331d9-5258-4c03-a28e-64b2cd1b0faf)

Edita el archivo de configuración para aplicar la autenticación:
```
<Directory /var/www/departamentos.centro.intranet>
	AuthType Basic
	AuthName "Restricted Content"
	AuthUserFile /etc/apache2/.htpasswd
	Require valid-user
</Directory>
```
![image](https://github.com/user-attachments/assets/796eea9c-ab51-4753-9ecb-ad642c08161f)

Recarga Apache para aplicar los cambios:
```
sudo systemctl reload apache2
```
----
## 7. Instalación y Configuración de Awstats

Instala awstats:
```
sudo apt install awstats
```
![image](https://github.com/user-attachments/assets/4f1f4a8f-0aad-4dce-b014-9cdaf95f843f)

Configura awstats:
```
sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.centro.intranet.conf
sudo nano /etc/awstats/awstats.centro.intranet.conf
```
Modifica el archivo de configuración con el nombre de dominio adecuado:
```
SiteDomain="centro.intranet"
HostAliases="localhost 127.0.0.1 centro.intranet"
```
Actualiza las estadísticas manualmente:
```
sudo /usr/lib/cgi-bin/awstats.pl -config=centro.intranet -update
```
![image](https://github.com/user-attachments/assets/6abfbdad-ee4c-41c1-9507-7aaa51b96931)

Para acceder a las estadísticas en el navegador, configura un alias en Apache:
```
Alias /awstats /usr/lib/cgi-bin
<Directory /usr/lib/cgi-bin>
	Options ExecCGI
	AllowOverride None
	Require all granted
</Directory>
```
![image](https://github.com/user-attachments/assets/90bbcd20-4ee2-48e0-96b3-798f7fde5f5b)

Recarga Apache:
```
sudo systemctl reload apache2
```
----
## 8. Instalación y Configuración de un Segundo Servidor (Nginx) en el Puerto 8080

Instala Nginx y PHP:
```
sudo apt install nginx php-fpm
```
![image](https://github.com/user-attachments/assets/5463027b-8da8-4a84-8a32-f920ea6840b4)

Configura Nginx para que sirva en el puerto 8080 bajo servidor2.centro.intranet:
```
sudo nano /etc/nginx/sites-available/servidor2.centro.intranet
```
Añade la configuración para el sitio:
```
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
```
![image](https://github.com/user-attachments/assets/ed10d6e4-b7eb-4322-aab0-1341beded43d)

Activa el sitio y recarga Nginx:
```
sudo ln -s /etc/nginx/sites-available/servidor2.centro.intranet /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```
-----

## 9. Instalación de phpMyAdmin

sudo apt install phpmyadmin

![image](https://github.com/user-attachments/assets/8cd1574b-c974-41a0-a537-72812cf19c43)

En Nginx, añade una localización para phpMyAdmin:

location /phpmyadmin {
	root /usr/share;
	index index.php index.html index.htm;
}

![image](https://github.com/user-attachments/assets/3f3993ae-a230-406b-a395-1b1b20a3ffad)

Por último, recarga Nginx:

sudo systemctl reload nginx
