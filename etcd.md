# etcd

coreos  
https://coreos.com/etcd/docs/latest/op-guide/clustering.html  
https://coreos.com/etcd/docs/latest/op-guide/security.html   
https://coreos.com/os/docs/latest/generate-self-signed-certificates.html  
https://coreos.com/etcd/docs/latest/op-guide/configuration.html  

github  
https://github.com/coreos/etcd/blob/master/Documentation/op-guide/container.md  

初试ETCD  
https://tonydeng.github.io/2015/11/24/etcd-the-first-using/  
etcd 2.0之后，规范了端口号的使用，并且写入了IANA组织的标准端口记录。etcd将提供给外部客户端的端口变为2379，而etcd服务间通信的端口变为2380（不过现在依然还是兼容原来4001和7001端口）。  


## run.sh
```
#docker run -d -p 2379:2379 -p 2380:2380 --name etcd quay.io/coreos/etcd:v2.3.8 \
#       -name etcd0 \
#       -advertise-client-urls http://10.101.15.191:2379 \
#       -listen-client-urls http://0.0.0.0:2379 \
#       -initial-advertise-peer-urls http://10.101.15.191:2380 \
#       -listen-peer-urls http://0.0.0.0:2380 \
#       -initial-cluster-token etcd-cluster \
#       -initial-cluster etcd0=http://10.101.15.191:2380 \
#       -initial-cluster-state new

docker run -d -p 2379:2379 -p 2380:2380 --name etcd quay.io/coreos/etcd \
        /usr/local/bin/etcd -name etcd0 \
        -advertise-client-urls http://10.101.15.191:2379 \
        -listen-client-urls http://0.0.0.0:2379 \
        -initial-advertise-peer-urls http://10.101.15.191:2380 \
        -listen-peer-urls http://0.0.0.0:2380 \
        -initial-cluster-token etcd-cluster \
        -initial-cluster etcd0=http://10.101.15.191:2380 \
        -initial-cluster-state new
```


