## Docker Initialisation

```bash
docker-compose -f cassandra-docker-compose.yaml up -d
```

Wait few seconds then check if the cluster is up :

```bash
nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.31.0.4  140.63 KB  8       58.1%             606d71dc-8309-44de-bdbe-e2600d05caaf  rack1
UN  172.31.0.3  140.81 KB  8       64.8%             5775764c-9c05-4105-a0fb-3fa83940ed0e  rack1
UN  172.31.0.2  144.27 KB  8       77.1%             5164ff1c-29e5-4a3b-8882-5858701d3ec2  rack
```

## Context Initialisation

from project root :

```bash

cd  /home/cql

docker cp cql node1:home/cql
docker exec -it node1 bash

cqlsh

source 'stockwatcher.cql';
```

## Single Partition Key

### Keyspace Stockwatcher

```sql
CREATE KEYSPACE StockWatcher WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': '2'
};
```

### User Table definition


```sql
CREATE TABLE stockwatcher.User (
  user_id TIMEUUID,
  first_name VARCHAR,
  last_name VARCHAR,
  display_name VARCHAR,
  email_address VARCHAR,
  postal_code VARCHAR,
  active BOOLEAN,
  updated TIMESTAMP,
  PRIMARY KEY (user_id)
);
```

#### Insert values (sample)

```sql
INSERT INTO stockwatcher.User (user_id, 
first_name, 
last_name, 
display_name, 
email_address, 
postal_code, 
active, 
updated) VALUES (dfdf8000-d6b4-11e2-992a-238715b9803d, 
'Eldrick', 
'Woods', 
'Tiger Woods', 
'user01@stockwatcher.org', 
'33455', 
true, 
dateOf(now()));
```

**PRIMARY KEY** = ***Partition Key*** (**mandatory**) + ***Clustering Columns*** (Optional)

For User Table only 1 column is defining the Primary Key then :

Primary Key = Partition Key

On the Insert Example Primary Key = **Partition Key** = **user_id** column



#### Select

```sql
use stockwatcher;


select * from user limit 2;

 user_id                              | active | display_name               | email_address           | first_name | last_name  | postal_code | updated
--------------------------------------+--------+----------------------------+-------------------------+------------+------------+-------------+--------------------------
 dfdf8009-d6b4-11e2-992a-238715b9803d |   True | Two-Time Super Bowl Winner | user10@stockwatcher.org |        Eli |    Manning |       07073 | 2020-11-23 21:59:58+0000
 dfdf8005-d6b4-11e2-992a-238715b9803d |   True |       Friend with Benefits | user06@stockwatcher.org |     Justin | Timberlake |       90210 | 2020-11-23 21:59:58+0000


```

with Expand on :

```sql
select * from user limit 2;

expand on;


@ Row 1
---------------+--------------------------------------
 user_id       | dfdf8009-d6b4-11e2-992a-238715b9803d
 active        | True
 display_name  | Two-Time Super Bowl Winner
 email_address | user10@stockwatcher.org
 first_name    | Eli
 last_name     | Manning
 postal_code   | 07073
 updated       | 2020-11-23 21:59:58+0000

@ Row 2
---------------+--------------------------------------
 user_id       | dfdf8005-d6b4-11e2-992a-238715b9803d
 active        | True
 display_name  | Friend with Benefits
 email_address | user06@stockwatcher.org
 first_name    | Justin
 last_name     | Timberlake
 postal_code   | 90210
 updated       | 2020-11-23 21:59:58+0000

```

##### Get Token value

```sql
select user_id, token(user_id) from user where user_id=dfdf8009-d6b4-11e2-992a-238715b9803d;

 user_id                              | token(user_id)
--------------------------------------+----------------------
 dfdf8009-d6b4-11e2-992a-238715b9803d | -6329350173379368567

```

Hash of the Murmur3 for **user_id** value **dfdf8009-d6b4-11e2-992a-238715b9803d** = ***token*** **-6329350173379368567**

##### Get Token nodes list

exit from CQLSH :

```sql
cqlsh:stockwatcher> exit;
```


```bash
nodetool getendpoints stockwatcher user dfdf8009-d6b4-11e2-992a-238715b9803d;

172.31.0.2
172.31.0.3

```

The token -6329350173379368567 the Murmur3 hash result of user_id dfdf8009-d6b4-11e2-992:a-238715b9803d is managed by 2 replicas :

172.31.0.2 (node1) 
172.31.0.3 (node3)


[Cassandra Parameters for Dummies](https://github.com/jalkanen/cassandracalculator) 


```bash
nodetool describering stockwatcher

Schema Version:346d43b0-ad4c-3316-ba45-9866e9383d29
TokenRange: 
	TokenRange(start_token:8617475369353249286, end_token:-9034880737563150698, endpoints:[172.31.0.4, 172.31.0.2], rpc_endpoints:[172.31.0.4, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-1380579787098828579, end_token:-1031267560278126039, endpoints:[172.31.0.2, 172.31.0.3], rpc_endpoints:[172.31.0.2, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-5449531261285041286, end_token:-5402663575363398807, endpoints:[172.31.0.3, 172.31.0.2], rpc_endpoints:[172.31.0.3, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-3026872043942537809, end_token:-2958726572654972884, endpoints:[172.31.0.3, 172.31.0.2], rpc_endpoints:[172.31.0.3, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-2366466135250000685, end_token:-1832545502013913081, endpoints:[172.31.0.3, 172.31.0.2], rpc_endpoints:[172.31.0.3, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:1274045411687906698, end_token:1646031069633593387, endpoints:[172.31.0.2, 172.31.0.3], rpc_endpoints:[172.31.0.2, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:4165917494782097329, end_token:5992581923948680702, endpoints:[172.31.0.4, 172.31.0.2], rpc_endpoints:[172.31.0.4, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-9034880737563150698, end_token:-8450636422565845295, endpoints:[172.31.0.4, 172.31.0.2], rpc_endpoints:[172.31.0.4, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-1832545502013913081, end_token:-1380579787098828579, endpoints:[172.31.0.2, 172.31.0.3], rpc_endpoints:[172.31.0.2, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:6518288310492567995, end_token:8617475369353249286, endpoints:[172.31.0.2, 172.31.0.4], rpc_endpoints:[172.31.0.2, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:3288351140465726615, end_token:3764651565730598595, endpoints:[172.31.0.3, 172.31.0.4], rpc_endpoints:[172.31.0.3, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:3764651565730598595, end_token:4165917494782097329, endpoints:[172.31.0.3, 172.31.0.4], rpc_endpoints:[172.31.0.3, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-4468886577494593637, end_token:-3071802124097626956, endpoints:[172.31.0.3, 172.31.0.4], rpc_endpoints:[172.31.0.3, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:2581814879857704256, end_token:2646094359852264245, endpoints:[172.31.0.4, 172.31.0.3], rpc_endpoints:[172.31.0.4, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:6156179155924644709, end_token:6518288310492567995, endpoints:[172.31.0.2, 172.31.0.4], rpc_endpoints:[172.31.0.2, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:5992581923948680702, end_token:6156179155924644709, endpoints:[172.31.0.4, 172.31.0.2], rpc_endpoints:[172.31.0.4, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-2958726572654972884, end_token:-2366466135250000685, endpoints:[172.31.0.2, 172.31.0.3], rpc_endpoints:[172.31.0.2, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-8450636422565845295, end_token:-5449531261285041286, endpoints:[172.31.0.2, 172.31.0.3], rpc_endpoints:[172.31.0.2, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:2646094359852264245, end_token:3288351140465726615, endpoints:[172.31.0.4, 172.31.0.3], rpc_endpoints:[172.31.0.4, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-4731131488596673626, end_token:-4468886577494593637, endpoints:[172.31.0.4, 172.31.0.3], rpc_endpoints:[172.31.0.4, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-1031267560278126039, end_token:1274045411687906698, endpoints:[172.31.0.3, 172.31.0.2], rpc_endpoints:[172.31.0.3, 172.31.0.2], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:1646031069633593387, end_token:2581814879857704256, endpoints:[172.31.0.3, 172.31.0.4], rpc_endpoints:[172.31.0.3, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-5402663575363398807, end_token:-4731131488596673626, endpoints:[172.31.0.2, 172.31.0.4], rpc_endpoints:[172.31.0.2, 172.31.0.4], endpoint_details:[EndpointDetails(host:172.31.0.2, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1)])
	TokenRange(start_token:-3071802124097626956, end_token:-3026872043942537809, endpoints:[172.31.0.4, 172.31.0.3], rpc_endpoints:[172.31.0.4, 172.31.0.3], endpoint_details:[EndpointDetails(host:172.31.0.4, datacenter:datacenter1, rack:rack1), EndpointDetails(host:172.31.0.3, datacenter:datacenter1, rack:rack1)])
```

There is a range of token handled per node :


```bash
nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.31.0.4  140.63 KB  8       58.1%             606d71dc-8309-44de-bdbe-e2600d05caaf  rack1
UN  172.31.0.3  140.81 KB  8       64.8%             5775764c-9c05-4105-a0fb-3fa83940ed0e  rack1
UN  172.31.0.2  144.27 KB  8       77.1%             5164ff1c-29e5-4a3b-8882-5858701d3ec2  rack
```

We can see that 8 token ranges are handled per node.
You can verify it checking the list of token ranges given by **nodetool describering stockwatcher** 

For each keyspace, you will get another token range.

#### SSTable for User

By default SSTables are located to **/var/lib/cassandra/data** , our **keyspace** is named **stockwatcher** :

```bash
ls -la /var/lib/cassandra/data/stockwatcher/
total 64
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 .
drwxr-xr-x  5 cassandra cassandra 4096 Nov 23 21:59 ..
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 applicationproperty-2e84b6402dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 dailysummary-2f1e5e302dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 exchange-2fbb13602dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 industry-3057a1802dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 sector-30ee63402dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 stock-318e73d02dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 stockcommentbysymbol-322c88902dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 stockcommentbyuser-32c657902dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 stockcount-3365f2f02dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 stocksearch-33e8b7802dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 trade-3480d8d02dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 user-351ca3a02dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 watchlist-35b70ee02dd711eb9fe4112a71a722b7
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 watchlistitem-36467da02dd711eb9fe4112a71a722b7
root@ddb2fe34736c:/# 

```

We can find here all the folders for each Tables located to the keyspace.

Let's focus on User Tables :

```bash
ls -la /var/lib/cassandra/data/stockwatcher/user-351ca3a02dd711eb9fe4112a71a722b7
total 8
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 .
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 ..
```

There is no SSTables ??!! Wy ? Datas are in the MemTable,on memory

Let's Flush the Memtable, it's automated but we have not inserted enought data to initiate a Memtable flush.

Let's do it by hand :

```bash
nodetool flush stockwatcher user
```

Now we can check our SSTable


```bash
total 44
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 22:56 .
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 ..
-rw-r--r--  1 cassandra cassandra   43 Nov 23 22:56 stockwatcher-user-ka-1-CompressionInfo.db
-rw-r--r--  1 cassandra cassandra  757 Nov 23 22:56 stockwatcher-user-ka-1-Data.db
-rw-r--r--  1 cassandra cassandra   10 Nov 23 22:56 stockwatcher-user-ka-1-Digest.sha1
-rw-r--r--  1 cassandra cassandra   24 Nov 23 22:56 stockwatcher-user-ka-1-Filter.db
-rw-r--r--  1 cassandra cassandra  180 Nov 23 22:56 stockwatcher-user-ka-1-Index.db
-rw-r--r--  1 cassandra cassandra 4452 Nov 23 22:56 stockwatcher-user-ka-1-Statistics.db
-rw-r--r--  1 cassandra cassandra  128 Nov 23 22:56 stockwatcher-user-ka-1-Summary.db
-rw-r--r--  1 cassandra cassandra   91 Nov 23 22:56 stockwatcher-user-ka-1-TOC.txt
```

#### User Table data structure

We are going to use a deprecated cassandra client, it has been since Cassandra 2.2.x version.
Also the SSTable structure has been enhanced in Cµ version 3.0, for more details you can read this blog from TLP :

[Introduction To The Apache Cassandra 3.x Storage Engine](https://thelastpickle.com/blog/2016/03/04/introductiont-to-the-apache-cassandra-3-storage-engine.html) 

But for understanding the underlying structure of data storage (SSTable), by experience, it has been very useful to use it to explain how things are stored.
With Docker-Compose the Cassandra version 2.1.22 is used that's why,we are going to use the old-deleted cassandra-cli :

```bash
cassandra-cli
Connected to: "Test Cluster" on 127.0.0.1/9160
Welcome to Cassandra CLI version 2.1.22

The CLI is deprecated and will be removed in Cassandra 2.2.  Consider migrating to cqlsh.
CQL is fully backwards compatible with Thrift data; see http://www.datastax.com/dev/blog/thrift-to-cql3

Type 'help;' or '?' for help.
Type 'quit;' or 'exit;' to quit.

[default@unknown] 

```

```bash
[default@unknown] use stockwatcher;
Authenticated to keyspace: stockwatcher
[default@stockwatcher] get user[dfdf8009-d6b4-11e2-992a-238715b9803d];
=> (name=, value=, timestamp=1606168798066782)
=> (name=active, value=01, timestamp=1606168798066782)
=> (name=display_name, value=54776f2d54696d6520537570657220426f776c2057696e6e6572, timestamp=1606168798066782)
=> (name=email_address, value=7573657231304073746f636b776174636865722e6f7267, timestamp=1606168798066782)
=> (name=first_name, value=456c69, timestamp=1606168798066782)
=> (name=last_name, value=4d616e6e696e67, timestamp=1606168798066782)
=> (name=postal_code, value=3037303733, timestamp=1606168798066782)
=> (name=updated, value=00000175f71ee373, timestamp=1606168798066782)
Returned 8 results.
Elapsed time: 29 msec(s).
[default@stockwatcher] 
```

To convert Hex to Text :

[Convert hexadecimal to text](http://www.unit-conversion.info/texttools/hexadecimal/) 


There is another way to read a SSTable :

```bash
/opt/cassandra/tools/bin/sstable2json stockwatcher-user-ka-1-Data.db

[
{"key": "dfdf8009-d6b4-11e2-992a-238715b9803d",
 "cells": [["","",1606168798066782],
           ["active","true",1606168798066782],
           ["display_name","Two-Time Super Bowl Winner",1606168798066782],
           ["email_address","user10@stockwatcher.org",1606168798066782],
           ["first_name","Eli",1606168798066782],
           ["last_name","Manning",1606168798066782],
           ["postal_code","07073",1606168798066782],
           ["updated","2020-11-23 21:59Z",1606168798066782]]},
{"key": "dfdf8008-d6b4-11e2-992a-238715b9803d",
 "cells": [["","",1606168798064791],
           ["active","true",1606168798064791],
           ["display_name","Two-Time World Series Winner",1606168798064791],
           ["email_address","user09@stockwatcher.org",1606168798064791],
           ["first_name","Buster",1606168798064791],
           ["last_name","Posey",1606168798064791],
           ["postal_code","94107",1606168798064791],
           ["updated","2020-11-23 21:59Z",1606168798064791]]},
{"key": "dfdf8001-d6b4-11e2-992a-238715b9803d",
 "cells": [["","",1606168798049069],
           ["active","true",1606168798049069],
           ["display_name","The Ex-Governator",1606168798049069],
           ["email_address","user02@stockwatcher.org",1606168798049069],
           ["first_name","Arnold",1606168798049069],
           ["last_name","Schwarzenegger",1606168798049069],
           ["postal_code","90049",1606168798049069],
           ["updated","2020-11-23 21:59Z",1606168798049069]]},
{"key": "dfdf8000-d6b4-11e2-992a-238715b9803d",
 "cells": [["","",1606168798046892],
           ["active","true",1606168798046892],
           ["display_name","Tiger Woods",1606168798046892],
           ["email_address","user01@stockwatcher.org",1606168798046892],
           ["first_name","Eldrick",1606168798046892],
           ["last_name","Woods",1606168798046892],
           ["postal_code","33455",1606168798046892],
           ["updated","2020-11-23 21:59Z",1606168798046892]]},
{"key": "dfdf8006-d6b4-11e2-992a-238715b9803d",
 "cells": [["","",1606168798060548],
           ["active","true",1606168798060548],
           ["display_name","Child of Destiny",1606168798060548],
           ["email_address","user07@stockwatcher.org",1606168798060548],
           ["first_name","Beyonce",1606168798060548],
           ["last_name","Knowles",1606168798060548],
           ["postal_code","10001",1606168798060548],
           ["updated","2020-11-23 21:59Z",1606168798060548]]},
{"key": "dfdf8007-d6b4-11e2-992a-238715b9803d",
 "cells": [["","",1606168798062697],
           ["active","true",1606168798062697],
           ["display_name","King James",1606168798062697],
           ["email_address","user08@stockwatcher.org",1606168798062697],
           ["first_name","Lebron",1606168798062697],
           ["last_name","James",1606168798062697],
           ["postal_code","33141",1606168798062697],
           ["updated","2020-11-23 21:59Z",1606168798062697]]}
]
```

Let's update for user_id dfdf8009-d6b4-11e2-992a-238715b9803d the firstname and lastname :

```bash
select * from user where user_id=dfdf8009-d6b4-11e2-992a-238715b9803d;

 user_id                              | active | display_name               | email_address           | first_name | last_name | postal_code | updated
--------------------------------------+--------+----------------------------+-------------------------+------------+-----------+-------------+--------------------------
 dfdf8009-d6b4-11e2-992a-238715b9803d |   True | Two-Time Super Bowl Winner | user10@stockwatcher.org |        Eli |   Manning |       07073 | 2020-11-23 21:59:58+0000

```

Let's do an UPDATE :


```bash
update user set first_name='Foo', last_name='Bar' where user_id=dfdf8009-d6b4-11e2-992a-238715b9803d;
```

```bash
select * from user where user_id=dfdf8009-d6b4-11e2-992a-238715b9803d;

user_id                              | active | display_name               | email_address           | first_name | last_name | postal_code | updated
--------------------------------------+--------+----------------------------+-------------------------+------------+-----------+-------------+--------------------------
 dfdf8009-d6b4-11e2-992a-238715b9803d |   True | Two-Time Super Bowl Winner | user10@stockwatcher.org |        Foo |       Bar |       07073 | 2020-11-23 21:59:58+0000

```
ow let's go to one of the replicas handling this partition :

```bash
nodetool getendpoints stockwatcher user dfdf8009-d6b4-11e2-992a-238715b9803d;

172.31.0.2
172.31.0.3

```
```bash
docker exec -it node1 bash
```

Let's flush the table User :

```bash
nodetool flush stockwatcher user
```
Get a closer look to User data folder :

```bash
cd /var/lib/cassandra/data/stockwatcher/user-351ca3a02dd711eb9fe4112a71a722b7

ls -la
total 80
drwxr-xr-x  2 cassandra cassandra 4096 Nov 24 10:56 .
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 ..
-rw-r--r--  1 cassandra cassandra   43 Nov 23 22:56 stockwatcher-user-ka-1-CompressionInfo.db
-rw-r--r--  1 cassandra cassandra  757 Nov 23 22:56 stockwatcher-user-ka-1-Data.db
-rw-r--r--  1 cassandra cassandra   10 Nov 23 22:56 stockwatcher-user-ka-1-Digest.sha1
-rw-r--r--  1 cassandra cassandra   24 Nov 23 22:56 stockwatcher-user-ka-1-Filter.db
-rw-r--r--  1 cassandra cassandra  180 Nov 23 22:56 stockwatcher-user-ka-1-Index.db
-rw-r--r--  1 cassandra cassandra 4452 Nov 23 22:56 stockwatcher-user-ka-1-Statistics.db
-rw-r--r--  1 cassandra cassandra  128 Nov 23 22:56 stockwatcher-user-ka-1-Summary.db
-rw-r--r--  1 cassandra cassandra   91 Nov 23 22:56 stockwatcher-user-ka-1-TOC.txt
-rw-r--r--  1 cassandra cassandra   43 Nov 24 10:56 stockwatcher-user-ka-2-CompressionInfo.db
-rw-r--r--  1 cassandra cassandra   83 Nov 24 10:56 stockwatcher-user-ka-2-Data.db
-rw-r--r--  1 cassandra cassandra   10 Nov 24 10:56 stockwatcher-user-ka-2-Digest.sha1
-rw-r--r--  1 cassandra cassandra   16 Nov 24 10:56 stockwatcher-user-ka-2-Filter.db
-rw-r--r--  1 cassandra cassandra   30 Nov 24 10:56 stockwatcher-user-ka-2-Index.db
-rw-r--r--  1 cassandra cassandra 4434 Nov 24 10:56 stockwatcher-user-ka-2-Statistics.db
-rw-r--r--  1 cassandra cassandra  128 Nov 24 10:56 stockwatcher-user-ka-2-Summary.db
-rw-r--r--  1 cassandra cassandra   91 Nov 24 10:56 stockwatcher-user-ka-2-TOC.txt

```

We can see another new SSTable :

-rw-r--r--  1 cassandra cassandra   83 Nov 24 10:56 stockwatcher-user-ka-2-Data.db

Let's check is content :

```bash
/opt/cassandra/tools/bin/sstable2json stockwatcher-user-ka-2-Data.db

[
{"key": "dfdf8009-d6b4-11e2-992a-238715b9803d",
 "cells": [["first_name","Foo",1606215186711437],
           ["last_name","Bar",1606215186711437]]}
]
```

As we can see only the colums updated are persisted, that's the **power** of **Cassandra** to be **Apend Only**

During the READ PATH C* will merge these 2 SSTables to get the updated view of this partition using last timestamp for each columns.
This explaination will describe in much more details in another chapter.


## Single Partition Key with Clustering Column

### User StockCommentBySymbol definition

```sql
CREATE TABLE stockwatcher.StockCommentBySymbol (
  stock_symbol VARCHAR,
  comment_id TIMEUUID,
  company_name VARCHAR,
  user_id TIMEUUID,
  user_display_name VARCHAR,
  comment VARCHAR,
  active BOOLEAN,
  PRIMARY KEY (stock_symbol, comment_id)
) WITH CLUSTERING ORDER BY (comment_id DESC);
```

The Primary Key is composed by :
1 Partition Key : column stock_symbol
1 Clustering Column : colmn comment_id

```sql
select * from StockCommentBySymbol limit 5;

 stock_symbol | comment_id                           | active | comment                                                          | company_name | user_display_name     | user_id
--------------+--------------------------------------+--------+------------------------------------------------------------------+--------------+-----------------------+--------------------------------------
          BGS | 39fcf5f0-2dd7-11eb-9fe4-112a71a722b7 |   True |                                 This stock is poised to explode! |         null |       Sister of Venus | dfdf8002-d6b4-11e2-992a-238715b9803d
         CBEY | 39ff18d0-2dd7-11eb-9fe4-112a71a722b7 |   True |                           This stock will eventually make money. |         null |        Doctor Liberty | dfdf8003-d6b4-11e2-992a-238715b9803d
         HOLL | 3a0162c0-2dd7-11eb-9fe4-112a71a722b7 |   True |                      I would suggest a buy and hold on this one. |         null |  Friend with Benefits | dfdf8005-d6b4-11e2-992a-238715b9803d
          EXC | 3a009f70-2dd7-11eb-9fe4-112a71a722b7 |   True |                                           Could be a big winner. |         null | Wizard of Wall Street | dfdf8004-d6b4-11e2-992a-238715b9803d
         DGLY | 3a024d20-2dd7-11eb-9fe4-112a71a722b7 |   True | Huge risk on this stock but also huge potential if they survive. |         null |      Child of Destiny | dfdf8006-d6b4-11e2-992a-238715b9803d

(5 rows)
```
```sql
cqlsh:stockwatcher> expand on;
Now Expanded output is enabled
cqlsh:stockwatcher> select * from StockCommentBySymbol limit 5;

@ Row 1
-------------------+------------------------------------------------------------------
 stock_symbol      | BGS
 comment_id        | 39fcf5f0-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | This stock is poised to explode!
 company_name      | null
 user_display_name | Sister of Venus
 user_id           | dfdf8002-d6b4-11e2-992a-238715b9803d

@ Row 2
-------------------+------------------------------------------------------------------
 stock_symbol      | CBEY
 comment_id        | 39ff18d0-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | This stock will eventually make money.
 company_name      | null
 user_display_name | Doctor Liberty
 user_id           | dfdf8003-d6b4-11e2-992a-238715b9803d

@ Row 3
-------------------+------------------------------------------------------------------
 stock_symbol      | HOLL
 comment_id        | 3a0162c0-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | I would suggest a buy and hold on this one.
 company_name      | null
 user_display_name | Friend with Benefits
 user_id           | dfdf8005-d6b4-11e2-992a-238715b9803d

@ Row 4
-------------------+------------------------------------------------------------------
 stock_symbol      | EXC
 comment_id        | 3a009f70-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | Could be a big winner.
 company_name      | null
 user_display_name | Wizard of Wall Street
 user_id           | dfdf8004-d6b4-11e2-992a-238715b9803d

@ Row 5
-------------------+------------------------------------------------------------------
 stock_symbol      | DGLY
 comment_id        | 3a024d20-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | Huge risk on this stock but also huge potential if they survive.
 company_name      | null
 user_display_name | Child of Destiny
 user_id           | dfdf8006-d6b4-11e2-992a-238715b9803d

(5 rows)
cqlsh:stockwatcher> 
```

```sql
cqlsh:stockwatcher> select stock_symbol, token(stock_symbol) from stockcommentbysymbol;

 stock_symbol | token(stock_symbol)
--------------+----------------------
          BGS | -9165165945071193398
         CBEY | -8347839709842205291
         HOLL | -8081613047217837851
          EXC | -7307067873798146258
         DGLY | -6538546626543511127
         NSSC | -4911858418781863771
          HRS | -3816466130909351153
         HCOM | -3649907427753692401
         AAPL | -3367223219348229195
         AAPL | -3367223219348229195
          HYF | -1433544300892360898
         FLIC | -1071947717604030790
         CJJD |  -886593793177581812
         ATRI |   450790228736567609
          TKR |   854399503562793201
         EPAM |  1322611372050383607
          CVR |  1888482621889768080
         CMRE |  2129807583828427104
         EDAP |  2194322523214221990
         SNCR |  2680266242262493798
          PZN |  3381123392269919738
         FISI |  4042823051114111921
          BHD |  5237421755070733502
         GEVA |  5584622203549135151
         LSTR |  6169388439398625265
         INFI |  6277815532313351799
         GLUU |  7198976408531766212
         OGEN |  7254698907632504471
          THM |  7489212158800975073
          HHC |  8433409320413452965
          EFT |  9090389204915228265

(31 rows)
cqlsh:stockwatcher> 
```

Let's focus on the stock for Apple (stock_symbol=AAPL)

```sql
select * from stockcommentbysymbol where stock_symbol='AAPL';

@ Row 1
-------------------+--------------------------------------------------
 stock_symbol      | AAPL
 comment_id        | 3a0644c0-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | Still a great company that will continue to win.
 company_name      | null
 user_display_name | Two-Time Super Bowl Winner
 user_id           | dfdf8009-d6b4-11e2-992a-238715b9803d
```

Let's do an Insert :

```sql
INSERT INTO stockwatcher.stockcommentbysymbol (stock_symbol, comment_id, user_id, user_display_name, active, comment) VALUES ('AAPL', now(), dfdf8009-d6b4-11e2-992a-238715b9803d, 'Two-Time Super Bowl Winner', true, 'Just an Update to play with.');
```

Check the result :

```sql
cqlsh:stockwatcher> select * from stockcommentbysymbol where stock_symbol='AAPL';

@ Row 1
-------------------+--------------------------------------------------
 stock_symbol      | AAPL
 comment_id        | 081f3230-2e47-11eb-9fe4-112a71a722b7
 active            | True
 comment           | Just an Update to play with.
 company_name      | null
 user_display_name | Two-Time Super Bowl Winner
 user_id           | dfdf8009-d6b4-11e2-992a-238715b9803d

@ Row 2
-------------------+--------------------------------------------------
 stock_symbol      | AAPL
 comment_id        | 3a0644c0-2dd7-11eb-9fe4-112a71a722b7
 active            | True
 comment           | Still a great company that will continue to win.
 company_name      | null
 user_display_name | Two-Time Super Bowl Winner
 user_id           | dfdf8009-d6b4-11e2-992a-238715b9803d

(2 rows)
cqlsh:stockwatcher>
```

Nice and simple, but what about the Partition ?
Now we are going to see what does it mean a '**PARTITION**'

Let's give a look with old casandra-cli :

```bash
root@ddb2fe34736c:/# cassandra-cli
Column Family assumptions read from /root/.cassandra/assumptions.json
Connected to: "Test Cluster" on 127.0.0.1/9160
Welcome to Cassandra CLI version 2.1.22

The CLI is deprecated and will be removed in Cassandra 2.2.  Consider migrating to cqlsh.
CQL is fully backwards compatible with Thrift data; see http://www.datastax.com/dev/blog/thrift-to-cql3

Type 'help;' or '?' for help.
Type 'quit;' or 'exit;' to quit.

[default@unknown] use stockwatcher;
Authenticated to keyspace: stockwatcher
[default@stockwatcher] 
```

```bash
get stockcommentbysymbol['AAPL'];
=> (name=081f3230-2e47-11eb-9fe4-112a71a722b7:, value=, timestamp=1606216817875072)
=> (name=081f3230-2e47-11eb-9fe4-112a71a722b7:active, value=01, timestamp=1606216817875072)
=> (name=081f3230-2e47-11eb-9fe4-112a71a722b7:comment, value=4a75737420616e2055706461746520746f20706c617920776974682e, timestamp=1606216817875072)
=> (name=081f3230-2e47-11eb-9fe4-112a71a722b7:user_display_name, value=54776f2d54696d6520537570657220426f776c2057696e6e6572, timestamp=1606216817875072)
=> (name=081f3230-2e47-11eb-9fe4-112a71a722b7:user_id, value=dfdf8009d6b411e2992a238715b9803d, timestamp=1606216817875072)
=> (name=3a0644c0-2dd7-11eb-9fe4-112a71a722b7:, value=, timestamp=1606168797963698)
=> (name=3a0644c0-2dd7-11eb-9fe4-112a71a722b7:active, value=01, timestamp=1606168797963698)
=> (name=3a0644c0-2dd7-11eb-9fe4-112a71a722b7:comment, value=5374696c6c206120677265617420636f6d70616e7920746861742077696c6c20636f6e74696e756520746f2077696e2e, timestamp=1606168797963698)
=> (name=3a0644c0-2dd7-11eb-9fe4-112a71a722b7:user_display_name, value=54776f2d54696d6520537570657220426f776c2057696e6e6572, timestamp=1606168797963698)
=> (name=3a0644c0-2dd7-11eb-9fe4-112a71a722b7:user_id, value=dfdf8009d6b411e2992a238715b9803d, timestamp=1606168797963698)
Returned 10 results.
Elapsed time: 31 msec(s).
```

Let's see in a Spreasheet this result given by cassandra-cli :

[Partition for stock symbol AAPL](https://docs.google.com/spreadsheets/d/116tjYH1oTok__MbX0XxbK-XV0hH2A_C17JWu4Hu069I/edit#gid=0) 





Question : Where is located this Partition 

Response :

```bash
nodetool getendpoints stockwatcher  'AAPL';

172.31.0.3
172.31.0.4
```

```bash
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)

/node3 - 172.31.0.3
/node2 - 172.31.0.4
/node1 - 172.31.0.2
```
Partition is located in Node3 and Node2

```bash
docker exec -it node2 bash
```

Get data folder to access stockcommentbysymbol folder data :

```bash
cd /var/lib/cassandra/data/stockwatcher/stockcommentbysymbol*
```

```bash	
root@a8a905478b99:/var/lib/cassandra/data/stockwatcher/stockcommentbysymbol-322c88902dd711eb9fe4112a71a722b7# ls -la
total 8
drwxr-xr-x  2 cassandra cassandra 4096 Nov 23 21:59 .
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 ..
```

No SSTables yet, let's flush everything for table stockcommentbysymbol :

```bash
nodetool flush stockwatcher stockcommentbysymbol
```
```bash
root@a8a905478b99:/var/lib/cassandra/data/stockwatcher/stockcommentbysymbol-322c88902dd711eb9fe4112a71a722b7# ls -la
total 44
drwxr-xr-x  2 cassandra cassandra 4096 Nov 24 11:35 .
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 ..
-rw-r--r--  1 cassandra cassandra   43 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-CompressionInfo.db
-rw-r--r--  1 cassandra cassandra 2395 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-Data.db
-rw-r--r--  1 cassandra cassandra   10 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-Digest.sha1
-rw-r--r--  1 cassandra cassandra   40 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-Filter.db
-rw-r--r--  1 cassandra cassandra  352 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-Index.db
-rw-r--r--  1 cassandra cassandra 4538 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-Statistics.db
-rw-r--r--  1 cassandra cassandra   89 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-Summary.db
-rw-r--r--  1 cassandra cassandra   91 Nov 24 11:35 stockwatcher-stockcommentbysymbol-ka-1-TOC.txt
root@a8a905478b99:/var/lib/cassandra/data/stockwatcher/stockcommentbysymbol-322c88902dd711eb9fe4112a71a722b7# 
```
#### User StockSearch data structure

```sql
CREATE TABLE stockwatcher.StockSearch (
  industry_id INT,
  exchange_id VARCHAR,
  stock_symbol VARCHAR,
  PRIMARY KEY (industry_id, exchange_id, stock_symbol)
);
```

```sql
select * from StockSearch where industry_id=769;

 industry_id | exchange_id | stock_symbol
-------------+-------------+--------------
         769 |      NASDAQ |         CRAI
         769 |      NASDAQ |         EXPO
         769 |      NASDAQ |         HPOL
         769 |      NASDAQ |          III
         769 |      NASDAQ |         RECN
         769 |      NASDAQ |         TMNG
         769 |        NYSE |           AH
         769 |        NYSE |          CXW
         769 |        NYSE |          FCN
         769 |        NYSE |          HIL
         769 |        NYSE |          NCI
         769 |        NYSE |           TW
```

```sql
root@ddb2fe34736c:/# cassandra-cli
Column Family assumptions read from /root/.cassandra/assumptions.json
Connected to: "Test Cluster" on 127.0.0.1/9160
Welcome to Cassandra CLI version 2.1.22

The CLI is deprecated and will be removed in Cassandra 2.2.  Consider migrating to cqlsh.
CQL is fully backwards compatible with Thrift data; see http://www.datastax.com/dev/blog/thrift-to-cql3

Type 'help;' or '?' for help.
Type 'quit;' or 'exit;' to quit.

[default@unknown] use stockwatcher;
Authenticated to keyspace: stockwatcher
[default@stockwatcher] get stocksearch[769];
=> (name=NASDAQ:CRAI:, value=, timestamp=1606168804035873)
=> (name=NASDAQ:EXPO:, value=, timestamp=1606168804038626)
=> (name=NASDAQ:HPOL:, value=, timestamp=1606168804038626)
=> (name=NASDAQ:III:, value=, timestamp=1606168804040137)
=> (name=NASDAQ:RECN:, value=, timestamp=1606168804042691)
=> (name=NASDAQ:TMNG:, value=, timestamp=1606168804043237)
=> (name=NYSE:AH:, value=, timestamp=1606168804034349)
=> (name=NYSE:CXW:, value=, timestamp=1606168804037362)
=> (name=NYSE:FCN:, value=, timestamp=1606168804038626)
=> (name=NYSE:HIL:, value=, timestamp=1606168804038626)
=> (name=NYSE:NCI:, value=, timestamp=1606168804041919)
=> (name=NYSE:TW:, value=, timestamp=1606168804043237)
Returned 12 results.
Elapsed time: 21 msec(s).
```



## Composed Partition Key with Clustering Column

#### User Trade data structure

```sql
CREATE TABLE stockwatcher.Trade (
  stock_symbol VARCHAR,
  trade_id TIMEUUID,
  trade_date TIMESTAMP,
  trade_timestamp TIMESTAMP,
  exchange_id VARCHAR,
  share_price DECIMAL,
  share_quantity INT,
  PRIMARY KEY ((stock_symbol, trade_date), trade_timestamp, trade_id)
);
```

The Primary Key is composed by :
1 Composed Partition Key : 2 columns (stock_symbol, trade_date)
2 Clustering Columns : column trade_timestamp and trade_id

Let's insert some trades :

```sql
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NYSE', 'PAA', 52.69, 100, f40cba92-2e73-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NYSE', 'PVA', 4.99, 806, 66fd003a-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NYSE', 'CTL', 40.50, 100, 66fd026a-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NASDAQ', 'AAPL', 501.325, 80, 66fd0364-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NASDAQ', 'FCCY', 10.01, 100, 66fd0440-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NASDAQ', 'PKT', 12.83, 50, 66fd0508-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NYSE', 'CBZ', 7.25, 300, 66fd0724-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 12:59:00', 'NASDAQ', 'FB', 37.155, 100, 66fd07f6-2e7d-11eb-adc1-0242ac120002, '2020-11-24');

INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:01:00', 'NYSE', 'WTI', 30.55, 80, 66fd08b4-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:01:00', 'NYSE', 'QIHU', 24.99, 230, 66fd0972-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:01:00', 'NYSE', 'CTL', 33.69, 100, 66fd0b5c-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:01:00', 'NASDAQ', 'AAPL', 200.325, 150, 66fd0c24-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:01:00', 'NASDAQ', 'BRCD', 40.5, 100, 66fd0ce2-2e7d-11eb-adc1-0242ac120002, '2020-11-24');

INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:02:00', 'NYSE', 'GM', 27.50, 120, 66fd0da0-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:02:00', 'NYSE', 'PVA', 17.90, 210, 66fd0e5e-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:02:00', 'NYSE', 'FSLR', 33.00, 100, 66fd0f1c-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:02:00', 'NASDAQ', 'AAPL', 210.50, 180, 66fd112e-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:02:00', 'NASDAQ', 'FB', 50.10, 90, 66fd11f6-2e7d-11eb-adc1-0242ac120002, '2020-11-24');

INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:03:00', 'NYSE', 'PAA', 33.00, 110, 66fd12b4-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:03:00', 'NYSE', 'APC', 5.00, 140, 66fd1372-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:03:00', 'NYSE', 'CTL', 30.00, 100, 66fd1430-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:03:00', 'NASDAQ', 'AAPL', 240.30, 220, 66fd1692-2e7d-11eb-adc1-0242ac120002, '2020-11-24');
INSERT INTO Trade (trade_timestamp, exchange_id, stock_symbol, share_price, share_quantity, trade_id, trade_date) VALUES ('2020-11-24 13:03:00', 'NASDAQ', 'FB', 66.55, 80, 66fd175a-2e7d-11eb-adc1-0242ac120002, '2020-11-24');

```

Let's get the list of tokens for these rows :


```sql
cqlsh:stockwatcher> select stock_symbol, trade_date, token(stock_symbol, trade_date) from trade;

 stock_symbol | trade_date               | token(stock_symbol, trade_date)
--------------+--------------------------+---------------------------------
          PAA | 2020-11-24 00:00:00+0000 |            -8869188060485178007
          PAA | 2020-11-24 00:00:00+0000 |            -8869188060485178007
         FCCY | 2020-11-24 00:00:00+0000 |            -4643951212538594641
          PKT | 2020-11-24 00:00:00+0000 |            -2272126211363618776
           GM | 2020-11-24 00:00:00+0000 |            -1953701761339545589
         FSLR | 2020-11-24 00:00:00+0000 |             -967105553814637581
          CTL | 2020-11-24 00:00:00+0000 |              542838003986218230
          CTL | 2020-11-24 00:00:00+0000 |              542838003986218230
          PVA | 2020-11-24 00:00:00+0000 |             1537958549549826337
          PVA | 2020-11-24 00:00:00+0000 |             1537958549549826337
         AAPL | 2020-11-24 00:00:00+0000 |             2050644165161903073
         AAPL | 2020-11-24 00:00:00+0000 |             2050644165161903073
         AAPL | 2020-11-24 00:00:00+0000 |             2050644165161903073
         AAPL | 2020-11-24 00:00:00+0000 |             2050644165161903073
          APC | 2020-11-24 00:00:00+0000 |             6013601871958943093
          CBZ | 2020-11-24 00:00:00+0000 |             6052939309865478914
          WTI | 2020-11-24 00:00:00+0000 |             6529726756968815792
         BRCD | 2020-11-24 00:00:00+0000 |             7058500133183191873
           FB | 2020-11-24 00:00:00+0000 |             8747189575396538967
           FB | 2020-11-24 00:00:00+0000 |             8747189575396538967
           FB | 2020-11-24 00:00:00+0000 |             8747189575396538967
         QIHU | 2020-11-24 00:00:00+0000 |             8941685029735615887

(22 rows)
cqlsh:stockwatcher> 
```

Now let's focus in one specific partition :


```sql
select * from trade where stock_symbol='AAPL' and trade_date='2020-11-24 00:00:00+0000';

 stock_symbol | trade_date               | trade_timestamp          | trade_id                             | exchange_id | share_price | share_quantity
--------------+--------------------------+--------------------------+--------------------------------------+-------------+-------------+----------------
         AAPL | 2020-11-24 00:00:00+0000 | 2020-11-24 12:59:00+0000 | 66fd0364-2e7d-11eb-adc1-0242ac120002 |      NASDAQ |     501.325 |             80
         AAPL | 2020-11-24 00:00:00+0000 | 2020-11-24 13:01:00+0000 | 66fd0c24-2e7d-11eb-adc1-0242ac120002 |      NASDAQ |     200.325 |            150
         AAPL | 2020-11-24 00:00:00+0000 | 2020-11-24 13:02:00+0000 | 66fd112e-2e7d-11eb-adc1-0242ac120002 |      NASDAQ |      210.50 |            180
         AAPL | 2020-11-24 00:00:00+0000 | 2020-11-24 13:03:00+0000 | 66fd1692-2e7d-11eb-adc1-0242ac120002 |      NASDAQ |      240.30 |            220

(4 rows)
cqlsh:stockwatcher> 
```

Who are the nodes handling this partition :

```bash
nodetool getendpoints stockwatcher trade 'AAPL:2020-11-24';

172.31.0.3
172.31.0.4
```

```bash
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)

/node3 - 172.31.0.3
/node2 - 172.31.0.4
/node1 - 172.31.0.2
```

Let's go to Node2 :

```bash
docker exec -it node2 bash
```

Inspection of the partition with cassandra-cli :

```bash
root@ddb2fe34736c:/# cassandra-cli
Column Family assumptions read from /root/.cassandra/assumptions.json
Connected to: "Test Cluster" on 127.0.0.1/9160
Welcome to Cassandra CLI version 2.1.22

The CLI is deprecated and will be removed in Cassandra 2.2.  Consider migrating to cqlsh.
CQL is fully backwards compatible with Thrift data; see http://www.datastax.com/dev/blog/thrift-to-cql3

Type 'help;' or '?' for help.
Type 'quit;' or 'exit;' to quit.

[default@unknown] use stockwatcher;
Authenticated to keyspace: stockwatcher
[default@stockwatcher] get trade['AAPL:2020-11-24'];
=> (name=2020-11-24 12\:59Z:66fd0364-2e7d-11eb-adc1-0242ac120002:, value=, timestamp=1606240518544806)
=> (name=2020-11-24 12\:59Z:66fd0364-2e7d-11eb-adc1-0242ac120002:exchange_id, value=4e4153444151, timestamp=1606240518544806)
=> (name=2020-11-24 12\:59Z:66fd0364-2e7d-11eb-adc1-0242ac120002:share_price, value=0000000307a64d, timestamp=1606240518544806)
=> (name=2020-11-24 12\:59Z:66fd0364-2e7d-11eb-adc1-0242ac120002:share_quantity, value=00000050, timestamp=1606240518544806)
=> (name=2020-11-24 13\:01Z:66fd0c24-2e7d-11eb-adc1-0242ac120002:, value=, timestamp=1606240518565107)
=> (name=2020-11-24 13\:01Z:66fd0c24-2e7d-11eb-adc1-0242ac120002:exchange_id, value=4e4153444151, timestamp=1606240518565107)
=> (name=2020-11-24 13\:01Z:66fd0c24-2e7d-11eb-adc1-0242ac120002:share_price, value=00000003030e85, timestamp=1606240518565107)
=> (name=2020-11-24 13\:01Z:66fd0c24-2e7d-11eb-adc1-0242ac120002:share_quantity, value=00000096, timestamp=1606240518565107)
=> (name=2020-11-24 13\:02Z:66fd112e-2e7d-11eb-adc1-0242ac120002:, value=, timestamp=1606240518579595)
=> (name=2020-11-24 13\:02Z:66fd112e-2e7d-11eb-adc1-0242ac120002:exchange_id, value=4e4153444151, timestamp=1606240518579595)
=> (name=2020-11-24 13\:02Z:66fd112e-2e7d-11eb-adc1-0242ac120002:share_price, value=00000002523a, timestamp=1606240518579595)
=> (name=2020-11-24 13\:02Z:66fd112e-2e7d-11eb-adc1-0242ac120002:share_quantity, value=000000b4, timestamp=1606240518579595)
=> (name=2020-11-24 13\:03Z:66fd1692-2e7d-11eb-adc1-0242ac120002:, value=, timestamp=1606240518594905)
=> (name=2020-11-24 13\:03Z:66fd1692-2e7d-11eb-adc1-0242ac120002:exchange_id, value=4e4153444151, timestamp=1606240518594905)
=> (name=2020-11-24 13\:03Z:66fd1692-2e7d-11eb-adc1-0242ac120002:share_price, value=000000025dde, timestamp=1606240518594905)
=> (name=2020-11-24 13\:03Z:66fd1692-2e7d-11eb-adc1-0242ac120002:share_quantity, value=000000dc, timestamp=1606240518594905)
Returned 16 results.
Elapsed time: 32 msec(s).
[default@stockwatcher] 

```

[Partition for trade ](https://docs.google.com/spreadsheets/d/116tjYH1oTok__MbX0XxbK-XV0hH2A_C17JWu4Hu069I/edit#gid=1341960572) 

Exit from cassandra-cli !

Let's see the SSTable on Node2 :

```bash
root@ddb2fe34736c:/# cd /var/lib/cassandra/data/stockwatcher/trade*
root@ddb2fe34736c:/var/lib/cassandra/data/stockwatcher/trade-3480d8d02dd711eb9fe4112a71a722b7#
```

Again flush the Table Trade :

```bash
nodetool flush stockwatcher trade
```

```bash
root@ddb2fe34736c:/var/lib/cassandra/data/stockwatcher/trade-3480d8d02dd711eb9fe4112a71a722b7# ls -la
total 48
drwxr-xr-x  3 cassandra cassandra 4096 Nov 24 18:45 .
drwxr-xr-x 16 cassandra cassandra 4096 Nov 23 21:59 ..
drwxr-xr-x  4 cassandra cassandra 4096 Nov 24 17:55 snapshots
-rw-r--r--  1 cassandra cassandra   43 Nov 24 18:45 stockwatcher-trade-ka-3-CompressionInfo.db
-rw-r--r--  1 cassandra cassandra 1302 Nov 24 18:45 stockwatcher-trade-ka-3-Data.db
-rw-r--r--  1 cassandra cassandra   10 Nov 24 18:45 stockwatcher-trade-ka-3-Digest.sha1
-rw-r--r--  1 cassandra cassandra   32 Nov 24 18:45 stockwatcher-trade-ka-3-Filter.db
-rw-r--r--  1 cassandra cassandra  373 Nov 24 18:45 stockwatcher-trade-ka-3-Index.db
-rw-r--r--  1 cassandra cassandra 4530 Nov 24 18:45 stockwatcher-trade-ka-3-Statistics.db
-rw-r--r--  1 cassandra cassandra  132 Nov 24 18:45 stockwatcher-trade-ka-3-Summary.db
-rw-r--r--  1 cassandra cassandra   91 Nov 24 18:45 stockwatcher-trade-ka-3-TOC.txt
root@ddb2fe34736c:/var/lib/cassandra/data/stockwatcher/trade-3480d8d02dd711eb9fe4112a71a722b7# 
```


I've got a folder 'snapshots' because I've done a truncate of the trade table during testing.
When performing a truncate of a Table, by default, Cassandra snapshot (backup) the table content is case of human error.

```bash
root@ddb2fe34736c:/var/lib/cassandra/data/stockwatcher/trade-3480d8d02dd711eb9fe4112a71a722b7# /opt/cassandra/tools/bin/sstable2json stockwatcher-trade-ka-3-Data.db
[
{"key": "PAA:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 12\\:59Z:f40cba92-2e73-11eb-adc1-0242ac120002:","",1606240518534797],
           ["2020-11-24 12\\:59Z:f40cba92-2e73-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518534797],
           ["2020-11-24 12\\:59Z:f40cba92-2e73-11eb-adc1-0242ac120002:share_price","52.69",1606240518534797],
           ["2020-11-24 12\\:59Z:f40cba92-2e73-11eb-adc1-0242ac120002:share_quantity","100",1606240518534797],
           ["2020-11-24 13\\:03Z:66fd12b4-2e7d-11eb-adc1-0242ac120002:","",1606240518585802],
           ["2020-11-24 13\\:03Z:66fd12b4-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518585802],
           ["2020-11-24 13\\:03Z:66fd12b4-2e7d-11eb-adc1-0242ac120002:share_price","33.00",1606240518585802],
           ["2020-11-24 13\\:03Z:66fd12b4-2e7d-11eb-adc1-0242ac120002:share_quantity","110",1606240518585802]]},
{"key": "PKT:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 12\\:59Z:66fd0508-2e7d-11eb-adc1-0242ac120002:","",1606240518550519],
           ["2020-11-24 12\\:59Z:66fd0508-2e7d-11eb-adc1-0242ac120002:exchange_id","NASDAQ",1606240518550519],
           ["2020-11-24 12\\:59Z:66fd0508-2e7d-11eb-adc1-0242ac120002:share_price","12.83",1606240518550519],
           ["2020-11-24 12\\:59Z:66fd0508-2e7d-11eb-adc1-0242ac120002:share_quantity","50",1606240518550519]]},
{"key": "GM:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 13\\:02Z:66fd0da0-2e7d-11eb-adc1-0242ac120002:","",1606240518572087],
           ["2020-11-24 13\\:02Z:66fd0da0-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518572087],
           ["2020-11-24 13\\:02Z:66fd0da0-2e7d-11eb-adc1-0242ac120002:share_price","27.50",1606240518572087],
           ["2020-11-24 13\\:02Z:66fd0da0-2e7d-11eb-adc1-0242ac120002:share_quantity","120",1606240518572087]]},
{"key": "FSLR:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 13\\:02Z:66fd0f1c-2e7d-11eb-adc1-0242ac120002:","",1606240518577054],
           ["2020-11-24 13\\:02Z:66fd0f1c-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518577054],
           ["2020-11-24 13\\:02Z:66fd0f1c-2e7d-11eb-adc1-0242ac120002:share_price","33.00",1606240518577054],
           ["2020-11-24 13\\:02Z:66fd0f1c-2e7d-11eb-adc1-0242ac120002:share_quantity","100",1606240518577054]]},
{"key": "CTL:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 12\\:59Z:66fd026a-2e7d-11eb-adc1-0242ac120002:","",1606240518541713],
           ["2020-11-24 12\\:59Z:66fd026a-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518541713],
           ["2020-11-24 12\\:59Z:66fd026a-2e7d-11eb-adc1-0242ac120002:share_price","40.50",1606240518541713],
           ["2020-11-24 12\\:59Z:66fd026a-2e7d-11eb-adc1-0242ac120002:share_quantity","100",1606240518541713],
           ["2020-11-24 13\\:03Z:66fd1430-2e7d-11eb-adc1-0242ac120002:","",1606240518591911],
           ["2020-11-24 13\\:03Z:66fd1430-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518591911],
           ["2020-11-24 13\\:03Z:66fd1430-2e7d-11eb-adc1-0242ac120002:share_price","30.00",1606240518591911],
           ["2020-11-24 13\\:03Z:66fd1430-2e7d-11eb-adc1-0242ac120002:share_quantity","100",1606240518591911]]},
{"key": "PVA:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 12\\:59Z:66fd003a-2e7d-11eb-adc1-0242ac120002:","",1606240518538807],
           ["2020-11-24 12\\:59Z:66fd003a-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518538807],
           ["2020-11-24 12\\:59Z:66fd003a-2e7d-11eb-adc1-0242ac120002:share_price","4.99",1606240518538807],
           ["2020-11-24 12\\:59Z:66fd003a-2e7d-11eb-adc1-0242ac120002:share_quantity","806",1606240518538807],
           ["2020-11-24 13\\:02Z:66fd0e5e-2e7d-11eb-adc1-0242ac120002:","",1606240518574762],
           ["2020-11-24 13\\:02Z:66fd0e5e-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518574762],
           ["2020-11-24 13\\:02Z:66fd0e5e-2e7d-11eb-adc1-0242ac120002:share_price","17.90",1606240518574762],
           ["2020-11-24 13\\:02Z:66fd0e5e-2e7d-11eb-adc1-0242ac120002:share_quantity","210",1606240518574762]]},
{"key": "APC:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 13\\:03Z:66fd1372-2e7d-11eb-adc1-0242ac120002:","",1606240518589148],
           ["2020-11-24 13\\:03Z:66fd1372-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518589148],
           ["2020-11-24 13\\:03Z:66fd1372-2e7d-11eb-adc1-0242ac120002:share_price","5.00",1606240518589148],
           ["2020-11-24 13\\:03Z:66fd1372-2e7d-11eb-adc1-0242ac120002:share_quantity","140",1606240518589148]]},
{"key": "CBZ:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 12\\:59Z:66fd0724-2e7d-11eb-adc1-0242ac120002:","",1606240518552850],
           ["2020-11-24 12\\:59Z:66fd0724-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518552850],
           ["2020-11-24 12\\:59Z:66fd0724-2e7d-11eb-adc1-0242ac120002:share_price","7.25",1606240518552850],
           ["2020-11-24 12\\:59Z:66fd0724-2e7d-11eb-adc1-0242ac120002:share_quantity","300",1606240518552850]]},
{"key": "WTI:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 13\\:01Z:66fd08b4-2e7d-11eb-adc1-0242ac120002:","",1606240518557388],
           ["2020-11-24 13\\:01Z:66fd08b4-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518557388],
           ["2020-11-24 13\\:01Z:66fd08b4-2e7d-11eb-adc1-0242ac120002:share_price","30.55",1606240518557388],
           ["2020-11-24 13\\:01Z:66fd08b4-2e7d-11eb-adc1-0242ac120002:share_quantity","80",1606240518557388]]},
{"key": "BRCD:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 13\\:01Z:66fd0ce2-2e7d-11eb-adc1-0242ac120002:","",1606240518568751],
           ["2020-11-24 13\\:01Z:66fd0ce2-2e7d-11eb-adc1-0242ac120002:exchange_id","NASDAQ",1606240518568751],
           ["2020-11-24 13\\:01Z:66fd0ce2-2e7d-11eb-adc1-0242ac120002:share_price","40.5",1606240518568751],
           ["2020-11-24 13\\:01Z:66fd0ce2-2e7d-11eb-adc1-0242ac120002:share_quantity","100",1606240518568751]]},
{"key": "FB:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 12\\:59Z:66fd07f6-2e7d-11eb-adc1-0242ac120002:","",1606240518555116],
           ["2020-11-24 12\\:59Z:66fd07f6-2e7d-11eb-adc1-0242ac120002:exchange_id","NASDAQ",1606240518555116],
           ["2020-11-24 12\\:59Z:66fd07f6-2e7d-11eb-adc1-0242ac120002:share_price","37.155",1606240518555116],
           ["2020-11-24 12\\:59Z:66fd07f6-2e7d-11eb-adc1-0242ac120002:share_quantity","100",1606240518555116],
           ["2020-11-24 13\\:02Z:66fd11f6-2e7d-11eb-adc1-0242ac120002:","",1606240518583246],
           ["2020-11-24 13\\:02Z:66fd11f6-2e7d-11eb-adc1-0242ac120002:exchange_id","NASDAQ",1606240518583246],
           ["2020-11-24 13\\:02Z:66fd11f6-2e7d-11eb-adc1-0242ac120002:share_price","50.10",1606240518583246],
           ["2020-11-24 13\\:02Z:66fd11f6-2e7d-11eb-adc1-0242ac120002:share_quantity","90",1606240518583246],
           ["2020-11-24 13\\:03Z:66fd175a-2e7d-11eb-adc1-0242ac120002:","",1606240519618582],
           ["2020-11-24 13\\:03Z:66fd175a-2e7d-11eb-adc1-0242ac120002:exchange_id","NASDAQ",1606240519618582],
           ["2020-11-24 13\\:03Z:66fd175a-2e7d-11eb-adc1-0242ac120002:share_price","66.55",1606240519618582],
           ["2020-11-24 13\\:03Z:66fd175a-2e7d-11eb-adc1-0242ac120002:share_quantity","80",1606240519618582]]},
{"key": "QIHU:2020-11-24 00\\:00Z",
 "cells": [["2020-11-24 13\\:01Z:66fd0972-2e7d-11eb-adc1-0242ac120002:","",1606240518559684],
           ["2020-11-24 13\\:01Z:66fd0972-2e7d-11eb-adc1-0242ac120002:exchange_id","NYSE",1606240518559684],
           ["2020-11-24 13\\:01Z:66fd0972-2e7d-11eb-adc1-0242ac120002:share_price","24.99",1606240518559684],
           ["2020-11-24 13\\:01Z:66fd0972-2e7d-11eb-adc1-0242ac120002:share_quantity","230",1606240518559684]]}
]
root@ddb2fe34736c:/var/lib/cassandra/data/stockwatcher/trade-3480d8d02dd711eb9fe4112a71a722b7#
```

You read the same structure has in cassandra-cli or the spreadsheet

Now you have done a deep overview of C* storage.

There is a lot of more, like static columns, tombstones, list, map, set ....

More are coming


