[![Build Status](https://travis-ci.org/SolaceLabs/mqtt-3.1.1-test.svg?branch=master)](https://travis-ci.org/SolaceLabs/mqtt-3.1.1-test)

## Synopsis

This project provides a simple example that shows how to use MQTT Paho Java 1.2.0 features of auto-reconnect and publish while offline.  These features enhance the MQTT system high availability. 

## Motivation

This project is the base example from which interoperability with Solace message router high availability features of router redundancy and disaster recovery where tested. 

## Checking out and Building

This project depends on maven for building. To build the class locally, check out the project and build from source by doing the following:

    git clone https://github.com/SolaceLabs/mqtt-3.1.1-test
    cd /mqtt-3.1.1-test
    mvn compile
      or
    gradle build
   
## Code Example

The interesting bits of code can be seen here, the initialize asyncClient, create host list with auto-reconnect and publish buffer:

```java
public class SampleMqttClient implements MqttCallbackExtended {

connOpt = new MqttConnectOptions();
connOpt.setCleanSession(true);
connOpt.setKeepAliveInterval(30);
connOpt.setConnectionTimeout(60);
connOpt.setAutomaticReconnect(true);

String[] brokerList = new String[2];
brokerList[0] = BROKER_URL1;
brokerList[1] = BROKER_URL2;
connOpt.setServerURIs(brokerList);

DisconnectedBufferOptions bufferOpts = new DisconnectedBufferOptions();
bufferOpts.setBufferEnabled(true);
bufferOpts.setBufferSize(100);            // 100 message buffer
bufferOpts.setDeleteOldestMessages(true); // Purge oldest messages when buffer is full
bufferOpts.setPersistBuffer(false);       // Do not buffer to disk

try {
myClient = new MqttAsyncClient(brokerList[0], clientID, persistence);
myClient.setBufferOpts(bufferOpts);
myClient.setCallback(this);
            myClient.connect(connOpt);
```

Also there is a new callback on connection complete:

```java
public void connectComplete(boolean reconnect, String serverURI) {
if (reconnect) {
System.out.println("Connection Reconnected! To: " + serverURI);
} else {
System.out.println("Initial Connection! To: " + serverURI);
}
addSubscriptions();
}
```

## Tests

There are no tests with this project **yet**. 
Tests should be: 

1. Publish before initial connection.  Async MQTT client calls connect then publish before connectComplete() is called.
- Ensure message is stored as per persistent model then published once connection is established.
2. Cause reconnection. Once connected, disconnect the client at the message broker.
- Ensure the client reconnects, validate subscription still exists for Clean=0 and do not exist for clean=1.
3. Publish while reconnecting.  Connect client, publish at a rate of ~20/sec.  Disconnect client at the broker.
- Ensure reconnection and messages losslessly published in correct order.
4. Cause failover.  Once connected, shut down access to connected MQTT router.
- Ensure failover to next broker in host list. Subscriptions should re-add as per clean flag.

## Contributing

Please read [CONTRIBUTING](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors

See the list of [contributors](https://github.com/SolaceLabs/mqtt-3.1.1-test/graphs/contributors) who participated in this project.

## License

This project is licensed under the Apache License, Version 2.0. - See the [LICENSE](LICENSE) file for details.

## Resources

For more information about working with Paho MQTT APIs:
- [Eclipse PAho MQTT APIs](http://www.eclipse.org/paho/)

For more information about Solace technology in general please visit these resources:
- The Solace Developer Portal website at: http://dev.solace.com
- Understanding [Solace technology.](http://dev.solace.com/tech/)
- Ask the [Solace community](http://dev.solace.com/community/).
