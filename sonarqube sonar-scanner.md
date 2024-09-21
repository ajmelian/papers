# SonarScanner CLI

## Descripción

El SonarScanner CLI es la herramienta de línea de comandos (Command Line Interface) oficial proporcionada por SonarQube para analizar la calidad del código fuente en proyectos donde no se dispone de un escáner específico para un sistema de construcción particular (como Maven, Gradle, o Jenkins). Es una herramienta versátil que se puede utilizar con cualquier proyecto, independientemente del lenguaje de programación.
**Nota:** Esta herramienta se debe instalar en la máquina donde se encuentre el codigo fuente a analizar, normalmente en la máquina de desarrollo.

### Principales Funciones del SonarScanner CLI

- **Análisis de Código**: SonarScanner CLI se utiliza para realizar análisis de calidad del código, detectando problemas como vulnerabilidades, bugs, code smells (malas prácticas), duplicación de código, y problemas de complejidad.

- **Integración con SonarQube**: El escáner envía los resultados del análisis directamente a una instancia de SonarQube, donde se procesan y se generan informes detallados que se pueden visualizar en el dashboard de SonarQube.

- **Soporte Multilenguaje**: Es compatible con una amplia variedad de lenguajes de programación, desde los más comunes como Java, Python, C#, y JavaScript, hasta otros menos convencionales.

- **Personalización**: Puedes configurar el análisis mediante un archivo de configuración llamado `sonar-project.properties`, donde se definen parámetros como la clave del proyecto, la codificación del código fuente, los directorios que deben ser analizados, y más.

- **Uso Flexible**: Al no depender de un entorno de construcción específico, se puede ejecutar en cualquier sistema operativo que soporte Java, lo que lo convierte en una opción muy flexible para integrar la inspección de código en diferentes entornos de desarrollo.

### Cómo Funciona SonarScanner CLI

1. **Configuración del Proyecto**: El usuario debe crear un archivo `sonar-project.properties` en el directorio raíz del proyecto. Este archivo contiene la configuración necesaria para realizar el análisis, como la clave del proyecto y las rutas de los archivos fuente.

2. **Ejecución del Escáner**: El usuario ejecuta el comando `sonar-scanner` desde la línea de comandos. Este comando inicia el proceso de análisis del código y envía los resultados al servidor SonarQube.

3. **Resultados en SonarQube**: Una vez completado el análisis, los resultados son almacenados en SonarQube, donde se pueden revisar detalladamente mediante su interfaz web. Los resultados incluyen información sobre vulnerabilidades, duplicación de código, cobertura de pruebas, y mucho más.

### Ejemplo de Uso Básico

Supongamos que tienes un proyecto configurado con un archivo `sonar-project.properties`, y deseas realizar un análisis de calidad del código:

1. Asegúrate de tener SonarScanner CLI instalado y configurado.

2. Ejecuta el comando:
   
   ```sh
   sonar-scanner -Dsonar.login=<tu_token_de_autenticación>
   ```
   
   Este comando lanzará el análisis del proyecto y enviará los resultados a la instancia de SonarQube especificada en la configuración.

## Instalación

### Requisito: Instalar Java

SonarQube requiere Java 17. Vamos a instalar OpenJDK 17.

```bash
sudo apt install openjdk-17-jdk -y
```

Verifica la instalación:

```bash
java -version
```

### Instalación del Escaner

1. Descargamos el comprimido ejecutable en `/opt`: 
   
   ```bash
   cd /opt
   wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-6.1.0.4477-linux-x64.zip?_gl=1*1jqqrtu*_gcl_au*ODUzNDMxNDIyLjE3MjQ0OTY2MDI.*_ga*MTkwNzY3MTM5OS4xNzI0NDk4MjMx*_ga_9JZ0GZ5TC6*MTcyNTIxNjQ1My4yLjEuMTcyNTIyODY5OS40Mi4wLjA -O sonar-scanner-cli-6.1.0.4477-linux-x64.zip
   unzip e sonar-scanner-cli-6.1.0.4477-linux-x64.zip
   mv sonar-scanner-6.1.0.4477-linux-x64 sonar-scanner
   ```
2. Creamos enlace simbólico para que el ejecutable sea accesible desde todo nuestro sistema.
   
   ```bash
   ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/bin/sonar-scanner
   ```
   
   Finalizado estos pasos, el ejecutable del escaner ya se encuentra debidamente instalado y accesible en todo el sistema.

## Configuración y Uso

### Configuración de tu proyecto

Crea un archivo de configuración en el directorio raíz de tu proyecto llamado `sonar-project.properties`. A continuación te muestro uno fichero con las variables posibles:

```txt
# Clave única del proyecto dentro de la instancia de SonarQube.
# Esta clave identifica de forma exclusiva tu proyecto en el servidor.
sonar.projectKey=mycompany:myproject

# Nombre del proyecto en SonarQube. Si no se especifica, se usará la clave del proyecto.
sonar.projectName=My Project

# Versión del proyecto. Útil para rastrear la evolución de la calidad del código a lo largo de diferentes versiones.
sonar.projectVersion=1.0

# Ruta relativa desde el archivo sonar-project.properties hasta el código fuente que debe ser analizado.
# Por defecto, SonarScanner busca en el directorio donde se encuentra este archivo.
sonar.sources=src

# Rutas de los archivos de pruebas, si los hay. Esto ayuda a calcular la cobertura de pruebas.
sonar.tests=tests

# Codificación del código fuente. Por defecto, usa la codificación del sistema, pero es importante especificarlo si el proyecto usa una codificación específica.
sonar.sourceEncoding=UTF-8

# Excluye ciertos archivos o directorios del análisis. Aquí puedes especificar patrones para excluir archivos que no necesitas analizar.
# Por ejemplo, archivos generados automáticamente, directorios de dependencias, etc.
sonar.exclusions=**/generated/**, **/libs/**

# Incluir directorios adicionales que contengan archivos relacionados que no son código fuente, pero que son relevantes para el análisis (por ejemplo, archivos de configuración).
sonar.inclusions=src/**/*.java, src/**/*.xml

# Directorio de los informes de cobertura de pruebas generados por herramientas externas (por ejemplo, Jacoco, Cobertura).
# Esto es necesario para integrar los resultados de cobertura de código en SonarQube.
sonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml

# Rutas de los archivos de duplicación de código generados por herramientas externas, si las hay.
# Estos informes pueden ser procesados por SonarQube para identificar duplicación de código.
sonar.cpd.exclusions=**/external-libraries/**

# Lenguaje principal del proyecto. Si el proyecto está escrito en un solo lenguaje, especifica cuál es.
# Si no se especifica, SonarQube intentará detectarlo automáticamente.
sonar.language=java

# URL del servidor SonarQube al que se enviarán los resultados del análisis. Esto puede ser configurado globalmente o en este archivo.
sonar.host.url=http://localhost:9000

# Token de autenticación para el análisis. Puedes especificarlo aquí o pasar el token directamente en la línea de comandos.
sonar.login=myAuthenticationToken

# Si estás utilizando un proxy para conectarte a SonarQube, debes configurar el proxy aquí.
# sonar.proxy.host=myproxy.example.com
# sonar.proxy.port=9000
# sonar.proxy.user=myProxyUser
# sonar.proxy.password=myProxyPassword

# Si el proyecto contiene submódulos, puedes especificar cómo deben ser analizados.
# sonar.modules=module1,module2
# sonar.moduleKey=module1
# sonar.sources=module1/src

# Directorio de salida de los informes generados por SonarQube.
# Este directorio contendrá los resultados intermedios antes de ser enviados al servidor.
sonar.working.directory=.scannerwork

# Habilitar o deshabilitar los plugins específicos para ciertos lenguajes.
# Esto es útil si deseas limitar el análisis solo a ciertos lenguajes soportados.
# sonar.exclusions=**/*.js

# Configuración de la rama a analizar, especialmente útil para proyectos que usan Git y tienen múltiples ramas.
sonar.branch.name=main

# Configuración para integración con sistemas de CI/CD como Jenkins, GitLab, etc.
# sonar.ci=true
# sonar.qualitygate.wait=true
```

### Ejecutando SonarScanner CLI

Para ejecutar SonarScanner CLI desde el archivo zip, sigue estos pasos:

1. Expande el archivo descargado en el directorio de tu elección. Nos referiremos a él como `<INSTALL_DIRECTORY>` en los siguientes pasos.

2. Actualiza la configuración global para apuntar a tu servidor SonarQube editando `$install_directory/conf/sonar-scanner.properties`:
   
   ```txt
    #----- Servidor SonarQube predeterminado
    #sonar.host.url=http://localhost:9000
   ```

3. Añade el directorio `<INSTALL_DIRECTORY>/bin` a tu PATH.

4. Verifica tu instalación abriendo una nueva terminal y ejecutando el comando `sonar-scanner -h`. Deberías obtener un resultado como este:
   
   ```bash
    usage: sonar-scanner [options]
   
    Options:
    -D,--define <arg>     Define propiedad
    -h,--help             Muestra la información de ayuda
    -v,--version          Muestra la información de la versión
    -X,--debug            Produce salida de depuración
   ```
   
    Si necesitas más información de depuración, puedes añadir una de las siguientes opciones a tu línea de comandos: `-X`, `--verbose`, o `-Dsonar.verbose=true`.

5. Ejecuta el siguiente comando desde el directorio base del proyecto para lanzar el análisis y pasar *tu token de autenticación*: `sonar-scanner -Dsonar.token=myAuthenticationToken`.
    Alternativamente, en lugar de pasar el token en tu línea de comandos, puedes crear la variable de entorno `SONAR_TOKEN` y asignarle el valor del token antes de lanzar el análisis.

## Escaneo de proyectos en C, C++ o Objective-C

El escaneo de proyectos que contienen código en C, C++ o Objective-C requiere algunos pasos adicionales de análisis. Puedes encontrar todos los detalles en la página del lenguaje [C/C++/Objective-C](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/languages/c-family/overview/ "C/C++/Objective-C").
