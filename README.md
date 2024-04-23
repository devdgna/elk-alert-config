# elk-alert-config
Stepwise process on how to enable Alerts for ELK Stack
## Prerequisites
1. [Docker](https://docs.docker.com/engine/install/ubuntu/) and [Docker Compose](https://docs.docker.com/compose/install/linux/)
2. SSL Certificate:- get one from [OpenSSL](https://www.openssl.org/)
It is assumed that you're on a Linux distribution as this doc uses Linux related commands.

## Docker Network
Docker network needs to be created for inter-container communication between ELK containers.

For this task, create a network as shown below.The network name is `elastic` but you can give any name of your choice.

Run: `docker network create elastic` inside your current working directory

To see the network was created successfully, run `docker network ls`, and you will see something like this:

```
NETWORK ID     NAME             DRIVER    SCOPE
5271435d0f3a   bridge           bridge    local
95bb21450d83   elastic          bridge    local
```
## Generate SSL
This document uses a self-signed certificate from OpenSSL as mentioned under prerequisites.

1. Create a new directory called `certs` inside the current working directory:
   
   `mkdir -p certs`

3. Change the current drectory to `certs`:
   
   `cd certs`

5. Generate the SSL.

> Note: You need to have a registered domain to route your ELK stack. Therefore, configure an appropriate DNS for yourself. This document uses a domain called `kibana-test.devdesign.com`

Once you have the domain, run the below command. Makes ure to put your domain in the command in place of mine:

`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout kibana.key -out kibana.crt -subj "/CN=kibana-test.devdesign.com"`

4. Come out of `certs` directory

   `cd ..`
## Creating necessary directories
From your current working directory, make a folder named `logstash/config`

`mkdir -p logstash/config`

`cd` to `logstash/config`

Now create a file named `logstash.yml` inside of it.

Add these two lines to it:

```
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
```
Now `cd ..` to `logstash` folder and create another directory called `pipeline`.

Inside `pipeline` directory, create a file named `logstash.conf`. Make sure to add `user` and `password` appropriately.

```
input {
  file {
    path => "/home/ubuntu/workspace/logstash/access.log"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logstash-logs-%{+YYYY.MM.dd}"
    user => "admin"
    password => "password"
  }
}
```

## Adding necessary file permissions
From the current working directory, add these permissions now:

`chmod 644 ./certs/*`

`chmod 644 ./logstash/config/logstash.yml`

`chmod 644 ./logstash/pipeline/logstash.conf`

## Create the docker file
```
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elasticnet

  logstash:
    image: docker.elastic.co/logstash/logstash:7.10.0
    container_name: logstash
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - 5000:5000
    networks:
      - elasticnet
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.10.0
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=<32_character_long>
    ports:
      - "5601:5601"
    networks:
      - elasticnet
    depends_on:
      - elasticsearch

volumes:
  esdata1:
    driver: local

networks:
  elasticnet:
    driver: bridge
```

## Deploy ELK
`docker compose up -d`
