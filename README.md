# MyKube
Combination of Vagrant and Ansible to spin up a Kubernetes cluster
Add velero, rook , ceph 

### Prerequisites
- Vagrant
- Ansible
- rook
- velero
...

### Define amount of nodes
in Vagrantfile:
```
N = 2
```

### Spin up cluster
  git clone https://github.com/chineyuan/mykube.git
  cd mykube
  vagrant up
  vagrant ssh k8s-master
  sudo cp /etc/kubernetes/admin.conf .
  sudo chown vagrant:vagrant admin.conf
  exit
  scp vagrant@192.168.52.10:/home/vagrant/admin.conf .
  # use vagrant as password
  export KUBECONFIG=`pwd`/admin.conf

### Verify on master
```
$ vagrant ssh k8s-master
$ kubectl get nodes
NAME         STATUS     ROLES    AGE     VERSION
k8s-master   Ready      master   3m43s   v1.14.0
node-1       Ready      <none>   118s    v1.14.0
node-2       NotReady   <none>   13s     v1.14.0
```

## install nginx
  kubectl apply -f nginx-app/base.yaml
  kubectl -n nginx-example get pod

## install rook
  kubectl apply -f rook/common.yaml
  kubectl apply -f rook/operator.yaml
  kubectl apply -f rook/cluster.yaml
  kubectl apply -f rook/cluster-test.yaml
  kubectl apply -f rook/storageclass-test.yaml
  kubectl apply -f rook/toolbox.yaml

  kubectl -n rook-ceph exec -it toolbox.... sh
  ceph status
  if not HEALTH_OK
  increate pg_num and pgp_num to 100
  ceph osd pool set replicapool pg_num 100
  ceph osd pool set replicapool pgp_num 100

## wordpress example
  kubectl apply -f rook/mysql.yaml
  kubectl apply -f rook/wordpress.yaml
  kubectl get all

### reinstall rook
  kubectl delete ns rook-cep

  rook-ceph         Terminating   2d2h
  Error from server (Conflict): Operation cannot be fulfilled on namespaces "rook-ceph":
  The system is ensuring all content is removed from this namespace.  
  Upon completion, this namespace will automatically be purged by the system.
  It turn out the crd issue.

  kubectl get crd 
  kubectl patch crd cephclusters.ceph.rook.io -p '{"metadata":{"finalizers": []}}' --type=merg
  kubectl delete crd --all


## install velero
  tar -xvzf velero-v0.11.0-darwin-amd64.tar.gz 
  sudo mv velero /usr/local/bin/ 
  velero help

  quick start - https://velero.io/docs/master/contributions/minio/
  
  These instructions start the Velero server and a Minio instance that is accessible from within the cluster only.
  kubectl apply -f velero/minio/00-minio-deployment.yaml

	velero install \
    --provider aws \ 
    --bucket velero \
    --secret-file ./credentials-velero \
    --use-volume-snapshots=false \
		--use-restic \
    --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://minio.velero.svc:9000 \
		--plugins=velero/velero-plugin-for-aws:v1.0.0

  one line command for ,r
  velero install  --provider aws  --bucket velero  --secret-file ./credentials-velero  --use-volume-snapshots=false  --use-restic  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://minio.velero.svc:9000 --plugins=velero/velero-plugin-for-aws:v1.0.0

* note: need to add --plugins *


## velero backup example
  kubectl apply -f velero/nginx-app/base.yaml

  velero backup create nginx-backup --selector app=nginx
  velero backup get
  kubectl -n nginx-example get all
  kubectl delete ns nginx-example
  kubectl get ns
  velero restore create --from-backup nginx-backup
  velero restore describe nginx-backup-20191215110415
  velero restore get
  kubectl get ns
  kubectl -n nginx-example get pod
  kubectl delete ns nginx-example

  kubectl apply -f velero/nginx-app/with-pv.yaml
  kubectl -n nginx-example get all
  kubectl -n nginx-example describe deploy nginx

	velero backup create nginx-backup2 --include-namespaces nginx-example
	kubectl get ns
  kubectl -n nginx-example get all
  velero backup get
	kubectl delete ns nginx-example   
	velero restore create --from-backup nginx-backup
	velero restore create --from-backup nginx-backup2
  velero restore describe nginx-backup2-20191213224830
  velero restore logs nginx-backup2-20191213224830

	velero restore get
	velero restore describe nginx-backup-20191212231330
	
## test rook wordpress backup
  remember add annotations: backup.velero.io/backup-volumes: mysql-persistent-storage
  kubectl apply -f rook/mysql.yaml
  kubectl apply -f rook/wordpress.yaml
  kubectl get pods  -l app=wordpress
  kubectl get svc
  kubectl get pv,pvc
  velero backup create wordpress-backup2   --selector app=wordpress
  velero backup describe wordpress-backup2

  kubectl delete -f rook/wordpress.yaml
  kubectl delete -f rook/mysql.yaml
  kubectl get pvc,pv,svc
  kubectl get pods
  velero restore create --from-backup wordpress-backup2 
  velero restore describe wordpress-backup2-20191215130621 
  kubectl get pods
  kubectl get pvc
  kubectl get svc
  velero --help

## debug velero restic
  kubectl -n velero get pod
  velero restic repo get
  velero restic 

	
## clean up velero
	kubectl delete namespace/velero clusterrolebinding/velero
	kubectl delete crds -l component=velero


## Not: add second hdd into worker node.

```
file_to_disk="node-#{i}-sdb.vmdk"
unless File.exist?(file_to_disk)
    v.customize ['createhd', '--filename', file_to_disk, '--size', 50 * 1024]
end
v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
```
