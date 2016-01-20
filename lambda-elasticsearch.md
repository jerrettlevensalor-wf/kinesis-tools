Title: Using elasticsearch with Kinesis and Lambda
Date: 2016-01-21
Category: InfRe
Tags: Engineering, Kinesis, elasticsearch
Slug: Using elasticsearch with Kinesis and Lambda
Author: Jerrett Levensalor
Summary: Notes on setting up elasticsearch with Kinesis leveraging lambda functions 

** Using elasticsearch with Kinesis and Lambda.**

Kinesis streams are pretty darn cool.  The ability to stream large amounts of data and do something useful with the
data is not only the challenge but also the fun in finding out new ways to leverage the data.  

This is just a quick overview on how we setup an elasticsearch service to be a consumer of a Kinesis stream levaraging 
AWS lambda functions and create some simple graphs using Kibana in front of elasticsearch.  

** What we built **
- A new kinesis stream for streaming data to it
- A lambda function that consumes the stream and sends to elasticsearch
- An elasticsearch service in AWS 
- A kibana graph for graphing the data

** What was done already **
- The endpoint for producing the data and sending to the stream 
- If you would like to find out more about the endpoint and streaming to kinesis see the monan team (Trenton Smith)


Setup the Kinesis stream
- image
- Notes

Setup the lambda function
- Image
- Notes

Setup an elasticsearch service in AWS
- Image
- Notes

Setup Kibana graphs
- Image
- Notes
