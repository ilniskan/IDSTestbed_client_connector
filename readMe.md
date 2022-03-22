This in an initial version of the instructions that describe how to install and deploy an IDS dataspace connector instance. The connector can be connected to VTT's IDS tesbed and it can be used to test e.g. IDS based data retrieval.

Prerequisites:  

The following software should be installed:

Docker and docker compose,
Java 11,
Maven,
Python3,
Ruby


Installation:

1. Clone the repository

git clone  https://github.com/IlkkaNis/IDSTestbed_client_connector.git

2. Build the connector

sudo docker build -t dsca .

3. Run the connector

sudo docker run --publish 8081:8081 --name connectora dsca

4. Open the swagger API
https://*serverURL*:8081/api/docs

5. Test the connection to the testbed IDS connector

Navigate to: _Messaging -> POST /api/ids/description
Define "https://broker.collab-cloud.eu:8081/api/ids/data" to the The recipient url -field

6. Test the connection to the testbed IDS broker

Navigate to: _Messaging -> POST /api/ids/query
Define "http://broker.collab-cloud.eu:8080/infrastructure" to the The recipient url -field
Add "select ?x ?t ?u where {?x ?t ?u}" to the request body
