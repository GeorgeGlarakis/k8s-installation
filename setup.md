## Create a Kubernetes Cluster manually, using the console

>Run the following on **all Nodes**
#### Disable the Swap memory
`sudo swapoff -a`

#### Add Docker's official GPG key:
`$ sudo apt-get update`

`$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg`

`$ sudo install -m 0755 -d /etc/apt/keyrings`

`$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`

`$ sudo chmod a+r /etc/apt/keyrings/docker.asc`

#### Add the Docker repository to Apt sources:
```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Install Docker:
`$ sudo apt-get update`

`$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

#### Install the CNI Network Plugins:
`$ wget https://github.com/containernetworking/plugins/releases/download/v1.4.1/cni-plugins-linux-amd64-v1.4.1.tgz`

`$ sudo mkdir -p /opt/cni/bin`

`$ sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.4.1.tgz`

#### Forward IPv4 and let iptables see bridged traffic:
```
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
```

`$ sudo modprobe overlay`

`$ sudo modprobe br_netfilter`

#### Set sysctl parameters required by setup:
```
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
```

#### Apply sysctl parameters without reboot:
`$ sudo sysctl --system`

#### Add Kuberentes's official GPG key and add it to the Apt sources:
```
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | 
    sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | 
    sudo tee /etc/apt/sources.list.d/kubernetes.list
```

#### Install Kubernetes:
`$ sudo apt-get update`

`$ sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni`

`$ sudo apt-mark hold kubelet kubeadm kubectl kubernetes-cni`

#### Change containerd config file:
`$ sudo vi /etc/containerd/config.toml`

Set this content:
```
version = 2
[plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
            runtime_type = "io.containerd.runc.v2"
            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
                SystemdCgroup = true
```

#### Restart containerd:
`$ sudo systemctl restart containerd`

#### Install NFS Server:
`$ sudo apt install nfs-kernel-server`

`$ sudo systemctl start nfs-kernel-server.service`

`$ sudo mkdir -p /mnt/data`

`$ echo "/mnt/data *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports`

`$ sudo exportfs -a`

The setup is complete.

---
>Run on the **Master Node**

#### Initialize Cluster
`$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16`

#### Give Kubernetes access to a specific user
`$ mkdir -p $HOME/.kube`

`$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`

`$ sudo chown $(id -u):$(id -g) $HOME/.kube/config`

Get the respone from `kubeadm init` command and save the `kubeadm join` command.

---
>Run an all **Worker Nodes**

#### Join the Node to the Cluster
```
$ sudo kubeadm join {master_node_ip}:6443 --token ********** \
        --discovery-token-ca-cert-hash sha256:**********
```
---
>Run on the **Master Node**

#### Apply Calico Kube Controller
`$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml`

#### Verify installation:
`$ kubectl get nodes`
The node status should be `Ready`