# RIH Clinical Imaging Research Registry Stack

Derek Merck  
<derek_merck@brown.edu>  
Rhode Island Hospital and Brown University  
Providence, RI  


## Usage

Set your environment vars (with `.env`, for example)

```bash
$ docker-compose -f setup.docker-compose.yml up  # Creates proxy and portainer
$ docker-compose up
$ docker-compose scale orthanc=3
```

Alternatively, you can log into Portainer and add the stack template "RIH-CIRR", although it does not support scaling.


## Required Environment Vars

```bash
DATA_DIR=/data
ORTHANC_PG_DATABASE=orthanc
ORTHANC_PASSWORD=passw0rd!
POSTGRES_PASSWORD=passw0rd!
```

## License

MIT