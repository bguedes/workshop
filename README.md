# workshop



```yml

version: '3'
services:

   node1:
    image: cassandra:2.1.22
    container_name: node1
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 60; fi && /docker-entrypoint.sh cassandra -f'
    environment:
     - JVM_EXTRA_OPTS=-Xms1g -Xmx2g
     - CASSANDRA_NUM_TOKENS=8
    ports:
     - 9042:9042
     - 7199:7199
    ulimits:
        memlock: -1
        nproc: 32768
        nofile: 100000

   node2:
    image: cassandra:2.1.22
    container_name: node2
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 60; fi && /docker-entrypoint.sh cassandra -f'
    links:
     - node1:seed
    environment:
     - CASSANDRA_SEEDS=seed
     - JVM_EXTRA_OPTS=-Xms1g -Xmx2g
     - CASSANDRA_NUM_TOKENS=8
    ports:
     - 9142:9042
    ulimits:
        memlock: -1
        nproc: 32768
        nofile: 100000

   node3:
    image: cassandra:2.1.22
    container_name: node3
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 120; fi && /docker-entrypoint.sh cassandra -f'    
    links:
     - node1:seed
    environment:
     - CASSANDRA_SEEDS=seed
     - JVM_EXTRA_OPTS=-Xms1g -Xmx2g
     - CASSANDRA_NUM_TOKENS=8
    ports:
     - 9242:9042
    ulimits:
        memlock: -1
        nproc: 32768
        nofile: 100000


```
