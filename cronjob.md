# CronJob

## What is a cron job?
A Cron Job manages time based Jobs, namely:  
- Once at a specified point in time
- Repeatedly at a specified point in time  

> Note: 
CronJob resource in batch/v2alpha1 API group has been deprecated starting from cluster version 1.8. 
You should switch to using batch/v1beta1, instead, which is enabled by default in the API server. 
Further in this document, we will be using batch/v1beta1 in all the examples.  

## 典型的用例
- 定时调度一个Job
- Create a periodic job, e.g. database backup, sending emails.  

## Prerequisites
You need a working Kubernetes cluster at version >= 1.8 (for CronJob).  

**1.8之前的版本，需要传递`--runtime-config=batch/v2alpha1=true`参数给apiservice，
以显式地使能batch/v2alpha1 API，并重启apiservice和controller manager组件。**  

## Creating a Cron Job
### yaml文件创建
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
### kubectl run
```
kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox 
-- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```
```
$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         <none>

$ kubectl get jobs --watch
NAME               DESIRED   SUCCESSFUL   AGE
hello-4111706356   1         1         2s

$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         Mon, 29 Aug 2016 14:34:00 -0700
```
### 查看job对应的Pod
```
# Replace "hello-4111706356" with the job name in your system
$ pods=$(kubectl get pods -a --selector=job-name=hello-4111706356 --output=jsonpath={.items..metadata.name})

$ echo $pods
hello-4111706356-o9qcm

$ kubectl logs $pods
Mon Aug 29 21:34:09 UTC 2016
Hello from the Kubernetes cluster
```

## Deleting a Cron Job
```
$ kubectl delete cronjob hello
```
This stops new jobs from being created and removes all the jobs and pods created by this cronjob.  

## Cron Job Limitations
cron job每次执行大约创建一个job对象。有些情况下，也可能创建两个或者不创建Job。我们尽量使得这种情况少发生，但仍不能完全阻止。  
**因此，Job必须是幕等的！**  

If `startingDeadlineSeconds` is set to a large value or left unset (the default) 
and if `concurrentPolicy` is set to AllowConcurrent, **the jobs will always run at least once**.  

如果Job controller没运行，或者controller在CronJob开始之前出问题了，且一直到start+startingDeadlineSeconds时都没有恢复，那么Job不会运行了；  

或者controller出问题的时间比超过了多个Job的开始时间，且`concurrencyPolicy`配置成了不允许并行，这样Job不会运行了。  

For example, suppose a cron job is set to start at exactly 08:30:00 and its startingDeadlineSeconds is set to 10, 
if the CronJob controller happens to be down from 08:29:00 to 08:42:00, the job will not start.   
 
**Set a longer startingDeadlineSeconds if starting later is better than not starting at all.**  
 
The Cronjob is only responsible for creating Jobs that match its schedule, 
and the Job in turn is responsible for the management of the Pods it represents.  

## Concurrency Policy
The `.spec.concurrencyPolicy` field is also optional. It specifies how to treat concurrent executions of a job created by this cron job. 
Only one of the following concurrent policies may be specified:  
- Allow (default): allows concurrently running jobs
- Forbid: forbids concurrent runs, skipping next run if previous hasn’t finished yet
- Replace: cancels currently running job and replaces it with a new one  
Note that concurrency policy only applies to the jobs created by the same cron job. 
If there are multiple cron jobs, their respective jobs are always allowed to run concurrently.  

## Suspend
The `.spec.suspend` field is also optional. If set to true, all subsequent executions will be suspended. 
It does not apply to already started executions. Defaults to false.  
意思是一个Job没有运行完成，后面的Job都将会被挂起？  

## Jobs History Limits
The `.spec.successfulJobsHistoryLimit` and `.spec.failedJobsHistoryLimit` fields are optional. 
These fields specify how many completed and failed jobs should be kept.  

## Writing a Cron Job Spec
```
apiVersion:
kind:
metadata:
 name:
spec:  # All modifications to a cron job, especially its .spec, will be applied only to the next run.
  schedule: "*/1 * * * *"  # Cron Job格式的字符串
  startingDeadlineSeconds:    # 不指定就没有deadline
  concurrencyPolicy:  # Allow, Forbid, Replace
  jobTemplate:
    spec:
      containers:
      - name:
        image:
        args:
        - /bin/sh
        - -c
        - date
      restartPolicy: 
```
