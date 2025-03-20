# AWS EC2 v1

## Descripción

Crear un instancia EC2 en AWS utilizando la capa gratuilta para poder desplegar la aplicación https://github.com/jonaygarciav/app-descubre-canarias-bs

## Software

* AWS EC2
* Ubuntu Server 24.04
* Nginx 

## Contenido

### Parte I

Para permitir acceso a las instancias, debes crear un __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-sg` con las siguientes reglas:

Para el tráfico de entrada:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| SSH   | TCP      | 22      | `0.0.0.0/0` | Acceso seguro por SSH   |
| HTTP  | TCP      | 80      | `0.0.0.0/0` | Permitir trafico web    |
| HTTPS | TCP      | 443     | `0.0.0.0/0` | Permitir trafico seguro |

Para el tráfico de salida:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| ALL   | ALL      | ALL     | `0.0.0.0/0` | Acceso hacia fuera      |

* __Notas__: para crear un _Security Group_ debes hacerlo desde la configuración VPC en AWS.

* __Notas__: sustituye '<github-user>' por tu nombre de usuario de _GitHub_.

### Parte II

Crear una instancia EC2 llamada `<github-user>-nginx-ec2` en AWS utilizando la capa gratuita de AWS:

| Servicio         | Tipo       | vCPU | RAM | Disco | AMI                 |
|------------------|------------|------|-----|-------|---------------------|
| **Nginx Server** | `t2.micro` | 1    | 1GB | 8GB   | Ubuntu Server 24.04 |

Aspectos a tener en cuenta:

* Crear la instancia EC2 a partir de la AMI basada en Ubuntu Server 24.04
* Se debe asociar el Security Group creado anteriormente a la instancia EC2.
* Crear una clave SSH nueva (.pem) llamada `<github-user>-nginx-key` y descargarla, la cual usaremos posteriormente para conectarnos a la instancia EC2 mediante el protocolo SSH.
* Crear la instancia en la única VPC que existe y la subred correspondiente a la zona us-east-1a.
* Asignarle una IP pública a la instancia.

🔹 **Nota:** La instancia `t2.micro` está incluida en la **capa gratuita de AWS**.  

Una vez creada la instancia EC2 y descargada la clave `<github-user>-nginx-key.pem`, nos conectamos a la instancia EC2 mediante SSH con el siguiente comando:

```bash
ssh -i <github-user>-nginx-key.pem" ubuntu@<ip-publica-instancia-ec2>
```

> __Nota__: si al ejecutar el comando anterior obtienes un error de permisos, corrige los permisos del archivo _.pem_ con:

```bash
chmod 400 <github-user>-nginx-key.pem
```

### Parte III

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
sudo git clone https://github.com/<github-user>/app-descrubre-canarias-bs /var/www/html
sudo chown -R ubuntu:www-data /var/www/html
```

Acceder a la aplicación a través de la URL `<ip-publica-instancia-ec2>` y comprobar que accedemos a la aplicación:

### Parte IV

Ahora en tu equipo, clona el repositorio `https://github.com/<github-user>/app-descrubre-canarias-bs` en local y ábrelo con Visual Studio Code: haz cambios en la aplicación añadiendo en el fichero `index.html` un nuevo `card` con un imagen correspondiente en la carpeta `img`, crea un commit para guardar estos cambios en local y luego sube los cambios al repositorio remoto de GitHub.

En la instancia EC2, actualizamos el repositorio trayéndonos el último commit del repositorio remoto:

```bash
cd /var/www/html
git pull
chown -R ubuntu:www-data /var/www/html
```

Acceder a la aplicación a través de la URL `http://<ip-publica-instancia-ec2>` y comprobar que se ven los cambios.

### Parte V

Actualización Automática con GitHub Actions. Para automatizar la actualización del repositorio en la instancia EC2 cada vez que subamos un commit al repositorio remoto, agregamos un __workflow__ de __GitHub Actions__.

Crear secretos en el repositorio `https://github.com/<github-user>/app-descrubre-canarias-bs` de GitHub:

Ir a Settings → Secrets and variables → Actions → Repository secrets:

* Crear un nuevo Repository Secret llamado `EC2_SSH_KEY` que contenga el contenido del fichero `<github-user>-nginx-key.pem`.
* Crear un nuevo Repository Secret llamado `EC2_HOST` con la `IP pública de la instancia EC2`.
* Crear un nuevo Repository Secret llamado `EC2_USER` con el usuario `ubunt`.

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

> __Notas__: antes de crear el archivo `deploy.yml` hay que crear el directorio `.github` y dentro de él el directorio  `workflows`.

Actualiza en tu equipo a través de Visual Studio Code el repositorio cambiándole el título de la página web de `Explora las Islas Canarias` a `Explora las 8 Islas Canarias`, crea un commit para guardar estos cambios en local y luego sube los cambios al repositorio remoto de GitHub.

Confirmar que los cambios en el repositorio activan el despliegue automático en la instancia EC2, para ello revisar GitHub Actions en `∫https://github.com/<github-user>/app-descubre-canarias-bs/actions`.

Acceder a la aplicación a través de la URL `<ip-publica-instancia-ec2>` y comprobar que se ven los cambios.

### Parte VI

Eliminar los siguientes elementos de AWS:

* Instancia EC2 `<github-user>-nginx-ec2`
* Security Group `<github-user>-nginx-sg`

Eliminar los siguientes repositorios de GitHub:

* `https://github.com/<github-user>/app-descrubre-canarias-bs`
