K8 SETUP on AWS

Master Node
- EC2 instance medium Amazon linux 2 OS
sudo yum update -y
sudo amazon-linux-extras install docker -y
sudo service docker start
systemctl enable docker.service
docker info

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
lsmod | grep br_netfilter
python -c 'import yum, pprint; yb = yum.YumBase(); pprint.pprint(yb.conf.yumvar, width=1)'
Get basearch variable - x86_64

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
Or yum install --nogpg -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
systemctl daemon-reload
systemctl restart kubelet

Run these only on MASTER
kubeadm init --pod-network-cidr=192.168.0.0/16
-Or- for flannel
kubeadm init --pod-network-cidr=10.244.0.0/16

CAPTURE JOIN TOKEN FOR WORKER NODES

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
-or- for flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

kubectl get pods --all-namespaces
kubectl taint nodes --all node-role.kubernetes.io/master-
kubeadm token list
kubeadm token create
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

