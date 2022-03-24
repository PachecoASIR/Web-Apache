# Web-Apache

Paso 0:

Instalaremos todos los repositorios necesarios para la práctica: httpd (apache), bind9 y kasmweb.

Paso 1:

Creamos nuestro docker-compose.yml y configuramos la network y creamos los volúmenes que vayamos a emplear:

docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 br02

docker volume create apache-data   

docker volume create apache-data2

docker volume create dns_web_apache

Ejecutamos el Docker-compose para que se generen los contenedores y configuramos nuestro DNS (bind9), para ello editaremos los documentos named.conf.local (añadimos la zona) y named.conf.options (añadimos los forwarders), a su vez, crearemos el documento db.mizona.com,

named.conf.local

//
// Do any local configuration here
//

zone "mizona.com" {
    type master;
    file "/etc/bind/db.mizona.com";
};

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

named.conf.options

options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	 forwarders {
	 	//8.8.8.8;
		//8.8.4.4;
	 };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	listen-on-v6 { any; };
};


db.mizona.com

;
; BIND data file for example.com
;
$TTL    604800
@       IN      SOA     mizona.com. root.mizona.com. (
                              5         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      pacheco
@       IN      A       5.5.5.5
@       IN      AAAA    ::1
pacheco      IN      A       10.1.0.3
www     IN      A 19.19.9.1
jeje    IN      A 18.18.8.2
pacheco2 IN TXT "Probamos si funciona"
pacheco2 IN A 2.2.2.2
@ IN TXT "texto de la zona"

Una vez configurados todos los ficheros procederemos a reiniciar todos los servicios y a comprobar que todo funciona correctamente.

HOSTS

Para crear un nuevo host virtual en primer lugar hemos de editar el archivo /etc/apache2/sites-available/000-default.conf y, para crear un nuevo host copiaremos dicho archivo en el directorio con un nombre de nuestra elección: 

sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/pacheco.conf

Una vez hecho esto solo faltaría configurar el nuevo sitio usando cualquiera de las directivas de nuestra preferencia. 
Por ejemplo, la siguiente configuración haría que nuestro sitio responda a cualquier solicitud de dominio que termine en .pacheco.com.
ServerAlias *.pacheco.com
Una vez tengamos todas nuestras directivas escritas, guardaremos el archivo y lo habilitaremos con el siguiente comando:
sudo a2ensite mynewsite
Una vez habilitado, hemos de reiniciar el servicio de Apache para aplicar los nuevos cambios:
sudo systemctl restart apache2.service

HTTPS

Para instalar el protocolo seguro HTTPS en nuestro Apache seguiremos los siguientes pasos:

1. sudo a2enmod ssl

2. sudo a2ensite default-ssl

3.  Reiniciaremos nuestro servidor Apache para habilitar la nueva configuración:
sudo systemctl restart apache2.service
