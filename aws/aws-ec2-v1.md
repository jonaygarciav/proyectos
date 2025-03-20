# AWS EC2 v1

## Descripción

Crear un instancia EC2 en AWS utilizando la capa gratuita que ofrece AWS para poder desplegar la aplicación web cuyo código fuente se encuentra en el repositorio de GitHub [https://github.com/jonaygarciav/app-descubre-canarias-bs](https://github.com/jonaygarciav/app-descubre-canarias-bs).

## Software

* __Servicios en AWS__: EC2, Security Group, Key Pair
* __Sistemas Operativos__: Ubuntu Server 24.04
* __Servidores Web__: Nginx 

## Contenido

### Parte I. Redes VPC, Subredes y zonas

Lecturas recomendadas:
* [¿Qué es Amazon VPC?](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/what-is-amazon-vpc.html)
* [Regiones y zonas en AWS](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
* [Concesión de acceso a Internet de la VPC con puertas de enlace de Internet](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/VPC_Internet_Gateway.html)

Existe una red __VPC__ ya creada en la región `us-east-1`:

| VPC     | Identificador         | CIDR          |
|---------|-----------------------|---------------|
| default | vpc-028d7d7ffaa513c00 | 172.31.0.0/16 |

La región `us-east-1` cuenta con 6 __zonas de disponibilidad__: `us-east-1a`, `us-east-1b`, `us-east-1c`, `us-east-1d`, `us-east-1e` y `us-east-1f`. Esta red _VPC_ cuenta con 6 _subredes_, donde cada _subred_ se encuentra en una _zona de disponibilidad distinta_:

| Zona de disponibilidad | Indentificador de Subred | CIDR           |
|------------------------|--------------------------|----------------|
| us-east-1a             | subnet-0c13c7bae2a27de4f | 172.31.16.0/20 |
| us-east-1b             | subnet-0c13c7bae2a27de4f | 172.31.32.0/20 |
| us-east-1c             | subnet-0c13c7bae2a27de4f | 172.31.48.0/20 |
| us-east-1d             | subnet-0c13c7bae2a27de4f | 172.31.64.0/20 |
| us-east-1e             | subnet-0c13c7bae2a27de4f | 172.31.80.0/20 |
| us-east-1f             | subnet-0c13c7bae2a27de4f | 172.31.96.0/20 |

Todas y cada una de las subredes de la red VPC son públicas ya que están conectadas a un __Internet Gateway__ (_IG_), el cual es un componente de red de AWS que permite quen las instancias EC2 dentro de una red _VPC_ puedan comunicarse hacia o desde Internet. Para que reciban tráfico de Internet deben tener una __dirección pública__ asociada o una __Elastic IP__. Esto lo hacen mediante la configuración de una __tabla de rutas__:

| Destino       | Target         | Descripción                                       |
|---------------|----------------|---------------------------------------------------|
| 172.31.0.0/16 | local          | Comunicación interna dentro de la VPC             |
| 0.0.0.0/0     | igw-1234567890 | Ruta hacia Internet a través del Internet Gateway |

> __Notas__: los identificadores tanto de redes _VPC_ como de _subredes_, así como de _Internet Gateway_ pueden no ser los mismos.

### Parte II. Creación de Security Group

Lecturas recomendadas:
* [Controlar el tráfico hacia los recursos de AWS mediante grupos de seguridad](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/vpc-security-groups.html)
* [Crear un Grupo de Seguridad](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/creating-security-groups.html)

Para permitir el acceso por SSH a la instancia EC2, debes crear un __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-sg` con las siguientes reglas:

Para el _tráfico de entrada_:

| Tipo  | Protocolo | Puerto | Origen               | Descripción             |
|-------|----------|---------|----------------------|-------------------------|
| SSH   | TCP      | 22      | `<tu-ip-publica>/32` | Acceso por SSH          |

> __Nota__: sustituye `<tu-ip-publica>` por tu IP pública. Para consultarla la IP Pública puedes hacerlo desde la siguiente URL: `https://www.cualesmiip.com/`
Para el _tráfico de salida_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| ALL   | ALL      | ALL     | `0.0.0.0/0` | Acceso hacia fuera      |

* __Nota__: para crear un _Security Group_ debes hacerlo desde la configuración _VPC_ en AWS.

* __Nota__: sustituye '<github-user>' por tu nombre de usuario de _GitHub_.

### Parte III. Creación de una instancia EC2

Lecturas recomendadas:
* [Introducción a Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Tipos de instancias de Amazon EC2](https://aws.amazon.com/es/ec2/instance-types/)
* [Par de claves e instancias de Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
* [Creación de un par de claves para la instancia de Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/create-key-pairs.html)

Crear una instancia EC2 llamada `<github-user>-ec2` en AWS utilizando la capa gratuita de AWS:

| Servicio         | Tipo       | vCPU | RAM | Disco | AMI                 |
|------------------|------------|------|-----|-------|---------------------|
| **Nginx Server** | `t2.micro` | 1    | 1GB | 8GB   | Ubuntu Server 24.04 |

Aspectos a tener en cuenta:

* Crear la instancia EC2 a partir de la AMI basada en Ubuntu Server 24.04
* Se debe asociar el Security Group creado anteriormente a la instancia EC2.
* Crear una clave SSH nueva (.pem) llamada `<github-user>-key` y descargarla, la cual usaremos posteriormente para conectarnos a la instancia EC2 mediante el protocolo SSH.
* Crear la instancia EC2 en la única red _VPC_ que existe y en la subred correspondiente a la zona de disponibilidad `us-east-1a`.
* Asignarle una IP pública a la instancia EC2.

> __Nota__: la instancia `t2.micro` está incluida en la **capa gratuita de AWS**.  

Una vez creada la instancia EC2 y descargada la clave `<github-user>-key.pem`, nos conectamos a la instancia EC2 mediante SSH con el siguiente comando:

```bash
ssh -i <github-user>-key.pem" ubuntu@<ip-publica-instancia-ec2>
```

> __Nota__: si al ejecutar el comando anterior obtienes un error de permisos, corrige los permisos del archivo _.pem_ con:

```bash
chmod 400 <github-user>-key.pem
```

### Parte IV. Configurar conexión instancia EC2 con GitHub mediante SSH Key

En la instancia EC2 generar un par de claves SSH:

```bash
ssh-keygen -t rsa
```

Este comando crea dos ficheros en la instancia EC2 con la clave SSH generada para el usuario `ubuntu`: `/home/ubuntu/.ssh/id_rsa` y `/home/ubuntu/.ssh/id_rsa.pub`.

En la consola de GitHub, en nuestro perfil en el menú _Settings - SSH and GPG Keys_, configuramos una nueva clave SSH:
* __Title__: `ubuntu@<ip-publica-instancia-ec2>`
* __Key Type__: `Authentication Key`
* __Key__: contenido del fichero `/home/ubuntu/.ssh/id_rsa.pub` de la instancia EC2.

__Nota__: para ver el contenido del fichero `/home/ubuntu/.ssh/id_rsa.pub` puedes usar el comando `cat`.

Ahora comprobamos la conexión desde la instancia EC2 con el usuario `ubuntu` a GitHub: 

```bash
ssh -T git@github.com

Hi <github-user>! You've successfully authenticated, but GitHub does not provide shell access.
```

### Parte V. Instalación de servidor web Nginx

Instalar y configurar _Nginx_ en la instancia EC2:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

En GitHub, hacer un `fork` del repositorio de GitHub `https://github.com/jonaygarciav/app-descubre-canarias-bs` a vuestra cuenta: ahora tenéis un repositorio llamado `app-descubre-canarias-bs` en vuestro repositorio de GitHub.

En la instancia EC2, clonar el repositorio `https://github.com/<github-user>/app-descubre-canarias-bs` de la aplicación en el directorio `html` de _Nginx_:

```bash
sudo rm -Rf /var/www/html
sudo git clone git@github.com:<github-user>/app-descubre-canarias-bs.git
sudo chown -R ubuntu:www-data /var/www/html
```

Para permitir acceso al servidor Nginx de la instancia EC2, debes añadir al __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-sg` la siguiente regla:

Para el _tráfico de entrada_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| HTTP  | TCP      | 80      | `0.0.0.0/0` | Acceso por HTTP         |
| HTTPS | TCP      | 443     | `0.0.0.0/0` | Acceso por HTTPs        |

Intenta acceder a la aplicación a través de la URL `<ip-publica-instancia-ec2>` y comprobar se ve la aplicación.

### Parte VI. Realizar cambios en el repositorio y subirlos manualmente a la instancia EC2

Ahora en tu equipo, clona el repositorio `https://github.com/<github-user>/app-descrubre-canarias-bs` en local y ábrelo con Visual Studio Code:
* Haz cambios en la aplicación añadiendo en el fichero `index.html` un nuevo `card` con su imagen correspondiente, la cual tendrás que almacenar dentro de la carpeta `img` ya creada.
* Crea un commit para guardar estos cambios en local y luego súbelos al repositorio remoto de GitHub.

En la instancia EC2, actualizamos el repositorio trayéndonos el último commit del repositorio remoto:

```bash
cd /var/www/html
git pull
chown -R ubuntu:www-data /var/www/html
```

Acceder a la aplicación a través de la URL `http://<ip-publica-instancia-ec2>` y comprobar que se ven los cambios.

### Parte VII. Automatizar cambios en la instancia EC2 mediante GitHub Actions

Lecturas recomendadas:
* [Documentación de GitHub Actions](https://docs.github.com/es/actions)
* [Entender GitHub Actions](https://docs.github.com/es/actions/about-github-actions/understanding-github-actions)

Actualización Automática con GitHub Actions. Para automatizar la actualización del repositorio en la instancia EC2 cada vez que subamos un commit al repositorio remoto, agregamos un __workflow__ de __GitHub Actions__.

Crear secretos en el repositorio `https://github.com/<github-user>/app-descrubre-canarias-bs` de GitHub:

Ir a Settings → Secrets and variables → Actions → Repository secrets:

* Crear un nuevo Repository Secret llamado `EC2_SSH_KEY` que contenga el contenido del fichero `<github-user>-key.pem`.
* Crear un nuevo Repository Secret llamado `EC2_HOST` con la `IP pública de la instancia EC2`.
* Crear un nuevo Repository Secret llamado `EC2_USER` con el usuario `ubuntu`.

Ahora en tu equipo a través de Visual Studio Code crea el archivo `.github/workflows/deploy.yml` dentro del repositorio local con el siguiente contenido:

```
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Deploy to EC2
      uses: appleboy/ssh-action@v0.1.10
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          cd /var/www/html
          git pull
          chown -R ubuntu:www-data /var/www/html
```

> __Nota__: hay que crear el directorio `.github` en la raíz del proyecto, luego el directorio `workflows` dentro del directorio `.github` y finalmente crear el archivo `deploy.yml` dentro del directorio `.github/workflows`.

Actualiza en tu equipo a través de Visual Studio Code el repositorio cambiándole el título de la página web de `Explora las Islas Canarias` a `Explora las 8 Islas Canarias`, crea un commit para guardar estos cambios en local y luego súbelos al repositorio remoto de GitHub.

Confirmar que los cambios en el repositorio activan el despliegue automático en la instancia EC2, para ello revisar GitHub Actions en `https://github.com/<github-user>/app-descubre-canarias-bs/actions`.

Acceder a la aplicación a través de la URL `<ip-publica-instancia-ec2>` y comprobar que se ven los cambios.

### Parte VIII. Borrar Instancia EC2, Security Group, Key Pair y Repositorio de GitHub

Eliminar los siguientes elementos de AWS:

* Instancia EC2 `<github-user>-ec2`
* Security Group `<github-user>-sg`
* Key Pair `<github-user>-key`

Eliminar los siguientes repositorios de GitHub:

* `https://github.com/<github-user>/app-descrubre-canarias-bs`
