# Proxmox VE Standalone v1

## Descripción

Crear un entorno Proxmox VE Standalone para poder desplegar la aplicación https://github.com/jonaygarciav/app-descrubre-canarias-bs

## Software

    • Proxmox VE 8.3.1
    • Ubuntu Server 24.04
    • InfluxDB
    • Grafana

## Máquinas

Crear las siguientes VMs dentro de VMware Workstation:

| Servicio      | CPU | RAM | HD          | IP              | FQDN                |
|---------------|-----|-----|-------------|-----------------|---------------------|
| Proxmox VE 1  | 3   | 16  | 128GB (OS)  | 192.168.X.111   | pve8-1.labs.loc     |
| InfluxDB      | 2   | 2   | 16GB (OS)   | 192.168.X.114   | influxdb.labs.loc   |
| Grafana       | 2   | 2   | 16GB (OS)   | 192.168.X.115   | grafana.labs.loc    |


Crear las siguientes VMs dentro de la VM pve8-1.labs.loc:


| Servicio                 | CPU | RAM | HD          | IP              | FQDN                | Descripción                              |
|--------------------------|-----|-----|-------------|-----------------|---------------------|------------------------------------------|
| Ubuntu Server 24.04 (TLP)| 2   | 2   | 16GB (OS)   | 192.168.X.112   | tlp.labs.loc       |                                          |
| Nginx                    | 2   | 2   | 16GB (OS)   | 192.168.X.113   | nginx-srv.labs.loc | Clonado a partir de Ubuntu Server 24.04 |


## Contenido

Parte I:

    • Crear VM pve8-1 e instalar OS
    • Configurar No-Subscription Repository en pve8-1
    • Actualizar sistema en pve8-1

Parte II:

    • Crear VM tlp.labs.loc en pve8-1 (id 100) e instalar OS:
        ○ Configurar parámetros de red
        ○ Instalar servicio SSH
    • Crear VM nginx-srv.labs.loc en pve8-1 con id 101 clonada a partir de VM con id 100.
        ○ Configurar parámetros de red
        ○ Instalar servicio Nginx

Parte III:

    • Hacer un fork del proyecto https://github.com/jonaygarciav/app-descrubre-canarias-bs 
    • Desplegar la aplicación app-descubre-canarias-bs la VM nginx-srv.labs.loc
        ○ Copiar aplicación en directorio raíz de Nginx
        ○ Comprobar que accedemos a la aplicación desde el navegador web: http://<ip-nginx-srv.labs.loc>
    • Añadir 3 cards nuevas a la aplicación:
        ○ Subir los cambios al repositorio remoto
        ○ Desplegar los nuevos cambios

Parte IV:

    • Hacer un backup simple de la VM nginx-srv.labs.loc con id 101
        ○ Borrar el directorio raíz de Nginx de la VM nginx-srv.labs.loc con id 101
        ○ Restaurar el backup simple para recuperar el directorio
        ○ Borrar el backup simple de la VM nginx-srv.labs.loc con id 101

Parte V (Opcional):

    • Crear VM influxdb.labs.loc
        ○ Configurar parámetros de red
        ○ Instalar servicio Prometheus
    • Crear VM influxdb.labs.loc
        ○ Configurar parámetros de red
        ○ Instalar servicio Grafana
    • Instalar telegraf en pve8-1
        ○ Conectar telegraf con servicio Prometheus para que envíe métricas
    • Instalar Dashboard https://grafana.com/grafana/dashboards/10048-proxmox/
        ○ Conectarlo con InfluxDB
        ○ Revisar que se ven las métricas de pve8-1 en el dashboard de Grafana
