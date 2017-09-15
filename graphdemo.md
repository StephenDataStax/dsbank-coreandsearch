# Graph Demo

# Loading Data:

Note: this demo requires DSE 5.1.3 and DGL 5.1.3.

The following github contains the code for the graph examples: https://github.com/phact/graph-examples/tree/custom-vertex-ids

Download the repo as zip file or ```git pull --all``` branches and the fraud project is in a separate branch. Unzip the file.

In DSE Studio, open and submit the ```schema.groovy``` to create the graph schema. 

Navigate to the fraud directory. Using DGL, submit the following command to the loader:

```graphloader -graph fraud -address localhost fraud-mapping.groovy -inputpath ~/graph-examples/fraud/data/```

In DSE Studio, submit the ```addAddressEdges.groovy``` file. 

# Graph Demo:

This demo will illustrate DSE's ability to detect fraud in real-time. The demo consists of three examples. 

* Scenario 1: Legitimate - User registers and eventually places a order
* Scenario 2: Suspicious - User registers and places an order with previously used device id (might be husband and wife)
* Scenario 3: Fraud - User registers and places an order with highly used device id

Scenario 1: Legitimate - User registers and eventually places a order

Traversal to Visualize: ```g.V().has("customer", "customerid", "10000000-0000-0000-0000-000000000001").emit().repeat(both().simplePath()).times(4)```

Scenario 2: Suspicious - User registers and places an order with previously used device id (might be husband and wife)

Traversal to Visualize: ```g.V().has("address", "address", "102 Bellevue Blvd").has("postalcode", "21201").emit().repeat(both().simplePath()).times(4)```

Scenario 3: Fraud - User registers and places an order with highly used device id

Travelral to Visualize: ```g.V().has("device", "deviceid", "30000000-0000-0000-0015-000000000004").emit().repeat(both()).times(3)```
