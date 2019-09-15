# Docker Splunk Clustering Architecture

docker run -d -p 8000:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunk --hostname splunk splunk/splunk:edge

## Container Clustering
docker run -d -p 8010:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkclm1 --hostname splunkclm1 splunk/splunk:edge
docker run -d -p 8020:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkidx1 --hostname splunkidx1 splunk/splunk:edge
docker run -d -p 8030:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkidx2 --hostname splunkidx2 splunk/splunk:edge
docker run -d -p 8040:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkidx3 --hostname splunkidx3 splunk/splunk:edge
docker run -d -p 8050:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkidx4 --hostname splunkidx4 splunk/splunk:edge
docker run -d -p 8060:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkseh1 --hostname splunkseh1 splunk/splunk:edge
docker run -d -p 8070:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=password" --name splunkseh2 --hostname splunkseh2 splunk/splunk:edge

## Remove all containers
docker rm -f splunkclm1
docker rm -f splunkidx1
docker rm -f splunkidx2
docker rm -f splunkidx3
docker rm -f splunkidx4
docker rm -f splunkseh1
docker rm -f splunkseh2

### CLi Conf
#### Master
./splunk edit cluster-config -mode master -multisite true -available_sites site1,site2 -site_replication_factor origin:2,total:3 -site site1 -site_search_factor origin:1,total:2 -cluster_label cliclusetersetup
#### Slaves/Indexes
./splunk edit cluster-config -mode slave -site site1 -master_uri https://172.17.0.2:8089 -replication_port 9887
./splunk edit cluster-config -mode slave -site site2 -master_uri https://172.17.0.2:8089 -replication_port 9887
#### Search Heads
./splunk edit cluster-config -mode searchead -site site1 -master_uri https://172.17.0.2
./splunk edit cluster-config -mode searchead -site site2 -master_uri https://172.17.0.2

### Conf File
#### Master
[general]
site = site#

[clustering]
available_sites = site1,site2
mode = master
multisite = true
site_replication_factor = origin:2,total:3
site_search_factor = origin:1,total:2
cluster_label = confclutersetup

#### Slaves/Indexes
[general]
site = site#

[replication_port://9887]

[clustering]
master_uri = https://172.17.0.2:8089
mode = slave

#### Search Heads
##### Clustered Search Heads
[clustering]
master_uri = clustermaster:172.17.0.2:8089
mode = searchhead

[clustermaster:172.17.0.2:8089]
master_uri = https://172.17.0.2:8089
multisite = true
site = site#

##### Single Search Head
[general]
site = site#

[clustering]
master_uri = https://172.17.0.2:8089
mode = searchhead
multisite = true