# kafka - Role to install kafka

#### Prerequisites
- **Java 1.8** - Can be installed using `java_1_8` role.
- **Zookeeper** - Can be installed using `zKeeper` role.

#### Settings/Variables

Following variables can be overwritten by passing `extra-vars` option, or in a template created using *ansible tower*

```yaml
---
kafka_home: "/etc/installers/kafka"
kafka:
  home: "{{ kafka_home }}"
  version: 2.5.0
  scala_version: 2.13
  data_dir: "{{ kafka_home }}/data"
  log_dir: "{{ kafka_home }}/logs"
  hosts_file: "/etc/hosts"
  daemon:
    name: "kafka"
    type: "service" # systemctl
    start_on_boot: "yes" # no
  zookeeper:
    chroot: kafka

```

#### Inventory/hosts
```
[kafka_hosts]
kafka1 ansible_host=25.201.182.24 alias=kafka1 private_ip=172.12.26.124 broker_id=1 public_ip=25.201.182.24 port=9092
kafka2 ansible_host=25.201.172.56 alias=kafka2 private_ip=172.18.37.115 broker_id=2 public_ip=25.201.172.56 port=9093
kafka3 ansible_host=25.201.176.52 alias=kafka3 private_ip=172.89.23.126 broker_id=3 public_ip=25.201.176.52 port=9094

[zookeeper_hosts]
zookeeper1 ansible_host=91.28.240.78 alias=zookeeper1 private_ip=172.56.29.25 zookeeper_id=1 client_port=2181
zookeeper2 ansible_host=91.85.139.81 alias=zookeeper2 private_ip=172.45.12.83 zookeeper_id=2 client_port=2181
zookeeper3 ansible_host=91.46.152.93 alias=zookeeper3 private_ip=172.84.68.97 zookeeper_id=3 client_port=2181
```

#### Playbook
Sample playbook can be found in `kafka/tests` directory. To install kafka along with all it's dependencies, use
`wfx_kafka.yml` playbook.

```yaml
- hosts: kafka_hosts
  vars:
    ansible_python_interpreter: /usr/bin/python3
  roles:
    - role: kafka
      become: yes
```

#### Test
```
# Create topic
bin/kafka-topics.sh --zookeeper zookeeper1:2181,zookeeper2:2181,zookeeper:2181/kafka --create --topic second_topic --replication-factor 3 --partitions 3

# Publish data to Kafka using the bootstrap server list!
bin/kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9093,kafka3:9094 --topic second_topic

# Read data using any broker too!
bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9093,kafka3:9094 --topic second_topic --from-beginning

# Delete the topic
bin/kafka-topics.sh --zookeeper zookeeper1:2181,zookeeper2:2181,zookeeper3:2181/kafka --delete --topic second_topic

# List of topics
bin/kafka-topics.sh --zookeeper zookeeper1:2181,zookeeper2:2181,zookeeper3:2181/kafka --list
```
