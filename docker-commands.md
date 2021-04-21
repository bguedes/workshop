## Get Docker Context

```bash
docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                     NAMES
902caca1fb09        cassandra:2.1.22    "docker-entrypoint.s…"   About an hour ago   Up About an hour    7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9242->9042/tcp                 node3
a8a905478b99        cassandra:2.1.22    "docker-entrypoint.s…"   About an hour ago   Up About an hour    7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9142->9042/tcp                 node2
ddb2fe34736c        cassandra:2.1.22    "docker-entrypoint.s…"   About an hour ago   Up About an hour    0.0.0.0:7199->7199/tcp, 7000-7001/tcp, 9160/tcp, 0.0.0.0:9042->9042/tcp   node1
```
## Launching OSS Cassandra

```bash
sudo docker-compose -f cassandra-docker-compose.yaml up &
```

## Launching DSE

Launch cassandra instances:

Launch the first node (seed one)

```bash
sudo docker-compose -f up -d --scale node=0
```

wait 3 minutes

Launch the second node

```bash
sudo docker-compose -f up -d --scale node=1
```

Launch the third node

```bash
sudo docker-compose -f up -d --scale node=1
```

Reproduce same steps if you mant more nodes, increasing the --scale node value

## Clear Docker Context

stop all containers :

```bash
docker container stop $(docker container ls -aq)
```

remove them :

```bash
docker container rm $(docker container ls -aq)
```

## Get container information

```bash
docker inspect node1
```
NetworkSettings -> Networks -> IPAddress

```bash
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress }}{{end}}' node1
```

To get all containers ip adresses

```bash
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```

## Get Cassandra cluster topology

```bash
docker exec node1 nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.31.0.4  72.83 KB   8       58.1%             606d71dc-8309-44de-bdbe-e2600d05caaf  rack1
UN  172.31.0.3  56.87 KB   8       64.8%             5775764c-9c05-4105-a0fb-3fa83940ed0e  rack1
UN  172.31.0.2  46.79 KB   8       77.1%             5164ff1c-29e5-4a3b-8882-5858701d3ec2  rack1
```

Using shell

```bash
docker exec -it node1 bash
root@ddb2fe34736c:/# nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.31.0.4  72.83 KB   8       58.1%             606d71dc-8309-44de-bdbe-e2600d05caaf  rack1
UN  172.31.0.3  56.87 KB   8       64.8%             5775764c-9c05-4105-a0fb-3fa83940ed0e  rack1
UN  172.31.0.2  46.79 KB   8       77.1%             5164ff1c-29e5-4a3b-8882-5858701d3ec2  rack1
```
