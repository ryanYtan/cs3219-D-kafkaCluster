# CS3219 Task D Report

## Running the Cluster

1. Clone the repo
2. `cd` into the repo directory
3. To start the cluster, run `docker-compose up -d`
4. To stop, run `docker-compose down`

## Sending and Receiving Messages

Connect to `kafka1` by running

```
$ docker exec -it kafka1 /bin/bash
```

To create a topic, run

```
$ kafka-topics.sh --create --topic techhelp \
        --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
Created topic techhelp.
```

To write to a topic, run

```
$ kafka-console-producer.sh --topic techhelp \
        --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
```


A prompt (`>`) will appear on-screen, type any number of lines (each line is one
message), and end with `Ctrl+C`. An example session is shown below:

```
bash-4.4# kafka-console-producer.sh --topic techhelp \
>         --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
>remove the ram
>install the new ram
>^Cbash-4.4#
```

Exit from `kafka` using `Ctrl+D`, and go into `kafka2` by running

```
$ docker exec -it kafka2 /bin/bash
```

To receive from a topic, run

```
$ kafka-console-consumer.sh --topic techhelp --from-beginning \
        --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
```

An example session is shwon below:

```
bash-4.4# kafka-console-consumer.sh --topic techhelp --from-beginning \
>         --bootstrap-server kafka1:9092, kafka2:9092, kafka3:9092
remove the ram
install the new ram
^CProcessed a total of 2 messages
bash-4.4#
```

## Failover
First, we need to figure out which zookeeper is the leader. Run the following
command

```
docker exec -it zookeeper1 /bin/bash
```

Now, run

```
$ zkServer.sh status
```

Look for `Mode`, it should either say `leader` or `follower`. An example
session is shown below:

```bash
root@zookeeper1:/apache-zookeeper-3.6.2-bin# zkServer.sh  status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower # find this
root@zookeeper1:/apache-zookeeper-3.6.2-bin#
```

Do this for each `zookeeper` until you find the `leader`. Once found, exit
from the `zookeeper` node and run

```bash
$ docker stop <ZOOKEEPER_CONTAINER_NAME> # e.g docker stop zookeeper3
```

Go back into the other zookeeper nodes and check if one of the nodes has
become the leader. Repeat the steps under "Sending and Receiving Messages"
to test if the pub/sub system still works.
