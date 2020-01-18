# テンプレート VM の準備。

OS

* Oracle Linux 7 / CentOS 7

Linux インストール後の、テンプレート VM の設定。

```
# nmcli c mod ens192 connection.autoconnect yes
# ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ''
# ssh-copy-id localhost
# yum update -y
# init 0
```

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

OS Firewall 設定

```
$ ansible-playbook -C --tags flannel step-2_firewall.yml
$ ansible-playbook --tags flannel step-2_firewall.yml
```

Kubernetes Master Node / Worker Node にするサーバにデモむけファイルを配置。

```
$ ansible-playbook -C step-3_copy-configs.yml
$ ansible-playbook step-3_copy-configs.yml
```

k8s クラスタ（Pod Network: flannel）の作成。

```
$ ansible-playbook -C --tags flannel step-4_kubeadm.yml
$ ansible-playbook --tags flannel step-4_kubeadm.yml
```

# 参考

Installing a Pod network add-on  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network

EOF
