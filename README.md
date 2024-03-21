## Introduction

Tazama is prefaced with the Transaction Monitoring Service (TMS) API which makes [Postman](https://www.postman.com/) a useful tool to test platform functionality. In days gone by, Tazama was also composed out of a series of forward-chaining microservices that all had their own RESTful interfaces to receive incoming requests, but we have since replaced our inter-services communication protocol with [NATS](http://nats.io) that connects all our internal processors via its pub/sub interface. While we still use Postman to test the internal processors, we now have to access the NATS pub/sub interface via a NATS REST Proxy that we also built. (You can read more about the NATS REST proxy in the [nats-utilities](https://github.com/frmscoe/nats-utilities)) repository.

The files and folders you see here can be used to test Tazama via both the TMS API and, as an example, rule processor 901, our sample template rule processor.

When you are deploying Tazama as a public user<sup>1</sup> you will be able to use the tests here to validate your deployment. The instructions to set up Tazama via Docker Compose into a Full Stack containerized implementation can be found in the [Full-Stack-Docker-Tazama](https://github.com/frmscoe/Full-Stack-Docker-Tazama) repository. The instructions there will provide specific guidance on how to test a deployment using the end-to-end test above.

If you are setting up your environment according to the instructions in the [Tazama Contribution Guide](https://github.com/frmscoe/docs/blob/main/Community/Tazama-Contribution-Guide.md#32-setting-up-the-development-environment), those instructions will also guide you on how to use these Postman tests to test our sample rule processor 901 in isolation using the NATS REST proxy.

To use any of these tests, you can clone this repository onto your local machine and either import the tests you want to run or work on into Postman, or you can run the tests from a command line via [Newman](https://learning.postman.com/docs/collections/using-newman-cli/installing-running-newman/#installing-newman).

Read on for some more information on the tests, and also guidance on how to write tests for your own processors.

## The folders

### archived-legacy-tests

Our use of Postman has evolved over the years. We are always finding new and better ways of using Postman for our testing. We are also hoarders, and we don't like to throw anything away. This folder contains tests that no longer work for testing Tazama, but the tests may contain some scripts that may again prove to be useful in the future. Browse at your leisure, or your peril.

### environments

If you're familiar with Postman then you will know that Postman tests are often executed within a specific environment configuration. Any environment variables that we use in our tests are defined in a number of environment files. The environment folder contains two environment files that we use for different purposes. If you are deploying Tazama into your own cloud environment, and you would like to create a Postman environment file that matches your deployed environment, you can modify either of these environment files for your needs.

> [!NOTE]  
> Postman contains two kinds of variables: there are environment variables and global variables. When you click the little icon in the top right in Postman, you'll get to see both types in two separate tables.
>
>As a rule in Tazama, the environment variables are just that: variables that define the parameters for running the Postman tests in a specific deployed environment. These variables should be static and persistent between tests. No Tazama Postman test (pre-request script or test) should ever use the `pm.environment.set()` method.
>
> The global variables are the ones that should be used when you are stashing values between tests. For example, if you want to set up the address to the API, it's an environment variable. If you want to remember an ID from one test to use as input into another test, it's a global variable.

#### Tazama-Docker-Compose-LOCAL.postman_environment.json

This environment will facilitate testing over the containerized Tazama that is deployed from the [Full-Stack-Docker-Tazama](https://github.com/frmscoe/Full-Stack-Docker-Tazama) repository. The environment is set up based on the default settings of the deployment.

#### Tazama-LOCAL.postman_environment.json

This environment will facilitate testing on a local installation of Tazama, mostly to support development and functional testing of a single processor at a time.

#### Tazama environment file contents

A functioning Tazama environment file for Postman will contain the following attributes. These attributes are used in our tests and allow us to make changes to the values via the environment file, instead of having to edit the scripts directly.

**General attributes:**

| Attribute(s) | Description | Example(s)
|:---:|---|---|
| `ofUrl` | The URL of the TMS API | localhost:5000 </br> https://tazama-tms.yourcompany.com |
| `natsUrl` | The URL of the NATS REST proxy for testing a specific processor directly. | localhost:3000 </br> https://tazama-tms.yourcompany.com:3000 |
| `arangoUrl` | The URL of the deployed instance of ArangoDB | localhost:18529 </br> https://tazama-arangodb.yourcompany.com:3000 |
| `arangoUsername` | The ArangoDB username for retrieving an Arango token to interact with ArangoDB via its native API | This value will depend on the implementation of Arango. Typically this value is blank for local deployments. |
| `arangoPassword` | The password associated with the ArangoDB username for retrieving an Arango token to interact with ArangoDB via its native API | This value will depend on the implementation of Arango. Typically this value is blank for local deployments. |
| `activePain001` | This attribute reflects whether the platform has been configured to include or exclude a quoting phase via pain.001 and pain.013 messages as part of a transaction set | `true` - quoting is included</br> `false` - quoting is excluded |
| `path-pain001` | The API path to receive a pain.001 request. Do not update this value unless the paths in your deployment are different. | `execute` |
| `path-pain013` | The API path to receive a pain.013 request. Do not update this value unless the paths in your deployment are different. | `quoteReply` |
| `path-pacs008` | The API path to receive a pacs.008 request. Do not update this value unless the paths in your deployment are different. | `transfer` |
| `path-pacs002` | The API path to receive a pacs.002 request. Do not update this value unless the paths in your deployment are different. | `transfer-response` |

**NATS REST proxy-specific attributes:**

The attributes below are only required if you are interacting with a specific processor directly via the NATS REST proxy. The variables here are a hangover from when the processors were invoked directly through their native RESTful APIs. With the update to NATS, the internal processors can only be accessed directly via the NATS REST proxy, and for that reason the folder path must be "natsPublish".

| Attribute(s) | Description | Example(s)
|:---:|---|---|
| `path-channel-router-setup-processor` | The folder path to the Channel Router & Setup Processor | natsPublish - since all  |
| `path-rule-001-rel-1-0-0` </br> to </br> `path-rule-901-rel-1-0-0` | The folder/path to the specific rule processor | natsPublish |
| `path-typology-processor` | The folder/path to the typology processor | natsPublish |
| `path-tadproc` | The folder/path to the Transaction Aggregation & Decisioning processor | natsPublish |
| `path-cms-service` | The folder/path to the Case Management egress service via NATS. This value should not be changed unless your deployment has an alternative destination configured for the CMS service. | `off-cms-service` |

**ArangoDB-specific attributes:**

The attributes below host a variety of ArangoDB variables for database and collection names to provide some abstraction in the code.In general, these values only need to change if the underlying database structure changes.

| Attribute(s) | Description | Example(s)
|:---:|---|---|
| `db_messagehistory` | The database name where the transaction history is stored | `transactionHistory` |
| `db_historygraph` | The database name where the transaction history graph data is stored | `pseudonyms` |
| `db_results` | The database name where the transaction evaluation results is stored | `evaluationResults` |
| `db_coll_msg_transactionHistoryPain001` | The collection name where the pain.001 transaction history is stored in the message history database | `transactionHistoryPain001` |
| `db_coll_msg_transactionHistoryPain013` | The collection name where the pain.013 transaction history is stored in the message history database | `transactionHistoryPain013` |
| `db_coll_msg_transactionHistoryPacs008` | The collection name where the pacs.008 transaction history is stored in the message history database | `transactionHistoryPacs008` |
| `db_coll_msg_transactionHistoryPacs002` | The collection name where the pacs.002 transaction history is stored in the message history database | `transactionHistoryPacs002` |
| `db_coll_graph_entities` | The collection name where the debtor and creditor information in a transaction is stored in the transaction history graph database | `entities` |
| `db_coll_graph_account_holders` | The edge collection name where the debtor and creditor account relationship information is stored in the transaction history graph database | `account_holder` |
| `db_coll_graph_accounts` | The collection name where the debtor and creditor account information in a transaction is stored in the transaction history graph database | `accounts` |
| `db_coll_graph_transactions` | The collection name where the debtor and creditor information in a transaction is stored in the transaction history graph database | `transactionRelationship` |
| `db_config_all` | The database name where processor configuration data is stored | `Configuration` |
| `db_config_route` | The database name where routing configuration data is stored | `networkmap` |
| `db_config_rules` | The collection name where the rule configurations will be stored in the processor configuration database | `configuration` |
| `db_config_typologies` | The collection name where the typology configurations will be stored in the processor configuration database | `typologyExpression` |
| `db_config_channel` | The collection name where the channel configurations will be stored in the processor configuration database. This is not currently in use. | `channelExpression` |
| `db_config_transactions` | The collection name where the channel configurations will be stored in the processor configuration database. This is no longer in use. | `transactionConfiguration` |
| `db_config_networkConfiguration` | The collection name where the routing configuration will be stored in the routing configuration database. | `networkConfiguration` |

**Postman testing-specific attributes:**

These attributes have an impact on how Postman tests are interpreted for testing purposes.

| Attribute(s) | Description | Example(s)
|:---:|---|---|
| `preconfigured` | This attribute is used to determine of tests of rule processors directly require that the rule configuration is to be loaded as part of the test (almost like a kind of pseudo-mocking on a rule-by-rule test basis), or if the rule configuration already exists in the database. | `true` - read the rule config from the database </br> `false` - recreate the rule config in the database before it is read |

### 1.1. Rule-901 End-to-End test - pain001-013 disabled.postman_collection.json

This test collection contains a collection of API requests that set up a randomly generated set of pacs.008 and pacs.002 transactions, then submits the transaction pair to the TMS API, and finally performs a number of tests to make sure that the databases were properly updated and the transaction evaluated successfully to the point where a result was posted to the results database.

Tazama has created a Javascript utility library to assist with the creation of valid pain.001, pain.013, pacs.008 and pacs.002 messages, one-by-one or in bulk. You can find the documentation for this utility further down in this page, or you can view the docstrings for each of the functions in the code in the test collection's Pre-Request Script tab.

The paragraphs below will provide a brief overview of each of the requests and tests in this test collection.

**Folder: Message creation sans pain.001/013**
 - Create messages in memory

    This request uses the Javascript utility library to create a pacs.008 message and a pacs.002 message. The messages in the set are linked via a common `EndToEndId` identifier.

    The complete messages, along with some specific attributes that we will be using in later requests, are stashed as global variables in Postman. You can also view the created information in the console when the request is executed.
    
    The default behaviour of the platform is to exclude the quoting steps via the `activePain001` environment variable and no pain.001/013 message will be created in this test.

    The message creation request is set up as a Postman test using a GET method and simultaneously checks that the TMS API is available. The TMS API will respond with: 
    ```json
    {
      "status": "UP"
    }
    ```

 - Post pacs.008 to TMS API

    The pacs.008 message is sent to the TMS API. The TMS API validates the incoming message and updates the database with the pacs.008 data. If the database update is successful, the message is routed to the Channel Router & Setup Processor (CRSP) and a response is generated by the API confirming that the message was successfully received. This response is not the result of the evaluation though, but only the successful receipt and ingestion of the message. The platform still has to do all the work to evaluate the message and the evaluation result will be posted to the results database once the evaluation is complete. The TMS API should respond with:

    ```json
    {
    "message": "Transaction is valid",
    "data": {
      "TxTp": "pacs.008.001.10",
      <a copy of the incoming message>
    }
    ```

 - Post pacs.002 to TMS API

    The pacs.002 message is sent to the TMS API. The TMS API validates the incoming message and updates the database with the pacs.008 data. If the database update is successful, the message is routed to the Channel Router & Setup Processor (CRSP) and a response is generated by the API confirming that the message was successfully received. This response is not the result of the evaluation though, but only the successful receipt and ingestion of the message. The platform still has to do all the work to evaluate the message and the evaluation result will be posted to the results database once the evaluation is complete. The TMS API should respond with:

    ```json
    {
    "message": "Transaction is valid",
    "data": {
      "TxTp": "pacs.002.001.12",
      <a copy of the incoming message>
    }
    ```

**Folder: DB update tests**

The tests in this folder interrogates the database to make sure that the datebases and collections were correctly updated when the message was received and that the transaction evaluation completed successfully.

 - Get Arango Token

    If your installation of Arango is secured with a username and password, this request will use the ArangoDB credentials from the environment file to fetch a short-term security token from Arango that you need to access the ArangoDB API for the queries below.

 - Fetch created pacs.008 from transactionHistoryPacs008

    This request checks if the pacs.008 message we had submitted was successfully written to the `transactionHistoryPacs008` collection in the `transactionHistory` database. The request attempts to retrieve the pacs.008 message via the `msgId` that we had used when we created the messages in memory. The response body will contain the retrieved message and the Test Results tab should show the successful result of the test assertion:

    ![successful test for pacs.008 retrieval](./images/successful-test-pacs008-retrieval.png)

 - Fetch created pacs.002 from transactionHistoryPacs002

    This request checks if the pacs.002 message we had submitted was successfully written to the `transactionHistoryPacs002` collection in the `transactionHistory` database. The request attempts to retrieve the pacs.002 message via the `msgId` that we had used when we created the messages in memory. The response body will contain the retrieved message and the Test Results tab should show the successful result of the test assertion:

    ![successful test for pacs.008 retrieval](./images/successful-test-pacs002-retrieval.png)

 - Check graph creation

    This request checks if al the graph database components of the incoming message were properly created in the `pseudonyms` database. The request attempts to retrieve the individual graph nodes and vertices from the database via the identifiers that we had used when we created the messages in memory. The response body will contain the retrieved components and the Test Results tab should show the successful result of the test assertions:

    ![successful test for pacs.008 retrieval](./images/successful-test-graph-retrieval.png)

 - Fetch evaluation results with msgId - Rule 901 Network Map Only

    This final test checks that the evaluation result was successfully produced by the system and safely stored in the `transactions` collection in the `evaluationResults` database. The response body will contain the retrieved evaluation result and the Test Results tab should show the successful result of the test assertions:

    ![successful test for pacs.008 retrieval](./images/successful-test-evaluation-retrieval.png)

---
**Footnotes:**
###### 1. A public user not a "member" of the Tazama GitHub organization and does not have access to private repositories that contain hidden rule processors