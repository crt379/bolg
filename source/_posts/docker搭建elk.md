---
title: docker搭建elk
date: 2024-06-12 22:03:22
tags:
- Elasticsearch
- kibana
- Logstash
---

## docker-compose

```yaml
version: '3.9'

networks:
  elk: {}

services:
  elasticsearch:
    image: elasticsearch:latest
    container_name: elasticsearch
    env_file:
      - ./environment/elasticsearch.env
    networks:
      - elk
    ports:
      - 9200:9200
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins

  kibana:
    image: kibana:latest
    container_name: kibana
    env_file:
      - ./environment/kibana.env
    networks:
      - elk
    ports:
      - 5601:5601
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml

  logstash:
    image: logstash:latest
    container_name: logstash
    env_file:
      - ./environment/logstash.env
    networks:
      - elk
    ports:
      - 9600:9600
      - 11000:11000
      - 11001:11001
      - 11002:11002
    volumes:
      - ./logstash/config/logstash.yml:/etc/logstash/logstash.yml
      - ./logstash/pipeline/:/usr/share/logstash/pipeline/
```

## environment

### elasticsearch
```env
# elasticsearch.env
TZ=Asia/Shanghai
discovery.type=single-node
ES_JAVA_OPTS=-Xms1024m -Xmx1024m
xpack.security.enabled=false
```

### kibana
```env
# kibana.env
ELASTICSEARCH_URL="http://192.168.31.112:9200"
```

### logstash
```env
TZ=Asia/Shanghai
```

## config

### kibana
```yaml
# kibana.yaml
server.host: "0.0.0.0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: [ "http://192.168.31.112:9200" ]
# elasticsearch.url: "http://192.168.31.112:9200" 
monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
elasticsearch.ssl.verificationMode: "none"
```

### logstash
- config
```yaml
# logstash.yml
path.config: /usr/share/logstash/pipeline
```

- pipeline
```conf
# logstash.conf
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 11000
    tags => ["prod"]
    codec => json_lines
  }
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 11001
    tags => ["dev"]
    codec => json_lines
  }
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 11002
    tags => ["test"]
    codec => json_lines
  }

  redis {
    data_type => "list"
		key => "service-collector_log"
		host => "192.168.31.207"
		port => 6379
		db => 0
		threads => 2
    password => "123456"
    tags => ["dev"]
    codec => json_lines
  }
}

filter {
  date {
    match => ["ts", "yyyy-MM-dd HH:mm:ss.SSS"]
    target => "@timestamp"
  }
}

output {
  if "prod" in [tags] {
    elasticsearch {
        hosts  => ["http://192.168.31.112:9200"]
        index  => "logstash-prod"
        codec  => "json"
    }
  } else if "dev" in [tags] {
    elasticsearch {
        hosts  => ["http://192.168.31.112:9200"]
        index  => "logstash-dev"
        codec  => "json"
    }
  } else if "test" in [tags] {
    elasticsearch {
        hosts  => ["http://192.168.31.112:9200"]
        index  => "logstash-test"
        codec  => "json"
    }
  }

  # stdout { codec => rubydebug }
}
```
