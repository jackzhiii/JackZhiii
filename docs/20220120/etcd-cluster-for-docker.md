## 在本地通过 docker 构建 etcd 集群的 docker-compose.yml 文件

```yml
version: "3.7"

services:
  etcd0:
    image: "bitnami/etcd"
    container_name: etcd0
    ports:
      - "23800:2380"
      - "23790:2379"
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd0
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.1.140:23790
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://etcd0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd0=http://etcd0:2380,etcd1=http://etcd1:2380,etcd2=http://etcd2:2380
      - ETCD_INITIAL_CLUSTER_STATE=new

  etcd1:
    image: "bitnami/etcd"
    container_name: etcd1
    ports:
      - "23801:2380"
      - "23791:2379"
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd1
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.1.140:23791
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://etcd1:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd0=http://etcd0:2380,etcd1=http://etcd1:2380,etcd2=http://etcd2:2380
      - ETCD_INITIAL_CLUSTER_STATE=new

  etcd2:
    image: "bitnami/etcd"
    container_name: etcd2
    ports:
      - "23802:2380"
      - "23792:2379"
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd2
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.1.140:23792
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://etcd2:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd0=http://etcd0:2380,etcd1=http://etcd1:2380,etcd2=http://etcd2:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
```
