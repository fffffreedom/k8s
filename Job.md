# Jobs - Run to Completion

## What is a Job?
Job会创建一个或者多个Pod，以确保指定数目的Pod能成功退出。当Pod成功完成时，Job能跟踪到这些成功的退出。当成功退出的Pod达到指定数目后，Job就完成了。
删除一个Job将会清除它创建的所有Job.  

一个简单的例子，就是创建一个Job对象，以确保一个Pod能成功地运行！因为如果一个Pod失败或者被删除了，Job对象会再创建一个Pod，直到其成功完成。
当然，Job可以一次并行运行多个Pod.

## Running an example Job
```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

### 查看所有Job的Pod
```
$ pods=$(kubectl get pods  --show-all --selector=job-name=pi --output=jsonpath={.items..metadata.name})
$ echo $pods
pi-aiw0a
```

## Writing a Job Spec
```
apiVersion:
kind: Job
metadata：
  name:
spec:
  selector:  # optional
  template:  # required
    spec:
      - name: 
        image:
    restartPolicy:  # Never or OnFailure is allowed.
```

## Parallel Jobs
There are three main types of jobs:  
- Non-parallel Jobs
  - normally only one pod is started, unless the pod fails.
  - job is complete as soon as Pod terminates successfully.
- Parallel Jobs with a fixed completion count
  - specify a non-zero positive value for `.spec.completions`
  - the job is complete when there is one successful pod for each value in the range 1 to `.spec.completions`.
  - not implemented yet: each pod passed a different index in the range 1 to `.spec.completions`.  
- Parallel Jobs with a work queue:
  - do not specify `.spec.completions`, default to `.spec.parallelism`
  - the pods must coordinate with themselves or an external service to determine what each should work on.  
    - each pod is independently capable of determining whether or not all its peers are done, thus the entire Job is done.
    - when any pod terminates with success, no new pods are created.
    - once at least one pod has terminated with success and all pods are terminated, then the job is completed with success.
    - once any pod has exited with success, no other pod should still be doing any work or writing any output. 
    They should all be in the process of exiting.  
 
### `.spec.completions` and `.spec.parallelism`
#### Non-parallel job
you can leave both `.spec.completions` and `.spec.parallelism` unset. When both are unset, both are defaulted to 1.
#### Fixed Completion Count job
you should set `.spec.completions` to the number of completions needed.  
You can set .spec.parallelism, or leave it unset and it will default to 1.  
#### Work Queue Job
you must leave `.spec.completions` unset, and set `.spec.parallelism` to a non-negative integer.

### Controlling Parallelism
The requested parallelism (.spec.parallelism) can be set to any non-negative value.  
If it is unspecified, it defaults to 1. If it is specified as 0, then the Job is effectively paused until it is increased.  

#### A job can be scaled up using the kubectl scale command
For example, the following command sets .spec.parallelism of a job called myjob to 10:  
```
$ kubectl scale  --replicas=10 jobs/myjob
job "myjob" scaled
```

### pod数和parallelism
Actual parallelism (number of pods running at any instant) may be **more or less** than requested parallelism（即pod数并不一定等于
parallelism的值）, for a variety of reasons:  
  - Fixed Completion Count jobs：实际并行运行的pod数量不会超过剩余要完成的数量；
  - work queue jobs：在任何pod成功完成之后，不允许再创建新的Pod，允许当前未完成的Pod继续执行；
  - 如果Job controller未响应
  - 因某原因Job controller创建Pod失败，如(lack of ResourceQuota, lack of permission, etc.)
  - 由于同一Job中有过多失败的Pod，Job controller可能会限制创建新的pod
  - pod清理需要时间

## Handling Pod and Container Failures
When a Pod fails, then the Job controller starts a new Pod.  
所以你的应用程序需要考虑这些情况：  
  - it needs to handle temporary files, locks, incomplete output and the like caused by previous runs.  
  - your pods must also be tolerant of concurrency.
  
## Pod Backoff failure policy
`.spec.backoffLimit`设置Pod失败后的重试次数！默认为6。  
Failed Pods associated with the Job are recreated by the Job controller 
with an exponential back-off delay (10s, 20s, 40s …) capped at six minutes.  

> Note: Due to a known issue #54870, when the spec.template.spec.restartPolicy field is set to “OnFailure”, 
the back-off limit may be ineffective. As a short-term workaround, set the restart policy for the embedded template to “Never”.  

## Job Termination and Cleanup
当Job完成后，不会再创建新的Pod，但是已经运行的Pod也不会删除！因为它们已经被终止了，通过`kubectl get pods`并不能看到它们，
需要使用`kubectl get pods -a`.  

保留这些Pod，你就可以查看Pod的日志，来检查错误、警告、或者其它输出信息。  
Job对象也被保留着，以便你能够查看它们的状态！  

### 删除Job
`kubectl delete jobs/pi` or `kubectl delete -f job.yaml`  
使用上面的命令去删除Job时，Job所创建的所有Pod都将被删除。  

### 重试控制配置(`spec.activeDeadlineSeconds`)
如果由于一些依赖原因，一个Job一直失败，可以让其一直重试；但如果不想让Job一直永远地重试下去，可以通过配置来指定：`spec.activeDeadlineSeconds`.  
该配置可以指定一个重试的时间限制。  

## Job Patterns(参见kukernetes实践指南！)
The Job object can be used to support reliable parallel execution of Pods. 
The Job object is not designed to support closely-communicating parallel processes, as commonly found in scientific computing.   

Job does support parallel processing of a set of independent but related work items（独立且相关的工作项）.   

In a complex system, there may be multiple different sets of work items. 
Here we are just considering one set of work items that the user wants to manage together — a batch job.

There are several different patterns for parallel computation, each with strengths and weaknesses.

#### One Job object for each work item vs a single Job object for all work items
The latter is better for large numbers of work items. The former creates some overhead for the user and for the system 
to manage large numbers of Job objects.  
Also, with the latter, the resource usage of the job (number of concurrently running pods) can be easily adjusted 
using the `kubectl scale` command.  

#### Number of pods created equals number of work items, vs. each pod can process multiple work items
The former typically requires less modification to existing code and containers. 
The latter is better for large numbers of work items, for similar reasons to the previous bullet.  

#### Several approaches(方法？) use a work queue
This requires running a queue service, and modifications to the existing program or container to make it use the work queue. 
Other approaches are easier to adapt to an existing containerised application.(什么意思？)  

> NOTE: W is the number of work items.

|Pattern|Single Job object|Fewer pods than work items?|Use app unmodified?|Works in Kube 1.1?|.spec.completions|.spec.parallelism|
|:------|:------|:------|:------|:------|:------|:------|
|Job Template Expansion|||✓|✓|1|should be 1|
|Queue with Pod Per Work Item|✓||sometimes|✓|W|any|
|Queue with Variable Pod Coun|✓|✓||✓|1|any|
|Single Job with Static Work Assignment|✓||✓||W|any|

## 高级用法
### Specifying your own pod selector
默认情况下，你不用给Job指定selector，当Job被创建时，系统会为其设置selector，且selector不会与其它Job重叠。
当然在某些情况下，你需要自己指定selector时，配置`spec.selector`属性即可。  

在自己指定selector时需要注意，确保你的selector是唯一的！  

具体实例见官网：  
https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/#job-patterns  

## Alternatives
#### Bare Pods
When the node that a pod is running on reboots or fails, the pod is terminated and will not be restarted.  
However, a Job will create new pods to replace terminated ones.  

#### Replication Controller
Jobs are complementary to Replication Controllers. rc用来管理长期运行的Pod，而Job是用来管理短期运行的Pod.
Job is only appropriate for pods with RestartPolicy equal to OnFailure or Never.  

#### Single Job starts Controller Pod
太复杂，支持不好。  

#### Cron Jobs
定期运行的Job，不是一个概念。

## Reference
https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/  




  
