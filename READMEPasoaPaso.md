# Proyecto-Final-Teleco3-Nginx

- primero que todo creamos una carpeta y dentro de CMD ingresamos el siguiente comando dentro de la ruta que se desee:

vagrant init


- El anterior comando nos generara un archivo llamado Vagranfile, dentro de este archivo creamos las maquinas que vayamos a utilizar, les mostraremos un ejemplo con todas las maquinas que en nuestro caso creamos para la realizacion de este proyecto:


Vagrant.configure("2") do |config|

 config.vm.define :cliente do |cliente|
    cliente.vm.box = "generic/centos8"
    cliente.vm.network :private_network, ip: "192.168.50.2"
    cliente.vm.hostname = "cliente"
  end


 config.vm.define :servidor do |servidor|
 servidor.vm.box = "centos/stream8"
 servidor.vm.network :private_network, ip: "192.168.50.3"
 servidor.vm.network :forwarded_port, guest: 80, host:5567
 servidor.vm.network :forwarded_port, guest: 443, host:5568
 servidor.vm.hostname = "servidor"
end



config.vm.define :servidor2 do |servidor2|
 servidor2.vm.box = "centos/stream8"
 servidor2.vm.network :private_network, ip: "192.168.50.4"
 servidor2.vm.hostname = "servidor2"
 end

config.vm.define :balanceador do |balanceador|
 balanceador.vm.box = "centos/stream8"
 balanceador.vm.network :private_network, ip: "192.168.50.5"
 balanceador.vm.hostname = "balanceador"
 end

config.vm.define :servidor3 do |servidor3|
 servidor3.vm.box = "centos/stream8"
 servidor3.vm.network :private_network, ip: "192.168.50.6"
 servidor3.vm.hostname = "servidor3"
 end

end


- Abre la terminal en tu sistema CentOS.



- Necesitaremos crear las paginas web con las que realizamos el balanceo de carga, por ello en cada servidor creado hacemos los siguientes pasos:

	vagrant ssh servidor

	sudo -i

	yum update

	yum install vim

	yum install httpd

 	service httpd start


- Luego de instalar lo requerido para httpd vamos a ubicarnos en:

	cd /var/www/html

- y creamos un archivo .html, por ejemplo:

	touch index.html

- Dentro de este archivo agregaremos nuestro codigo para la pagina web, en nuestro caso realizamos una pagina web que genera numeros aleatorios:

	<!DOCTYPE html>
<html>
<head>
    <title>Generador de Números Automático</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        h1 {
            color: #333;
        }
        #number {
            font-size: 48px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>Generador de Números Automático</h1>
    <p id="number"></p>

    <script>
        function generateNumber() {
            var randomNumber = Math.floor(Math.random() * 100) + 1;
            document.getElementById("number").innerHTML = "El número generado es: " + randomNumber;
        }

        // Generar número automáticamente cada 2 segundos
        setInterval(generateNumber, 2000);
    </script>
<p><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b8/Logo_UAO.svg/1200px-Logo_UAO.svg.png" width="500" height="485" alt="" /></p>

</body>
</html>





- Luego para instalar Nginx en CentOS, puedes seguir estos pasos:



- Instala Nginx utilizando el siguiente comando:

	sudo yum install nginx -y


- Una vez finalizada la instalación, puedes iniciar el servicio de Nginx utilizando el siguiente comando:


	sudo systemctl start nginx

- Para asegurarte de que Nginx se inicie automáticamente al arrancar el sistema, ejecuta el siguiente comando:


	sudo systemctl enable nginx

- Para verificar si Nginx se instaló correctamente, abre un navegador web e ingresa la dirección IP de tu servidor CentOS. Si ves la página de bienvenida de Nginx, significa que la instalación fue exitosa.

- ingresar a la ruta:

	vim /etc/nginx/nginx.conf

- por defecto nuestro archivo se va a ver asi:

server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }


- al realizar unas modificaciones nuestro archivo va a quedar:


user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {


    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
         root         /usr/share/nginx/html;


         include /etc/nginx/default.d/*.conf;

        location / {


        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }


}




- ingresar a la ruta:

	cd /etc/nginx/conf.d/


- crear un archivo llamado loadbalancer.conf:

	touch loadbalancer.conf

- luego agregar en el archivo lo siguiente utilizando vim loadbalancer.conf:


upstream backend {
        server 192.168.50.3;
        server 192.168.50.4;
        server 192.168.50.6;
    }

    server {
        listen      80 default_server;
        listen      [::]:80 default_server;
        server_name frontal.bitsandlinux.com;

        location / {
            proxy_redirect      off;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    Host $http_host;
        proxy_pass http://backend;
    }
}


- Finalmente le damos un:

	sudo systemctl restart nginx

	setenforce 0

- para reinciar el servicio guardando todos los cambios realizados anteriormente y ya podriamos ingresar la ip de nuestro balanceador de carga en un buscador como por ejemplo google chrome y al recargar la pagina realizariamos un redireccionamiento.

- Ahora vamos a explicar como utilizar Artillery para saturar el balanceador y ver como reaccionan los servidores. Primero en el servidor creador llamado cliente vamos a instalar vim para poder editar archivos:

	yum install vim


- Como en esta ocasión en el cliente estamos utilizando el centos8/generic, tendremos que apagar el firewalld utilizando el siguiente comando:

	service firewalld stop

- Instala Node.js Artillery requiere Node.js para ejecutarse. Puedes instalar Node.js en CentOS ejecutando los siguientes comandos:

	curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -

	sudo yum install -y nodejs


- Ya tenemos todo lo necesario para instalar el artillery, entonces usamos:

	sudo npm install -g artillery

- Verificamos que haya sido instalado correctamente:

	artillery --version


- Luego creamos un archivo YAML dentro de nuestro cliente y lo editamos:

	touch test.yaml

	vim test.yaml

- y le agregamos a nuestro archivo las configuraciones deseadas para el funcionamiento de artillery con los respectivos IP's o paginas http, el siguiente codigo es un ejemplo:



	config:
  target: "http://192.168.50.5"
  phases:
    - duration: 60
      arrivalRate: 10
scenarios:
  - name: "Mi escenario de prueba"
    flow:
      - get:
          url: "/"



- Finalmente utilizamos el siguiente comando para el funcionamiento del Artillery:

	artillery run test.yaml

