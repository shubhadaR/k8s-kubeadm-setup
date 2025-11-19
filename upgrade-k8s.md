first take backup
```
sudo ETCDCTL_API=3 etcdctl snapshot save backup.db   --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/peer.crt   --key=/etc/kubernetes/pki/etcd/peer.key
```
ensure db status
```
etcdctl snapshot status /path/to/backup.db
```

save all resources in all namespaces
```
kubectl get all --all-namespaces -o yaml > kubernetes-backup.yaml
```
check kubernetes version:
```
kubectl version
```

pull new version repo for k8s and change it in gpg file

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

now check you have updated kubeadm version pointed
```
sudo apt update
sudo apt-cache madison kubeadm
```

now install new kubeadm version
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.14-1.1' && \
sudo apt-mark hold kubeadm
```

check kubeadm version
```
kubeadm version
```

upgrade kubeadm
```
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.31.14
```
upgrade kubelet on cntrolplane 
```
kubectl drain controlplane --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.14-1.1' kubectl='1.31.14-1.1' && \
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
kubectl uncordon controlplane

```

now check controlpnae upgraded to new version
```
kubectl get nodes
```
upgrade worketr nodes

now change repo to new version on worker node
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

install new version of kubeadm on worker node
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.14-1.1' && \
sudo apt-mark hold kubeadm
```
upgrade node with new version
```
sudo kubeadm upgrade node
kubectl drain node01 --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.14-1.1' kubectl='1.31.14-1.1' && \
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon node01
```


