# ElasticSearch Stack

## Command and setup

`sudo su -` to edit config file use root user

`cat /etc/elasticsearch/elasticsearch.yml` - Elasticsearch Configuration

`/var/log/elasticsearch` - log files locations

`systemctl start elasticsearch.service` - start elasticsearch

`ps -eaf | grep elast` - find elastic on running process using ps

`curl -XGET 'localhost:9200/_cluster/health?pretty'` - get information about cluster

`curl -XGET 'localhost:9200/_cluster/stats?human&pretty'` - information about elastic

`/etc/kibana` - location of config file

`sudo systemctl start kibana.service` - start kibana

`5601` - default kibana port
`http://localhost:9200` - default elasticsearch port

By default ElasticSearch use 4 Gb for SWAP and 4 Gb for heap.

`documents` data is stored in documents

`attributes`

`values`

### Questions

- Що таке Elasticsearch і які завдання він виконує?
- Що таке кластер?
- Що таке нода?
- Що таке індекс?
- Що таке тип?
- Що таке документ?
- Що таке шарди і репліки?
- Що таке Query DSL?
- Що таке мапинг?
