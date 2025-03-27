# AWS EC2 RDS v1

* _Paso I_: Redes VPC, Subredes y zonas en AWS
* _Paso II_: Creación de Par de claves en AWS
* _Paso III_: Creación de Security Group para instancia EC2 en AWS
* _Paso IV_: Creación de una instancia EC2 en AWS
* _Paso V_: Creación de Security Group para instancia RDS en AWS
* _Paso VI_: Creación de una instancia RDS de MySQL en AWS
* _Paso VII_: Configurar conexión desde instancia EC2 en AWS hacia GitHub mediante SSH Key
* _Paso VIII_: Instalalación y configuración del repositorio en la instancia EC2, configuración servicio Systemd y configuración de Security Group de AWS
* _Paso IX_: Realizar cambios en el repositorio y subirlos manualmente a la instancia EC2
* _Paso X_: Automatizar cambios en el repositorio de la instancia EC2 cada vez que subimos un commit al repositorio remoto mediante GitHub Actions
* _Paso XI_: Borrar Instancia EC2, Security Group, Key Pair y Repositorio de GitHub

## Descripción

Crear un instancia EC2 y una instancia RDS de MySQL en AWS utilizando la capa gratuita que ofrece AWS para poder desplegar la aplicación web cuyo código fuente se encuentra en el repositorio de GitHub [https://github.com/jonaygarciav/app-contactos-py](https://github.com/jonaygarciav/app-contactos-py).

## Software

* __Servicios en AWS__: EC2, RDS, Security Group, Key Pair
* __Sistemas Operativos__: Ubuntu Server 24.04
* __Servicios__: MySQL 8

## Contenido

### Paso I. Redes VPC, Subredes y zonas en AWS

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

### Paso II. Creación de Par de claves en AWS

Lecturas recomendadas:
* [Par de claves e instancias de Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
* [Creación de un par de claves para la instancia de Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/create-key-pairs.html)

![][01]

Crea una clave SSH nueva (.pem) llamada `<github-user>-key` y descargarla en tu equipo, la cual usaremos posteriormente para conectarnos a la instancia EC2 mediante el protocolo SSH.

### Paso III. Creación de Security Group para instancia EC2 en AWS

Lecturas recomendadas:
* [Controlar el tráfico hacia los recursos de AWS mediante grupos de seguridad](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/vpc-security-groups.html)
* [Crear un Grupo de Seguridad](https://docs.aws.amazon.com/es_es/vpc/latest/userguide/creating-security-groups.html)

![][02]

Para permitir el acceso por SSH a la instancia EC2, en la consola web de AWS crea un __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-ec2-sg` con las siguientes reglas:

Para el _tráfico de entrada_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| SSH   | TCP      | 22      | `0.0.0.0/0` | Acceso por SSH          |

Para el _tráfico de salida_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| ALL   | ALL      | ALL     | `0.0.0.0/0` | Acceso hacia fuera      |

* __Nota__: para crear un _Security Group_ debes hacerlo desde la configuración _VPC_ en AWS.

* __Nota__: sustituye `<github-user>` por tu nombre de usuario de _GitHub_.

### Paso IV. Creación de una instancia EC2 en AWS

Lecturas recomendadas:
* [Introducción a Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Tipos de instancias de Amazon EC2](https://aws.amazon.com/es/ec2/instance-types/)

![][03]

En la consola web de AWS crea una instancia EC2 llamada `<github-user>-ec2` utilizando la capa gratuita de AWS:

| Tipo       | vCPU | RAM | Disco | AMI                 |
|------------|------|-----|-------|---------------------|
| `t2.micro` | 1    | 1GB | 8GB   | Ubuntu Server 24.04 |

Aspectos a tener en cuenta:

* Crea la instancia EC2 a partir de la AMI basada en Ubuntu Server 24.04
* Asocia el _Par de claves_ `<github-user>-key` creado anteriormente a la instancia EC2.
* Asocia el _Security Group_ `<github-user>-ec2-sg` creado anteriormente a la instancia EC2.
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

### Paso V. Creación de Security Group para instancia RDS en AWS

Lecturas recomendadas:
* [¿Qué es Amazon Relational Database Service (Amazon RDS)?](https://docs.aws.amazon.com/es_es/AmazonRDS/latest/UserGuide/Welcome.html)

![][02]

Para permitir el acceso por SSH desde la instancia RDS de MySQL a la instancia EC2, en la consola web de AWS crea un __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-mysql-sg` con las siguientes reglas:

Para el _tráfico de entrada_:

| Tipo           | Protocolo | Puerto | Origen                         | Descripción    |
|----------------|-----------|--------|--------------------------------|----------------|
| Personalizado  | TCP       | 3306   | `<ip-private-instancia-ec2/32` | Acceso a MySQL |

Para el _tráfico de salida_:

| Tipo  | Protocolo | Puerto | Origen      | Descripción             |
|-------|----------|---------|-------------|-------------------------|
| ALL   | ALL      | ALL     | `0.0.0.0/0` | Acceso hacia fuera      |

### Paso VI. Creación de una instancia RDS de MySQL en AWS

Lecturas recomendadas:
* [Introducción a Amazon EC2](https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Tipos de instancias de Amazon EC2](https://aws.amazon.com/es/ec2/instance-types/)

![][04]

En la consola web de AWS crea una instancia RDS de MySQL llamada `<github-user>-mysql` utilizando la capa gratuita de AWS:

| Tipo           | vCPU | RAM | Disco |
|----------------|------|-----|-------|
| `db.t4g.micro` | 2    | 1GB | 8GB   |

Aspectos a tener en cuenta:

* Elige la plantilla `capa gratuita`
* Master username: `contactos_user`
* Password: elige un password para la base de datos y apúntalo
* Configura la instancia a `db.t4g.micro`
* Configura la instancia RDS en la única red _VPC_ que existe y en la subred correspondiente a la zona de disponibilidad `us-east-1a`
* No le asignes una IP pública a la instancia EC2
* Asígnale el Security Group creado en el paso anterior

> __Nota__: la instancia `db.t4g.micro` está incluida en la **capa gratuita de AWS**.  

Una vez creada la instancia RDS, probaremos la conexión desde la instancia EC2 a la instancia RDS, para ello, conéctate a la instancia EC2 utilizando el protocolo SSH con el siguiente comando:

```bash
ssh -i <github-user>-key.pem ubuntu@<ip-publica-instancia-ec2>
```

En la instancia EC2 instala el cliente MySQL:

```bash
sudo apt update
sudo apt install mysql-client
```

En la instancia EC2 conéctate a la instancia RDS:

```bash
$ mysql -h <github-user>-mysql.c18o4wyiy79q.us-east-1.rds.amazonaws.com -u contactos_user -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 27
Server version: 8.0.40 Source distribution

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

> __Nota__: para conectarte a la instancia RDS desde la instancia EC2 necesitarás el `endpoint` de la instancia RDS, el cual tiene la siguiente forma: `<github-user>-mysql.c18o4wyiy79q.us-east-1.rds.amazonaws.com` (¡OJO! No tiene por que ser exactamente igual)

### Paso VII. Configurar conexión desde instancia EC2 en AWS hacia GitHub mediante SSH Key

En la instancia EC2 genera un par de claves SSH:

```bash
ssh-keygen -t rsa
```

Este comando crea dos ficheros en la instancia EC2 con la clave SSH generada para el usuario `ubuntu`: `/home/ubuntu/.ssh/id_rsa` y `/home/ubuntu/.ssh/id_rsa.pub`.

En la consola de GitHub, en nuestro perfil, en el menú _Settings → SSH and GPG Keys_, configuramos una nueva clave SSH:
* __Title__: `ubuntu@<ip-publica-instancia-ec2>`
* __Key Type__: `Authentication Key`
* __Key__: contenido del fichero `/home/ubuntu/.ssh/id_rsa.pub` de la instancia EC2.

![][05]

__Nota__: para ver el contenido del fichero `/home/ubuntu/.ssh/id_rsa.pub` de la instancia EC2 puedes usar el comando `cat`.

En la instancia EC2 comprueba la conexión a GitHub: 

```bash
ssh -T git@github.com

Hi <github-user>! You've successfully authenticated, but GitHub does not provide shell access.
```

### Paso VIII. Instalalación y configuración del repositorio en la instancia EC2, configuración servicio Systemd y configuración de Security Group de AWS

En tu equipo, desde un navegador web, accede a la web de GitHub y loguéate con tu usuario, accede al repositorio de GitHub `https://github.com/jonaygarciav/app-contactos-py` y haz un `fork` del repositorio: ahora deberías tener un repositorio llamado `app-contactos-py` en tu cuenta de GitHub y puedes acceder a él desde la URL `https://github.com/<github-user>/app-contactos-py`

En la instancia EC2, clona el repositorio `https://github.com/<github-user>/app-contactos-py`  en el directorio `/home/ubuntu/flask-app`:

```bash
git clone git@github.com:<github-user>/app-contactos-experience.git /home/ubuntu/flask-app
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

En la instancia EC2 activa el servicio:

```bash
sudo systemctl daemon-reload

sudo systemctl enable flask-app
```

En la instancia EC2 carga el fichero `flask-app/seed.sql` en la Base de Datos MySQL:

```bash
cd /home/ubuntu/flask-app
mysql -h <github-user>-db.c18o4wyiy79q.us-east-1.rds.amazonaws.com -u contactos_user -p < seed.sql
```

En la instancia EC2, comprueba que el fichero `seed.sql` se ha cargado correctamente:

```sql
mysql -h <github-user>-db.c18o4wyiy79q.us-east-1.rds.amazonaws.com -u contactos_user -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 31
Server version: 8.0.40 Source distribution

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database          |
+--------------------+
| contactos_db      |
| information_schema |
| mysql             |
| performance_schema |
| sys               |
+--------------------+
5 rows in set (0.01 sec)

mysql> use contactos_db;
Database changed
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

mysql> show tables;
+----------------+
| Tables_in_contactos_db |
+----------------+
| contacto      |
+----------------+
1 row in set (0.00 sec)

mysql> select * from contacto;
+----+--------+-----------+------------+-------------------------+---------------+
| id | nombre | apellido1 | apellido2  | email                   | telefono      |
+----+--------+-----------+------------+-------------------------+---------------+
|  1 | Juan   | Perez     | Gomez      | juan.perez@example.com  | +34612345678  |
|  2 | Ana    | Martinez  | Lopez      | ana.martinez@example.com | +34687654321  |
|  3 | Carlos | Rodriguez | Fernandez  | carlos.rodriguez@example.com | +34711122334  |
|  4 | Elena  | Gonzalez  | Hernandez  | elena.gonzalez@example.com | +34755667788  |
|  5 | Pedro  | Sanchez   | Diaz       | pedro.sanchez@example.com | +34766778899  |
|  6 | Laura  | Ramirez   | Castro     | laura.ramirez@example.com | +34998877665  |
|  7 | Diego  | Torres    | Vega       | diego.torres@example.com | +34112233445  |
|  8 | Marta  | Flores    | Ortiz      | marta.flores@example.com | +34223344556  |
|  9 | Luis   | Moreno    | Ruiz       | luis.moreno@example.com | +34334455667  |
| 10 | Sara   | Jimenez   | Navarro    | sara.jimenez@example.com | +34445566778  |
+----+--------+-----------+------------+-------------------------+---------------+
10 rows in set (0.00 sec)
```

En la instancia EC2, en el directorio  `flask-app` crea un fichero llamado `.env` que contenga los parámetros de conexión a Base de Datos. Utiliza el fichero `.env.example` que se encuentra dentro del repositorio como referencia:

* __DB_USER__: `contactos_user`
* __DB_PASSWORD__: password del usuario `contactos_user`
* __DB_HOST__: `endpoint de la instancia RDS de MySQL`
* __DB_PORT__: `3306`
* __DB_NAME__: `contactos_db`

En la instancia EC2 arranca la aplicación:

```bash
sudo systemctl start flask-app
```

En la instancia EC2, dentro del directorio `flask-app` ejecuta el test `test_app.py` para probar la conectividad con la Base de Datos:

```bash
python3 test_app.py
```

En la instancia EC2 comprueba que la aplicación se haya levantado correctamente:

```bash
curl http://localhost:5000
```

Para poder acceder a la aplicación de la instancia EC2, en la consola web de AWS debes añadir al __Grupo de Seguridad__ (_Security Group_) llamado `<github-user>-ec2-sg` la siguiente regla:

Para el _tráfico de entrada_:

| Tipo           | Protocolo | Puerto | Origen      | Descripción             |
|----------------|-----------|--------|-------------|-------------------------|
| Personalizado  | TCP       | 5000   | `0.0.0.0/0` | Acceso a Flask App      |

Ahora, desde tu equipo, abre un navegador web e intenta acceder a la aplicación a través de la URL `http://<ip-publica-instancia-ec2>:5000` y comprueba que se muestra la aplicación.

### Paso IX. Realizar cambios en el repositorio y subirlos manualmente a la instancia EC2

Ahora desde tu equipo, clona el repositorio `https://github.com/<github-user>/app-contactos-py` en local y ábrelo con Visual Studio Code:
* Haz cambios en la aplicación modificando el título en el fichero `templates/index.html`.
* Crea un commit para guardar estos cambios en local y luego súbelos al repositorio remoto de GitHub.

En la instancia EC2, actualiza el repositorio trayéndote el último commit del repositorio remoto:

```bash
cd /home/ubuntu/flask-app

git pull
```

Posteriormente, en la instancia EC2, reiniciamos el servicio:

```bash
sudo systemctl restart flask-app
```

Ahora, desde tu equipo, abre un navegador web e intenta acceder a la aplicación a través de la URL  `http://<ip-publica-instancia-ec2>:5000` y comprueba que se muestran los cambios.

### Paso X. Automatizar cambios en el repositorio de la instancia EC2 cada vez que subimos un commit al repositorio remoto mediante GitHub Actions

Lecturas recomendadas:
* [Documentación de GitHub Actions](https://docs.github.com/es/actions)
* [Entender GitHub Actions](https://docs.github.com/es/actions/about-github-actions/understanding-github-actions)

Para automatizar la actualización del repositorio en la instancia EC2 cada vez que subamos un commit al repositorio remoto en GitHub, configuraremos un __workflow__ en _GitHub Actions_.

En la consola web de GitHub crea secretos en el repositorio `https://github.com/<github-user>/app-contactos-experience-py` de GitHub:

En la consola de GitHub, accede al repositorio `https://github.com/<github-user>/app-contactos-experience-py`, en el menú del repositorio _Settings → Secrets and variables → Actions → Repository secrets_, crea  los siguientes _Repository Secrets_:

* __EC2_SSH_KEY__: contenido del fichero `<github-user>-key.pem`.
* __EC2_HOST__: `IP pública de la instancia EC2`.
* __EC2_USER__: usuario `ubuntu`.

![][06]

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

Actualiza en tu equipo a través de Visual Studio Code tu copia del repositorio en local cambiándole el título de la página web, crea un commit para guardar estos cambios en local y luego súbelos al repositorio remoto de GitHub.

Confirma que los cambios en el repositorio activan el despliegue automático en la instancia EC2, para ello revisar GitHub Actions en `https://github.com/<github-user>/app-contactos-py/actions`.

Accede a la aplicación a través de la URL `http://<ip-publica-instancia-ec2>:5000` y comprueba que se muestran los cambios.

### Paso XI. Borrar Instancia EC2, Security Group, Key Pair y Repositorio de GitHub

Elimina los siguientes elementos de AWS:

* Instancia RDS `<github-user>-mysql`
* Instancia EC2 `<github-user>-ec2`
* Security Group `<github-user>-mysql-sg`
* Security Group `<github-user>-ec2-sg`
* Key Pair `<github-user>-key`

Elimina los siguientes repositorios de GitHub:

* `https://github.com/<github-user>/app-contactos-py`

Elimina las siguientes claves SSH de GitHub:

* En la cuenta personal, en el menú _Settings → SSH and GPG Keys_, eliminar la clave SSH `ubuntu@<ip-publica-instancia-ec2>`

[01]: ../img/aws/ec2-par-de-claves.png "01"
[02]: ../img/aws/ec2-security-group.png "02"
[03]: ../img/aws/ec2-instancia.png "03"
[04]: ../img/aws/rds.png "04"
[05]: ../img/github/ssh-key.png "05"
[06]: ../img/github/secrets.png "06"
