---
title: Sending JSON to NodeJS to Multiple Topics in Kafka - using kafka-node.
category: ['Linux', 'Webserver', 'Nodejs']
tags: ['linux', 'webserver', 'nodejs', 'kafka-node', 'json', 'kafka']
---

Kafka-node is a Node.js client with Zookeeper integration for Apache Kafka 0.8.1 and later. The Zookeeper integration does the following jobs:

* Loads broker metadata from Zookeeper before we can communicate with the Kafka server
* Watches broker state, if broker changes, the client will refresh broker and topic metadata stored in the client

What we are trying to achieve ?

1. Send `json` from and browser/`curl` to `nodejs`.
2. `nodejs` will redirect `json` data based on url to each `topic` in `kafka`. Example : URL `/upload/topic/A` will send the `json` to `topic_a` in `kafka`
3. Further processing is done on `kafka`.
4. We can then see the `json` arrival in `kafka`, using `kafka-console-consumer.sh` script.

### Step 1 : Get the `json_nodejs_multiple_topics.js` from git

Code Link : [json_nodejs_multiple_topics.js](https://raw.githubusercontent.com/zubayr/kafka-nodejs/master/send_json_multiple_topics/json_nodejs_multiple_topics.js)


### Step 2 : Start above script on the `nodejs` server.

	[nodejs-admin@nodejs nodejs]$ vim json_nodejs_multiple_topics.js
	[nodejs-admin@nodejs nodejs]$ node json_nodejs_multiple_topics.js


### Step 3 : Execute `curl` command to send the JSON to `nodejs`.

	[nodejs-admin@nodejs nodejs]$ curl -H "Content-Type: application/json" \
                    -d '{"username":"xyz","password":"xyz"}' http://localhost:8125/upload/topic/A
	[nodejs-admin@nodejs nodejs]$ curl -H "Content-Type: application/json" \
                    -d '{"username":"abc","password":"xyz"}' http://localhost:8125/upload/topic/B
	[nodejs-admin@nodejs nodejs]$ curl -H "Content-Type: application/json" \
                    -d '{"username":"efg","password":"xyz"}' http://localhost:8125/upload/topic/C
	[nodejs-admin@nodejs nodejs]$ curl -H "Content-Type: application/json" \
                    -d '{"username":"efg","password":"xyz"}' http://localhost:8125/upload/topic/D


### Step 4 :  Output on nodejs console

    [nginx-admin@nginx nodejs]$ node json_nodejs_multiple_topics.js 
    For Topic A
    {"username":"xyz","password":"xyz"}
    { topic_a: { '0': 16 } }
    For Topic B
    {"username":"abc","password":"xyz"}
    { topic_b: { '0': 1 } }
    For Topic C
    {"username":"efg","password":"xyz"}
    { topic_c: { '0': 0 } }
    ERROR: Could not Process this URL :/upload/topic/D
    {"username":"efg","password":"xyz"}

`{"username":"xyz","password":"xyz"}` request from the `curl` command.
`{ test: { '0': 29 } }` response from the kafka cluster that, it has received the `json`.


#### Step5 : Output on the `kafka` consumer side.

NOTE : Assuming that we have already created topics in kafka. using below command. 
    
    [kafka-admin@kafka kafka]$ bin/kafka-topics.sh --create --zookeeper localhost:2181 \
                                            --replication-factor 1 --partitions 1 --topic topic_a
    [kafka-admin@kafka kafka]$ bin/kafka-topics.sh --create --zookeeper localhost:2181 \
                                            --replication-factor 1 --partitions 1 --topic topic_b
    [kafka-admin@kafka kafka]$ bin/kafka-topics.sh --create --zookeeper localhost:2181 \
                                            --replication-factor 1 --partitions 1 --topic topic_c
    [kafka-admin@kafka kafka_2.9.2-0.8.2.0]$ bin/kafka-topics.sh --list --zookeeper localhost:2181
    topic_a
    topic_b
    topic_c
    [kafka-admin@kafka kafka_2.9.2-0.8.2.0]$ 

Here is the output after running `curl` command on the `nodejs` server

	[kafka-admin@kafka kafka_2.9.2-0.8.2.0]$ bin/kafka-console-consumer.sh \
                                        --zookeeper localhost:2181 --topic topic_a --from-beginning
	{"username":"xyz","password":"xyz"}

	[kafka-admin@kafka kafka_2.9.2-0.8.2.0]$ bin/kafka-console-consumer.sh \
                                        --zookeeper localhost:2181 --topic topic_b --from-beginning
	{"username":"abc","password":"xyz"}
	
	[kafka-admin@kafka kafka_2.9.2-0.8.2.0]$ bin/kafka-console-consumer.sh \
                                        --zookeeper localhost:2181 --topic topic_c --from-beginning
	{"username":"efg","password":"xyz"}	

`{"username":"xyz","password":"xyz"}` data received from `nodejs` server.
`{"username":"abc","password":"xyz"}` data received from `nodejs` server.
`{"username":"efg","password":"xyz"}` data received from `nodejs` server.



##### Useful Links.

	http://whatizee.blogspot.in/2015/02/installing-kafka-single-node-quick-start.html
	http://whatizee.blogspot.in/2015/02/installing-nodejs-on-centos-66.html
	http://whatizee.blogspot.in/2015/02/sending-json-nodejs-kafka.html
	https://github.com/zubayr/kafka-nodejs/blob/master/send_json_multiple_topics/README.md
