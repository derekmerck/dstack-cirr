# Image Registry Stacks

Derek Merck  
<derek_merck@brown.edu>  
Rhode Island Hospital and Brown University  
Providence, RI  


Examples of deploying DICOM image registry stacks using Docker Swarm.


## The RIH Clincial Imaging Research Registry Core

The RIH CIRR was the initial development site for these configurations.

### Overview

Deploys:

- A SQL backend service with bind-mounted storage (Postgresql)
- A DICOM mass archive (Orthanc)
- A simple DICOM routing service that multiplexes to the archive and $MOD_WRKSTN (Orthanc)
- A DICOM Q/R bridge to $MOD_BRIDGE for external data pulls (Orthanc)

The bridge service can be manipulated using DIANA watcher scripts to monitor and index the PACS, or to exfiltrate and anonymize large data collections.


### Usage

At RIH, the CIRR runs in production on a pair of 16-core Xeon servers with 200GB of RAM each.  One node has an attached iSCSI interface to a 45TB StorSimple.  For staging, we use two disposable desktop-type machines with 4-8GB of RAM and about 1TB of disk.


#### Provision A Development Environment

1. Provision a couple of nodes with docker and `docker-compose`.

See cloud-init: <https://gist.github.com/derekmerck/7b55c34c91954e84aa155e487ffe2e8d>  Requires Docker v>18 for ingress routing.

The node that supports storage-bound operations (PostgreSQL, Splunk) should have directories pre-created for Docker to use as persistent storage.

```yaml
$ mkdir -p /$DATA_DIR/splunk
$ mkdir -p /$DATA_DIR/postgres
```

#### Setup the Swarm

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

#### Install the Administrative Backend

This only needs to be done once and then all other services can share the same cluster manager management systems.

3. Add the Portainer agent so you can watch the system

```bash
$ curl -L https://portainer.io/download/portainer-agent-stack.yml -o portainer-agent-stack.yml
$ docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```

4. Add a Traefik reverse proxy and Splunk server for log aggregation and data indexing.

```bash
$ docker stack deploy --compose-file=admin-stack.yml splunk
```

#### Setup the CIRR

4. Set Variables for Abstractions and Secrets

Create a `cirr.env` file on the master and source it.

```yaml
DATA_DIR=/data
ORTHANC_PG_DATABASE=orthanc
ORTHANC_PASSWORD=orthanc
POSTGRES_PASSWORD=postgres
MOD_PACS=PACS,10.0.0.1,11112
MOD_WORKSTATION=TERARECON,10.0.0.2,11112
```

5. Start Up the Service Stack

```bash
$ . cirr.env && docker stack deploy --compose-file=cirr-stack.yml cirr
```

Note that if volumes are created on a node, they are not removed when the stack is removed.  Manually remove them to clear errors about directories not found.

Alternatively, you can log into Portainer after installing the admin stack and add the template file for the RIH CIRR from this repository.


#### Augmenting the CIRR

The CIRR can have additional Orthanc and DIANA nodes attached to it for various tasks.  

- `derekmerck/orthanc-wbv` images can be used as research project mini-PACS servers.
- `derekmerck/diana` or `derekmerck/diana-ai` images can be used for automated postprocessing and to drive continuous data monitoring tasks


## The SIREN Stack

Differences:  
- certficate validation
- anon and compress on data ingress


## License

MIT