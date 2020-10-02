# kustomize

https://github.com/kubernetes-sigs/kustomize

## 什么是kustomize

`kustomize` lets you customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.

kustomizes可以让你在不修改原始YAML文件的情况下，定制化原始的、无模板的YAML文件，以满足你的多种目的或使用场景。

`kustomize` targets kubernetes; it understands and can patch [kubernetes style](https://kubernetes-sigs.github.io/kustomize/api-reference/glossary#kubernetes-style-object) API objects. It's like [`make`](https://www.gnu.org/software/make), in that what it does is declared in a file, and it's like [`sed`](https://www.gnu.org/software/sed), in that it emits edited text.

kustomize是为kubernetes开发的，它了解并可以为kubernetes样式的API对象打补丁。就像make一样，它所做的事情在文件中声明，就像sed一样，它生成编辑后的文本。

## kustomize vs helm

https://cloud.tencent.com/developer/article/1484121

他们的区别主要在工作流程上：

- Helm 的基础流程比较`瀑布`：定义 Chart->填充->运行，**在 Chart 中没有定义的内容是无法更改的**；
- Kustomize 的用法比较`迭代`：Base 和 Overlay 都是可以独立运作的，增加新对象，或者对编写 Base 时未预料的内容进行变更，都不在话下，即可以不用预定义要修改的内容。

## 官方示例

https://github.com/kubernetes-sigs/kustomize/blob/master/examples/helloWorld/README.md

Steps:

1. Clone an existing configuration as a [base](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/glossary.md#base).
2. Customize it.
3. Create two different [overlays](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/glossary.md#overlay) (*staging* and *production*) from the customized base.
4. Run kustomize and kubectl to deploy staging and production.

```
mkdir demo
cd demo; mkdir base; cd base

curl -s -o "#1.yaml" "https://raw.githubusercontent.com\
/kubernetes-sigs/kustomize\
/master/examples/helloWorld\
/{configMap,deployment,kustomization,service}.yaml"

or

wget "https://raw.githubusercontent.com\
/kubernetes-sigs/kustomize\
/master/examples/helloWorld\
/{configMap,deployment,kustomization,service}.yaml"

fffffreedom:demo jonny$ tree base
base
├── configMap.yaml
├── deployment.yaml
├── kustomization.yaml
└── service.yaml

## Customize the base

sed -i.bak 's/app: hello/app: my-hello/' \
    base/kustomization.yaml

## Create Overlays
mkdir overlays
mkdir -p overlays/staging
mkdir -p overlays/production

## Staging Kustomization
## 在staging目录下生成kustomization.yaml和map.yaml文件
cat <<'EOF' >overlays/staging/kustomization.yaml
namePrefix: staging-
commonLabels:
  variant: staging
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am staging!
bases:
- ../../base
patchesStrategicMerge:
- map.yaml
EOF

cat <<EOF >overlays/staging/map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
EOF

## 运行 kustomize build overlays/staging 查看patch后的输出

## 生产环境
cat <<EOF >overlays/production/kustomization.yaml
namePrefix: production-
commonLabels:
  variant: production
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am production!
bases:
- ../../base
patchesStrategicMerge:
- deployment.yaml
EOF

cat <<EOF >overlays/production/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  replicas: 10
EOF

## 查看当前目录文件层次：
fffffreedom:demo jonny$ tree .
.
├── base
│   ├── configMap.yaml
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   ├── kustomization.yaml.bak
│   └── service.yaml
└── overlays
    ├── production
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── staging
        ├── kustomization.yaml
        └── map.yaml
        
## 最后查看staging和production的patch结果
diff \
  <(kustomize build overlays/staging) \
  <(kustomize build overlays/production) |\
  more

## 部署
To deploy, pipe the above commands to kubectl apply:

kustomize build overlays/staging |\
    kubectl apply -f -
kustomize build overlays/production |\
   kubectl apply -f -
```

## kustomize用法

https://kubernetes-sigs.github.io/kustomize/api-reference/

可以修改YAML文件中定义的K8s资源对象，详见上面的API文档：

- annotations
- labels
- images
- and so on

