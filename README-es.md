# ApacheAtlasDocker

Éste `Apache Atlas` está construido a partir del tarball de origen de la versión 2.1.0 y parcheado para ejecutarse en un contenedor Docker.

Atlas está construido con `embedded HBase + Solr` y está preinicializado, por lo que puede usarlo justo después de la descarga de la imagen sin pasos adicionales.

Si desea utilizar backends externos de Atlas, configúrelos de acuerdo con [la documentación](https://atlas.apache.org/#/Configuration).

## Uso básico

1.  Extraiga la última imagen de lanzamiento:

```bash
docker pull 
```

2.  Inicie Apache Atlas en un contenedor que expone el puerto Web-UI 21000:

```bash
docker run -d \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Tenga en cuenta que el primer inicio de Atlas puede tardar hasta unos minutos dependiendo del rendimiento de la máquina host antes de que la interfaz web esté disponible en `http://localhost:21000/`

Credenciales predeterminadas de la interfaz de usuario web: `admin / admin`

## Opciones de uso

Detenga con gracia Atlas:

```bash
docker exec -ti atlas /opt/apache-atlas-2.1.0/bin/atlas_stop.py
```

Compruebe el resultado del script de inicio de Atlas:

```bash
docker logs atlas
```

Compruebe de forma interactiva la aplicación Atlas.log (útil en la primera ejecución y para la depuración durante la carga de trabajo):

```bash
docker exec -ti atlas tail -f /opt/apache-atlas-2.1.0/logs/application.log
```

Ejecute el ejemplo (esto agregará tipos e instancias de ejemplo junto con rasgos):

```bash
docker exec -ti atlas /opt/apache-atlas-2.1.0/bin/quick_start.py
```

Iniciar Atlas reemplazando la configuración por variables de entorno
(para admitir un gran número de objetos de metadatos, por ejemplo):

```bash
docker run --detach \
    -e "ATLAS_SERVER_OPTS=-server -XX:SoftRefLRUPolicyMSPerMB=0 \
    -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC \
    -XX:+CMSParallelRemarkEnabled -XX:+PrintTenuringDistribution \
    -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=dumps/atlas_server.hprof \
    -Xloggc:logs/gc-worker.log -verbose:gc -XX:+UseGCLogFileRotation \
    -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=1m -XX:+PrintGCDetails \
    -XX:+PrintHeapAtGC -XX:+PrintGCTimeStamps" \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Inicie Atlas exponiendo el directorio de registros en el host para verlos directamente:

```bash
docker run --detach \
    -v ${PWD}/atlas-logs:/opt/apache-atlas-2.1.0/logs \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Inicie Atlas exponiendo el directorio conf en el host para colocar y editar archivos de configuración directamente:

```bash
docker run --detach \
    -v ${PWD}/pre-conf:/opt/apache-atlas-2.1.0/conf \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Inicie Atlas con el directorio de datos montado en el host para proporcionar su persistencia:

```bash
docker run --detach \
    -v ${PWD}/data:/opt/apache-atlas-2.1.0/data \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Puede combinar la configuración anterior togethor

```bash
docker run --detach \
    -v ${PWD}/atlas-logs:/opt/apache-atlas-2.1.0/logs \
    -v ${PWD}/data:/opt/apache-atlas-2.1.0/data \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

## Soporte técnico de Tinkerpop Gremlin

La imagen contiene extras incorporados para aquellos que quieran jugar con Janusgraph y artefactos Atlas usando Apache Tinkerpop Gremlin Console (gremlin CLI).

1.  Necesita que el contenedor Atlas esté en funcionamiento como se muestra arriba.

2.  Instalar `gremlin-server` y `gremlin-console` en el contenedor ejecutando el script de automatización incluido:

```bash
docker exec -ti atlas /opt/gremlin/install-gremlin.sh
```

3.  Empezar `gremlin-server` en el mismo contenedor:

```bash
docker exec -d atlas /opt/gremlin/start-gremlin-server.sh
```

4.  Finalmente, corre `gremlin-console` interactivamente:

```bash
docker exec -ti atlas /opt/gremlin/run-gremlin-console.sh
```

Ejemplo de uso de la consola Gremlin:

```bash
         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----

gremlin>:remote connect tinkerpop.server conf/remote.yaml session
==>Configured localhost/127.0.0.1:8182-[d1b2d9de-da1f-471f-be14-34d8ea769ae8]
gremlin> :remote console
==>All scripts will now be sent to Gremlin Server - [localhost/127.0.0.1:8182]-[d1b2d9de-da1f-471f-be14-34d8ea769ae8] - type ':remote console' to return to local mode
gremlin> g = graph.traversal()
==>graphtraversalsource[standardjanusgraph[hbase:[localhost]], standard]
gremlin> g.V().has('__typeName','hdfs_path').count()
```

## Variables de entorno

Las siguientes variables de entorno están disponibles para la configuración:

| Nombre | | predeterminada Descripción |
|------|---------|-------------|
| JAVA_HOME | /usr/lib/jvm/java-8-openjdk-amd64 | La implementación de java a utilizar. Si no se encuentra JAVA_HOME esperamos que java y jar estén en camino
| ATLAS_OPTS | <none> | cualquier opción java adicional que desee establecer. Esto se aplicará tanto a las operaciones de cliente como de servidor.
| ATLAS_CLIENT_OPTS | <none> | Cualquier opción java adicional que desee establecer sólo para el cliente
| ATLAS_CLIENT_HEAP | <none> | Tamaño del montón de java que queremos establecer para el cliente. El valor predeterminado es 1024 MB
| ATLAS_SERVER_OPTS | <none> |  cualquier opción adicional que desee establecer para el servicio atlas.
| ATLAS_SERVER_HEAP | <none> | Tamaño del montón de java que queremos establecer para el servidor atlas. El valor predeterminado es 1024 MB
| ATLAS_HOME_DIR | <none> | Lo que se considera como atlas home dir. El valor predeterminado es la ubicación base del software instalado
| ATLAS_LOG_DIR | <none> | Dónde se almacenan los archivos de registro. Defatult es el directorio de registros bajo la ubicación de instalación base
| ATLAS_PID_DIR | <none> | Donde se almacenan los archivos pid. Defatult es el directorio de registros bajo la ubicación de instalación base
| ATLAS_EXPANDED_WEBAPP_DIR | <none> | ¿Dónde quieres expandir el archivo de guerra? De forma predeterminada, está en /server/webapp dir bajo el dir de instalación base.

## Rastreador de errores

Los errores se rastrean en [Problemas de GitHub](https://github.com/sburn/docker-apache-atlas/issues).
En caso de problemas, verifique allí para ver si su problema ya ha sido reportado.
Si lo viste primero, ayúdanos a aplastarlo proporcionándonos comentarios detallados y bienvenidos.

## Mantenedor

Esta imagen es mantenida por [Vadim Korchagin](mailto:vadim@clusterside.com)

*   https://github.com/sburn/docker-apache-atlas
