Kubernetes 环境安装
=====================
注意：如果环境启动过程中出现问题，请参考https://github.com/xiaods/kubernetes-the-hard-way-vagrant/README。已经在MacOS下验证。


Virtualbox  v6.0
https://www.virtualbox.org/wiki/Downloads

=====================

Vagrant
https://www.vagrantup.com/downloads.html


Linux镜像：
--------------------

vagrant box add ./box/bionic-server-cloudimg-amd64-vagrant.box --name ubuntu/bionic64


=====================
集群环境代码：
https://github.com/xiaods/kubernetes-the-hard-way-vagrant

  readonly k8s_version="v1.14.2"
  readonly etcd_version="v3.3.9"
  readonly cfssl_version="R1.2"
  readonly traefik_version="v1.3.8"
  readonly crio_version="v1.14.0"


需要的软件二进制，下载，放到binary目录

https://storage.googleapis.com/kubernetes-release/release/v1.14.2/bin/linux/amd64/kube-apiserver
https://storage.googleapis.com/kubernetes-release/release/v1.14.2/bin/linux/amd64/kube-controller-manager
https://storage.googleapis.com/kubernetes-release/release/v1.14.2/bin/linux/amd64/kube-scheduler
https://storage.googleapis.com/kubernetes-release/release/v1.14.2/bin/linux/amd64/kube-proxy
https://storage.googleapis.com/kubernetes-release/release/v1.14.2/bin/linux/amd64/kubelet
https://storage.googleapis.com/kubernetes-release/release/v1.14.2/bin/linux/amd64/kubectl


https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz


https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz
https://github.com/opencontainers/runc/releases/download/v1.0.0-rc4/runc.amd64
https://files.schu.io/pub/cri-o/crio-amd64-v1.14.0.tar.gz

https://github.com/containous/traefik/releases/download/v1.3.8/traefik_linux-amd64

====================
前置环境要求：
Requirements Host
```
Vagrant (with VirtualBox)
Minimum of 7x 512MB of free RAM
cfssl, cfssljson and kubectl (scripts/install-tools can be used to download and install the binaries to /usr/local/bin)
```

安装步骤：
```
git clone https://github.com/xiaods/kubernetes-the-hard-way-vagrant.git
cd kubernetes-the-hard-way-vagrant
vagrant destroy -f   # remove previous setup
./scripts/setup      # takes about 5 minutes or more
```

测试：
```
kubectl create -f ./manifests/kube-dns.yaml
[...]
kubectl get pods -l k8s-app=kube-dns -n kube-system
[...]
kubectl run --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
[...]
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
kubectl exec -ti $POD_NAME -- nslookup kubernetes

kubectl create -f ./manifests/nginx.yaml
deployment "nginx" created
service "nginx" created

NODE_PORT=$(kubectl get svc nginx --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
for i in {0..2}; do curl -sS 192.168.199.2${i}:${NODE_PORT} | awk '/<h1>/{gsub("<[/]*h1>", ""); print $0}'; done
Welcome to nginx!
Welcome to nginx!
Welcome to nginx!
```
====================
注意事情:

1. cfssl下载包在macos有兼容问题无法使用，请使用brew覆盖安装
brew install cfssl
Bug细节：
https://github.com/kubernetes/kubernetes/issues/66995

2. error: enable-admission-plugins plugin "Initializers" is unknown
k8s 1.4 不在支持Initializers，去掉

3. k8s排错命令参考：https://zhuanlan.zhihu.com/p/34323536

4. kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/











