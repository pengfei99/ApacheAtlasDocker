# Gremlin support

In the standard image, the Gremlin support has been removed.

Tinkerpop Gremlin support
-------------------------

Image contains build-in extras for those who want to play with Janusgraph, and Atlas artifacts using Apache Tinkerpop Gremlin Console (gremlin CLI).

1. You need Atlas container up and running as shown above.

2. Install `gremlin-server` and `gremlin-console` into the container by running included automation script:
```bash
docker exec -ti atlas /opt/gremlin/install-gremlin.sh
```
3. Start `gremlin-server` in the same container:
```bash
docker exec -d atlas /opt/gremlin/start-gremlin-server.sh
```
4. Finally, run `gremlin-console` interactively:
```bash
docker exec -ti atlas /opt/gremlin/run-gremlin-console.sh
```
Gremlin-console usage example:
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
