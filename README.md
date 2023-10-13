# Data Replication to Elasticsearch from MySQL via Logstash

The demonstration uses Docker for all databases and services in the project:
- MySQL
- Elasticsearch
- Logstash
- Kibana

1. Start a MySQL container with `init.sql` loaded
```
docker run --name mysql-demo -p 3306:3306 \
-v ./mysql_init:/docker-entrypoint-initdb.d/:ro \
-e MYSQL_ROOT_PASSWORD=root -d mysql:8.1.0

```
2. Create Docker network for ELK stack

```
docker network create elastic
```

3. Create a new directory to be mounted by Elasticsearch 
```
mkdir es_data
```

4. Start an Elasticsearch container
```
docker run --name es-demo -d -p 9200:9200 --log-driver local --net elastic \
-e "bootstrap.memory_lock=true" \
-e ES_JAVA_OPTS="-Xms4g -Xmx4g" --ulimit memlock=-1:-1 \
-v ./es_data:/usr/share/elasticsearch/data \
docker.elastic.co/elasticsearch/elasticsearch:8.10.3-arm64
```

5. Reset the password for user `elastic` on the Elasticsearch container
```
docker exec -it -u 0 es-demo elasticsearch-reset-password --auto -u elastic
```

6. Start a Kibana container 
```
docker run -d --name kib-demo --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.10.3-arm64
```

7. Go to http://localhost:5601 and enter enrolment token from Elasticsearch
``` bash
# Generate enrolment token on the Elasticsearch container
docker exec -it -u 0 es-demo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

8. Modify the password in Elasticsearch output plugin using the generated password in Step 5

9. Start a Logstash container
```
docker run --rm -it --net elastic \
-v ./logstash_settings/:/usr/share/logstash/config/ \
docker.elastic.co/logstash/logstash:8.10.3-arm64 \
logstash -f /usr/share/logstash/config/jdbc.conf
```

10. Go to http://localhost:5601 and login in as `elastic` using the password in Step 5

11. Go to Stack Management > Kibana > Data Views

12. Create a data view using the index pattern generated

13. Go to Discovery, you should be able to see the data from MySQL

---

Reference: [Elastic doc](https://www.elastic.co/guide/en/cloud/current/ec-getting-started-search-use-cases-db-logstash.html)