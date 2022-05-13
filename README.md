# Bootcamp_DevOps_Docker_Clase2

*ADJUNTO UN SOLO DOCKERFILE QUE COMPARTO CON TODOS LOS SERVICIOS YA QUE CON LA VARIABLE DE ENTORNO EJECUTO EL JAR INTERNO DE CADA SERVICIO*

## Anotaciones y resumen de la segunda clase de Docker: Docker Compose y Docker Registry


### En la clase de hoy hemos creado una copia (priceservice-mysql) de un servicio existente (priceservice) para agregarle la dependencia de mysql manualmente accediendo al pom.xml. También modificamos el application.yml del nuevo proyecto para configurarlo con mysql. Una vez creado el proyecto modificado para que trabaje con mysql, lo compilamos. Creamos un Dockerfile para cada uno de los servicios, y hacemos los build con cada uno de ellos par crear las imágenes. Una vez creadas las imágenes, creamos dentro de la carpeta Microservicios un documento .yml llamado docker-compose que será el encargado de desplegar nuestros contenedores con las imágenes creadas anteriormente y orquestar todos ellos siguiendo las instrucciones del fichero docker-compose. Por último solo quedaría probar en el navegador que todos nuestros micros han sido creados. Finalmente hemos creado un Docker registry que es un repositorio local con el que podemos hacer push y pull de todos nuestras imágenes.


**1. En el nuevo proyecto priceservicemysql nos dirigimimos a la raíz /source y dentro del pom.xml añadimos la siguiente dependencia de mysql:**

   ` <dependency> `

   ` <groupId>mysql</groupId>`

   ` <artifactId>mysql-connector-java</artifactId> `

   ` <scope>runtime</scope> `

**2. Dentro de application.yml (/servicio/src/main/resources/) añadimos las siguientes configuraciones de spring (en jpa configuramos la bd mysql, con datasource implementamos el patrón de conexión hiraki) y eureka (indicamos la ruta en la que levantará)**:

 ```yml
 spring:
  application:
    name: bootcamp-priceservice-mysql
  datasource:
    type: com.zaxxer.hikari.HikariDataSource    
    initialization-mode: ALWAYS
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
    hikari:
      data-source-properties:
        cachePrepStmts: true
        prepStmtCacheSize: 250
        prepStmtCacheSqlLimit: 2048
        useServerPrepStmts: true
  jpa:
    database-platform: org.hibernate.dialect.MySQL8Dialect
    database: MYSQL
    show-sql: false
    properties:
      hibernate.id.new_generator_mappings: true
      hibernate.cache.use_second_level_cache: false
      hibernate.cache.use_query_cache: false
      hibernate.generate_statistics: false
  cloud:
    config:
      import-check:
        enabled: false
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: ALWAYS 
   ```
      
**3. Con las configuraciones hechas, compilamos el proyecto para crear el nuevo .jar con el siguiente comando situándonos dentro del directorio de los binarios de nuestro priceservice-mysql:**

   ` cd /home/devops/Road2Cloud/00.Microservices/binaries/priceservice-mysql/ `

   ` docker run -it --rm --network host --name priceservice-maven -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.8.1-openjdk-11 mvn clean install -Dmaven.test.skip=true `

**4. Ahora creamos dentro de cada una de la carpeta de binarios de cada servicio, un archivo dockerfile con la configuración de las imágenes de cada servicio que vamos a crear. Las instrucciones que vamos a indicar son:**

  > FROM openjdk:11 

   *Conecta la imagen que vamos a crear con la imagen del jdk 11 previamente montada en nuestra máquina (en nuestro caso la trae montada, pero se podría pullear de docker hub)*
   
   > ENV SERVER_PORT=9090 \
       APPLICATION=inner-spring-boot-app 
       
   *Definimos las variables de entorno para indicar un puerto y que vamos a utilizar la imagen jar interna (con esto evitamos errores y facilitamos la reutilización del fichero). Las variables de entorno sirven para facilitar el trabajo con archivos complejos y ofrecer ficheros más legibles.*
   
   > EXPOSE ${SERVER_PORT} 

   *Expose no es obligatorio y no afecta al funcionamiento, solo documenta el puerto de escucha del contenedor.*
   
   > COPY *.jar /jars/${APPLICATION}.jar 

   *Con COPY copiamos el jar interno a la nueva carpeta jars.*
   
   > CMD java -jar /jars/${APPLICATION}.jar
 
   *Con CMD ejecutamos el jar copiado dentro de la carpeta jars.*
   
**5. Con todos los dockerfile creados dentro de cada servicio, levantamos cada una de las imágenes de los servicios: **
(Cada build lo ejecutamos dentro del path del dockerfile de cada servicio)

` docker build -t eurekaservice:1.0 . `

` docker build -t priceservicemysql:1.0 . `

` docker build -t productservice:1.0 . `

` docker build -t adminservice:1.0 . `

` docker build -t zuulservice:1.0 . `

**6. Creamos el fichero docker-compose.yml:**

` cd /home/devops/Road2Cloud/00.Microservices/binaries/ `

**7. Definimos la configuración del docker-compose: **
   - services: declaramos todos nombres de los servicios que vamos a usar
   - image: es el nombre de la imagen con su tag, por ejemplo adminserver:1.0. Con esta imagen se despliega el servicio
   - ports: declaramos el puerto por donde va a levantar el contenedor en el host
   - depends_on: aquí indicamos los nombres de nos servicios que tienen que estar desplegados para que cada uno funcione
   - environment: para añadir variables de entorno que nos permiten setear valores
 
 **8. Creamos todos los contenedores orquestados por el fichero docker-compose utilizando un solo comando. Para ello nos colocamos en el path del fichero y ejecutamos:**
 
 ` cd /home/devops/Road2Cloud/00.Microservices/binaries/ `
 
 ` docker-compose up `
 
 ![image](https://user-images.githubusercontent.com/69739273/168346897-5eebc7a6-9297-4799-938f-bf5108585049.png)

**9. Para crear un repositorio local de nuestras imágenes hacemos uso de Docker registry. Lo levantamos con el comando:

` docker run -d -p 5000:5000 -v /mnt/registry:/var/lib/registry  --restart=always --name bootcamp-registry registry:2 `

**10. Para tagear una imagen a nuestro nuevo repositorio local usamos el comando docker tag y comprobamos con docker images que se ha levantado, por ejemplo:**

` docker tag adminservice:1.0 172.28.128.80:5000/imagenadminregistry `

![image](https://user-images.githubusercontent.com/69739273/168373540-e8a5fec3-3c42-4629-9b75-f7276248e144.png)

