# Sophora UGC

This chart deploys Sophora UGC. Optionally, the UGC Multimedia Service addon is also deployed.

## What you need (requirements)

This chart requires the following already present in the target namespace:

* An ImagePullSecret for the subshell Docker Registry
* A secret containing username and password for the sophora server and a username and password for the database.

## Network configuration

This chart contains a preconfigured ingress. It can be used to make the editorial UI available to editors.

If using the UGC Multimedia Service, public access is needed to the `/public/` endpoints for users to upload multimedia objects directly to the microservice.  
Always ensure that access to all other endpoints is protected. Adjust your ingress and network configurations accordingly.

For details, check the [UGC Multimedia Service documentation page](https://subshell.com/docs/ugc/4/ugc192.html).

## Example values.yaml

```yaml
service:
  jolokia:
    clusterIP: None
  webapp:
    type: LoadBalancer

ingress:
  enabled: false
  ingressClassName: nginx
  hosts:
    - host: "my-ugc.com"
  annotations: {}

authentication:
  secret:
    server:
      name: secret-name
      usernameKey: username
      passwordKey: password
    database:
      name: secret-name
      usernameKey: database-user
      passwordKey: database-password
    binarystore:
      name: secret-name
      accessKeyKey: binarystore-access-key
      secretIdKey: binarystore-secret-id
    s3bucket:
      name: s3bucket-secret
      accessKeyKey: s3bucket-access-key
      secretIdKey: s3bucket-secret-id

ugc:
  image:
    repository: docker.subshell.com/ugc/ugc
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "latest"
  binariesStorage:
    size: 1G
    storageClass: standard

  logback: |
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration scan="true">
        <jmxConfigurator />
        <!-- Propagate level settings to java.util.logging. -->
        <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator"/>
        <appender name="logfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <File>/logs/ugc-webapp.log</File>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>/logs/ugc.%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>[%d{"yyyy-MM-dd HH:mm:ss.SSSX",UTC}, %-5p] [%t] [%X{ID}] %.30c:%L: %m%n</pattern>
            </encoder>
        </appender>
        <appender name="errorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                <level>ERROR</level>
            </filter>
            <File>/logs/error.log</File>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>/logs/error.%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>[%d{"yyyy-MM-dd HH:mm:ss.SSSX",UTC}, %-5p] [%t] [%X{ID}] %.40c:%L: %m%n</pattern>
            </encoder>
        </appender>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level [%X{ID}] %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <logger name="com.subshell.sophora.ugc" level="INFO" />
        <root level="WARN">
            <appender-ref ref="STDOUT" />
            <appender-ref ref="logfile" />
            <appender-ref ref="errorLog" />
        </root>
    </configuration>
  config:
    sophora-server:
      host: "http://localhost:1196"

    # configuration of the database connection. Adapt this to match with your settings
    database:
      # url of the jdbc connection, the last part is the name of the database
      url: "jdbc:mysql://localhost:3306/usercontent"

    # configure the port of the embedded jetty
    server:
      port: 9080
      
    previewUrl: "https://localhost:8090/demosite/demosite/system/preview.jsp?sophoraid="

    # ratings, image uploads and comments can be enabled for a list of node types
    rating:
      primaryTypes: ["sophora-content-nt:story"]


ugcMultimedia:
  enabled: false # enable to deploy UGC Multimedia Service
  image:
    repository: docker.subshell.com/ugc/ugc-multimedia
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: "latest"
  hostname:
  logback: |
    <?xml version="1.0" encoding="UTF-8"?>

    <configuration scan="true" scanPeriod="60 seconds">
    <jmxConfigurator />

    <!-- Propagate level settings to java.util.logging. -->
    <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator"/>

    <appender name="logfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <File>logs/ugc-multimedia.log</File>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>logs/ugc-multimedia.%d{yyyy-MM-dd}.log.gz</fileNamePattern>
    <maxHistory>7</maxHistory>
    </rollingPolicy>
    <encoder>
    <pattern>[%d{ISO8601}, %-5p] [%t] [%X{ID}] %.30c:%L: %m%n</pattern>
    </encoder>
    </appender>
    <appender name="errorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>ERROR</level>
    </filter>
    <File>logs/ugc-multimedia-error.log</File>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>logs/ugc-multimedia-error.%d{yyyy-MM-dd}.log.gz</fileNamePattern>
    <maxHistory>7</maxHistory>
    </rollingPolicy>
    <encoder>
    <pattern>[%d{ISO8601}, %-5p] [%t] [%X{ID}] %.40c:%L: %m%n</pattern>
    </encoder>
    </appender>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level [%X{ID}] %logger{36} - %msg%n</pattern>
    </encoder>
    </appender>

    <logger name="com.subshell.sophora.ugc.services" level="DEBUG" />
    <logger name="org.springframework" level="INFO" />

    <root level="INFO">
    <appender-ref ref="logfile" />
    <appender-ref ref="STDOUT" />
    <appender-ref ref="errorLog" />
    </root>
    </configuration>
  config:
    spring:
      jpa:
        hibernate:
          ddl-auto: update
        database-platform: org.hibernate.dialect.MySQL5InnoDBDialect # org.hibernate.dialect.Oracle12cDialect
        database: mysql # oracle
      datasource:
        url: jdbc:mysql://host.docker.internal:3306/ugc-multimedia
        username: ${DATABASE_USER}
        password: ${DATABASE_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver # oracle.jdbc.OracleDriver
      servlet:
        multipart:
          max-file-size: 20MB
          max-request-size: 20MB

    management:
      endpoints:
        web:
          exposure:
            include: "prometheus"
      endpoint:
        prometheus:
          enabled: true

    sophora:
      client:
        server-connection:
          urls: http://server:1196

    storage:
      type: fs
      fs:
        baseDir: /binaries/
      # Configuration for s3
      # type: s3
      # s3:
      #   accessKeyId: ${BINARY_STORE_ACCESS_KEY}
      #   secretAccessKey: ${BINARY_STORE_SECRET_ID}
      #   bucketName: <bucketName>
      #   host: <host>

    multimedia:
      allowedMimeTypes:
        - audio
        - video
      headlineProperty: sophora-content:title
      markers:
        - key: "key1"
          label: "topic 1"
          colorValue: "Red"
        - key: "key 2"
          label: "topic 2"
          colorValue: "Gold"
        - key: "key 3"
          label: "topic 3"
          colorValue: "Green"

    clamd:
      clamdHost:
      clamdPort: 0
      timeoutInMS: 10
      clamdFileStorageDir: clamdTest/
      httpStatusCodeForVirus: 200

    cleanup:
      binaries:
        enabled: false
        cronSchedule: "0 59  * * * *"
        regexForBinaries: "[a-z0-9-]{36,}" #Regex for UUIDs
        binaryWasAtLeastNotModifiedSinceHours: 0
      metadata:
        enabled: false
        cronSchedule: "0 59  * * * *"
        minimumAgeInHours: 0

    headers:
      crossOriginUrl: "http://localhost:8080"

    websocket:
      allowedOrigin: http://localhost:9000

    springdoc:
      swagger-ui:
        url: "/v3/api-docs"
        disable-swagger-default-url: true
        csrf:
          enabled: true

# Additional environment variables can be defined here
env:

resources:
  requests:
    memory: "3.5G"
    cpu: "0.5"
  limits:
    memory: "3.5G"
```
