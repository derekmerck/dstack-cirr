# RIH Clinical Imaging Research Registry Stack

Derek Merck  
<derek_merck@brown.edu>  
Rhode Island Hospital and Brown University  
Providence, RI  


## Overview

Deploys:

- A SQL backend service with bind-mounted storage (Postgresql)
- A DICOM mass archive with bind-mounted storage (Orthanc)
- A DICOM Q/R mass archive interface (Orthanc)
- A DICOM ingress and simple routing mesh (Orthanc)
- A DICOM Q/R brige service for external services (PACS)

The bridge service can be manipulated using DIANA watcher scripts to monitor and index the PACS or exfiltrate and anonymize large data collections.


## Usage

### Provision Some Nodes

1. Setup some nodes with docker and `docker-compose`.

See cloud-init: <https://gist.github.com/derekmerck/7b55c34c91954e84aa155e487ffe2e8d>

Needs Docker >18? for ingress routing.

The node(s) that support storage-bound operations (database, image archive) should have paths pre-created for Docker to use as persistent storage.

```yaml
$ mkdir -p /data/orthanc/db
$ mkdir -p /data/postgres/data
```

### Setup the Swarm

2. Create a Docker swarm

```bash
$ docker swarm init --advertise-addr <ip_addr>
$ ssh host2
> docker swarm join ... etc
```

2. Tag unique nodes for the scheduler

The `storage` node will be assigned the database backend and DICOM mass archive accessors.  
Any `bridge` nodes will be assigned DICOM ingress, routing, and bridging services (b/c typically modalities register and allow endpoint access by specific IP addrs)

```bash
$ docker node update --label-add storage=true host1   # mounts mass storage
$ docker node update --label-add bridge=true host2    # registered IP address for DICOM receipt
```

3. Add the Portainer agent so you can watch the system

```bash
$ curl -L https://portainer.io/download/portainer-agent-stack.yml -o portainer-agent-stack.yml
$ docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```


### Setup the CIRR

4. Set Variables for Abstractions and Secrets

Create a `cirr.env` file and copy and source it to 

```yaml
DATA_DIR=/data
ORTHANC_PG_DATABASE=orthanc
ORTHANC_PASSWORD=orthanc
POSTGRES_PASSWORD=postgres
```

5. Start Up the Service Stack

```bash
$ . cirr.env && docker stack deploy --compose-file=cirr-stack.yml cirr
```

Note that if volumes are created on a node, they are not removed when the stack is removed.  Manually remove them to clear errors about directories not found.





### Deploy CIRR Stack Template by Hand


Set your environment vars (with `.env`, for example)

```bash
$ docker-compose -f setup.docker-compose.yml up  # Creates reverse-proxy and portainer
$ docker-compose up
$ docker-compose scale orthanc=3
```

Alternatively, you can log into Portainer and add the stack template "RIH-CIRR", although it does not support service scaling.




## License

MIT