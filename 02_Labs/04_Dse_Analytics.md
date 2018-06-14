**[Back to Agenda](./../README.md)**

Hands On DSE Analytics
--------------------

Spark is general cluster compute engine. You can think of it in two pieces: **Streaming** and **Batch**.
**Streaming** is the processing of incoming data (in micro batches) before it gets written to Cassandra (or any database).
**Batch** includes both data crunching code and **SparkSQL**, a hive compliant SQL abstraction for **Batch** jobs.



### *Spark SQL*

Spark SQL allows you to perform relational queries over data stored in DSE clusters, and executed using Spark. Spark SQL is a unified relational query language and supports a variation of the SQL language used in relational databases. You can use traditional Spark applications in conjunction with Spark SQL queries to analyze large data sets. This lets you use the power of Spark with an already familiar language.

Start the DSE Spark SQL prompt (run from the SSH command prompt on any cluster node)

```
dse spark-sql --executor-memory 1G --total-executor-cores 2
```

Find the list of merchants that Betty spends the most money with.

```
SELECT merchant, sum(amount) AS total FROM transactions WHERE account_number = '1234123412341240' GROUP BY merchant ORDER BY total DESC;
```

Across the entire bank find the accounts that are doing the most transactions

```
SELECT account_number, count(account_number) AS total_trans FROM transactions GROUP BY account_number ORDER BY total_trans DESC;
```

Find the merchants doing the most transactions with the bank, and the total value of all transactions for each merchant.
```
SELECT merchant, sum(amount) AS total_amount, count(amount) AS total_trans FROM transactions GROUP BY merchant ORDER BY total_trans;
```

Or you can order your results by total value of each merchant
```
SELECT merchant, sum(amount) AS total_amount, count(amount) AS total_trans FROM transactions GROUP BY merchant ORDER BY total_amount;
```

Spark-SQL can also do JOINs across DSE tables. The table  merchant_category contains an index of all merchants and their merchandise categories. Query for all transactions with merchants that sell electronics.
```
SELECT * FROM merchant_category JOIN  transactions ON merchant_category.merchant = transactions.merchant WHERE category = "electronics";
```

Put it all together. Find the merchants Betty bought electronics from and the total paid to each merchant.
```
SELECT merchant_category.merchant, sum(amount) AS total_amount FROM merchant_category JOIN  transactions ON merchant_category.merchant = transactions.merchant WHERE category = "electronics" AND account_number = '1234123412341240' GROUP BY merchant_category.merchant ORDER BY total_amount DESC;
```

(Press CTRL+C now to quit the DSE Spark-SQL prompt)




#### **Spark Batch Hands On:**

Spark-SQL is great for doing analytical queries with a familiar language, but has a limited set of functionality. When you need to be able to work directly with your data and do more complex transformations Spark supports using Scala, a general-purpose programming language providing support for functional programming. It is a concise language that is not hard to learn, but is extremely powerful. Spark Scala supports everything from basic ETL operations, real-time streaming, and even full machine learning piplines. Scala will be used for the rest of this hands-on, but DSE Spark also supports other languages such as Java, Python, and R.

##### Exercise:

DS Bank is developing a new offering for it's customers allowing them to view realtime reports about any possible fraudulent events against their accounts.

This new use case will be using new data access patterns requiring additional data models to be created.

A data model for the Spark stream to look up transactions by their id (transactions_by_id)
A data model to support the fraud report dashboard (account_fraud)

#### *Batch Data Load*
The  transactions_by_id table will be used to lookup existing transactions by their transaction_id. This table needs to be populated by the existing data in the  transactions table. This can be handled by Spark using some simple Scala.

Start the spark scala prompt (run from the SSH command prompt on any cluster node)
```
dse spark spark.ui.port=<Pick a random 4 digit number> --total-executor-cores 2 --executor-memory 1G
```

>Notice the spark.ui.port flag - Because we are on a shared cluster, we need to specify a radom port so we don't clash with other users. We're also setting max cores = 1 or else one job will hog all the resources.

Create a varible that points to the table you want to source from. This is a “lazy evaluation”, meaning Spark won't execute the work that needs to be done until it is needed. Assinging the table to a varible allows you to iterate through the data later and perform operations such as aggregations, transformations, and enrichment.

```
val transactions = sc.cassandraTable("<your keyspace>", "transactions")
```

Write all of the rows from “transactions” into “transactions_by_id”. This will actually execute the evaluation of the transactions varible we created in the previous step. This is a basic ETL that does not require any transformation, but we could easily add them here if needed.
```
transactions.saveToCassandra("<your keyspace>", "transactions_by_id")
```

Verify that the table “transactions_by_id” has data. Note that adding .collect() to the statement makes this no longer be a “lazy evaluation” and executes the work right away. You can the use .foreach() to iterate through data in a variable.
```
val sample = sc.cassandraTable("<your keysapce>", "transactions_by_id").limit(20).collect()
sample.foreach(println)
```

(Press CTRL+C now to quit the DSE Spark-SQL prompt)


**[Back to Agenda](./../README.md)**
