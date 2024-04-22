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
