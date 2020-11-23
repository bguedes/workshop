## Clear Docker Context

stop all containers :

```bash
docker container stop $(docker container ls -aq)
```

remove them :

```bash
docker container rm $(docker container ls -aq)
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
