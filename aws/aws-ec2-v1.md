# AWS EC2 v1

## Descripción

Crear un instancia EC2 en AWS utilizando la capa gratuilta para poder desplegar la aplicación https://github.com/jonaygarciav/app-descubre-canarias-bs

## Software

    • AWS EC2
    • Ubuntu Server 24.04
    • InfluxDB
    • Grafana

## Contenido

### Parte I

Para permitir acceso a las instancias, debes configurar un **Security Group** con las siguientes reglas para el tráfico de entrada:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| SSH   | TCP      | 22      | `0.0.0.0/0` | Acceso seguro por SSH   |
| HTTP  | TCP      | 80      | `0.0.0.0/0` | Permitir tráfico web    |
| HTTPS | TCP      | 443     | `0.0.0.0/0` | Permitir tráfico seguro |

### Parte II

Crear instancia EC2 en AWS utilizando la capa gratuilta de AWS:

| Servicio         | Tipo (Capa Gratuita)     | vCPU | RAM | Disco | AMI                 |
|------------------|--------------------------|------|-----|-------|---------------------|
| **Nginx Server** | `t2.micro`               | 1    | 1GB | 8GB   | Ubuntu Server 24.04 |

Aspectos a tener en cuenta:

* Crear la instancia EC2 a partir de la AMI basada en Ubuntu Server 24.04
* Se debe asociar el Security Group creado anteriormente a la instancia EC2.
* Crear una clave SSH nueva (.pem) y descargarla, la cual usaremos para conectarnos al servidore por SSH posteriormente.
* Crear la instancia en la única VPC que existe y la subred la correspondiente a la zona us-east-1a.
* Asignarle una IP pública a la instancia.

🔹 **Nota:** La instancia `t2.micro` está incluida en la **capa gratuita de AWS**.  

Una vez creada la instancia y descargado el archivo **`mi-clave.pem`**, usa este comando para conectarte:

```sh
ssh -i "mi-clave.pem" ubuntu@<ip-publica-aws-ec2>
```

> __Nota__: Si obtienes un error de permisos, corrige los permisos del archivo .pem con:

```bash
chmod 400 mi-clave.pem
```

### Parte III

Instalar y configurar _Nginx_:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Hacer un fork del repositorio de GitHub a vuestro `https://github.com/jonaygarciav/app-descubre-canarias-bs`: ahora tenéis un repositorio llamado app-descubre-canarias-bs en vuestro repositorio de GitHub.

Clonar el repositorio `https://github.com/<vuestro-usuario>/app-descubre-canarias-bs` de la aplicación en el directorio `html` de _Nginx_:

```bash
sudo rm -Rf /var/www/html/*
sudo git clone https://github.com/jonaygarciav/app-descrubre-canarias-bs /var/www/html/
sudo chown -R www-data:www-data /var/www/html
```

Acceder a la aplicación a través de la URL http://<ip-publica> y comprobar que accedemos a la aplicación:

### Parte IV

Actualiza en local la aplicación añadiéndole 3 cards bootstrap más y subir los cambios al repositorio.

Actualizar el repositorio de la instancia EC2 y comprobar que se ven los cambios:

```bash
cd /var/www/html/
sudo git pull
sudo chown -R www-data:www-data /var/www/html
```

Acceder a la aplicación a través de la URL http://<ip-publica> y comprobar que se ven los cambios.
