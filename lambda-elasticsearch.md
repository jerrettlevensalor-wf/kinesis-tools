Title: Using elasticsearch with Kinesis and Lambda
Date: 2016-01-21
Category: InfRe
Tags: Engineering, Kinesis, elasticsearch
Slug: Using elasticsearch with Kinesis and Lambda
Author: Jerrett Levensalor
Summary: Notes on setting up elasticsearch with Kinesis leveraging lambda functions

** Using elasticsearch with Kinesis and Lambda.**

I admit, kinesis streams are pretty darn cool.  The ability to stream large amounts of data and do something useful with the data is not only a challenge but can be quite fun as well.  It's exciting to find new ways to leverage the data.  

This is just a quick overview on how we setup an elasticsearch service to be a consumer of a Kinesis stream levaraging AWS lambda functions and create simple graphs using Kibana in front of elasticsearch for a recent monan jam.  Kudo's to the monan team specifically, Shawn Russaw, Thian-Peng Ter, and Trenton Smith who helped in setting this up.  The following is an overview on the notes throughtout the process.      

** What we built **
1.  A new kinesis stream for streaming data to it
2.  A lambda function that consumes the stream and sends to elasticsearch
3.  An elasticsearch service in AWS
4.  A kibana graph for graphing the data

** What was done already  **
- The endpoint for producing the data and sending to the stream
- If you would like to find out more about the endpoint and streaming to kinesis see the monan team.  



### 1. Setup the Kinesis stream
![Setting up the Kinesis stream](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/kinesis-stream.png "Setting up the Kinesis stream")

### 2. Setup the lambda function
![Setting up the lambda function](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/lambda-function.png "Setting up the lambda function")

##### If you would like more on lambda functions check out [AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
#### Breaking down the node.js file.  
###### Setup the the Globals

```javascript
/* == Imports == */
var AWS = require('aws-sdk');
var path = require('path');

/* == Globals == */
var esDomain = {
    region: 'us-east-1',
    endpoint: 'generated link from elasticsearch service below under Elasticsearch and put here before the region.us-east-1.es.amazonaws.com',
    index: 'monan-jam-index',
    doctype: 'mytype'
};
var endpoint = new AWS.Endpoint(esDomain.endpoint);
/*
 * The AWS credentials are picked up from the environment.
 * They belong to the IAM role assigned to the Lambda function.
 * Since the ES requests are signed using these credentials,
 * make sure to apply a policy that allows ES domain operations
 * to the role.
 */
var creds = new AWS.EnvironmentCredentials('AWS');
```
###### Stetup lambda function to process kinesis records
```javascript
/* Lambda "main": Execution begins here */
exports.handler = function(event, context) {
    console.log(JSON.stringify(event, null, '  '));
    var count = 0;
    jsonDocs = "";
    event.Records.forEach(function(record) {
        count = count + 1;
        var jsonDoc = new Buffer(record.kinesis.data, 'base64');
        var header = {
            "index":{
                "_index": esDomain.index,
                "_type": esDomain.mytype,
                // "_id": record.eventTime + "-" + record.requestID
            }
        };
        jsonDocs += JSON.stringify(header) + "\n";
        jsonDocs += jsonDoc.toString('ascii') + "\n";
        // console.log(jsonDoc.toString('ascii'));
        // json_object = JSON.parse(jsonDoc.toString('ascii'));
        // element = json_object[0];
        // element_string = JSON.stringify(element, null, '  ');
        // console.log(element_string);
        // postToES(element_string, context);
        // postToES(jsonDoc.toString('ascii'), context);
    });
    postToESBulk(jsonDocs, context);
    console.log("MONAN-LOG Processed " + count + " records");
}
```
###### Post the document to Elasticsearch
```javascript
/*
 * Post the given document to Elasticsearch
 */
function postToESBulk(doc, context) {
    var req = new AWS.HttpRequest(endpoint);
    console.log("MONAN-LOG Posting to ES");

    req.method = 'POST';
    req.path = path.join('/', esDomain.index, esDomain.doctype, '_bulk');
    req.region = esDomain.region;
    req.headers['presigned-expires'] = false;
    req.headers['Host'] = endpoint.host;
    req.body = doc;

    var signer = new AWS.Signers.V4(req , 'es');  // es: service code
    signer.addAuthorization(creds, new Date());

    var send = new AWS.NodeHttpClient();
    send.handleRequest(req, null, function(httpResp) {
        console.log("MONAN-LOG ES PostResponseStatusCode: " + httpResp.statusCode);
        var respBody = '';
        httpResp.on('data', function (chunk) {
            respBody += chunk;
        });
        httpResp.on('end', function (chunk) {
            console.log('Response: ' + respBody);
            context.succeed('Lambda added document ' + doc);
        });
    }, function(err) {
        console.log('Error: ' + err);
        context.fail('Lambda failed with error ' + err);
    });
}
```
#### Use monitoring tab to view lambda function performance
![Monitoring lambda](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/lambda-monitor.png "Monitoring lambda function")

#### Using the log output from running test action on lambda function
![Monitoring lambda logs](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/monitor-lambda-log.png "Monitoring lambda logs")


### Setup an elasticsearch service in AWS
![Setting up elasticsearch](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/elasticsearch-initial.png "Setting up elasticsearch")

-  Follow online remaining steps to setup the elasticsearch instance
-  The Kibana interface is used to setup and configure the elasticsearch index

![Setting up elasticsearch-index](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/elasticsearch-initial.png "Setting up elasticsearch-index")

### Setup Kibana graphs
![Validate data in elasticsearch from Kinesis](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/kibana-graphs.png "Kibana graphs")

### Example of a pretty graph using the data streamed to Kinesis.
![Kibana graphs example](/images/2016-01-21-using-elasticsearch-with-kinesis-and-lambda/kibana-graph-example.png "Kibana graphs example")
