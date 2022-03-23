This in an initial version of the instructions that describe how to install and deploy an IDS dataspace connector instance. The connector can be connected to VTT's IDS tesbed and it can be used to test e.g. IDS based data retrieval.

<b>Prerequisites: </b>

The following software should be installed:

Docker and docker compose,
Java 11,
Maven,
Python3,
Ruby


<h2> 1. Installation: </h2>

<b>1.1 Clone the repository and unzip the package</b>

git clone  https://github.com/IlkkaNis/IDSTestbed_client_connector.git

Unzip the package and navigate to IDSTestbed_client_connector/DataspaceConnector

<b>1.2 Build the connector</b>

docker build -t dsca .

<b>1.3 Run the connector</b>

docker run --publish 8081:8081 --name connectora dsca

<h2> 2. Usage of the connector: </h2>

<b>2.1 Open the swagger API</b>
https://*yourURL*:8081/api/docs

<b>2.2 Test the connection to the testbed IDS connector</b>

Navigate to: _Messaging -> POST /api/ids/description
Define "https://broker.collab-cloud.eu:8081/api/ids/data" to the recipient url -field

Examine the results and find the URL endpoint for a data resource catalogue (e.g. https://broker.collab-cloud.eu:8081/api/catalogs/ba16f31a-7ff7-4fb4-aad6-9e4a5fcd0124) 

Send the same request again but this time, add the resource catalogue URL into the "id of the requested resource" -field. 

Examine the results. You should see more information about the data resources offered by the connector deployed in the VTT's IDS testbed

<b>2.3 Negotiate a contract to retrieve the example data resource</b>

Navigate to: _Messaging -> POST /api/ids/contract
Define "https://broker.collab-cloud.eu:8081/api/ids/data" to the The recipient url -field

Next, define the URL of the data resource you would like to receive (e.g. https://broker.collab-cloud.eu:8081/api/offers/8df5375e-bbd1-449f-bb81-768e9c711d52) 
NOTE! You can find all the required information / URLs from the response you received in the previous step.

Next, add the URLs of the artifacts you would like to retrieve (e.g. https://broker.collab-cloud.eu:8081/api/artifacts/869a9f91-00da-42af-aa30-df70a8f0b201)

In the automatic download section you can select "false"

To retrieve the actual data from another container, you must accept the contract offer specified by the data provider. You can find the contract offer from the response of /api/ids/description

To do so, copy the following statement into the "Request body" field of the /api/ids/contract:


[
  {
  "@type" : "ids:Permission",
  "@id" : "*URL of the permission as specified in the response of /api/ids/description (e.g. https://broker.collab-cloud.eu:8081/api/rules/8a0142db-b040-473e-a715-b2441af76de2)*",
  "ids:description" : [ {
  "@value" : "provide-access",
  "@type" : "http://www.w3.org/2001/XMLSchema#string"
  } ],
  "ids:title" : [ {
  "@value" : "Example Usage Policy",
  "@type" : "http://www.w3.org/2001/XMLSchema#string"
  } ],
  "ids:action" : [ {
  "@id" : "https://w3id.org/idsa/code/USE"
  } ],
  "ids:target": "*URL of the artifact you would like to receive*"
  }
]

Finally, execute the request



<b> 7. Test the connection to the testbed IDS broker</b>

Navigate to: _Messaging -> POST /api/ids/query
Define "http://broker.collab-cloud.eu:8080/infrastructure" to the The recipient url -field
Add "select ?x ?t ?u where {?x ?t ?u}" to the request body




