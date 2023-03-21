# erddap-config
Template repository containing boilerplate ERDDAP configuration including:

* basic ERDDAP configuration boilerplate.
* github action configuration using [7yl4r/erddap-datasetsxml-builder](https://github.com/7yl4r/erddap-datasetsxml-builder) to build `datsets.xml` out of more human-readable dataset documentation. 
* [docker-erddap](https://hub.docker.com/r/axiom/docker-erddap/) docker-compose.yml to run it all on a container easily

# (1) Workflow
## (1.0) Initial setup:
0. create new repo using 7yl4r/erddap-config-template as the github template
1. modify the following files to fit your organization's settings
  1. `_pre.xml` : this is the header section of `datasets.xml`
  2. `setup.xml`
  3. `web.xml`
  4. `/images/erddapStart2.css`
2. enable github actions in the settings of your repo
3. set up datasets as described in workflow below

### (1.1) Dataset editing:
1. modify dataset files in `/datasets/` dir
    * NOTE: the `dataset.xml` file is the only file that matters to ERDDAP. A `README.md` and other files can be included for human convenience.
2. erddap-datasets-xml-builder will run to `datasets.xml` and commit it to github.
3. pull the new datasets.xml onto your erddap server (if using docker a `docker-compose restart` is not needed)
    * NOTE: A linux ERDDAP server can be set to do automatically using `cron`.[^1]
    * NOTE: Changes beyond just `datasets.xml` edits *may* require the docker container be restarted or rebuilt.

NOTE: in the docs below:
      `${HOSTNAME}` is the name of the dockerhost system.
      `${USERNAME}` is your username (must have docker permissions).

## (1.2) Adding a new DataSet Checklist:
1. create a file in this repo `/datasets/{dataset_name}/README.md` where `{dataset_name}` is the name you have chosen for your dataset; try to follow the patterns of existing dataset names in `/datasets/`.
1. create a file `/datasets/{dataset_name}/dataset.xml`:
    1. connect to docker host:
        * `ssh ${USERNAME}@${HOSTNAME}
        * for user `alice` accessing the 2023 IMaRS ERDDAP hypervisor this is : `alice@dune.marine.usf.edu`
    1. use `GenerateDatasetsXml` tool to auto-generate dataset xml:
        * `docker exec -it erddap  bash -c "cd webapps/erddap/WEB-INF/ && bash GenerateDatasetsXml.sh -verbose"`
    1. copy the xml into your new `/datasets/{dataset_name}/dataset.xml` file
    1. modify the xml
        1. change the name in the xml to match your chosen `dataset_name`  
1. see the `Dataset editing (1.1)` section above to finish applying the changes

## (1.3) checking for errors on the ERDDAP server:
1. look at `{erddap_server_url/status.html` to see a list of datasets that are failing
1. search the `/erddapdata/logs/log.txt` for errors related to your dataset
1. run `DasDds` to find errors
    * `docker exec -it erddap  bash -c "cd webapps/erddap/WEB-INF/ && bash DasDds.sh -verbose"`
1. once DasDds has no errors, you may need to restart the ERDDAP container to update volumes:
    * `docker-compose restart`
    * in some cases resetting the volumes is needed too:
        * `docker-compose down --rmi all --volumes && docker-compose up -d`
1. watch [${HOSTNAME}/erddap/status.hml](http://${HOSTNAME}.marine.usf.edu:8080/erddap/status.html) for `LoadDatasets` to finish & that all is well.

# additional links
* [setup.xml spec](https://coastwatch.pfeg.noaa.gov/erddap/download/setup.html#setup.xml)
* [datasets.xml spec](https://coastwatch.pfeg.noaa.gov/erddap/download/setupDatasetsXml.html)
* [example content directory](https://github.com/BobSimons/erddapContent)
* [updates to ERDDAP code](https://coastwatch.pfeg.noaa.gov/erddap/download/changes.html)
* [IOOS "Gold Standard" Example configs](https://github.com/ioos/erddap-gold-standard)

----------------------

[^1]: example crontab entry to pull new changes every 30min: `*/30 * * * * cd /root/docker_volumes/erddap-config ; /usr/bin/git pull`
