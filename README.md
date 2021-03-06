# What is DSE Core and DSE Search?

* DSE Core is a scalable, distributed, fault-tolerant NoSQL database designed for real-time record lookup. DSE contain an easy to use, powerful data manipulation language called CQL which shares a lot of the syntactical language features of SQL.
* DSE Search is an integrated search engine that automatically sync with changes in the data and requires no ETL.
  * Enable Search with a single command
  * Search indexes are maintained with every insert, update, and delete operation.
  * Text/Fuzzy search, Faceting, Geospatial, Type ahead

# Dataset used for the example

The dataset used for this example is simulated credit card transaction data. The columns are account number, transaction time, amount, items, location, merchant, notes, status, tag, transaction ID, and user ID. The data is pre-loaded on the VM for this workshop.

Dataset: [dsbank.csv](https://github.com/StephenDataStax/dsbank-coreandsearch/blob/master/dsbank.csv)

# Creating schema with the CREATE KEYSPACE command

Keyspaces (or schemas) are the highest-level object in the database. They store a collection of tables or views. The keyspace determines how the data is distributed (in which data centers, geographic or cloud), and the level of replication and fault-tolerance the data should have. In this example, we have a single data center with a replication factor of 3 (3 copies of the data). 

`CREATE KEYSPACE if not exists dsbank WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3};`

Note: The dsbank keyspae has already been created for this demo.

The use keyspace command can set the default keyspace for the session:

`use dsbank;`

# Create a DSE table

To create a table with CQL, the developer will issue commands very similar to SQL. A CREATE TABLE statement is issued, columns are defined with data types, and some data distribution method is determined by the primary key (PK). The rows of the table are automatically distributed bsed on a hash of the PK. Additionally, clustering columns (CK) determine the sort order of the table. In this example, the PK is cc_no and transaction_time. This combination is a unique identifier for each row of data, which is a requirement for the PK. The data is then stored DESC by the CK transaction_time since most queries will want the latest transactions to appear first.

Some rules to remember when creating a table in DSE:
* Every row in a DSE table must contain a unique primary key.
* Distribution of data is automatically controlled though PK.
* The data can also be sorted with clustering columns (CK).

Note: The table has been already created for this demo.

```
CREATE TABLE if not exists dsbank.transactions (
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

# Copying data into the table

DSE has several mechanisms for data ingest. This example uses the COPY command to import the sample CSV file provided with this example. With COPY, the target table is named, the column structure is defined, and the source file is provided. Note, in this example, the data has a column header which is handled with the HEADER = TRUE option.

Note: The data has been pre loaded for this demo.

```
COPY dsbank.transactions (account_number, transaction_time, amount, items, location, merchant, notes, status, tags, transaction_id, user_id) FROM '/tmp/dse-halfday-workshop/resources/cql/dsbank.csv' WITH HEADER = TRUE ;
```

# Querying data with CQL

Querying data with CQL is very similar to SQL. Data should be located using the PK for efficient lookup and ranges can be applied to the CK. The database is designed for fast lookup and transactions based on the PK.

To see the transactions for Betty's account:

`select * from dsbank.transactions where account_number = '1234123412341240' limit 10;`

To see how much Betty spent over Labor Day weekend transactions within a range of transaction_time:

```
select sum(amount) from dsbank.transactions where account_number = '1234123412341240' and transaction_time >= '2017-09-01' and transaction_time <= '2017-09-04';
```

To see how many transactions have occured to date in September:

```
select count(*) from dsbank.transactions where account_number = '1234123412341240' and transaction_time >= '2017-09-01'
```

# Enabling Search

Search is an integrated part of the DSE platform. Only a single line of code is required to enable search:

`CREATE SEARCH INDEX IF NOT EXISTS ON dsbank.transactions;`

Once search is enabled, the query flexibility increases greatly. We can now search on any part of the record. To leverage search use the solr_query predicate as part of the where clause.

To search for all of Betty's transactions at Macys in West Palm Beach. This is use a fuzzy search for West Palm Beach:

`select * from dsbank.transactions where solr_query = 'merchant:Macys location:West*';`

To search all of Betty's cancelled transactions:

`select count(*) from dsbank.transactions where solr_query = 'status:cancelled';`

# Real-time search index updates

As records are inserted, updated, or deleted the search indexes are automatically updated. This allow DataStax to offer real-time search without the need for an external culster and complex ETL operations.

Let's search for any merchants that are McDonalds. Note there are 0 records returned.

`select * from dsbank.transactions where solr_query='merchant:mcdonalds';`

Now insert a record with a merchant of McDonalds:

```
INSERT INTO dsbank.transactions (account_number, transaction_time, amount, location, merchant, notes, transaction_id, user_id)
  VALUES ('1234123412341240','2017-09-06 16:31:46.959+0000',58.32,'Tampa','McDonalds','HouseHold','31f4d4cc-8519-4982-be7b-b8aa06523ae3','banderson');
 ```
Run the search query again and the newly inserted record will appear:

`select * from dsbank.transactions where solr_query='merchant:mcdonalds';`











