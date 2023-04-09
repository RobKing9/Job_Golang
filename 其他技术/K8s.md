# K8s指南

## Mac使用MiniKube搭建K8s环境

1. 首先配置好docker环境

2. 安装kubectl

   ```shell
   brew install kubectl 
   ```

3. 在[官网](https://www.virtualbox.org/wiki/Downloads)下载virtualbox，配置为默认驱动

   ```shell
   minikube config set driver virtualbox
   ```

4. 或者可以使用docker为默认驱动

   ```
   minikube config set driver docker
   ```

5. [下载minikube](https://minikube.sigs.k8s.io/docs/start/)

   ```shell
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
   sudo install minikube-darwin-amd64 /usr/local/bin/minikube
   ```

6. 启动minikube

   ```
   minikube start --image-mirror-country=cn --registry-mirror=https://ug267z0y.mirror.aliyuncs.com
   ```

   `--image-mirror-country=cn`自动使用阿里云服务来支持minikube的环境配置

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20220925010556.png)

   常用配置参数如下

   - `--driver=***` 从1.5.0版本开始，Minikube缺省使用系统优选的驱动来创建Kubernetes本地环境，比如您已经安装过Docker环境，minikube 将使用 `docker` 驱动
   - `--cpus=2`: 为minikube虚拟机分配CPU核数
   - `--memory=2048mb`: 为minikube虚拟机分配内存数
   - `--registry-mirror=***` 为了提升拉取Docker Hub镜像的稳定性，可以为 Docker daemon 配置镜像加速，参考[阿里云镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
   - `--kubernetes-version=***`: minikube 虚拟机将使用的 kubernetes 版本

7. 验证是否成功

   ```
   kubectl get po -A
   ```

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/%E6%88%AA%E5%B1%8F2022-09-25%2001.11.59.png)

8. 打开Kubernetes控制台

   ```
   minikube dashboard
   ```

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/%E6%88%AA%E5%B1%8F2022-09-25%2001.08.33.png)

自动会跳转到网页控制台

## Kubectl指令

- 查看集群配置 `kubectl config view`

- 与deployment相关

  编写deployment.yaml文件，创建

  ```sh
  kubectl create -f deployment.yaml
  ```

  查看所有的deployment `kubectl get deployments`

  删除指定的pod `kubectl delete deployments <deploymentName>`

- 应用升级

  ```
  kubectl set image deloyment web nginx=nginx:1.15
  ```

  查看是否成功

  ```
  kubectl rollout status deloyment web 
  ```

  查看历史版本

  ```
  kubectl rollout history deloyment web
  ```

  回滚到上一个版本

  ```
  kubectl rollout undo deloyment web
  ```

  后面加上 --to-version = 2 回滚到指定版本

  弹性伸缩

  ```
  kubectl scale deloyment web --repicas = 10 
  ```

  

- 与pod相关

  查看所有的pod `kubectl get pods`

  删除指定的pod `kubectl delete pod <podName>`

- 与service相关

  查看services `kubectl get services` 或者`kubectl get svc`可以知道映射到端口号

  删除指定的service `kubectl delete service <serviceName>`

## Minikube基本指令

- 停止一个集群 `minikube stop`
-  删除一个集群 `minikube delete`, 删除所有集群 `minikube delete --all`
- 查看ip地址：`minikube ip`

## 通过kubectl发布服务

1. 创建服务` hello-minikubel`

   ```sh
   kubectl create deployment hello-minikube1 --image=registry.cn-hangzhou.aliyuncs.com/google_containers/echoserver:1.10
   ```

2. 暴露端口

   ```sh
   kubectl expose deployment hello-minikube1 --type=LoadBalancer --port=8080
   ```

3. 访问端口

   ```sh
   minikube service hello-minikube1
   ```

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20220925094057.png)

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20220925094149.png)

   

4. 查看所有的deployment服务 `kubectl get deployments`

   查看pods `kubectl get pods`

   查看services `kubectl get services` 或者`kubectl get svc`可以知道映射到端口号

   查看创建的 Pod 和 Service `kubectl get pod,svc -n kube-system`

5. 删除服务

   ```sh
   kubectl delete deployment -n default hello-minikube1 --force --grace-period=0
   ```

### 部署Nginx服务

```sh
# 创建一个部署，使用镜像nginx:latest 部署名称为hello-nginx
kubectl create deployment hello-nginx --image=nginx:latest

# 查看部署列表
kubectl get deployments

# 查看pods列表
kubectl get pods

# 查看时间
kubectl get events

# 查看集群配置
kubectl config view

# 导出一个部署的应用 80是容器对外暴露的端口，会被分配一个随机端口以供访问
kubectl expose deployment hello-nginx --type=NodePort --port=80

# 列出hello-nginx的信息 自动打开浏览器
minikube service hello-nginx
```

[参考链接](https://cloud.tencent.com/developer/article/1939368)

## 通过minikube部署go语言项目

### 制作程序镜像

1. 编写dockefile文件

2. 根据dockerfile文件build镜像 

3. 验证进行，将其运行起来成容器

4. 推送镜像到DockerHub

   登录[DockerHub](https://hub.docker.com/) 使用账号和密码登录即可 `docker login`

   ```sh
   //用Dockerfile重新构建镜像，指定镜像仓库名。构建完成后将镜像然后推送到DockerHub上
   docker build -t robking/minukube-go .
   docker push robking/minikube-go
   ```

### Kubernets部署应用

1. 定义预期状态,编写**deployment.yaml**文件

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-go-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: go-app
     template:
       metadata:
         labels:
           app: go-app
       spec:
         containers:
           - name: go-app-container
             image: rob1king1/minikube-go
             resources:
               limits:
                 memory: "128Mi"
                 cpu: "500m"
             ports:
               - containerPort: 3000
   ```

   `Kubernetes Deployment` 对象（清单文件的`kind`里指定的）表示运行在集群中的应用。文件里还指定了应用需要一个副本运行（`replicas`），以及运行的容器名和容器的镜像、资源大小等信息

   指令生成

   ```
   kubectl create deployment web --image=nginx -o yaml --dry-run  > nginx.yaml
   ```

   

2. 使用`deployment.yaml`创建`Deployment`对象来运行`Go`应用程序的容器

   ```
   kubuctl apply -f delpyment.yaml
   ```

   ```sh
   kubectl create -f deployment.yaml
   ```

3. 可以查看pods和services的状态，event的信息

4. 暴露应用,可以输出到文件中yaml，然后apply即可

   ```
   kubectl expose deployment my-go-app --type=NodePort --name=go-app-svc --target-port=3000 
   ```

5. 查看应用的端口和ip地址

   ```
   //查看ip
   minikube ip
   //查看端口
   kubectl get svc
   ```

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20220925210559.png)

6. 列出hello-nginx的信息 自动打开浏览器

   ```
   minkube service go-app-svc
   ```

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20220925210943.png)

   

## 参考链接

- [配置minikube环境](https://github.com/AliyunContainerService/minikube/wiki)
- [Minikube安装成功Kubernetes,一次过！](https://www.cnblogs.com/sanshengshui/p/11228985.html)
- [Minikube 快速入门手册](https://www.jianshu.com/p/ef400bfea973?u_atoken=bcfdb46e-4276-4aef-8ffe-de3766992479&u_asession=01VfFY4apfRkCAOhyqWkZma3OboX-GO8c-jTRhLWSX7Uv_CoVzgdWStTxKEzDNNEF5X0KNBwm7Lovlpxjd_P_q4JsKWYrT3W_NKPr8w6oU7K_fe2ACVfnep7PkD1dtUFBikC1LUOsbnJoxzzl_EpVkQGBkFo3NEHBv0PZUm6pbxQU&u_asig=05znWRq41MaGvV2vCfboGRsEQUT2H40OFFzr4tieCuXsKbnoCsvtap_mGUXtCyJdFADdkGZht6azHd8DrUIdzK4svffEY1am4tU0FLzMhIFTHVMt9ZKZ-6hLU0j9spJJqiKIrrG2DvsgltUUSJ5olBGdmDlZHdu7OlyKAjPJHeFyr9JS7q8ZD7Xtz2Ly-b0kmuyAKRFSVJkkdwVUnyHAIJzauGK_4xK-5m-37CTHDCnGd6io4DON8gC-EU9-tIUMKRUDrMH9U5oL9icxwFsJSlo-3h9VXwMyh6PgyDIVSG1W-HEAamnBy1z2kSIMsrNaRGFpy3JlBHANnS6YW7W30lsfC1v6niF1ZBJ7nV-05ozjJUQpu_6H8HKbXpBOWJ1vGNmWspDxyAEEo4kbsryBKb9Q&u_aref=xwGuv9MSUEJjUvIYxyE82JbdDtI%3D)
- [你好，Minikube](https://kubernetes.io/zh-cn/docs/tutorials/hello-minikube/)
- [Kubernetes入门之本地集群minikube](https://loocode.com/post/kubernetes-ru-men-zhi-ben-di-ji-qun-minikube)
- [Kubectl基本概念&常用命令](https://huweicai.com/kubectl-concepts-and-commands/)
- [使用minikube部署一个应用 并通过浏览器访问](https://cloud.tencent.com/developer/article/1939368)
- [Kubernetes入门实践--部署运行Go项目](https://juejin.cn/post/6844904205598081037)

