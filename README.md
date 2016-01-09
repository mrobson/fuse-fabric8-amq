ActiveMQ Producer and Consumer using Camel (part 3)
===================================================
Author: Matt Robson

Technologies: Fuse, Fabric8, ActiveMQ, Camel, fabric8-maven-plugin, Profiles

Product: Fuse 6.2.1, ActiveMQ 6.2.1

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
* Java 1.7 or 1.8
* JBoss Fuse 6.2.1
* JBoss ActiveMQ 6.2.1

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

4) deploy the bundles and feature file

	mvn bundle:deploy

5) create and upload the profiles

	mvn fabric8:deploy -Dfabric8.jolokiaUrl=http://fusefabric1.lab.com:8181/jolokia

This will create and upload both producer and consumer profiles.

	example-fabric8-amq-producer
	example-fabric8-amq-consumer

The first, 'example-fabric8-amq-producer', is a basic camel route that uses a timer to generate an event every second, passes it to a bean that sets a message and date and then finishes by sending it to an ActiveMQ queue.  The second, 'example-fabric8-amq-consumer', is also a basic camel route that then consumes from the queue and writes the message out to a file.

The profiles are built and deployed with the necessary dependancies using the fabric8 maven plugin.

	JBossFuse:admin@root> profile-display example-fabric8-amq-producer 
	Profile id: example-fabric8-amq-producer
	Version   : 1.0
	Attributes: 
		abstract: false
		parents: feature-camel
	Containers: fuse-services1

	Container settings
	----------------------------
	Repositories : 
		mvn:org.mrobson.example.fabric8-amq/amq-pc-features/1.0-SNAPSHOT/xml/features

	Features : 
		fuse-fabric8-amq-producer

	Configuration details
	----------------------------

	Other resources
	----------------------------
	Resource: README.md

	JBossFuse:admin@root> profile-display example-fabric8-amq-consumer
	Profile id: example-fabric8-amq-consumer
	Version   : 1.0
	Attributes: 
		abstract: false
		parents: feature-camel
	Containers: fuse-services1

	Container settings
	----------------------------
	Repositories : 
		mvn:org.mrobson.example.fabric8-amq/amq-pc-features/1.0-SNAPSHOT/xml/features

	Features : 
		fuse-fabric8-amq-consumer

	Configuration details
	----------------------------
	Resource: README.md


6) Now that the profiles have been deployed into the fabric, you can assign them to the services container.

	JBossFuse:admin@root> container-add-profile fuse-services example-fabric8-amq-producer

Once the profile loads, you will see it start to produce a message every second.

	2016-01-09 09:54:34,607 | INFO  | /productionTimer | produce-random-message | 144 - org.apache.camel.camel-core - 2.15.1.redhat-621084 | The message time is: Sat Jan 09 09:54:34 EST 2016

Now, assign the consumer profile to the same container.  You could assign the consumer to any container you wanted and it would automatically the brokers and consume the messages.

	JBossFuse:admin@root> container-add-profile fuse-services example-fabric8-amq-consumer

Once the profile loads, you will see it start to consume all the previously queued messages and subsequently all messages thereafter.

	2016-01-09 09:54:34,611 | INFO  | m.redhat.queue1] | consume-messages | 144 - org.apache.camel.camel-core - 2.15.1.redhat-621084 | The message time is: Sat Jan 09 09:54:34 EST 2016

7) Verify everyone was successfully provisioned

	JBossFuse:admin@root> container-list 
	[id]                           [version] [connected] [profiles]                                                                                         [provision status]
	amq-broker1                    1.0       true        default, mq-broker-redHatBrokerNetwork.redhat-broker1                                              success
	amq-broker2                    1.0       true        default, mq-broker-redHatBrokerNetwork.redhat-broker2                                              success
	amq-broker3                    1.0       true        default, mq-broker-redHatBrokerNetwork.redhat-broker3                                              success
	fuse-services1                 1.0       true        default, mq-client-redHatBrokerNetwork, example-fabric8-amq-producer, example-fabric8-amq-consumer success
	root*                          1.0       true        fabric, fabric-ensemble-0001-1                                                                     success
	root2                          1.0       true        fabric, fabric-ensemble-0001-2                                                                     success
	root3                          1.0       true        fabric, fabric-ensemble-0001-3                                                                     success

8) You can check the ActiveMQ statistics to see the brokers in action. From 'containers/amq-broker1/fabric8-karaf-1.0.0.redhat-431/bin/' start a './client' session.

	Fabric8:admin@amq-broker3> activemq:dstat 
	Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #
	ActiveMQ.Advisory.Connection                                 0           0           0          10           0           0
	ActiveMQ.Advisory.Consumer.Queue.com.redhat.queue1           0           0           2           5           0           0
	ActiveMQ.Advisory.MasterBroker                               0           0           0           1           0           0
	ActiveMQ.Advisory.NetworkBridge                              0           0           0           4           0           0
	ActiveMQ.Advisory.Queue                                      0           0           0           2           0           0
	com.redhat.queue1                                            0           0           1         520         520           0

9) You can also check out the messages being written to the filesystem on the 'fuse-services' node under 'containers/fuse-services/fabric8-karaf-1.0.0.redhat-431/data/activemq'.

	[root@fusefabric5 activemq]# tail -n 5 RedHatConsumer-20160109.txt
	The message time is: Sat Jan 09 09:02:15 EST 2016
	The message time is: Sat Jan 09 09:02:16 EST 2016
	The message time is: Sat Jan 09 09:02:17 EST 2016
	The message time is: Sat Jan 09 09:02:18 EST 2016
	The message time is: Sat Jan 09 09:02:19 EST 2016

Done!
