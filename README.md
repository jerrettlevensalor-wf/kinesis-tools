# kinesis-tools
Title: Lessons learned with Kinesis troubleshooting
Date: 2016-01-18 10:00
Category: InfRe
Tags: Engineering, Kinesis, troubleshooting
Slug: Lessons-learned-with-kinesis-troubleshooting
Author: Jerrett Levensalor
Summary: A quick note about how to troubleshoot Kinesis streams. 

** Lessons learned with Kinesis troubleshooting.**

Setting up a Kinesis stream in AWS is very straightforward using either the GUI or the awscli tools.  [AWS Kinesis](https://docs.aws.amazon.com/kinesis/latest/dev/introduction.html)

One of the biggest challenges to learning Kinesis was how to troubleshoot the stream after it was setup.  You can send small or large amounts of data to the stream but unless you have an existing consumer of the data or a lambda function to push the data to an endpoint like S3, splunk, elasticsearch, redshift or others it becomes important to validate what is actually in the stream if you want to troubleshoot.  There are simple graphs from within the AWS GUI that comes bundled with Kinesis so you can validate that some kind of data is getting to the stream, but we found that we needed to troubleshoot further, and how do you validate your actual data?

In light of a recent Jam with the monan team, we needed to come up with a way to actively watch the stream to validate what was being written to the stream, before we used AWS lambda functions to push it to elasticsearch.  A few tools we found useful were kinesis-cat(go,ruby) and kinesis-tools(ruby) known as kinesis-tail proved to be very helpful.

**Kinesis-tools**
(Ruby)
Use the steps referenced here [kinesis-tools](https://github.com/AutoScout24/kinesis-tools) to be able to setup this up.

**Here is an example:**

`WF13367:kinesis-tools jerrettlevensalor$ ./kinesis-tail.rb harbour-wk-dev-west

Provides output that will look like this: 

{"host":{"fqdn":"mark-manness-admin-1.internal.harbour","zone":"us-west-2a"},"level":"info","message":"Starting Generate /run/coreos/motd...","name":"daemon","process":{"pid":1,"cmd":"/usr/lib64/systemd/systemd"},"source":"host","timestamp":"2016-01-18T18:39:31.221199Z","type":"log","version":"1.0"}
{"host":{"fqdn":"mark-manness-admin-1.internal.harbour","zone":"us-west-2a"},"level":"info","message":"Started Generate /run/coreos/motd.","name":"daemon","process":{"pid":1,"cmd":"/usr/lib64/systemd/systemd"},"source":"host","timestamp":"2016-01-18T18:39:31.230732Z","type":"log","version":"1.0"}`
Notes:
- Don't forget to update your creds with okta generated keys
- Export your region $ export AWS_REGION=us-west-2 

**Here are a few others if you would like to set those up depending on your preference.**

Kinesis-cat

- [(Ruby)](https://github.com/winebarrel/kinesis_cat)
- [(Go)](https://github.com/winebarrel/kinesis-cat-go)

If you have any plans to work with AWS lambda, then you will end up needing to validate your payloads to the stream, and lambda does have some tools for troubleshooting but if you find you need to dig further in then you may want to consider setting up one of these tools.
Hope this helps.  


