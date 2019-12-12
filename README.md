# MyKube
Combination of Vagrant and Ansible to spin up a Kubernetes cluster
Add velero, rook , ceph 

### Prerequisites
- Vagrant
- Ansible
...

### Define amount of nodes
in Vagrantfile:
```
N = 2
```


### Spin up cluster
```
$ vagrant up
```

### Verify on master
```
$ vagrant ssh k8s-master
$ kubectl get nodes
NAME         STATUS     ROLES    AGE     VERSION
k8s-master   Ready      master   3m43s   v1.13.4
node-1       Ready      <none>   118s    v1.13.4
node-2       NotReady   <none>   13s     v1.13.4
```

### Note$ 
git clone https://github.com/chineyuan/mykube.git
$ cd mykube
$ vagrant up
$ vagrant ssh k8s-master
$ sudo cp /etc/kubernetes/admin.conf .
$ sudo chown vagrant:vagrant admin.conf
$ exit
$ scp vagrant@192.168.52.10:/home/vagrant/admin.conf .
# use vagrant as password
$ export KUBECONFIG=`pwd`/admin.conf
$ k get nodes


## add second hdd into worker node.

```
file_to_disk="node-#{i}-sdb.vmdk"
unless File.exist?(file_to_disk)
    v.customize ['createhd', '--filename', file_to_disk, '--size', 50 * 1024]
end
v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
```
