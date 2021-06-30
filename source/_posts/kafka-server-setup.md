---
title: Kafka server setup
slug: kafka-server-setup
date: 2021-03-08T13:24:34.000Z
date_updated: 2021-03-08T13:24:34.000Z
tags: kafka
---

Script to start a kafka server
{% codeblock lang:bash %}
#!/bin/sh

setProperty(){
    awk -v pat="^$1=" -v value="$1=$2" '{ if ($0 ~ pat) print value; else print $0; }' $3 > $3.tmp
    mv $3.tmp $3
}

downloadAndPrepare() {
    DIR="kafka/"
    if [[ -d "$DIR" ]]; then
        cd kafka
        rm -rf nohup.out
    else
        wget https://www-us.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz
        tar -xzf kafka_2.12-2.3.0.tgz
        mv kafka_2.12-2.3.0 kafka
        cd kafka
        rm -rf kafka_2.12-2.3.0.tgz
    fi
}

stopServer() {
    PIDS=$(ps ax | grep -i 'kafka\.Kafka' | grep java | grep -v grep | awk '{print $1}')

    if [[ -z "$PIDS" ]]; then
        echo "No kafka server to stop"
    else
        for pid in $PIDS; do kill -9 $pid; done
    fi

    PIDS=$(ps ax | grep java | grep -i QuorumPeerMain | grep -v grep | awk '{print $1}')

    if [[ -z "$PIDS" ]]; then
        echo "No zookeeper server to stop"
    else
        for pid in $PIDS; do kill -9 $pid; done
    fi

}

startServer() {
    nohup bin/zookeeper-server-start.sh config/zookeeper.properties &
    nohup bin/kafka-server-start.sh config/server.properties &
}

helpFunction()
{
    echo ""
    echo "Usage: $0 -n number"
    echo -e "\t-n Number of kafka server to launch"
    exit 1
}

while getopts "n:" opt
do
    case "$opt" in
        n ) number="$OPTARG" ;;
        ? ) helpFunction ;;
    esac
done

if [[ -z "$number" ]] || ! [[ "$number" =~ ^[0-9]+$ ]]
then
    helpFunction
fi

downloadAndPrepare
stopServer
startServer

counter=1
while [[ ${counter} -lt ${number} ]]
do
    cp config/server.properties config/"server-$counter.properties"

    setProperty "broker.id" "$counter" config/"server-$counter.properties"
    setProperty "log.dirs" "/tmp/kafka-logs-$counter" config/"server-$counter.properties"
    echo "listeners=PLAINTEXT://:$((counter+9092))" >> config/"server-$counter.properties"

    nohup bin/kafka-server-start.sh config/"server-$counter.properties" &

    counter=$(( counter+1 ))
done
{% endcodeblock %}


