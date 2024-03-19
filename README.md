# ApacheAtlasDocker

This repo is used to build the latest `Apache Atlas` docker image.

> Current latest version is v2.3.0

Atlas is built with `embedded HBase + Solr` and it is pre-initialized, so you can use it right after image 
download without additional steps.

If you want to use external Atlas backends, set them up according to [the documentation](https://atlas.apache.org/#/Configuration).

Basic usage
-----------
1. Pull the latest release image:

```bash
docker pull 
```

2. Start Apache Atlas in a container exposing Web-UI port 21000:

```bash
docker run -d \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Please, take into account that the first startup of Atlas may take up to few minunts depending on host machine performance before web-interface become available at `http://localhost:21000/`

Web-UI default credentials: `admin / admin`

Usage options
-------------

Gracefully stop Atlas:

```bash
docker exec -ti atlas /opt/apache-atlas-2.1.0/bin/atlas_stop.py
```

Check Atlas startup script output:

```bash
docker logs atlas
```

Check interactively Atlas application.log (useful at the first run and for debugging during workload):

```bash
docker exec -ti atlas tail -f /opt/apache-atlas-2.1.0/logs/application.log
```

Run the example (this will add sample types and instances along with traits):

```bash
docker exec -ti atlas /opt/apache-atlas-2.1.0/bin/quick_start.py
```

Start Atlas overriding settings by environment variables 
(to support large number of metadata objects for example):

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

Start Atlas exposing logs directory on the host to view them directly:

```bash
docker run --detach \
    -v ${PWD}/atlas-logs:/opt/apache-atlas-2.1.0/logs \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

Start Atlas exposing conf directory on the host to place and edit configuration files directly:

```bash
docker run --detach \
    -v ${PWD}/pre-conf:/opt/apache-atlas-2.1.0/conf \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```
Start Atlas with data directory mounted on the host to provide its persistency:
```bash
docker run --detach \
    -v ${PWD}/data:/opt/apache-atlas-2.1.0/data \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```

You can combine the above config togethor

```bash
docker run --detach \
    -v ${PWD}/atlas-logs:/opt/apache-atlas-2.1.0/logs \
    -v ${PWD}/data:/opt/apache-atlas-2.1.0/data \
    -p 21000:21000 \
    --name atlas \
    sburn/apache-atlas \
    /opt/apache-atlas-2.1.0/bin/atlas_start.py
```


Environment Variables
---------------------

The following environment variables are available for configuration:

| Name                      | Default                           | Description                                                                                                  |
|---------------------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------|
| JAVA_HOME                 | /usr/lib/jvm/java-8-openjdk-amd64 | The java implementation to use. If JAVA_HOME is not found we expect java and jar to be in path               |
| ATLAS_OPTS                | <none>                            | any additional java opts you want to set. This will apply to both client and server operations               |
| ATLAS_CLIENT_OPTS         | <none>                            | any additional java opts that you want to set for client only                                                |
| ATLAS_CLIENT_HEAP         | <none>                            | java heap size we want to set for the client. Default is 1024MB                                              |
| ATLAS_SERVER_OPTS         | <none>                            | any additional opts you want to set for atlas service.                                                       |
| ATLAS_SERVER_HEAP         | <none>                            | java heap size we want to set for the atlas server. Default is 1024MB                                        |
| ATLAS_HOME_DIR            | <none>                            | What is is considered as atlas home dir. Default is the base location of the installed software              |
| ATLAS_LOG_DIR             | <none>                            | Where log files are stored. Defatult is logs directory under the base install location                       |
| ATLAS_PID_DIR             | <none>                            | Where pid files are stored. Defatult is logs directory under the base install location                       |
| ATLAS_EXPANDED_WEBAPP_DIR | <none>                            | Where do you want to expand the war file. By Default it is in /server/webapp dir under the base install dir. |

Bug Tracker
-----------

Bugs are tracked on [GitHub Issues](https://github.com/sburn/docker-apache-atlas/issues).
In case of trouble, please check there to see if your issue has already been reported.
If you spotted it first, help us smash it by providing detailed and welcomed feedback.

Maintainer
----------

This image is maintained by [Vadim Korchagin](mailto:vadim@clusterside.com)

* https://github.com/sburn/docker-apache-atlas
