# Configure Service Accounts for Pods

> https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/  

A service account provides an identity for processes that run in a Pod.  
Service account 是为了方便 Pod 里面的进程调用 Kubernetes API 或其他外部服务而设计的。  

## User Account

When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account
(currently this is usually admin, unless your cluster administrator has customized your cluster).   

## Service Account

Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account
(for example, default).  

### 查看默认的service account

**Every namespace has a default service account resource called default.**  
You can list this and any other serviceAccount resources in the namespace with this command:  
```
$ kubectl get serviceAccounts
NAME      SECRETS    AGE
default   1          1d
```

### 创建service account

You can create additional ServiceAccount objects like this:  
```
$ cat > /tmp/serviceaccount.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF
$ kubectl create -f /tmp/serviceaccount.yaml
serviceaccount "build-robot" created
```

### 获取service account完整的yaml 

If you get a complete dump of the service account object, like this:  
```
$ kubectl get serviceaccounts/build-robot -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2015-06-16T00:12:59Z
  name: build-robot
  namespace: default
  resourceVersion: "272500"
  selfLink: /api/v1/namespaces/default/serviceaccounts/build-robot
  uid: 721ab723-13bc-11e5-aec2-42010af0021e
secrets:
- name: build-robot-token-bvbk5
```
then you will see that a token has automatically been created and is referenced by the service account.  

### 删除service account

```
$ kubectl delete serviceaccount/build-robot
```

## 一些要点

- You may use authorization plugins to set permissions on service accounts.  
- To use a non-default service account, simply set the `spec.serviceAccountName` field of a pod to the name of the service account 
you wish to use.  
- The service account has to exist at the time the pod is created, or it will be rejected.  
- You cannot update the service account of an already created pod.  

## Use the Default Service Account to access the API server

When you create a pod, if you do not specify a service account, it is automatically assigned the default service account 
in the same namespace. If you get the raw json or yaml for a pod you have created (for example, kubectl get pods/podname -o yaml), 
you can see the `spec.serviceAccountName` field has been automatically set.  

You can access the API from inside a pod using automatically mounted service account credentials, as described in Accessing the Cluster. 
The API permissions a service account has depend on the authorization plugin and policy in use.  

In version 1.6+, you can opt out of automounting API credentials for a service account 
by setting `automountServiceAccountToken: false` on the service account:  
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
```

In version 1.6+, you can also opt out of automounting API credentials for a particular pod:  
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
  ...
```
The pod spec takes precedence over the service account if both specify a automountServiceAccountToken value.  

## Manually create a service account API token

在上面创建service account小节创建出来的service account，会自带一个token：  
```
secrets:
- name: build-robot-token-bvbk5
```

自己要手动创建一个新的secret，可按如下进行
```
$ cat > /tmp/build-robot-secret.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
EOF
$ kubectl create -f /tmp/build-robot-secret.yaml
secret "build-robot-secret" created
```
Now you can confirm that the newly built secret is populated with an API token for the “build-robot” service account.  
```
$ kubectl describe secrets/build-robot-secret
```

## Add ImagePullSecrets to a service account

- 创建imagePullSecret  
```
$ kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
secret "myregistrykey" created.
```

- modify the default service account for the namespace to use this secret as an imagePullSecret  
```
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'
```

- 手动编辑  
```
$ kubectl get serviceaccounts default -o yaml > ./sa.yaml
$ cat sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2015-08-07T22:02:39Z
  name: default
  namespace: default
  resourceVersion: "243024"
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
  uid: 052fb0f4-3d50-11e5-b066-42010af0d7b6
secrets:
- name: default-token-uudge
$ vi sa.yaml
[editor session not shown]
[delete line with key "resourceVersion"]
[add lines with "imagePullSecret:"]
$ cat sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2015-08-07T22:02:39Z
  name: default
  namespace: default
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
  uid: 052fb0f4-3d50-11e5-b066-42010af0d7b6
secrets:
- name: default-token-uudge
imagePullSecrets:
- name: myregistrykey
$ kubectl replace serviceaccount default -f ./sa.yaml
serviceaccounts/default
```

Now, any new pods created in the current namespace will have this added to their spec:  

```
spec:
  imagePullSecrets:
  - name: myregistrykey
```
