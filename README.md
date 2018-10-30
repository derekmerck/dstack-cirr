# RIH Clinical Imaging Research Registry Stack

Derek Merck  
<derek_merck@brown.edu>  
Rhode Island Hospital and Brown University  
Providence, RI  


## Usage

### Setup Some Nodes

See cloud-init: <https://gist.github.com/derekmerck/7b55c34c91954e84aa155e487ffe2e8d>

### Setup Swarm

1. Create a Docker swarm

```bash
$ docker swarm init --advertise-addr <ip_addr>
$ ssh host2
> docker swarm join ... etc
```

2. Tag the storage node

```bash
$ docker node update --label-add storage=true host1
$ docker node update --label-add storage_dir=/data host1
```

2. Add the Portainer agent

```bash
$ curl -L https://portainer.io/download/portainer-agent-stack.yml -o portainer-agent-stack.yml
$ docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```




TODO: Need to insert our stack template

Now you can login to <http://host1:9000> and deploy the CIRR stack


### Setup Cluster Secrets

$ docker stack deploy --compose-file=docker-stack.yml postgres


### Deploy CIRR Stack Template by Hand


Set your environment vars (with `.env`, for example)

```bash
$ docker-compose -f setup.docker-compose.yml up  # Creates reverse-proxy and portainer
$ docker-compose up
$ docker-compose scale orthanc=3
```

Alternatively, you can log into Portainer and add the stack template "RIH-CIRR", although it does not support service scaling.


## Required Environment Vars

```bash
DATA_DIR=/data
ORTHANC_PG_DATABASE=orthanc
ORTHANC_PASSWORD=passw0rd!
POSTGRES_PASSWORD=passw0rd!
```

## License

MIT