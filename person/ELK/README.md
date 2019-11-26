# ELK简介及基本配置



## Docker容器



### logstash

`docker run --name logstash -v "$PWD/logstash/pipeline/:/usr/share/logstash/pipeline" -v "$PWD/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml" -p 5044:5044 docker.elastic.co/logstash/logstash:7.4.0`

### filebeat

`docker run --name filebeat --network elk --user=root --volume="$PWD/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro" -v "$PWD/configs:/usr/share/filebeat/configs"  -v "$PWD/data:/data" --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" --volume="/var/run/docker.sock:/var/run/docker.sock:ro" docker.elastic.co/beats/filebeat:7.4.0 filebeat -e -strict.perms=false`


### 访问Kibana

`http://yourIpAddr:5601/`