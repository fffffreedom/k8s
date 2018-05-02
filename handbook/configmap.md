# Pod 的配置管理 -- ConifgMap

## 应用场景

- 生成为容器的环境变量  
- 设置容器启动命令的启动参数（需设置为环境变量）  
- 以 volume 的形式挂载为容器内部的文件或者目录  

## 创建方式

### kubectl 命令行

kubectl create configmap <map-name> <data-source>

Use the kubectl create configmap command to create configmaps from:

- directories

```
kubectl create configmap NAME --from-file=/path/to/directories
```

- files

```
kubectl create configmap NAME --from-file=[key=]/path/to/files ... --from-file=[key=]/path/to/files
```
当未指定key=时，文件名为key!

- literal values

```
kubectl create configmap NAME --from-literal=[key=]value ... --from-literal=[key=]value
```

### yaml 文件

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: NAME
data:
  key: value
  ...
  key: value
```

## 使用方式

- 环境变量

定义 ConfigMap, 然后在 Pod 中引用：

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO

---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.how
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: log_level
  restartPolicy: Never
```

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      **envFrom**:
      - configMapRef:
          name: special-config
  restartPolicy: Never
````

- volume 方式

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.level: very
  special.type: charm

---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never

---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: special.level
          path: keys    # /etc/config/keys 文件的值为 very
  restartPolicy: Never
```

## 指定路径和权限

[secret](https://kubernetes.io/docs/concepts/configuration/secret/)

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 0777
```

In this case, the file resulting in /etc/foo/my-group/my-username will have permission value of 0777.

## 参考资料

[configure-pod-configmap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
