---
title: docker搭建etcd集群
date: 2024-06-12 22:10:18
tags:
- Etcd
---

## docker-compose
```yaml
version: '1.0'

networks:
  etcd: {}

services:
  etcd0:
    image: bitnami/etcd
    container_name: etcd0
    networks:
      - etcd
    ports:
      - 4001:4001
      - 2380:2380
      - 2379:2379
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd0
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.31.112:2379,http://192.168.31.112:4001
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379,http://0.0.0.0:4001
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.31.112:2380
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
      - ETCD_INITIAL_CLUSTER=etcd0=http://192.168.31.112:2380,etcd1=http://192.168.31.112:12380,etcd2=http://192.168.31.112:22380
      - ETCD_INITIAL_CLUSTER_STATE=new
    volumes:
      - ./data0:/bitnami/etcd

  etcd1:
    image: bitnami/etcd
    container_name: etcd1
    networks:
      - etcd
    ports:
      - 14001:4001
      - 12380:2380
      - 12379:2379
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd1
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.31.112:12379,http://192.168.31.112:14001
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379,http://0.0.0.0:4001
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.31.112:12380
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
      - ETCD_INITIAL_CLUSTER=etcd0=http://192.168.31.112:2380,etcd1=http://192.168.31.112:12380,etcd2=http://192.168.31.112:22380
      - ETCD_INITIAL_CLUSTER_STATE=new
    volumes:
      - ./data1:/bitnami/etcd

  etcd2:
    image: bitnami/etcd
    container_name: etcd2
    networks:
      - etcd
    ports:
      - 24001:4001
      - 22380:2380
      - 22379:2379
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd2
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.31.112:22379,http://192.168.31.112:24001
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379,http://0.0.0.0:4001
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.31.112:22380
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
      - ETCD_INITIAL_CLUSTER=etcd0=http://192.168.31.112:2380,etcd1=http://192.168.31.112:12380,etcd2=http://192.168.31.112:22380
      - ETCD_INITIAL_CLUSTER_STATE=new
    volumes:
      - ./data2:/bitnami/etcd

```