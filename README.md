# テンプレート VM の準備。

## 想定 OS

* Oracle Linux 7 / CentOS 7

## Linux インストール後のテンプレート VM 設定。

```
# nmcli c mod ens192 connection.autoconnect yes
# ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ''
# ssh-copy-id localhost
# yum install -y open-vm-tools
# yum update -y
# init 0
```

## VM のクローン

https://github.com/gowatana/deploy-k8s-demo-vms

# 実行環境 Ansible / git の準備。

```
## Oracle Linux 7
# yum install -y --enablerepo=ol7_addons ansible git

## CentOS 7
# yum install -y ansible git
```

# k8s のセットアップ。

Ansible を実行するサーバに、Playbook などをダウンロード。

```
$ git clone https://github.com/gowatana/k8s-pod-nw-demo.git
```

設定対象の指定と確認。

```
$ cd ./k8s-pod-nw-demo/
$ cp inventory/hosts.template inventory/hosts
$ vi inventory/hosts
$ ansible all -m ping
```

OS 設定と、docker / kubeadm　などのインストール。  
設定内容によっては、最後に OS が自動リブートされることあり。

```
$ ansible-playbook -C step-1_os-base.yml
$ ansible-playbook step-1_os-base.yml
```

## Pod Network: flannel パターン

OS Firewall 設定。

```
$ ansible-playbook -C --tags flannel step-2_firewall.yml
$ ansible-playbook --tags flannel step-2_firewall.yml
```

Kubernetes Master Node / Worker Node にファイルを配置。  

```
$ ansible-playbook -C --tags flannel step-3_copy-configs.yml
$ ansible-playbook --tags flannel step-3_copy-configs.yml
```

k8s クラスタの作成。

```
$ ansible-playbook -C --tags flannel step-4_kubeadm.yml
$ ansible-playbook --tags flannel step-4_kubeadm.yml
```

## Pod Network: Calico パターン

OS Firewall 設定。

```
$ ansible-playbook -C --tags calico step-2_firewall.yml
$ ansible-playbook --tags calico step-2_firewall.yml
```

Kubernetes Master Node / Worker Node にファイルを配置。  

```
$ ansible-playbook -C --tags calico step-3_copy-configs.yml
$ ansible-playbook --tags calico step-3_copy-configs.yml
```

k8s クラスタの作成。

```
$ ansible-playbook -C --tags calico step-4_kubeadm.yml
$ ansible-playbook --tags calico step-4_kubeadm.yml
```

## Pod Network: Canal パターン

OS Firewall 設定。

```
$ ansible-playbook -C --tags canal step-2_firewall.yml
$ ansible-playbook --tags canal step-2_firewall.yml
```


Kubernetes Master Node / Worker Node にファイルを配置。  

```
$ ansible-playbook -C --tags canal step-3_copy-configs.yml
$ ansible-playbook --tags canal step-3_copy-configs.yml
```

k8s クラスタの作成。

```
$ ansible-playbook -C --tags canal step-4_kubeadm.yml
$ ansible-playbook --tags canal step-4_kubeadm.yml
```

## Pod Network: Antrea (VXLAN) パターン

OS Firewall 設定。

```
$ ansible-playbook -C --tags antrea_vxlan step-2_firewall.yml
$ ansible-playbook --tags antrea_vxlan step-2_firewall.yml
```

Kubernetes Master Node / Worker Node にファイルを配置。  

```
$ ansible-playbook -C --tags antrea_vxlan step-3_copy-configs.yml
$ ansible-playbook --tags antrea_vxlan step-3_copy-configs.yml
```

k8s クラスタの作成。

```
$ ansible-playbook -C --tags antrea_vxlan step-4_kubeadm.yml
$ ansible-playbook --tags antrea_vxlan step-4_kubeadm.yml
```

## Pod Network: Antrea (Geneve) パターン

OS Firewall 設定。

```
$ ansible-playbook -C --tags antrea_geneve step-2_firewall.yml
$ ansible-playbook --tags antrea_geneve step-2_firewall.yml
```

Kubernetes Master Node / Worker Node にファイルを配置。  

```
$ ansible-playbook -C --tags antrea_geneve step-3_copy-configs.yml
$ ansible-playbook --tags antrea_geneve step-3_copy-configs.yml
```

k8s クラスタの作成。

```
$ ansible-playbook -C --tags antrea_geneve step-4_kubeadm.yml
$ ansible-playbook --tags antrea_geneve step-4_kubeadm.yml
```

# 確認。

Master Node に root ユーザで SSH ログインする。

```
$ ssh root@10.0.3.181
[root@k8s-m-05-01 ~]#
```

クラスタのノード構成確認。

```
[root@k8s-m-05-01 ~]# kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
k8s-m-05-01   Ready    master   27m   v1.14.2
k8s-w-05-01   Ready    <none>   26m   v1.14.2
k8s-w-05-02   Ready    <none>   26m   v1.14.2
k8s-w-05-03   Ready    <none>   26m   v1.14.2
[root@k8s-m-05-01 ~]# kubectl describe nodes | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
```

デモ ネームスペースの作成。

```
[root@k8s-m-05-01 ~]# kubectl create namespace demo-01
namespace/demo-01 created
```

デモ アプリの起動。

```
[root@k8s-m-05-01 ~]# kubectl apply -f 03_demo/yelb.yaml -n demo-01
service/redis-server created
service/yelb-db created
service/yelb-appserver created
service/yelb-ui created
deployment.extensions/yelb-ui created
deployment.extensions/redis-server created
deployment.extensions/yelb-db created
deployment.extensions/yelb-appserver created
```

Pod の起動確認。

```
[root@k8s-m-05-01 ~]# kubectl get pods -n demo-01
NAME                              READY   STATUS    RESTARTS   AGE
redis-server-6bd9dc4f99-lr5pb     1/1     Running   0          3m3s
yelb-appserver-5fd98f7df7-587ts   1/1     Running   0          3m3s
yelb-db-6569f794f4-6mwjd          1/1     Running   0          3m3s
yelb-ui-57474ddfc4-stf9b          1/1     Running   0          3m3s
```

ノード ポート番号の確認。例では yelb-ui が 31872 番ポート。  
Web ブラウザから ``` http:// Worker Node IP:31872 ``` にアクセスできる。

```
[root@k8s-m-05-01 ~]# kubectl get service -n demo-01
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
redis-server     ClusterIP   10.102.84.250    <none>        6379/TCP       56s
yelb-appserver   ClusterIP   10.99.118.66     <none>        4567/TCP       56s
yelb-db          ClusterIP   10.108.248.155   <none>        5432/TCP       56s
yelb-ui          NodePort    10.104.37.233    <none>        80:31872/TCP   56s
```

デモ アプリの削除。

```
[root@k8s-m-05-01 ~]# kubectl delete -f 03_demo/yelb.yaml -n demo-01
service "redis-server" deleted
service "yelb-db" deleted
service "yelb-appserver" deleted
service "yelb-ui" deleted
deployment.extensions "yelb-ui" deleted
deployment.extensions "redis-server" deleted
deployment.extensions "yelb-db" deleted
deployment.extensions "yelb-appserver" deleted
```

デモ ネームスペースの削除。

```
[root@k8s-m-05-01 ~]# kubectl delete namespaces demo-01
namespace "demo-01" deleted
```

# 参考

Installing a Pod network add-on  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network

EOF
