# kafka

Why do we have to balance topics in a Kafka cluster?

Whenever a Kafka node is down, the load of that server is distributed to the other nodes in the cluster and this distribution is not even, i.e the load is not distributed evenly across all nodes in the cluster. We need to do some steps to achieve this balancing (also called rebalancing).

There are two things that people usually mean when they talk about rebalancing. One is leader re-election, or preferred replica election and the other one is partition rebalancing. The first one is more of a case when a node is down and is brought back again within a certain period of time . The other one is when we want to either decrease or increase the number of nodes in the cluster. Let’s see how to deal with these different scenarios to balance topics.

All the examples below assume a 5-broker Kafka cluster.

Scenario 1: When the broker is down because of maintenance or due to server failure and is brought back within a certain period of time.

There are two ways to handle this scenario. One is adding the following line to the broker configuration “auto.leader.rebalance.enable” to automatically rebalance, but this is reported to have issues. The other way is to manually use the “kafka-preferred-replica-election.sh” tool.

Edit server.properties of Kafka to add the following line,auto.leader.rebalance.enable = false

ajdakdha

Let us bring broker4 down and see how the topic load is distributed.image01

Here we could see that the load is not evenly distributed. Let’s bring back broker 4 online.image12

Now we can see that even though the node 4 is back online, it is not serving as a leader for any of the partitions. Let’s run the kafka-preferred-replica-election.sh tool to balance the loadimage11

image02

Now we can see that topics are evenly balanced.

Scenario 2: When a node is down and not recoverable.

We create a new broker and update the broker.id with the previous one’s id which was not recoverable and manually run “kafka-preferred-replica-election.sh” for topic balancing.

Scenario 3: To increase or decrease the number of nodes in a Kafka cluster.

Following are the steps to  balance topics when increase or decreasing number of nodes.

    Using partition reassignment tool (kafka-reassign-partition.sh), generate (with the –generate option) the candidate assignment configuration. This shows the current and the proposed replica assignments.
    Copy the proposed assignment to a JSON file.
    Execute the partition reassignment tool (kafka-reassign-partition.sh –execute) to update the metadata for balancing. Make sure run this at a time when there is not much load on the cluster as this moves the data between different available nodes.
    Wait for a while (based on the amount of data that has to move around) and verify that the balancing is completed successfully with –verify option.
    Once the partition reassignment is completed, run “kafka-preferred-replica-election.sh” tool to complete the balancing.

Let’s us assume that we want to decrease the number of nodes in the cluster and bring down one of the nodes (broker 5). Once the broker is terminated, we generate a candidate assignment config using the partition reassignment tool which shows current and proposed replica assignments as JSONs.

image10

Copy the proposed reassignment configuration to a JSON file and execute the partition reassignment tool. This updates the metadata and then starts to move data around to balance the load. image05

image09

Next we verify that the rebalance/reassignment is successful.image00

Once the reassignment is successful for all partitions, we run the preferred replica election tool to balance the topics and then run ‘describe topic’ to check balancing of topics.image03

image07

Now we can see that the topics (along with leaders and replicas) are all balanced.
