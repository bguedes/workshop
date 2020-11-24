### Context Initialisation

from project root :

```bash
docker cp cql node1:home/cql

docker exec -it node1 bash

cd  /home
cqlsh

source 'cql/stockwatcher.cql';
```

## Keyspace and Tables for Workshop

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







