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

2. Unzip the package and navigate to IDSTestbed_client_connector/DataspaceConnector

3. Build the connector

docker build -t dsca .

4. Run the connector

docker run --publish 8081:8081 --name connectora dsca

5. Open the swagger API
https://*yourURL*:8081/api/docs

6. Test the connection to the testbed IDS connector

Navigate to: _Messaging -> POST /api/ids/description
Define "https://broker.collab-cloud.eu:8081/api/ids/data" to the The recipient url -field

Examine the results and find the URL endpoint for a data resource catalogue (e.g. https://broker.collab-cloud.eu:8081/api/catalogs/ba16f31a-7ff7-4fb4-aad6-9e4a5fcd0124) 

Send the same request again but this time, add the resource catalogue URL into "The id of the requested resource" field. 

Examine the results. You should see more information about the data resource offered by the connector hosted by VTT's IDS testbed

8. Negotiate a contract to retrieve an example data resource

Navigate to: _Messaging -> POST /api/ids/contract
Define "https://broker.collab-cloud.eu:8081/api/ids/data" to the The recipient url -field

Next, define the data resource you would like to receive. First add the URL of the resource you are interested in. 
NOTE! You can find all the required URLs from the response you received in the previous step.





7. Test the connection to the testbed IDS broker

Navigate to: _Messaging -> POST /api/ids/query
Define "http://broker.collab-cloud.eu:8080/infrastructure" to the The recipient url -field
Add "select ?x ?t ?u where {?x ?t ?u}" to the request body




