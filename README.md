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
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10251/tcp
firewall-cmd --permanent --add-port=10252/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --reload
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

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

  
AWX Install on K8S:
  
install the helm to manage to create the database pod:
  $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
  
Create Separate Cluster Context for AWX:
  $ mkdir -p /home/ansible/.kube/custom-contexts/awx-config/
$ cd /home/ansible/.kube/custom-contexts/awx-config/
  
Creating Certificates:
  $ openssl genrsa -out ansible.key 4096
  $ openssl req -new -newkey rsa:4096 -nodes -keyout ansible.key -out ansible.csr -subj "/CN=ansible/O=system:authenticated"
  $ sudo openssl x509 -req -in ansible.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ansible.crt -days 365
  to verify
  $ openssl x509  -noout -text -in /home/ansible/.kube/custom-contexts/awx-config/ansible.crt
  
  
Create Namespace for AWX
  # kubectl create namespace awx
  cat > awx_cluster_config.yml
---
apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  name: awxcls

contexts:
- context:
  name: awx-contx

users:
- name: root

 
Add context details to the configuration file.

KUBECONFIG=awx_cluster_config.yml kubectl config set-cluster awxcls --server=https://192.168.0.115:6443 --client-certificate=/home/ansible/.kube/custom-contexts/awx-config/ansible.crt --client-key=/home/ansible/.kube/custom-contexts/awx-config/ansible.key

KUBECONFIG=awx_cluster_config.yml kubectl config set-credentials ansible --client-certificate=/home/ansible/.kube/custom-contexts/awx-config/ansible.crt --client-key=/home/ansible/.kube/custom-contexts/awx-config/ansible.key

KUBECONFIG=awx_cluster_config.yml kubectl config set-context awx-contx --cluster awxcls --namespace=awx --user ansible

  
  
Now the custom config file will looks like below. (certificate-authority-data: –> This value can be get from exiting config $ sudo cat /etc/kubernetes/admin.conf file)

# vi ~/.kube/custom-contexts/awx-config/awx_cluster_config.yml
  apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc 
    server: https://192.168.0.42:6443
  name: awxcls
contexts:
- context:
    cluster: awxcls
    namespace: awx
    user: ansible
  name: awx-contx
current-context: ""
kind: Config
preferences: {}
users:
- name: ansible
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUR6VENDQXJVQ0ZCNWJOTmFmU1c4Um9RblpxQlVreGNDZ25sTy9NQTBHQ1NxR=
    client-key-data: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpRd0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1Mwd2dna3BBZ0VBQW9o=
  
Custom Context Env:
  # vim ~/.bashrc
  # Set the default Kube context if present
DEFAULT_KUBE_CONTEXTS="$HOME/.kube/config"
if test -f "${DEFAULT_KUBE_CONTEXTS}"
then
  export KUBECONFIG="$DEFAULT_KUBE_CONTEXTS"
fi
# Additional contexts should be in ~/.kube/custom-contexts/ 
CUSTOM_KUBE_CONTEXTS="$HOME/.kube/custom-contexts"
mkdir -p "${CUSTOM_KUBE_CONTEXTS}"
OIFS="$IFS"
IFS=$'\n'
for contextFile in `find "${CUSTOM_KUBE_CONTEXTS}" -type f -name "*.yml"`  
do
    export KUBECONFIG="$contextFile:$KUBECONFIG"
done
IFS="$OIFS"
  
$ source ~/.bashrc (Source it activate the multi context switching)
  
 $ kubectl config get-contexts (to check avalible contexts)

define a role under the awx namespace:
  
<!--$ kubectl create role ansible-role --namespace=awx --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods -o yaml --dry-run=client  > role.yaml -->
  
vi role.yml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ansible-role
  namespace: awx
rules:
- apiGroups:
  - ""
  - extensions
  - apps
  resources:
  - "*"
  verbs:
  - create
  - get
  - list
  - update
  - delete
  - patch
...

$ kubectl create -f role.yaml
$ kubectl describe role -n awx ansible-role
  
Create RoleBinding:
$ kubectl create rolebinding ansible-rolebinding --role=ansible-role --user=ansible -n awx
$ kubectl describe rolebindings.rbac.authorization.k8s.io -n awx ansible-rolebinding

########Creating PersistentVolume:
We are going to use the NFS as our storage solution, install the NFS client package and list the exported share from the NFS server. If you have installed as part of resolve dependencies, ignore. In case, If you are looking to set up an NFS server look at this How to Setup NFS Server on CentOS 8.

The share we are about to create in NFS server should have below NFS options, replace with your UID and GID and it should match on all the k8s nodes.

/k8sdata *(rw,all_squash,anonuid=1001,anongid=1001)
List the exported NFS shares.

ansible@awx1:~$ showmount -e 192.168.0.155
Export list for 192.168.0.55:
/nfspub   *
/k8sdata  (everyone)
/nfslimit 192.168.0.[42-44]/24
/nfsshare (everyone)
ansible@awx1:~$
As part of AWX installer, the PersistentVolumeClaim will be created, However, it will be in a pending state if we don’t have a persistent Volume, So let’s create one.

$ cat > persistentVolume_k8sdata.yaml
Create a file and append the YAML content.

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: k8sdata
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: dbstorageclass
  mountOptions:
    - hard
    - nfsvers=4.2
  nfs:
    path: /k8sdata
    server: 192.168.0.55
...
Let’s create the PV

$ kubectl create -f persistentVolume_k8sdata.yaml
Right after creating it, let’s verify.

ansible@awx1:~$ kubectl get persistentvolumes 
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS     REASON   AGE
k8sdata   10Gi       RWO            Retain           Available           dbstorageclass            12s
ansible@awx1:~$
Creating PersistentVolumeClaim
Create a persistent volume claim for PostgreSQL database.

$ cat > postgrespvc.yaml

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgrespvc
  namespace: awx
spec:
  storageClassName: dbstorageclass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
...
We are good to move forward.

ansible@awx1:~$ kubectl get pvc
NAME          STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS     AGE
postgrespvc   Bound    k8sdata   10Gi       RWO            dbstorageclass   5s
#####################################
  
  
Switching to AWX Context:
$ kubectl config get-contexts
$ kubectl config use-context awx-contx
$ kubectl config current-context
  
$ kubectl config use-context awx-contx
Switched to context "awx-contx".

$ kubectl config current-context 
awx-contx

Downloading AWX:
  $ wget https://github.com/ansible/awx/archive/*.*.*.zip or git clone -b "*.*.*" https://github.com/ansible/awx.git
$ unzip 14.1.0.zip
$ cd awx/

  $vi installer/inventory
  localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python3"

[all:vars]

dockerhub_base=ansible

# Kubernetes Install
kubernetes_context=awx-contx
kubernetes_namespace=awx
kubernetes_web_svc_type=NodePort
#Optional Kubernetes Variables
pg_image_registry=docker.io
pg_serviceaccount=awx
pg_volume_capacity=5
pg_persistence_storageClass=dbstorageclass
pg_persistence_existingclaim=postgrespvc
pg_cpu_limit=1000
pg_mem_limit=2

postgres_data_dir="~/.awx/pgdocker"

pg_username=awx
pg_password=awxpass
pg_database=awx
pg_port=5432

admin_user=admin
admin_password=password

create_preload_data=True
secret_key=awxsecret
  

Edit the role yaml to add/remove parameters.
 $ vim installer/roles/kubernetes/defaults/main.yml
  AWX is resource hungry and they are with the below default value.

web_mem_request: 1
web_cpu_request: 500
task_mem_request: 2
task_cpu_request: 1500
redis_mem_request: 2
redis_cpu_request: 500

Change them to 0 for all the containers or assign with less value than the default.

web_mem_request: 0
web_cpu_request: 0
task_mem_request: 0
task_cpu_request: 0
redis_mem_request: 0
redis_cpu_request: 0

  
Change from 60 seconds to 90 seconds.

postgress_activate_wait: 90
We have additionally added this parameter.

postgress_migrate_wait: 90
  

Added an additional task to wait for 90 seconds before starting with migrating the database:

$ vim installer/roles/kubernetes/tasks/main.yml
- name: Wait for management pod to start
  shell: |
  
  
$ ansible-playbook -i installer/inventory installer/install.yml
    {{ kubectl_or_oc }} -n {{ kubernetes_namespace }} \
      get pod ansible-tower-management -o jsonpath="{.status.phase}"
  register: result
  until: result.stdout == "Running"
  retries: 60
  delay: 10

  - name: Wait for two minutes before starting with migrate database
  pause:
    seconds: "{{ postgress_migrate_wait }}"
  when: openshift_pg_activate.changed or kubernetes_pg_activate.changed
  
  
  ansible-playbook -i installer/inventory installer/install.yml
