
## Adaptations necessary for indexing terminologies for SemLookP
The [EMBL EBI OLS4 repository](https://github.com/EBISPOT/ols4) is forked in a [ZB MED ols4 repository](https://github.com/zbmed/ols4).
The dev branch is kept up to date with the EBISPOT/ols4:dev upstream branch. 
The dev-zbmed branch contains the adaptations necessary for indexing terminologies for SemLookP.

The following changes should be kept intact when merging the dev branch into the dev-zbmed branch:


- Add in `dataload/rdf2json/src/main/java/uk/ac/ebi/rdf2json/annotators/LabelAnnotator.java` to clear default properties (solves Radlex issue):
```
if(configLabelProperties instanceof Collection<?>) {
            labelProperties.clear();
            labelProperties.addAll((Collection<String>) configLabelProperties);
        }
```
- Matomo tracking

## Indexing
Indexing is done on the ZB MED VM 10.X.X.X (256 GB RAM, 64 GB swap, 32 CPU, 1 TB disk space).  

`ts/ols4`: the dev-zbmed branch of the forked OLS4 repository  
`ts/ols4/dataload/terminologies/health`: terminologies to index   
`ts/ols4/dataload/configs`: config.json must be here  
`ts/database-archives`: final neo4j and solr archives
`ts/configs`: all configs
`ts/terminologies`: all terminologies

Clone the repository if not already done:
```
git clone https://github.com/zbmed/ols4.git
```
Copy terminologies and configs to the ols4 project in the dataload directory if not already done:
```
mkdir ols4-march26/dataload/terminologies/ && cp -r terminologies/health ols4-march26/dataload/terminologies/
cp configs/all-health-configs.json ols4-march26/dataload/configs
```


### Clean disk space 
To prevent any unwanted behavior following an update to the repository, delete all containers, images and volumes:

Stop and remove all (running, stopped and attached anonymous (v))
```
docker rm -vf $(docker ps -aq)
docker image prune -af
docker volume prune -af
docker system prune -a
```
Hint: deleting networks with system prune can result in errors during indexing, a reboot solves the problem

Status overview of what's left:
```
docker ps -a && docker images -a && docker volume ls && docker network ls
```

## Run
In the config.json set the absolute path for a local terminology:
```
"ontology_purl": "file:///home/user/ts/ols4/dataload/terminologies/health/maelstrom.owl"
```
```
export OLS4_CONFIG=dataload/configs/all-health-configs-may26.json
rm nohup.out
rm -r out
rm -r tmp
nohup ./dataload.sh 2>&1 | ts '[%Y-%m-%d %H:%M:%S]' > nohup.out &
```
or 
```
nohup bash -lc './dataload.sh 2>&1 -resume | ts "[%Y-%m-%d %H:%M:%S]" > nohup.out' >/dev/null 2>&1 & disown
```

After fail and solving the error:
```
nohup ./dataload.sh 2>&1 -resume | ts '[%Y-%m-%d %H:%M:%S]' > nohup.out &
```
After indexing, save the nohup.out file for documentation.  
Result data is in `/var/lib/docker/volumes.`

---


## Post-processing

- copy to the cluster:
```
TNAME="health-may26"
DATASERVER="zbmed-ts-health-ols4-dataserver-69644dbfdb-6hzwt"
NEO4J="/home/sasse/ts/ols4/tmp/work/b5/d1ab22ec2580db2655fc5c6f312d65/neo4j.tgz"
SOLR="/home/sasse/ts/ols4/tmp/work/aa/cf740c2a3230bbdf5c050d86deba3b/solr.tgz"
kubectl cp ${NEO4J} ${DATASERVER}:/usr/share/nginx/html/${TNAME}_neo4j.tgz -n zbmed-ts-health
kubectl cp ${SOLR} ${DATASERVER}:/usr/share/nginx/html/${TNAME}_solr.tgz -n zbmed-ts-health
```



---

### OLS4 system without Nextflow:

- Dockerfile: `ENV JAVA_TOOL_OPTIONS="--add-opens=java.base/java.nio=ALL-UNNAMED"`
- In `ols4/dataload/load_into_neo4j.sh` change
  `--read-buffer-size=16777216` to
  `--read-buffer-size=67108864` (solves Loinc issue).
- `dataload/Dockerfile` is adapted to mount dataload/terminologies/ into /tmp/media of the dataload Docker container. Add in `ols4-dataload/Dockerfile`:
```
RUN rm -rf /tmp/media
RUN mkdir /tmp/media
ADD dataload/terminologies/ /tmp/media/
```

- for indexing, run
```
export OLS4_CONFIG=dataload/configs/terminologies.json
export JAVA_OPTS="-Xms10G -Xmx35G"
rm nohup.out
nohup docker compose up --force-recreate --build --always-recreate-deps --attach-dependencies ols4-solr ols4-neo4j | ts '[%Y-%m-%d %H:%M:%S]' > nohup.out &
```

- compress:
```
sudo tar --use-compress-program="pigz --fast --recursive" -cf ts/database-archives/example_neo4j.tgz -C /var/lib/docker/volumes/ols4_ols4-neo4j-data/_data .
sudo tar --use-compress-program="pigz --fast --recursive" -cf ts/database-archives/example_solr.tgz -C /var/lib/docker/volumes/ols4_ols4-solr-data/_data .
```

### Disk space requirements for the NFDI4Health terminologies (August 2024):
Neo4j Docker volume: 94 GiB  
Solr Docker volume: 21 GiB  
Neo4j archive: 10 GiB  
Solr archive: 11 GiB  
Docker container : ~ 300 GiB

NCBITAXON  
140 GiB Docker container  
44 GiB neo4j result  
10 GiB solr result
