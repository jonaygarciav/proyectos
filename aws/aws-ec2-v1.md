# AWS EC2 v1

## Descripción

Crear un instancia EC2 en AWS utilizando la capa gratuita que ofrece AWS para poder desplegar la aplicación web cuyo código fuente se encuentra en el repositorio de GitHub [https://github.com/jonaygarciav/app-canarias-experience-py](https://github.com/jonaygarciav/app-canarias-experience-py).

## Software

* __Servicios en AWS__: EC2, Security Group, Key Pair
* __Sistemas Operativos__: Ubuntu Server 24.04

## Contenido

### Parte I. Redes VPC, Subredes y zonas en AWS

Lecturas recomendadas:
* [¿Qué es Amazon VPC?](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/what-is-amazon-vpc.html)
* [Regiones y zonas en AWS](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
* [Concesión de acceso a Internet de la VPC con puertas de enlace de Internet](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/VPC_Internet_Gateway.html)

Existe una red __VPC__ ya creada en AWS en la región `us-east-1`:

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

### Parte II. Creación de Security Group en AWS

Lecturas recomendadas:
* [Controlar el tráfico hacia los recursos de AWS mediante grupos de seguridad](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/vpc-security-groups.html)
* [Crear un Grupo de Seguridad](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/creating-security-groups.html)

Para permitir el acceso por SSH a la instancia EC2, en la consola web de AWS crea un __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-sg` con las siguientes reglas:

Para el _tráfico de entrada_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| SSH   | TCP      | 22      | `0.0.0.0/0` | Acceso por SSH          |

Para el _tráfico de salida_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| ALL   | ALL      | ALL     | `0.0.0.0/0` | Acceso hacia fuera      |

* __Nota__: para crear un _Security Group_ debes hacerlo desde la configuración _VPC_ en AWS.

* __Nota__: sustituye '<github-user>' por tu nombre de usuario de _GitHub_.

### Parte III. Creación de una instancia EC2 en AWS

Lecturas recomendadas:
* [Introducción a Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Tipos de instancias de Amazon EC2](https://aws.amazon.com/es/ec2/instance-types/)
* [Par de claves e instancias de Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
* [Creación de un par de claves para la instancia de Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/create-key-pairs.html)

En la consola web de AWS crea una instancia EC2 llamada `<github-user>-ec2` utilizando la capa gratuita de AWS:

| Tipo       | vCPU | RAM | Disco | AMI                 |
|------------|------|-----|-------|---------------------|
| `t2.micro` | 1    | 1GB | 8GB   | Ubuntu Server 24.04 |

Aspectos a tener en cuenta:

* Crea la instancia EC2 a partir de la AMI basada en Ubuntu Server 24.04
* Asocia el Security Group creado anteriormente a la instancia EC2.
* Crea una clave SSH nueva (.pem) llamada `<github-user>-key` y descargarla, la cual usaremos posteriormente para conectarnos a la instancia EC2 mediante el protocolo SSH.
* Crea la instancia EC2 en la única red _VPC_ que existe y en la subred correspondiente a la zona de disponibilidad `us-east-1a`.
* Asígnale una IP pública a la instancia EC2.

> __Nota__: la instancia `t2.micro` está incluida en la **capa gratuita de AWS**.  

Una vez creada la instancia EC2 y descargada la clave `<github-user>-key.pem`, desde tu equipo conéctate a la instancia EC2 utilizando el protocolo SSH con el siguiente comando:

```bash
ssh -i <github-user>-key.pem ubuntu@<ip-publica-instancia-ec2>
```

> __Nota__: si al ejecutar el comando anterior obtienes un error de permisos, corrige los permisos del archivo _.pem_ con:

```bash
chmod 400 <github-user>-key.pem
```

### Parte IV. Configurar conexión desde instancia EC2 en AWS hacia GitHub mediante SSH Key

En la instancia EC2 genera un par de claves SSH:

```bash
ssh-keygen -t rsa
```

Este comando crea dos ficheros en la instancia EC2 con la clave SSH generada para el usuario `ubuntu`: `/home/ubuntu/.ssh/id_rsa` y `/home/ubuntu/.ssh/id_rsa.pub`.

En la consola de GitHub, en nuestro perfil, en el menú _Settings → SSH and GPG Keys_, configuramos una nueva clave SSH:
* __Title__: `ubuntu@<ip-publica-instancia-ec2>`
* __Key Type__: `Authentication Key`
* __Key__: contenido del fichero `/home/ubuntu/.ssh/id_rsa.pub` de la instancia EC2.

__Nota__: para ver el contenido del fichero `/home/ubuntu/.ssh/id_rsa.pub` de la instancia EC2 puedes usar el comando `cat`.

En la instancia EC2 comprueba la conexión a GitHub: 

```bash
ssh -T git@github.com

Hi <github-user>! You've successfully authenticated, but GitHub does not provide shell access.
```

### Parte V. Instalalación y configuración del repositorio en la instancia EC2, configuración servicio Systemd y configuración de Security Group de AWS

En tu equipo, desde un navegador web, accede a la web de GitHub y loguéate con tu usuario, accede al repositorio de GitHub `https://github.com/jonaygarciav/app-canarias-experience-py` y haz un `fork` del repositorio: ahora deberías tener un repositorio llamado `app-canarias-experience-py` en tu cuenta de GitHub y puedes acceder a él desde la URL `https://github.com/<github-user>/app-canarias-experience-py`

En la instancia EC2, clona el repositorio `https://github.com/<github-user>/app-canarias-experience-py`  en el directorio `/home/ubuntu/flask-app`:

```bash
git clone git@github.com:<github-user>/app-canarias-experience.git /home/ubuntu/flask-app
```

En la instancia EC2, instala el paquete `python3.12-venv` para crear entornos virtuales:

```bash
sudo apt update
sudo apt install python3.12-venv -y
```

En la instancia EC2, crea un entorno virtual e instala las dependencias de la aplicación:

```bash
cd /home/ubuntu/flask-app
python3 -m venv .venv

source .venv/bin/activate
pip3 install -r requirements.txt
```

En la instancia EC2 configura un servicio para poder parar y arrancar la aplicación con el comando `systemctl`, para ello, crea el archivo `/etc/systemd/system/flask-app.service` con el siguiente contenido:

```bash
[Unit]
Description=Flask App Service
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/flask-app
Environment="PATH=/home/ubuntu/flask-app/.venv/bin"
ExecStart=/home/ubuntu/flask-app/.venv/bin/python3 /home/ubuntu/flask-app/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

En la instancia EC2 activa el servicio y arranca la aplicación:

```bash
sudo systemctl daemon-reload

sudo systemctl enable flask-app
sudo systemctl start flask-app
```

En la instancia EC2 prueba que la aplicación se haya levantado correctamente:

```bash
curl http://localhost:5000
¡Hola, bienvenido a mi aplicación Flask!
```

Para poder acceder a la aplicación de la instancia EC2, en la consola web de AWS debes añadir al __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-sg` la siguiente regla:

Para el _tráfico de entrada_:

| Tipo           | Protocolo | Puerto | Origen      | Descripción             |
|----------------|-----------|--------|-------------|-------------------------|
| Personalizado  | TCP       | 5000   | `0.0.0.0/0` | Acceso a Flask App      |

Ahora, desde tu equipo, abre un navegador web e intenta acceder a la aplicación a través de la URL `http://<ip-publica-instancia-ec2>:5000` y comprueba que se muestra la aplicación.

### Parte VI. Realizar cambios en el repositorio y subirlos manualmente a la instancia EC2

Ahora desde tu equipo, clona el repositorio `https://github.com/<github-user>/app-canarias-experience-py` en local y ábrelo con Visual Studio Code:
* Haz cambios en la aplicación añadiendo en el fichero `templates/index.html` un nuevo `card` con su imagen correspondiente, la cual tendrás que almacenar dentro de la carpeta `static/img` ya creada.
* Crea un commit para guardar estos cambios en local y luego súbelos al repositorio remoto de GitHub.

En la instancia EC2, actualiza el repositorio trayéndonos el último commit del repositorio remoto:

```bash
cd /home/ubuntu/flask-app

git pull
```

Posteriormente, en la instancia EC2, reiniciamos el servicio:

```bash
sudo systemctl restart flask-app
```

Ahora, desde tu equipo, abre un navegador web e intenta acceder a la aplicación a través de la URL  `http://<ip-publica-instancia-ec2>:5000` y comprueba que se muestran los cambios.

### Parte VII. Automatizar cambios en la instancia EC2 mediante GitHub Actions

Lecturas recomendadas:
* [Documentación de GitHub Actions](https://docs.github.com/es/actions)
* [Entender GitHub Actions](https://docs.github.com/es/actions/about-github-actions/understanding-github-actions)

Para automatizar la actualización del repositorio en la instancia EC2 cada vez que subamos un commit al repositorio remoto en GitHub, configuraremos un __workflow__ en _GitHub Actions_.

En la consola web de GitHub crea secretos en el repositorio `https://github.com/<github-user>/app-canarias-experience-py` de GitHub:

En la consola de GitHub, accede al repositorio `https://github.com/<github-user>/app-canarias-experience-py`, en el menú del repositorio _Settings → Secrets and variables → Actions → Repository secrets_, crea  los siguientes _Repository Secrets_:

* __EC2_SSH_KEY__: con el contenido del fichero `<github-user>-key.pem`.
* __EC2_HOST__: la `IP pública de la instancia EC2`.
* __EC2_USER__: el usuario `ubuntu`.

Ahora en tu equipo, a través de Visual Studio Code crea el archivo `.github/workflows/deploy.yml` dentro de tu copia del repositorio en local con el siguiente contenido:

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
      uses: appleboy/ssh-action@v1.2.2
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          export GIT_SSH_COMMAND="ssh -o StrictHostKeyChecking=no"
          cd /home/ubuntu/flask-app
          echo "GIT PULL:"
          git pull
          sudo systemctl restart flask-app
          echo "ESTADO DEL SERVICIO flask-app:"
          sudo systemctl status flask-app --no-pager
```

> __Nota__: crea el directorio `.github` en la raíz del proyecto, luego el directorio `workflows` dentro del directorio `.github` y finalmente crear el archivo `deploy.yml` dentro del directorio `.github/workflows`.

Actualiza en tu equipo a través de Visual Studio Code tu copia del repositorio en local cambiándole el título de la página web de `Canarias Experience` a `Canarias 8 Experience`, crea un commit para guardar estos cambios en local y luego súbelos al repositorio remoto de GitHub.

Confirma que los cambios en el repositorio activan el despliegue automático en la instancia EC2, para ello revisar GitHub Actions en `https://github.com/<github-user>/app-canarias-experience-py/actions`.

Accede a la aplicación a través de la URL `http://<ip-publica-instancia-ec2>:5000` y comprueba que se muestran los cambios.

### Parte VIII. Borrar Instancia EC2, Security Group, Key Pair y Repositorio de GitHub

Elimina los siguientes elementos de AWS:

* Instancia EC2 `<github-user>-ec2`
* Security Group `<github-user>-sg`
* Key Pair `<github-user>-key`

Elimina los siguientes repositorios de GitHub:

* `https://github.com/<github-user>/app-canarias-experience-py`

Elimina las siguientes claves SSH de GitHub:

* En la cuenta personal, en el menú Settings → SSH and GPG Keys, eliminar la clave SSH `ubuntu@<ip-publica-instancia-ec2>`
