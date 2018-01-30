---
title: NodeJS Kafka Producer - Using `kafka-node`
category: ['Linux', 'Webserver', 'Nodejs']
tags: ['linux', 'webserver', 'nodejs', 'kafka-node', 'kafka']
---

Now that we have Kafka and NodeJS ready. Lets some data to our Kafka Cluster.

Below is a basic producer code. 

below are the Server Details.

1. `nodejs` is the nodejs server.
2. `kafka` is the kafka server (single node).


#### Step 1: Copy the below script in a file called `producer_nodejs.js`.

	/*
		Basic producer to send data to kafka from nodejs.
		More Information Here : https://www.npmjs.com/package/kafka-node
	*/
	 
	//	Using kafka-node - really nice library
	//	create a producer and connect to a Zookeeper to send the payloads.
	var kafka = require('kafka-node'),
	    Producer = kafka.Producer,
	    client = new kafka.Client('kafka:2181'),
	    producer = new Producer(client);
	
		/*
			Creating a payload, which takes below information
			'topic' 	-->	this is the topic we have created in kafka. (test)
			'messages' 	-->	data which needs to be sent to kafka. (JSON in our case)
			'partition' -->	which partition should we send the request to. (default)
							
							example command to create a topic in kafka: 
							[kafka@kafka kafka]$ bin/kafka-topics.sh \
										--create --zookeeper localhost:2181 \
										--replication-factor 1 \
										--partitions 1 \
										--topic test
							
							If there are multiple partition, then we optimize the code here,
							so that we send request to different partitions. 
							
		*/
		payloads = [
	        { topic: 'test', messages: 'This is the First Message I am sending', partition: 0 },
	    ];
	
	
	//	producer 'on' ready to send payload to kafka.
	producer.on('ready', function(){
		producer.send(payloads, function(err, data){
			console.log(data)
		});
	});
	
	producer.on('error', function(err){}


#### Step 2 : Start the `kafka` cluster as we already did in `Installation of Kafka`. Assuming topic as `test`

#### Step 3 : Start the consumer service as in the below command.

	[kafka-admin@kafka kafka]$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 \
                                                                        --topic test --from-beginning

#### Step 4 : Execute below command. This will send `This is the First Message I am sending` Message to the Kafka consumer. 


	[nodejs-admin@nodejs nodejs]$ node producer_nodejs.js


#### Step 5 : Check on the consumer you will see the message sent from `nodejs`.


	[kafka-admin@kafka kafka]$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 \
                                                                      --topic test --from-beginning
	This is a message
	This is another message here 
	This is the First Message I am sending

