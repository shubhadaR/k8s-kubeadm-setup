first take backup
sudo ETCDCTL_API=3 etcdctl snapshot save backup.db   --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/peer.crt   --key=/etc/kubernetes/pki/etcd/peer.key

ensure db status
etcdctl snapshot status /path/to/backup.db

save all resources in all namespaces
kubectl get all --all-namespaces -o yaml > kubernetes-backup.yaml

