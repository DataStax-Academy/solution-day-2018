**[Back to Agenda](./../README.md)**

# Lab 2 - CQL

Use SSH to connect to one of your nodes.  We're now going to start the cqlsh client.

And before we go on, a quick explanation of what CQL, CQLSH and other aspects of Cassandra and DataStax Enterprise is in order:

The Cassandra Query Language (CQL) is the primary language for communicating with the Cassandra database. The most basic way to interact with Cassandra is using the CQL shell, cqlsh. Using cqlsh, you can create keyspaces and tables, insert and query tables, plus much more. If you prefer a graphical tool, you can use DataStax DevCenter. For production, DataStax supplies a number of drivers so that CQL statements can be passed from client to cluster and back.

To start the cqlsh client run the command:

```
# use the specific connected node name or `hostname -I`

cqlsh node0
```

![](./img/lab2-1cqlsh.png )

Let's make our first Cassandra Keyspace! If you are using uppercase letters, use double quotes around the keyspace.

```

CREATE KEYSPACE if not exists </ your_keyspace /> WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3};

```

![](./img/lab2-2createkeyspace.png )

And just like that, any data within any table you create under your keyspace will automatically be replicated 3 times. Let's keep going and create ourselves a table. You can follow my example or be a rebel and roll your own.

```
use </ your_keyspace />;

CREATE TABLE if not exists transactions (
    account_number text,
    transaction_time timestamp,
    amount double,
    items map<text, double>,
    location text,
    merchant text,
    notes text,
    status text,
    tags set<text>,
    transaction_id text,
    user_id text,
    PRIMARY KEY (account_number, transaction_time)
) WITH CLUSTERING ORDER BY (transaction_time DESC);

```

![](./img/lab2-2createtable.png)**/

Yup. This table is very simple but don't worry, we'll play with some more interesting tables in just a minute.

Let's get some data into your table! Cut and paste these inserts into DevCenter or CQLSH. Feel free to insert your own data values, as well.

```
INSERT INTO transactions (account_number,transaction_time,amount,merchant, notes, status) VALUES ('1', '2018-01-11', 299.0, 'Target', 'Shopping', 'SUCCESS');
INSERT INTO transactions (account_number,transaction_time,amount,merchant, notes, status) VALUES ('2', '2018-01-12', 1999.0, 'Wal-Mart', 'Houshold', 'CANCELLED');
INSERT INTO transactions (account_number,transaction_time,amount,merchant, notes, status) VALUES ('1', '2018-01-13',  1499.00, 'Amazon','Shopping', 'SUCCESS');
INSERT INTO transactions (account_number,transaction_time,amount,merchant, notes, status) VALUES ('1', '2018-01-04', 399.0, 'Amazon','Shopping', 'CANCELLED');
INSERT INTO transactions (account_number,transaction_time,amount,merchant, notes, status) VALUES ('2', '2018-01-15', 560.0, 'Macys', 'Shopping', 'SUCCESS');
```

Now, to retrieve data from the database run:

```

select * from transactions WHERE account_number='1' AND transaction_time >='2018-01-01';

select sum(amount) from transactions where account_number = '1' and transaction_time >= '2017-09-01' and transaction_time < '2018-02-01';

```

![](./img/lab2-4select.png)

See what I did there? You can do range scans on clustering keys! Give it a try.

Hey you can also use standard aggregate functions.

```  
select account_number, sum(amount) from transactions group by account_number;
```

If you have time checkout the Upsert, the TTL and LWT.

1. What happens if you insert the same rows multiple times?
2. Insert a new partition with the TTL of 10 seconds.
3. Insert a duplicate row of the available partitions with IF NOT EXISTS statement. What can you see?
4. What happens when you insert a NULL value?



Let's put some workload on the cluster.
Cassandra stress is a tool which can be used to verify the scalability and latency performance of a specific keyspace and table.

We've prepared a cassandra-stress profile to customize the workload as we would expect it in the system.

Login in On one of the nodes of your cluster and copy the casstress-sales.yaml profile. Add your keyspace to it.

```
vi cassandra-stress-transactions.yaml
```
Copy the content to the file.

```
# Keyspace Name

keyspace: <your keyspace name>

keyspace_definition:

  CREATE KEYSPACE IF NOT EXISTS <your keyspace name> WITH replication = { 'class':'NetworkTopologyStrategy', 'DC1':3};


# Table name
table: stress_transaction_by_txtime

# The CQL for creating a table you wish to stress (optional if it already exists)
table_definition:
  CREATE TABLE if not exists stress_transaction_by_txtime (
    account_number text,
    transaction_time timestamp,
    amount double,
    location text,
    merchant text,
    notes text,
    status text,
    tags set<text>,
    transaction_id text,
    user_id text,
    PRIMARY KEY (account_number, transaction_time)
  ) WITH CLUSTERING ORDER BY (transaction_time DESC);

### Column Distribution Specifications ###

columnspec:
- name: account_number
  size: uniform(10..30)
- name: transaction_time
  cluster: uniform(20...40)
- name: amount
  cluster: uniform(3...5)
- name: merchant
  size: uniform(1..2)
- name: notes
  size: uniform(10..30)
- name: status
  size: uniform(10..30)

insert:
  partitions: fixed(1) # 1 partition per batch
  batchtype: UNLOGGED # use unlogged batches
  select: fixed(10)/10 # no chance of skipping a row when generating inserts

#
# A list of queries you wish to run against the schema
#
queries:
   read1:
    cql: SELECT * FROM stress_transaction_by_txtime WHERE account_number = ?
    fields: samerow
   read1:
    cql: SELECT * FROM stress_transaction_by_txtime WHERE account_number = ? AND transaction_time = ?
    fields: samerow
   read1:
    cql: SELECT * FROM stress_transaction_by_txtime WHERE account_number = ? AND transaction_time >= ? AND transaction_time <= ?
    fields: samerow
```

To start cassandra-stress you can use the following command:  
**Please note that you have to put your IP address of one of your cluster nodes for the *-node* parameter**

```
cassandra-stress user profile=/<path_to_file>/cassandra-stress-transactions.yaml ops\(insert=3,read1=5\) no-warmup cl=LOCAL_ONE -node < node IPs address >
```

![](./img/lab2-5casstress.png)

Now you can start monitoring your cluster via OpsCenter: http://<cluster_node1_ip>:8888/opscenter/

![](./img/lab2-6opscenter.png)

For further assistance and to create a more complex stress yaml file please consult the following page:   
**[Cassandra Stress Data Modeler](http://www.sestevez.com/sestevez/CassandraDataModeler/)**

**[Back to Agenda](./../README.md)**
