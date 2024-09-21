# SonarQube: Instalación y Configuración

========================================

## Descripción

[SonarQube](https://www.sonarsource.com/) es una plataforma de código abierto para la inspección continua de la calidad del código que realiza revisiones automáticas con análisis estático del código para detectar errores, malos olores en el código y vulnerabilidades de seguridad. 

Este tutorial te guiará a través de los pasos para instalar y configurar SonarQube Community Edition en un sistema Ubuntu Linux 22.04.

## Prerrequisitos

* Un servidor dedicado [Ubuntu 22.04](https://hostnextra.com/dedicated-server) nuevo
* Un usuario con privilegios sudo
* Java 17 instalado (SonarQube requiere una versión específica de Java)
* Node.JS Runtime (necesario en máquinas donde se realizarán los análisis)

Para este tutorial, hemos utilizado un servidor en la nube con la configuración de 2vCPU, 4GB de RAM, 80GB SSD. Debe tener al menos 2GB de RAM, 1 núcleo de CPU y 30GB de espacio libre.

## Paso 1: Actualizar el Sistema

Primero, asegúrate de que tu sistema esté actualizado:

```bash
sudo apt update  
sudo apt upgrade -y
```

## Paso 2: Instalar Java

SonarQube requiere Java 11 o 17. Vamos a instalar OpenJDK 17.

```bash
sudo apt install openjdk-17-jdk -y
```

Verifica la instalación:

```bash
java -version
```

## Paso 3: Instalar PostgreSQL

SonarQube utiliza PostgreSQL como su base de datos. Instala PostgreSQL 15. Ejecuta el siguiente conjunto de comandos:

```bash
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

Actualiza e instala PostgreSQL 15:

```bash
sudo apt update  
sudo apt install postgresql-15 -y
```

Ahora, configuremos PostgreSQL

Cambia al usuario de PostgreSQL:

```bash
sudo -i -u postgres
```

Crea un nuevo usuario y base de datos para SonarQube:

```bash
createuser sonar  
createdb sonar -O sonar  
psql
```

Dentro del shell de PostgreSQL, establece una contraseña para el usuario sonar:

```sql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'tu_contraseña';  
\q
```

Sal del usuario PostgreSQL:

```sql
exit
```

## Paso 4: Instalar SonarQube

Descarga la [última versión de SonarQube](https://www.sonarsource.com/products/sonarqube/downloads/success-download-community-edition/):

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
```

Extrae el paquete de SonarQube:

```bash
unzip sonarqube-10.5.1.90531.zip  
sudo mv sonarqube-10.5.1.90531 /opt/sonarqube
```

Crea un usuario para SonarQube:

```bash
sudo adduser --system --no-create-home --group --disabled-login sonarqube
```

Cambia la propiedad del directorio de SonarQube:

```bash
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

Ahora, configuremos SonarQube

Edita el archivo de configuración de SonarQube:

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

Descomenta y configura las siguientes propiedades:

```
sonar.jdbc.username=sonar  
sonar.jdbc.password=tu_contraseña  
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
```

## Paso 5: Crear un Archivo de Servicio Systemd

Crea un nuevo archivo de servicio para SonarQube:

```bash
sudo vi /etc/systemd/system/sonarqube.service
```

Añade el siguiente contenido:

```
[Unit]  
Description=Servicio SonarQube  
After=syslog.target network.target  

[Service]  
Type=forking  

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start  
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop  

User=sonarqube  
Group=sonarqube  
Restart=always  

LimitNOFILE=65536  
LimitNPROC=4096  

[Install]  
WantedBy=multi-user.target
```

Recarga el demonio systemd y arranca SonarQube:

```bash
sudo systemctl daemon-reload  
sudo systemctl start sonarqube  
sudo systemctl enable sonarqube
```

## Paso 6: Descriptores de Archivos

Verifica el límite actual:

```bash
ulimit -n
```

Debe ser al menos `65536`. Para incrementarlo, añade lo siguiente a `/etc/security/limits.conf`:

```bash
sudo vi /etc/security/limits.conf
```

Añade las siguientes líneas:

```
sonarqube    -    nofile    65536  
sonarqube    -    nproc    4096
```

Verifica y establece el límite de memoria virtual:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Para hacer este cambio permanente, añádelo a `/etc/sysctl.conf`:

```bash
sudo vi /etc/sysctl.conf
```

Añade la siguiente línea:

```
vm.max_map_count=262144
```

Aplica los cambios:

```bash
sudo sysctl -p
```

## Paso 7: Instalar y Configurar Nginx

Instala Nginx:

```bash
sudo apt install nginx -y
```

Crea un nuevo archivo de configuración de Nginx para SonarQube:

```bash
sudo vi /etc/nginx/sites-available/sonarqube
```

Añade el siguiente contenido:

```bash
server {  
    listen 80;  
    server_name sonarqube.local;  

    access_log /var/log/nginx/sonarqube.access.log;  
    error_log /var/log/nginx/sonarqube.error.log;  

    location / {  
        proxy_pass http://localhost:9000;  
        proxy_set_header Host $host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_set_header X-Forwarded-Proto $scheme;  
    }  
}
```

**Nota:** Reemplaza _sonarqube.ejemplo.com_ con tu nombre de dominio o comenta si trabajas con la dirección IP.

Habilita la nueva configuración:

```bash
sudo ln -s /etc/nginx/sites-available/sonarqube /etc/nginx/sites-enabled/
```

Prueba la configuración de Nginx y reinicia Nginx:

```bash
sudo nginx -t  
sudo systemctl restart nginx
```

## Paso 8: Acceder a SonarQube

Abre tu navegador web y ve a `https://tu_dominio_o_ip`. Deberías ver la página de inicio de sesión de SonarQube. Las credenciales predeterminadas son:

Usuario: admin  
Contraseña: admin

Al primer inicio de sesión, se te pedirá que cambies la contraseña predeterminada.

## Conclusión

Llegado este paso, se ha finalizado con la instalación del SonarQube en nuestro servidor particular. 

Pero, ¿cómo hacemos un análisis? Para ello debemos acudir a una aplicación satélite llamada SonarScanner.

------

# SonarScanner-cli

SonarScanner-cli es la aplicación que realiza el análisis del código fuente. Este programa autónomo se ejecuta en el host CI/CD y envía los resultados del análisis al servidor SonarQube, que los computa, calcula la calidad y genera informes. Para realizar el análisis, SonarScanner-cli utiliza los analizadores de lenguaje que descarga del servidor SonarQube durante la instalación.
