# ElasticSearch Stack

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

By default ElasticSearch use 4 Gb for SWAP.
