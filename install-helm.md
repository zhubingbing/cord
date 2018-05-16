## helm 安装

Helm是一个kubernetes应用的包管理工具，用来管理charts——预先配置好的安装包资源，有点类似于Ubuntu的APT和CentOS中的yum。

Helm chart是用来封装kubernetes原生应用程序的yaml文件，可以在你部署应用的时候自定义应用程序的一些metadata，便与应用程序的分发。

Helm和charts的主要作用：
-  应用程序封装
-  版本管理
-  依赖检查
-  便于应用程序分发

### 下载helm安装脚本
```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

创建tiller的serviceaccount和clusterrolebinding
```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```

然后安装helm服务端tiller，(目前最新版v2.8.2，可以使用阿里云镜像，如： helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.8.2 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts）
```
helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.8.2 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

我们使用-i指定自己的镜像，因为官方的镜像因为某些原因无法拉取，官方镜像地址是：gcr.io/kubernetes-helm/tiller:v2.8.1 该镜像的版本与helm客户端的版本相同，使用helm version可查看helm客户端版本。

为应用程序设置serviceAccount：

```
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

检查是否安装成功：
```
$ kubectl -n kube-system get pods|grep tiller
tiller-deploy-2372561459-f6p0z         1/1       Running   0          1h
$ helm version
Client: &version.Version{SemVer:"v2.3.1", GitCommit:"32562a3040bb5ca690339b9840b6f60f8ce25da4", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.3.1", GitCommit:"32562a3040bb5ca690339b9840b6f60f8ce25da4", GitTreeState:"clean"}
```

问题备注：
```
[root@os-k8s- 1 ~]# helm version
Client: &amp;version.Version{SemVer:&quot;v2.8.2&quot;,
GitCommit:&quot;a80231648a1473929271764b920a8e346f6de844&quot;,
GitTreeState:&quot;clean&quot;}
E0330 07:23:56.629068 19376 portforward.go:331] an error occurred
forwarding 36439 -&gt; 44134: error forwarding port 44134 to pod
bed7fcf1f11f86b5f6d1da916d2bdd04139ab5818616b41e8104937f6643edc4, uid :
unable to do port forwarding: socat not found
## 需要安装socat
yum install –y socat
```
