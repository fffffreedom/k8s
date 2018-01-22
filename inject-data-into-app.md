# Inject Data Into Applications
> https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/  
## Define a Command and Arguments for a Container
```
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    #=====
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
    #=====
    ## another way
    #command:
    #  - "printenv"
    #  - "HOSTNAME"
    #  - "KUBERNETES_PORT"
    #=====
  restartPolicy: OnFailure
```
## Define Environment Variables for a Container
```
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
get shell cmdline and print evn:
```
kubectl exec -it envar-demo -- /bin/bash
printenv
```
## Expose Pod Information to Containers Through Environment Variables
**a Pod can use environment variables to expose information about itself to Containers running in the Pod !**
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-fieldref
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
          printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
          sleep 10;
        done;
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: **spec.nodeName**
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: **metadata.name**
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: **metadata.namespace**
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: **status.podIP**
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: **spec.serviceAccountName**
  restartPolicy: Never
```
> https://kubernetes.io/docs/api-reference/v1.9/#envvar-v1-core  
## Expose Pod Information to Containers Through Files
## Distribute Credentials Securely Using Secrets
## Inject Information into Pods Using a PodPreset
