# Bootcamp_DevOps_Docker_Clase2
## Anotaciones y resumen de la segunda clase de Docker: Docker Compose y Docker Registry

### En la clase de hoy hemos creado una copia (priceservicemysql) de un servicio existente (priceservice) para agregarle la dependencia de mysql manualmente accediendo al pom.xml. También modificamos el application.yml del nuevo proyecto para configurarlo con mysql. Una vez creado el proyecto modificado para que trabaje con mysql, lo compilamos. Creamos un Dockerfile para cada uno de los servicios, y hacemos los build con cada uno de ellos par crear las imágenes. Una vez creadas las imágenes, creamos dentro de la carpeta Microservicios un documento .yml llamado docker-compose que será el encargado de desplegar nuestros contenedores con las imágenes creadas anteriormente y orquestar todos ellos siguiendo las instrucciones del fichero docker-compose. Por último solo quedaría probar en el navegador que todos nuestros micros han sido creados. Finalmente hemos creado un Docker registry que es un repositorio local con el que podemos hacer push y pull de todos nuestras imágenes.


**1. En el nuevo proyecto priceservicemysql nos dirigimimos a la raíz /source y dentro del pom.xml añadimos la siguiente dependencia de mysql:**

` <dependency>`

` <groupId>mysql</groupId>`

` <artifactId>mysql-connector-java</artifactId> `

` <scope>runtime</scope> `

** 2. Dentro de application.yml (/servicio/src/main/resources/) añadimos las siguientes propiedades:

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
      show-details: ALWAYS ```
