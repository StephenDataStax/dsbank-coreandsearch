# What is DSE Core and DSE Search?

* DSE Core is a scalable, distributed, fault-tolerant NoSQL database designed for real-time record lookup. DSE contain an easy to use, powerful data manipulation language called CQL which shares a lot of syntactical language freatures of SQL.
* DSE Search is an integrated search engine that automatically sync with DML and requires no ETL.
Enable real-time search with a single command
  * Search indexes are maintained with every insert, update, and delete operation.
  * Text/Fuzzy search, Faceting, Geospatial, Type ahead

# Dataset used for the exmaple

The dataset used for this example is simulated credit card tansaction data. The columns are credit card number, tansaction time, amount, items, location, merchant, notes, status, tag, transaction ID, and user ID.

Dataset: https://drive.google.com/a/datastax.com/file/d/0B56saSJLWZYETnBvQUoxWi1OZ0k/view?usp=sharing
** note this needs to move so people can access outside of DSE G Drive **

# Starting DSE with Search enabled

To start DSE with Search enabled issue the following command:

`dse cassandra -s`

Once DSE has started, invoke the cqlsh client to interact with the database:

`clqsh`

# Creating Schema with the the CREATE KEYSPACE command

Keyspaces (or schemas) are the highest-level object in the database. They store a collection of tables or views. The keyspace determines how the data is distributed (in which data centers, geographic or cloud), and Also the level of replication and fault-tolernace the data should have. In this example, we have a single data center with a replication factor of 3 (3 copies of the data). 

`CREATE KEYSPACE dsbank WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3};`

The use keyspace command can set the default keyspace for the session:

`use dsbank;`

# Create a DSE table

To create a table with CQL, the developer will issues commands very similar to SQL. A Create TABLE statement is used, columns are defined with data types, and some distribution model is determined by the primamry key. Aditionally, clustering columns determine the sort order of the data. In this example, our PRIMARY KEY is cc_no and transaction_time. This combination is a unique indentifer for each row of data. The data is then stored DESC by transaction_time since most queries will want the latest transcations to appear first.

Some  rules to remeber when create a table in DSE:
* Every row in a DSE table must contain a unique primary key. The PK determines the primary key. 
* Distribution of data is automatically controlled though PK.
* The data can also be sorted with clustering columns (CK).

```CREATE TABLE dsbank.transactions (
    cc_no text,
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
    PRIMARY KEY (cc_no, transaction_time)
) WITH CLUSTERING ORDER BY (transaction_time DESC);
```

# Copying data into the table

DSE has several mechanisms for data ingest. This example uses the COPY command to import the CSV file. With COPY, the target table is named, the columns structre is defined, and the source file is provided. Note, in this example, the data has a column hearder which is handled with the HEADER = TURE option.

`COPY dsbank.transactions (cc_no, transaction_time, amount, items, location, merchant, notes, status, tags, transaction_id, user_id) FROM '/tmp/dsbank.csv' WITH HEADER = TRUE ;`

# Querying data with CQL

//select a specific account (primary key)

`select * from dsbank.transactions where cc_no = '1234123412341240' limit 10;`

//count all transactions within a range of transaction_time (clustering column)

`select count(transaction_id) from dsbank.transactions where cc_no = '1234123412341240' and transaction_time > '2017-09-01' and transaction_time < '2017-09-03';`

//enable search

`CREATE SEARCH INDEX IF NOT EXISTS ON dsbank.transactions;`

//search for the sum amount of all transactions at Macys in Atlanta

`select sum (amount) from dsbank.transactions where solr_query = 'merchant:Macys location:Atlanta';`

//search all cancelled transactions

`select count(*) from dsbank.transactions where solr_query = 'status:cancelled';`

//insert a record

`INSERT INTO dsbank.transactions (cc_no, transaction_time, amount, location, merchant, notes, transaction_id, user_id)
  VALUES ('1234123412341240','2017-09-06 16:31:46.959+0000',58.32,'Tampa','McDonalds','HouseHold','31f4d4cc-8519-4982-be7b-b8aa06523ae3','banderson');`

//search for new record

`select * from transactions where solr_query='merchant:mcdonalds';`










