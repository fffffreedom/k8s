# DNS for Services and Pods

> https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/  

## intro

Kubernetes DNS schedules **a DNS Pod and Service** on the cluster, and configures the kubelets to tell individual containers 
to **use the DNS Service’s IP to resolve DNS names.**  

### What things get DNS names?

Every Service defined in the cluster (including the DNS server itself) is assigned a DNS name. 
By default, a client Pod’s DNS search list will include the Pod’s own namespace and the cluster’s default domain.  

集群中的每一个 service（包括DNS service自己）会被分配一个 DNS name, Pod 的 DNS 解析列表中会包含 Pod 自己的 namespace 和 集群的 default domain. 

Assume a Service named `foo` in the Kubernetes namespace `bar`. A Pod running in namespace `bar` can look up this service 
by simply doing a DNS query for `foo`. A Pod running in namespace `quux` can look up this service by doing a DNS query for `foo.bar`.  

#### DNS type and layout

> https://github.com/kubernetes/dns/blob/master/docs/specification.md  

### service

#### A record

“Normal” (not headless) Services are assigned a DNS A record for a name of the form `my-svc.my-namespace.svc.cluster.local`. 
This resolves to the cluster IP of the Service.

“Headless” (without a cluster IP) Services are also assigned a DNS A record for a name of the form 
`my-svc.my-namespace.svc.cluster.local`. 
Unlike normal Services, this resolves to the set of IPs of the pods selected by the Service.  

#### SRV records

SRV Records are created for **named ports** that are part of normal or Headless Services. For each named port, 
the SRV record would have the form `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`.  

For a regular service, this resolves to the port number and the CNAME: `my-svc.my-namespace.svc.cluster.local`.  

For a headless service, this resolves to multiple answers, one for each pod that is backing the service, 
and contains the port number and a CNAME of the pod of the form `auto-generated-name.my-svc.my-namespace.svc.cluster.local`.  

### Pods

#### A Records

When enabled, pods are assigned a DNS A record in the form of `pod-ip-address.my-namespace.pod.cluster.local`.  

- Pod’s hostname and subdomain fields  

Currently when a pod is created, its hostname is the Pod’s metadata.name value. The Pod spec has an optional hostname field, 
which can be used to specify the Pod’s hostname. When specified, it takes precedence over the Pod’s name to be the hostname 
of the pod. For example, given a Pod with hostname set to “my-host”, the Pod will have its hostname set to “my-host”.  

The Pod spec also has an optional subdomain field which can be used to specify its subdomain. 
For example, a Pod with hostname set to “foo”, and subdomain set to “bar”, in namespace “my-namespace”,
will have the fully qualified domain name (FQDN) “foo.bar.my-namespace.svc.cluster.local”.  

```
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # Actually, no port is needed.
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
```

在同一个 namespace 下，如果 Pod 的 spec.subdomain 和 headless service 的 metadata.name 一样，集群的 kubeDNS 仍然会返回 Pod FQDN 对应
的 A Record。 举个例子：  

For example, given a Pod with the hostname set to “busybox-1” and the subdomain set to “default-subdomain”, 
and a headless Service named “default-subdomain” in the same namespace, the pod will see its own FQDN as 
“busybox-1.default-subdomain.my-namespace.svc.cluster.local”.   
DNS serves an A record at that name, pointing to the Pod’s IP. Both pods “busybox1” and “busybox2” can have their distinct A records.  

#### Pod’s DNS Policy

DNS policies can be set on a per-pod basis. Currently Kubernetes supports the following pod-specific DNS policies. 
These policies are specified in the dnsPolicy field of a Pod Spec.  

- Default  

The Pod inherits the name resolution configuration from the node that the pods run on. See related discussion for more details.  
> https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/#inheriting-dns-from-the-node  

- ClusterFirst (Default Policy)  

Any DNS query that does not match the configured cluster domain suffix, such as “www.kubernetes.io”, 
is forwarded to the upstream nameserver inherited from the node. Cluster administrators may have extra stub-domain 
and upstream DNS servers configured. See related discussion for details on how DNS queries are handled in those cases.  

- ClusterFirstWithHostNet  

For Pods running with hostNetwork, you should explicitly set its DNS policy “ClusterFirstWithHostNet”.  

- None  

A new option value introduced in Kubernetes **v1.9**. This Alpha feature allows a Pod to ignore DNS settings 
from the Kubernetes environment. All DNS settings are supposed to be provided using the dnsConfig field in the `Pod Spec`. 
See DNS config subsection below.  

NOTE: “Default” is not the default DNS policy. If dnsPolicy is not explicitly specified, then “ClusterFirst” is used.  

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```

#### Pod’s DNS Config

Kubernetes **v1.9** introduces an Alpha feature that allows users more control on the DNS settings for a Pod. 
To enable this feature, the cluster administrator needs to enable the CustomPodDNS feature gate on the apiserver and the kubelet.  
for example, “--feature-gates=CustomPodDNS=true,...”.  When the feature gate is enabled, 
users can set the dnsPolicy field of a Pod to “None” and they can add a new field dnsConfig to a Pod Spec.  

The dnsConfig field is optional and it can work with any dnsPolicy settings. 
However, when a Pod’s dnsPolicy is set to “None”, the dnsConfig field has to be specified.  

> https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/  





