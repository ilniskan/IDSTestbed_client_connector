This in an initial version of the instructions that describe how to install and deploy an IDS dataspace connector instance. The connector can be connected to VTT's IDS tesbed and it can be used to test e.g. IDS based data retrieval.

<b>Prerequisites: </b>

The following software should be installed:

Docker and docker compose,
Java 11,
Maven,
Python3

<h2> 1. Installation: </h2>

<b>1.1 Clone the repository and unzip the package</b>

git clone  https://github.com/IlkkaNis/IDSTestbed_client_connector.git

Unzip the package and navigate to IDSTestbed_client_connector/DataspaceConnector

<b>1.2 Build the connector</b>

docker build -t dsca .

<b>1.3 Run the connector</b>

docker run --publish 8081:8081 --name connectora dsca

<h2> 2. Usage of the connector: </h2>

You can test the functionality of the connector by following the workflow shown in the picture below.

![Testbed workflow](https://github.com/IlkkaNis/IDSTestbed_client_connector/blob/main/testbedworkflow.png)

Next the different steps are explained in detail.

<b>2.1 Open the swagger API </b>

- Navigate with your browser to https://*yourURL*:8081/api/docs (Username = "admin", Password = "password")

<b>2.2 Retrieve connector metadata from the broker </b>
- In the swagger UI, navigate to:  _Messaging -> POST /api/ids/query
- Define "http://broker.collab-cloud.eu:8080/infrastructure" to the The recipient url -field
- Add the query statement shown below to the request body. This query will return the endpoints URLs of the connectors registered to the broker

```yaml 
SELECT ?accessURL WHERE {<https://localhost/connectors/-1475001399> <https://w3id.org/idsa/core/hasDefaultEndpoint> ?x. ?x <https://w3id.org/idsa/core/accessURL> ?accessURL} 
```


- From the response, copy an endpoint URL to be used in the next step

<b>2.3 Send description request message to the data provider connector </b>
- Navigate to:  _Messaging -> POST /api/ids/description
- Define the URL you received as response in the previous step (2.2)to the recipient url -field ( i.e. "https://broker.collab-cloud.eu:8081/api/ids/data")
- Examine the results and find the URL endpoint for a data resource catalogue (e.g. https://broker.collab-cloud.eu:8081/api/catalogs/ba16f31a-7ff7-4fb4-aad6-9e4a5fcd0124) 
- Send the same request again but this time, add the resource catalogue URL into the "id of the requested resource" -field. 
- Examine the results. You should see more information about the data resources offered by the connector deployed in the VTT's IDS testbed

<b>2.4 Negotiate a contract to retrieve the example data resource</b>

- Navigate to:  _Messaging -> POST /api/ids/contract and fill-in the required fields:
- Define the testbed connector endpoint URL (i.e. "https://broker.collab-cloud.eu:8081/api/ids/data") to the The recipient url -field
- Define the URL of the data resource you would like to receive (e.g. https://broker.collab-cloud.eu:8081/api/offers/8df5375e-bbd1-449f-bb81-768e9c711d52) <br><br>
<b> NOTE! You can find all the required information / URLs from the response you received in the previous step </b><br><br>
- Add the URLs of the artifacts you would like to retrieve (e.g. https://broker.collab-cloud.eu:8081/api/artifacts/869a9f91-00da-42af-aa30-df70a8f0b201)
- In the automatic download section you can select "false"

- To retrieve data from another container, you must accept the contract offer specified by the data provider (i.e. IDS testbed connector). You can find the specified contract offer from the response of /api/ids/description (step 2.3). The contract definition must be added to the "Request body" field. In this case, the contract statement should be defined as in the example shown below (remember to add the correct URLs into the JSON structure):

```yaml
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
```

<b>2.5 Access the received data resource </b>

- If the contract negotiation was successfull you will receive a response with the code "201". From the response body you should copy the URL that ends with the word "artifacts", e.g. "https://130.188.160.90:8081/api/agreements/3a4585cb-9119-4e39-ae8e-68eb773305d9/artifacts" (exclude the "{?page,size}" -part). 

- Paste this URL into your web browser and from the opened page click the link that ends with the word "data" e.g. 	"https://130.188.160.90:8081/api/artifacts/3b552084-a6cc-4845-8972-350647a624f2/data". This allows you to save and examine the data submitted by the data provider connector (i.e. the testbed connector). 

<h2> 3. Using the connector as a data provider (work in progress) </h2>

The testbed client connector can also be used as a data provider. This requires defining a data resource that can be requested by other connectors. The data structure used in the connector follows the data model shown below. 
A more detailed description of the data model can be found, for example, at https://international-data-spaces-association.github.io/DataspaceConnector/Documentation/v6/DataModel

![Connector data model](https://github.com/IlkkaNis/IDSTestbed_client_connector/blob/main/datamodel2.png)

As can be seen in the picture, the highest element of the data model is "Catalog". A catalog can contain multiple data resources and it is the starting point when connectors start acquiring more information about 
data elements provided by other connectors. "Data resource" can be described with a "Representation" that specifies, for example, the media type of a data resource. "Artifact" contains the actual "raw data" of the resource. An artifact also has a reference to an Agreement, 
that describe the agreed usage between data provider and data consumer. Data resource must always be linked with a "Contract". A usage contract comprises a set of usage policies that describes a certain permission or obligation of a data resource. The policies are specified with "Rules", and each rule may represent one of pre-defined IDS Usage Control Patterns.

Next, the creation of a new data resource using the Swagger API (https://*yourURL*:8081/api/docs) is explained 

<b>3.1 Defining a catalog </b>

- Navigate to:  Catalogs -> POST /api/catalogs and define a title and a description to the request body
- Copy the catalog URL from the response to be used in later phases (e.g. https://130.188.160.82:8081/api/catalogs/766e7048-66ad-4ece-9ed6-86eb4fb53773)

<b>3.2 Defining a data resource </b>

- Navigate to:  Offered Resources  -> POST /api/offers and define necessary data elements. Below is a concise example of the required request body
```yaml
{
  "title": "IDS data resource", 
  "description": "Example IDS data resource", 
  "keywords": ["data", "json","example","testing"], 
  "sovereign": "https://www.example.com/", 
  "publisher": "https://www.example.com/"
 }
```
- Copy the resource URL from the response to be used in later phases (e.g. https://130.188.160.82:8081/api/offers/cf26d3ea-9899-4725-b83b-b07ca4158d9f)

<b>3.3 Defining a representation </b>

- Navigate to:  Representations  -> POST /api/representations and define necessary data elements. Below is a concise example of the required request body
```yaml
{
  "title": "IDS representation",
  "description": "Example representation of a data resource",
  "mediaType": "json",
  "language": "English"
}
```
- Copy the representation URL from the response to be used in later phases (e.g. https://130.188.160.82:8081/api/representations/9455a243-e35c-4cd0-83d7-12084f386a3c)

<b>3.4 Defining a contract </b>

- Navigate to:  Contracts -> POST /api/contracts and define necessary data elements. Below is a concise example of the required request body
```yaml
{
  "title": "Example IDS contract",
  "description": "This an example IDS contract",
  "start": "2022-03-30T13:07:28.954Z",
  "end": "2023-03-30T13:07:28.954Z"
}
```
NOTE! If you preset the consumer attribute, the connector will compare this with the issuerConnector of an incoming contract request. If both values do not match, the contract request will be rejected. 

- Copy the contract URL from the response to be used in later phases (e.g. https://130.188.160.82:8081/api/contracts/ec380097-2b4d-4464-963b-71d3897112a9)

<b>3.5 Defining a rule </b>

- Navigate to:  Rules -> POST /api/rules and define necessary data elements. Below is an example of the required request body
```yaml
{
"title": "Example Rule Provide Access", 
"description": "This is an example rule containing a provide access usage policy", 
"value": "{\n \"@context\" : {\n \"ids\" : \"https://w3id.org/idsa/core/\",\n \"idsc\" : \"https://w3id.org/idsa/code/\"\n },\n \"@type\": \"ids:Permission\",\n \"@id\": \"https://w3id.org/idsa/autogen/permission/cf1cb758-b96d-4486-b0a7-f3ac0e289588\",\n \"ids:action\": [\n {\n \"@id\": \"idsc:USE\"\n }\n ],\n \"ids:description\": [\n {\n \"@value\": \"provide-access\",\n \"@type\": \"http://www.w3.org/2001/XMLSchema#string\"\n }\n ],\n \"ids:title\": [\n {\n \"@value\": \"Example Usage Policy\",\n \"@type\": \"http://www.w3.org/2001/XMLSchema#string\"\n }\n ]\n }"
}
```
- Copy the rule URL from the response to be used in later phases (e.g. https://130.188.160.82:8081/api/rules/b325c8e9-7cc0-4467-b0a7-268a69e59c90)


<b>3.6 Defining the artifact </b>

The connector enables defining different types of artifacts. Artifacts can consist, for example, arbitrary text strings, files or remote data that is acquired from external APIs. This example describes how
to define an artifact that contains a file retrieved from a local file system. 


- Navigate to:  Artifacts -> POST /api/artifact and define necessary data elements. Below is an example of the required request body (Please note that if you attach data from an external API you can include e.g. authorization or
API key data into the request body).
 
```yaml
{
  "title": "MyTestArtifact",
  "description": "This is an example artifact that contains an json file  ",
  "automatedDownload": "false"
}

```
- Copy the artifact URL from the response to be used in later phases (e.g. https://130.188.160.82:8081/api/rules/b325c8e9-7cc0-4467-b0a7-268a69e59c90)

- Add a file to the artifact with PUT /api/artifacts/{id}/data. Define the id of the artifact you just created into the "id" field (e.g. "df200253-7613-4e80-ba3d-d4d98a5bb2b3") and select a file from your local file system. After pressing
the execute button you should get the "204" response.

<b>3.7 Link different elements with each other </b>
To complete the data resource you should create linkages between different data elements (catalog, representation, artifact etc.)

- Add resource to catalog 
  - Navigate Catalogs -> POST /api/catalogs/{id}/offers 
  - Add catalog id to the "id" field (e.g. 03774974-6892-49b4-b170-207a9b2347b5) and offer URL to the request body (e.g. "https://130.188.160.82:8081/api/offers/80213233-f20d-42ac-b15e-f5e9ee87665e")

- Add representation to resource 
  - Navigate Offered Resources -> POST /api/offers/{id}/representations
  - Add resource id (i.e. offer id) to the "id" field and representation URL to the request body (e.g. "https://130.188.160.82:8081/api/representations/5f69fc08-7efb-4980-8b28-4973a11331ca")

- Add contract to resource 
  - Navigate Offered Resources -> POST /api/offers/{id}/contracts
  - Add resource id (i.e. offer id) to the "id" field and contract URL to the request body (e.g. "https://130.188.160.82:8081/api/contracts/858a449d-25f4-42d1-bd29-ef12fe5d7106")

- Add rule to contract 
  - Navigate Contracts -> POST /api/contracts/{id}/rules
  - Add contract id to the "id" field and rule URL to the request body (e.g. "https://130.188.160.82:8081/api/rules/b325c8e9-7cc0-4467-b0a7-268a69e59c90")

- Add artifact to representation 
  - Navigate Representation -> POST /api/representations/{id}/artifacts
  - Add representation id to the "id" field and artifact URL to the request body (e.g. "https://130.188.160.82:8081/api/artifacts/df200253-7613-4e80-ba3d-d4d98a5bb2b3")



