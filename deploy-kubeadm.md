1. Initialize the nodes.

    The following steps must be performed on each of the three nodes, so `ssh` to `kubemaster` and run the steps, then to `kubenode01`, then to `kubenode02`

      1. Configure kernel parameters

            ```
            {
            cat <<EOF | sudo tee /etc/modules-load.d/11-k8s.conf
            br_netfilter
            EOF

            sudo modprobe br_netfilter

            cat <<EOF | sudo tee /etc/sysctl.d/11-k8s.conf
            net.bridge.bridge-nf-call-ip6tables = 1
            net.bridge.bridge-nf-call-iptables = 1
            net.ipv4.ip_forward = 1
            EOF

            sudo sysctl --system
            }
            ```

    1. Install `containerd` container driver and Kubeadm, kubelet, kubectl.

        ```bash
        {
            sudo apt update
            sudo apt install -y apt-transport-https ca-certificates curl
            curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
            echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
            sudo apt-get install -y containerd
            sudo apt-get update
            sudo apt-get install -y kubelet kubeadm kubectl
            sudo apt-mark hold kubelet kubeadm kubectl
            sudo crictl config \
                --set runtime-endpoint=unix:///run/containerd/containerd.sock \
                --set image-endpoint=unix:///run/containerd/containerd.sock
        }
        ```

  1. Initialize controlplane node


      1. Get the IP address of the `eth0` adapter of the controlplane

         ```
         ip addr show dev enp0s8
         ```

         Take the value printed for `inet` in the output. This should be:

         > 192.168.0.118

      1. Create a config file for `kubeadm` to get settings from 

          ```yaml
          kind: ClusterConfiguration
          apiVersion: kubeadm.k8s.io/v1beta3
          kubernetesVersion: v1.32.3          # <- At time of writing. Change as appropriate
          controlPlaneEndpoint: 192.168.0.118:6443
          networking:
            serviceSubnet: "10.96.0.0/16"
            podSubnet: "10.244.0.0/16"
            dnsDomain: "cluster.local"
          controllerManager:
            extraArgs:
              "node-cidr-mask-size": "24"
          apiServer:
            extraArgs:
              authorization-mode: "Node,RBAC"
            certSANs:
              - "192.168.0.118"
              - "kubemaster"
              - "kubernetes"
              
          ---
          kind: KubeletConfiguration
          apiVersion: kubelet.config.k8s.io/v1beta1
          cgroupDriver: systemd
          serverTLSBootstrap: true
          ```

      1. Run `kubeadm init` using the IP address determined above for `--apiserver-advertise-address`

         ```
         sudo kubeadm init \
            --apiserver-cert-extra-sans=kubemaster01 \
            --apiserver-advertise-address 192.168.0.118 \
            --pod-network-cidr=10.244.0.0/16
         ```

         Note the `kubeadm join` command output at the end of this run. You will require it for the step `Initialize the worker nodes` below

      1. Set up the default kubeconfig file

         ```
         {
         mkdir ~/.kube
         sudo cp /etc/kubernetes/admin.conf ~/.kube/config
         sudo chown vagrant:vagrant ~/.kube/config
         }
         ```

1. Initialize the worker nodes

    The following steps must be performed on both worker nodes, so `ssh` to `kubenode01` and run the steps, then to `kubenode02`

    * Paste the `kubeadm join` command from above step to the command prompt and enter it.
      e.g.
    ```
        kubeadm join 192.168.0.118:6443 --token ncog0i.6uvizek7ev79vd5q \
	        --discovery-token-ca-cert-hash sha256:d4d87b3b0ab2373aa2f95f3366deec5f903829d29f0885150429713c081fd939 
    ```