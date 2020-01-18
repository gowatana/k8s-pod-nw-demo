# k8s Pod Network を試す。

* Oracle Linux 7 / CentOS 7

Ansible の準備。

```
## Oracle Linux 7
# yum install -y --enablerepo=ol7_addons ansible git

## CentOS 7
# yum install -y ansible git
```

Ansible を実行するサーバに、Playbook などをダウンロード。

```
$ git clone https://github.com/gowatana/k8s-pod-nw-demo.git
```

設定対象の指定と確認。

```
$ cd ./k8s-pod-nw-demo/
$ cp inventory/hosts.template inventory/hosts
$ vi inventory/hosts
$ ssh-copy-id root@xxx.xxx.xxx.xxx
$ ansible all -m ping
```

OS 設定と、docker / kubeadm　などのインストール。  
設定内容によっては、最後に OS が自動リブートされることあり。

```
$ ansible-playbook setup_os.yml
```

Kubernetes Master / Worker にするサーバにデモむけファイルを配置。

```
$ ansible-playbook copy_config_files.yml
```

# 参考

Installing a Pod network add-on  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network

EOF
