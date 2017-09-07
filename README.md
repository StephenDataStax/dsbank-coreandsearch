# What is DSE Core and DSE Search?

* DSE Core is a scalable, distributed, fault-tolerant NoSQL database designed for real-time record lookup.
* DSE Search is an integrated search engine that automatically sync with DML and requires no ETL.
Enable real-time search with a single command
  * Search indexes are maintained with every insert, update, and delete operation.
  * Text/Fuzzy search, Faceting, Geospatial, Type ahead

# Dataset used for the exmaple

The dataset used for this example is simulated credit card tansaction data. The columns are credit card number, tansaction time, amount, items, location, merchant, notes, status, tag, transaction ID, and user ID.

Dataset: https://drive.google.com/a/datastax.com/file/d/0B56saSJLWZYETnBvQUoxWi1OZ0k/view?usp=sharing
** note this needs to move so people can access outside of DSE G Drive **

`CREATE KEYSPACE dsbank WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3};`

`use dsbank;`

`CREATE TABLE dsbank.transactions (
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
) WITH CLUSTERING ORDER BY (transaction_time DESC);`

`COPY dsbank.transactions (cc_no, transaction_time, amount, items, location, merchant, notes, status, tags, transaction_id, user_id) FROM '/tmp/dsbank.csv' WITH HEADER = TRUE ;`

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










