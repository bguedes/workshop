## Docker Initialisation

```bash
docker-compose -f cassandra-docker-compose.yaml up -d
```

Wait few seconds then check if the cluster is up :

```bash
nodetool status

docker exec mygithub_seed_node_1 nodetool status
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving/Stopped
--  Address     Load       Owns (effective)  Host ID                               Token                                    Rack
UN  172.31.0.2  137.52 KiB  100.0%            d5813810-4f86-497c-9b80-56ace04cbe8e  -179757114937392923                      rack1


```

go to http://localhost:9091/
