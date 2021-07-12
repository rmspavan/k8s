# k8s

Ansible


hostnamectl set-hostname kubernetes-master
poweroff
clear
sudo dnf makecache
sudo dnf install epel-release
sudo dnf makecache
sudo dnf install ansible

K8s-using playbook


cat /etc/hosts
192.168.2.1 kubernetes-master.learnitguide.net kubernetes-master
192.168.2.2 kubernetes-worker1.learnitguide.net kubernetes-worker1
192.168.2.3 kubernetes-worker2.learnitguide.net kubernetes-worker2


ansible-playbook settingup_kubernetes_cluster.yml

ansible-playbook join_kubernetes_workers_nodes.yml
Make your servers ready (one master node and multiple worker nodes).

Make an entry of your each hosts in /etc/hosts file for name resolution.

Make sure kubernetes master node and other worker nodes are reachable between each other.

Internet connection must be enabled in all nodes, required packages will be downloaded from kubernetes official yum repository.

Clone this repository into your master node.

git clone https://github.com/learnitguide/kubernetes-and-ansible.git

once it is cloned, get into the directory

cd kubernetes-and-ansible/centos

There is a file "hosts" available in "centos" directory, Just make your entries of your all kubernetes nodes.

Provide your server details in "env_variables" available in "centos" directory.

Deploy the ssh key from master node to other nodes for password less authentication.

ssh-keygen

Copy the public key to all nodes including your master node and make sure you are able to login into any nodes without password.

Run "settingup_kubernetes_cluster.yml" playbook to setup all nodes and kubernetes master configuration.

ansible-playbook settingup_kubernetes_cluster.yml

Run "join_kubernetes_workers_nodes.yml" playbook to join the worker nodes with kubernetes master node once "settingup_kubernetes_cluster.yml" playbook tasks are completed.

ansible-playbook join_kubernetes_workers_nodes.yml

Verify the configuration from master node.

kubectl get nodes



K8s-manually:


hostnamectl set-hostname worker-node-1
vi /etc/hosts
ping master-node
sudo yum install -y yum-utils
ip addr
poweroff
yum install lsof git curl wget net-tools -y
hostnamectl set-hostname kube-master
vi /etc/fstab
swapoff -a
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=10250/tcp
systemctl status firewalld
sudo firewall-cmd --reload


cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
EOF
sysctl --system


cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo 
[kubernetes] 
name=Kubernetes 
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch 
enabled=1 
gpgcheck=1 
repo_gpgcheck=1 
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg 
exclude=kubelet kubeadm kubectl 
EOF


setenforce 0 

sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sestatus

yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
yum update -y && yum install containerd.io docker-ce docker-ce-cli


mkdir /etc/docker 
cat > /etc/docker/daemon.json <<EOF 
{ 
  "exec-opts": ["native.cgroupdriver=systemd"], 
  "log-driver": "json-file", 
  "log-opts": { 
    "max-size": "100m" 
  }, 
  "storage-driver": "overlay2", 
  "storage-opts": [ 
    "overlay2.override_kernel_check=true" 
  ] 
} 
EOF


mkdir -p /etc/systemd/system/docker.service.d 
systemctl daemon-reload 
systemctl restart docker 
systemctl enable docker


yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes 
systemctl enable --now kubelet

cat /etc/sysconfig/network-scripts/ifcfg-enp0s3


kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=192.168.0.120

watch docker images



kubeadm join 192.168.0.123:6443 --token st94e6.61c5env8jwg123n9 \\", "\t--discovery-token-ca-cert-hash sha256:88e1595ece46440e70efc89aaad2ab42490f741f1a4d68b6d624f667357fd9cc
kubeadm join 192.168.0.123:6443 --token st94e6.61c5env8jwg123n9 --discovery-token-ca-cert-hash sha256:88e1595ece46440e70efc89aaad2ab42490f741f1a4d68b6d624f667357fd9cc





kubeadm join 192.168.0.120:6443 --token 77h1ak.dkb8iu8lvwb3btrg --discovery-token-ca-cert-hash sha256:fe300860ee55a41082aa068d874d876dcb3e596318d604baf898b16ce9ef7c92


mkdir -p $HOME/ .kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

## to create network
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


yum install bash-completion 
echo "source <(kubectl completion bash)" >> ~/.bashrc

Add Role

kubectl label node <node name> node-role.kubernetes.io/<role name>=<key - (any name)>

kubectl label node kubernetes-worker1 node-role.kubernetes.io/worker1=worker1
  
  kubectl get node --show-labels
  
kubectl get pod -owide all-namespaces
  
kubectl get node -owide

  Kubectl token create --print-join-command

 
