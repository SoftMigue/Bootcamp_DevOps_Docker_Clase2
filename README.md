# Bootcamp_DevOps_Docker_Clase2
## Anotaciones y resumen de la segunda clase de Docker. Docker Compose y Docker Registry

#### En la clase de hoy hemos creado una copia (priceservicemysql) de un servicio existente (priceservice) para agregarle la dependencia de mysql manualmente accediendo al pom.xml. También modificamos el application.yml del nuevo proyecto para configurarlo con mysql. Una vez creado el proyecto modificado para que trabaje con mysql, lo compilamos. Creamos un Dockerfile para cada uno de los servicios, y hacemos los build con cada uno de ellos par crear las imágenes. Una vez creadas las imágenes, creamos dentro de la carpeta Microservicios un documento .yml llamado docker-compose que será el que orqueste todas 
