# 理解 kubernetes Objects
这一页我们将介绍 K8s objects 在 Kubernetes API 是如何表示的，以及在 .yaml 格式中你该如何表达它们。

## 理解 Kubernetes objects
在 K8s 系统中，Kubernetes objects 是持久化的实体。K8s 使用这些实体代表集群中的状态。尤其的，它们可以描述为：
1. 正在运行那些容器化的应用程序（以及在哪一个节点上）

2. 这些应用程序可用的资源

3. 围绕这些应用程序的行为方式的策略，例如重启策略，升级策略，容错策略。

一个 K8s 对象是一个 “意图策略” —— 一旦你创建该对象，K8s 系统将会不断的工作确保对象存在。通过创建一个对象，你能够高效的告诉 K8s 系统你希望集群中的工作负载看起来是怎么样的；这就是你集群期望的状态。

为了使用 K8s 对象 —— 无论是创建，修改，删除它们 —— 你需要使用 K8s API。当你使用 kubectl 命令行接口，举个例子，CLI 为你请求了必要的 K8s API。你也同样可以在应用程序中使用客户端库来直接调用 K8s API。

### Object Spec and Status 对象指定和状态
几乎每一个 K8s object 包括两个内嵌对象字段，用来管理对象的配置：对象的 spec 和 对象的 status。对于拥有 spec 字段的对象，当你创建该对象的时候必须设置该它，提供您希望资源具有的特征的描述：你期望的状态。

status 描述了该对象的当前状态，该对象由 K8s 系统和组件提供和更新。K8s 的 control plane 不断地，积极地管理每个真实的状态知道匹配你提供期望的状态。

举个例子：在 K8s 中， 一个 Deployment 是一个对象，用来代表运行在你集群中的应用程序。当你创建了该 Deployment, 你可能设置该 Deployment spec 字段来指定你想要运行应用程序的副本集。K8s 系统读取该 Deploymemnt spec，然后启动三个期望应用程序的副本集 —— 更新当前的状态知道匹配你的指定。如果这些实例中任何一个失败了（或者状态改变），K8s 系统将会通过修订错误来响应指定和当前状态的不同 —— 在这个例子中，启动了一个 replacement 实例。

### Describing a Kubernetes object
当你在 K8s 中创建了一个对象，你必须提供对象的 spec来描述对象期望的状态，同时还有关于该对象简单的信息（例如 name）。当你使用 K8s API 创建该对象（要么直接创建，或者利用 kubectl）, 该 API 请求必须在请求正文中以 JSON 形式包含该信息。很多时候，你在一个 .yaml 文件给 kubectl 提供信息。kubectl 将信息转化为 JSON，当你创建一个 API 请求的实际。

这是一个 .yaml 文件的例子，展示 K8s Deployment 要求的字段以及对象 spec。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

使用 .yaml 文件创建 Deployment 的一种方法是使用 kubectl 命令行界面中的 kubectl apply 命令，将 .yaml 文件作为参数传递。 下面是一个例子：

```bash
kubectl apply -f https://k8s.io/examples/application/deployment.yaml --record
```

输出类似于这样：
```bash
deployment.apps/nginx-deployment created
```

### Required Fields
在你为了创建 K8s 对象的 .yaml 文件中，你需要为以下字段设置值：

1. apiVersion —— 你想用哪一个 K8s API 版本来创建该对象。

2. kind —— 你想要创建对象的类型是什么

3. metadata —— 独一无二的定义该对象的数据，包括 name, UID, 以及可选的 namespace

4. spec —— 你期望对象的状态

对象的精确的格式在每个 K8s 对象中都是相同的，并且包含内嵌的字段对于指定的对象。