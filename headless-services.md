# Headless Services
## what's headless services
Sometimes you don’t need or want load-balancing and a single service IP. In this case, you can create “headless” services by specifying 
"None" for the cluster IP (spec.clusterIP).  
For such Services, a cluster IP is not allocated, kube-proxy does not handle these services, and there is no load balancing or proxying 
done by the platform for them. How DNS is automatically configured depends on whether the service has selectors defined.  
  - With selectors  
  For headless services that define selectors, the endpoints controller creates Endpoints records in the API,
  and modifies the DNS configuration to return A records (addresses) that point directly to the Pods backing the Service.
  - Without selectors  
  For headless services that do not define selectors, the endpoints controller does not create Endpoints records. However, 
  the DNS system looks for and configures either:
    - CNAME records for ExternalName-type services.
    - A records for any Endpoints that share a name with the service, for all other types.
> https://kubernetes.io/docs/concepts/services-networking/service/ 
## normal service 和 headless service的区别
- normal service  
service对应多个endpoint，但是dns查询时只会返回service的地址（虚ip）。具体client访问的是哪个Real Server，是由iptables来决定的。
- headless service  
dns查询会如实的返回2个真实的endpoint。Headless Service的对应的每一个Endpoints，即每一个Pod，都会有对应的DNS域名；这样Pod之间就可以互相访问。
> https://ieevee.com/tech/2017/03/22/k8s-headless-service.html  
