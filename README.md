ActiveMQ Producer and Consumer using Camel (part 3)
====================================================
Author: Matt Robson

Technologies: Fuse, Fabric8, ActiveMQ, Camel, fabric8-maven-plugin, Profiles

Product: Fuse 6.1, ActiveMQ 6.1

Breakdown                                                                                                                     
---------                                                                                                                     
This is a code and profile based example to demonstrate how to build on the Fabric ensemble and broker network you created in part 1, 'fuse-fabric8-getting-started' and part 2, 'fuse-fabric8-ssh-containers'.  We demonstrate how to create an ActiveMQ producer and consumer in Camel that utilizes the ‘amq:’ fabric discovery endpoint. We will also demonstrate how to deploy the code to our fabric using the fabric8-maven-plugin.

For more information see:

* <https://access.redhat.com/site/documentation/JBoss_Fuse/> for more information about Red Hat JBoss Fuse
* <http://www.jboss.org/products/fuse/overview/> for more information about the upstream community
* <http://fabric8.io/> for more information about fabric8

System Requirements
-------------------
Before building out your Fabric, you will need:
* Java 1.7
* JBoss Fuse 6.1
* JBoss ActiveMQ 6.1

Prerequisites
-------------
* A working Fabric and AMQ Broker Network as outlined in part 1, 'fuse-fabric8-getting-started' <https://github.com/mrobson/fuse-fabric8-getting-started> and part 2, 'fuse-fabric8-ssh-containers' <https://github.com/mrobson/fuse-fabric8-ssh-containers>

Build and Deploy
----------------

1) clone the project

        git clone https://github.com/mrobson/fuse-fabric8-amq.git

2) change to project directory

        cd fuse-fabric8-amq/

3) build the project

	mvn clean install

4) deploy the project

	mvn fabric8:deploy -Dfabric8.jolokiaUrl=http://fusefabric1.lab.com:8181/jolokia

This will deploy both producer and consumer profiles.

	fabric-amq-producer
	fabric-amq-consumer

The first, 'fabric-amq-producer', is a basic camel route that uses a timer to generate an event every second, passes it to a bean that sets a message and date and then finishes by sending it to an ActiveMQ queue.  The second, 'fabric-amq-consumer', is also a basic camel route that then consumes from the queue and writes the message out to a file.

The profiles are built and deployed with the necessary dependancies using the fabric8 maven plugin.

	JBossFuse:admin@root> profile-display example-fabric8-amq-producer 
	Profile id: example-fabric8-amq-producer
	Version   : 1.5
	Attributes: 
		parents: feature-camel
	Containers: fuse-services

	Container settings
	----------------------------
	Features : 
		activemq-client
		camel-jms
		camel-blueprint
		fabric-camel
		mq-fabric-camel

	Bundles : 
		mvn:org.mrobson.example.fabric8-amq/producer/1.0-SNAPSHOT

	JBossFuse:admin@root> profile-display example-fabric8-amq-consumer 
	Profile id: example-fabric8-amq-consumer
	Version   : 1.5
	Attributes: 
		parents: feature-camel
	Containers: fuse-services

	Container settings
	----------------------------
	Features : 
		activemq-client
		camel-jms
		camel-blueprint
		fabric-camel
		mq-fabric-camel

	Bundles : 
		mvn:org.mrobson.example.fabric8-amq/consumer/1.0-SNAPSHOT

5) Now that the profiles have been deployed into the fabric, you can assign them to the services container.

	JBossFuse:admin@root> container-add-profile fuse-services example-fabric8-amq-producer

Once the profile loads, you will see it start to produce a message every second.

	2015-06-14 16:05:13,730 | INFO  | /productionTimer | produce-random-message           | rg.apache.camel.util.CamelLogger  176 | 134 - org.apache.camel.camel-core - 2.12.0.redhat-611431 | The message time is: Sun Jun 14 16:05:13 EDT 2015

Now, assign the consumer profile to the same container.  You could assign the consumer to any container you wanted and it would automatically the brokers and consume the messages.

	JBossFuse:admin@root> container-add-profile fuse-services example-fabric8-amq-consumer

Once the profile loads, you will see it start to consume all the previously queued messages and subsequently all messages thereafter.

	2015-06-14 16:05:13,735 | INFO  | er[redhat.queue] | consume-messages                 | rg.apache.camel.util.CamelLogger  176 | 134 - org.apache.camel.camel-core - 2.12.0.redhat-611431 | The message time is: Sun Jun 14 16:05:13 EDT 2015

6) Verify everyone was successfully provisioned

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                                                                         [provision status]
	amq-broker1                    1.5       true        default, mq-broker-redHatBrokerNetwork.redhat-broker1                                              success
	amq-broker2                    1.5       true        default, mq-broker-redHatBrokerNetwork.redhat-broker2                                              success
	fuse-services                  1.5       true        default, mq-client-redHatBrokerNetwork, example-fabric8-amq-producer, example-fabric8-amq-consumer success
	root*                          1.5       true        fabric, fabric-ensemble-0001-1                                                                     success
	root2                          1.5       true        fabric, fabric-ensemble-0001-2                                                                     success
	root3                          1.5       true        fabric, fabric-ensemble-0001-3                                                                     success
	root4                          1.5       true        fabric, fabric-ensemble-0001-4                                                                     success
	root5                          1.5       true        fabric, fabric-ensemble-0001-5                                                                     success

7) You can check the ActiveMQ statistics to see the brokers in action. From 'containers/amq-broker1/fabric8-karaf-1.0.0.redhat-431/bin/' start a './client' session.

	Fabric8:admin@amq-broker1> activemq:dstat 
	Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #    Memory %
	ActiveMQ.Advisory.Connection                                 0           0           0          11           0           0
	ActiveMQ.Advisory.Consumer.Queue.redhat.queue                0           0           1           5           0           0
	ActiveMQ.Advisory.MasterBroker                               0           0           0           1           0           0
	ActiveMQ.Advisory.NetworkBridge                              0           0           0           3           0           0
	ActiveMQ.Advisory.Queue                                      0           0           0           1           0           0
	redhat.queue                                                 0           0           1        4728        4728           0

8) You can also check out the messages being written to the filesystem on the 'fuse-services' node under 'containers/fuse-services/fabric8-karaf-1.0.0.redhat-431/data/activemq'.

	[root@fusefabric5 activemq]# tail -n 5 RedHatConsumer-20150614.txt 
	The message time is: Sun Jun 14 16:12:09 EDT 2015 
	The message time is: Sun Jun 14 16:12:10 EDT 2015 
	The message time is: Sun Jun 14 16:12:11 EDT 2015 
	The message time is: Sun Jun 14 16:12:12 EDT 2015 
	The message time is: Sun Jun 14 16:12:13 EDT 2015

Done!
