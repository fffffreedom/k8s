# kubelet

## 问题

https://my.oschina.net/jxcdwangtao/blog/1629059

## 配置

```
[service]
...
ExecStartPre=/usr/bin/mkdir -p /sys/fs/cgroup/cpuset/system.slice/kubelet.service
ExecStartPre=/usr/bin/mkdir -p /sys/fs/cgroup/hugetlb/system.slice/kubelet.service
ExecStart=/usr/local/bin/kubelet \
  ...
  --kube-reserved=cpu=2,memory=8Gi \
  --system-reserved=cpu=10,memory=24Gi \
  --enforce-node-allocatable=pods,kube-reserved,system-reserved \
  --kube-reserved-cgroup=/system.slice/kubelet.service \
  --system-reserved-cgroup=/system.slice \
  ...
```

## Reserve Compute Resources for System Daemons

> https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/  
> https://k8smeetup.github.io/docs/tasks/administer-cluster/reserve-compute-resources/  
