**[Back to Agenda](./../README.md)**


# Lab 3 - Data Modelling with DSE Search

## Search Essentials

DSE Search is awesome. You can configure which columns of which Cassandra tables you'd like indexed in Lucene format to make extended searches more efficient while enabling features such as text search and geospatial search.

First import some more data to the table we've already made. For this please use the COPY command in cqlsh to import a CSV data into the sales_by_customer table.


Please start to copy the repository first:
```
git clone https://github.com/norim/DataStaxDay
cd DataStaxDay/labs/res
```

** You can find the file transactions.csv under DataStaxDay/data/ **

Start `cqlsh` commandline and follow the instructions.
```
use <your_keyspace>;

COPY transactions (account_number,transaction_time,amount,items,location,merchant,notes,status,tags,transaction_id,user_id) FROM 'transactions.csv' ;
```


**Functional queries:**

1. For a given account_number, get all transactions

2. For a given account_number, get all transactions in a specific date range

3. For a given account_number, get all transactions with a 'SUCCESS' status

4. For a given account_number, get all transactions with amount greater then 15.0


Functional query from 1-5

```
select account_number, transaction_time, amount, location, merchant, notes, status from transactions where account_number = '1234123412341240';

select account_number, transaction_time, amount, location, merchant, notes, status from transactions where account_number = '1234123412341240' and transaction_time >= '2017-09-01' and transaction_time <= '2017-09-04';

select account_number, transaction_time, amount, location, merchant, notes, status from transactions where account_number = '1234123412341240' and status ='SUCCESS' ALLOW FILTERING;

select account_number, transaction_time, amount, location, merchant, notes, status from transactions where account_number = '1234123412341240' and amount> 3500 ALLOW FILTERING;

```

Now lets find the answer of the following functional queries.

6. For a given status, get all transactions


Let's start off by indexing the tables we've already made. Here's where DSE Search really comes in handy.  From cqlsh on one of your nodes run:

```
use <your_keyspace>;

CREATE SEARCH INDEX IF NOT EXISTS ON transactions WITH COLUMNS status;

```
Anyone familiar with Solr knows that there's a REST API for querying data. In DSE Search, we embed that into CQL so you can take advantage of all the goodness CQL brings. Let's give it a shot.



**Filter Queries**    
Now lets check the functional query with a solr query again:


```
select account_number, transaction_time, amount, location, merchant, notes, status from transactions where solr_query='{"q":"status:SUCCESS"}';
```

or **Wildcard**

```
select account_number, transaction_time, amount, location, merchant, notes, status from transactions where solr_query='{"q":"status:fai*"}';
```

or **Negation**

```
select account_number, transaction_time, amount, location, merchant, notes, status from transactions where solr_query='{"q":"account_number:1234123412341240", "fq":"-status:fail*"}';
```

To answer the following functional query we might need to index another column

7. For a given merchant, get all transactions with a amount greater 2000

Now add the merchant column to the indexing

```
ALTER SEARCH INDEX SCHEMA ON transactions ADD field[ @name='merchant', @type='TextField', @docValues='true'];
RELOAD SEARCH INDEX ON transactions;
REBUILD SEARCH INDEX ON transactions WITH OPTIONS { deleteAll:false };
```
**Range Query**    
Now lets check the functional query with a solr query again:    

```
select account_number, transaction_time, amount, location, merchant, notes, status from transactions where solr_query='{"q":"merchant:Ama*"}';

```

**Facet Search**    
Lets check how many orders with positive sentiment are in the postalcodes:   

```
select account_number, transaction_time, amount, location, merchant, notes, status from transactions where  solr_query='{"q":"status:*" ,"facet":{"field":"merchant"} , "useFieldCache":true}';
```


If you've ever created your own Solr cluster, you know you need to create the core and upload a schema and config.xml. That generateResources tag does that for you. For production use, you'll want to take the resources and edit them to your needs but it does save you a few steps.



### Real-time search index updates
As records are inserted, updated, or deleted the search indexes are automatically updated. This allow DataStax to offer real-time search without the need for an external culster and complex ETL operations.

Let's search for any merchants that are McDonalds. Note there are 0 records returned.

```
select * from transactions where solr_query='merchant:mcdonalds';
```

Now insert a record with a merchant of McDonalds:

```
INSERT INTO transactions (account_number, transaction_time, amount, location, merchant, notes, transaction_id, user_id)
  VALUES ('1234123412341240','2017-09-06 16:31:46.959+0000',58.32,'Tampa','McDonalds','HouseHold','31f4d4cc-8519-4982-be7b-b8aa06523ae3','banderson');
  ```

Run the search query again and the newly inserted record will appear:

```
select * from dsbank.transactions where solr_query='merchant:mcdonalds';
```


Search indexes are automatically maintained during update operations.

Update the newly inserted record and change the merchant to In-n-Out Burger:

```
update dsbank.transactions set merchant='In-n-Out Burger' where account_number='1234123412341240' and transaction_time='2017-09-06 16:31:46.959+0000';
```

Now search for the merchant In-n-Out Burger:

```
select * from dsbank.transactions where solr_query='merchant:in*';
```



**As a side note:**   
Similar to what we've done on cqlsh you can run the dsetool on the command line e.g. dsetool create_core retailer.sales generateResources=true reindex=true
You can update the schema and reload the configuration and schema.

This by default will map Cassandra types to Solr types for you.  I

For Geo Spatial Search you need to update the schema similar to the example in the docs:
https://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/search/queriesGeoSpatial.html

For your reference, here's the doc that shows some of things you can do: http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/search/queriesAbout.html

Want to see a really cool example of a live DSE Search app? Check out [KillrVideo](http://www.killrvideo.com/) and its [Git](https://github.com/luketillman/killrvideo-csharp) to see it in action.

**[Back to Agenda](./../README.md)**
