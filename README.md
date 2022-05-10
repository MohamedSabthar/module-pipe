# Pipe Package for Ballerina

## Summary

The pipe package is primarily a blocking queue based on producer-consumer architecture. Data can be simultaneously sent and received through these pipes. The proposal is to introduce this model as a new package for Ballerina.

## Goals

* Implement a new package to provide APIs to send/receive data concurrently. 


## Motivation

Ballerina has some features that are running on event-driven architecture. For example, the subscription feature in GraphQL is designed to return data when a specific event is triggered. 
To satisfy this requirement, there must be a data transmission medium with APIs to return data, whenever data is received. The pipe model is designed to implement this functionality. 

## Description

The Pipe allows you to send data from one place to another. Following are the APIs of the Pipe. There is one method to produce and two methods to consume. Users can use either option as they wish but not both at once. The implementation of the `consumeStream` usually uses the `consume` method to provide its functionality.


```ballerina 
    public class Pipe {

        public function init(int 'limit) { ...//sets a limit }
        
        // produces data into the pipe
        // if full, blocks
        public isolated function produce(any data, decimal timeout) = external;
    
        // returns data in the pipe
        // if empty, blocks
        public isolated function consume(decimal timeout, typedesc<any> t = <>) returns t|error = external;
    
        // returns a stream 
        // data produced to the pipe can be fetched by the stream
        public isolated function consumeStream(decimal timeout, typedesc<any> t|error = <>) returns stream<t, error?> = external;
    
    }
```


The pipe can hold up to n number of data. In case the pipe is full, the `producer` method blocks until there is a free slot to produce data. On the other hand, in case the pipe is empty, the `consumer` method blocks until there is some data to consume. This behavior is somewhat similar to `go channels`. A timeout must be set to the `produce` and `consume` methods to regulate the waiting time in the blocking state.


### Closing Pipes

Closing a pipe can be complicated because there can be APIs running during the closing process. Therefore when the closing method is invoked, no data can be sent to the pipe. But the remaining data in the pipe can be consumed for a specific period. After that period, all the data is removed and the pipe instance is taken by the garbage collector. 
