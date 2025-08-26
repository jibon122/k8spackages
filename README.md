<h1> Single Node Cluster in Kubernetes</h1>
✅ To Make the server IP Static:
<p># ifconfig 				-> to check your DHCP IP
To check the config file under the directory location: 
[root@localhost ~]# ls /etc/sysconfig/network-scripts/ifcfg-ens33 
/etc/sysconfig/network-scripts/ifcfg-ens33</p>
<hr>
✅  Take a backup edit and save: <p>
[root@localhost ~]# ip route | grep default
default via 192.168.140.2 dev ens33 proto dhcp metric 100
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
Add as Below:
IPADDR=192.168.60.51 		->Change as your server
GATEWAY=192.168.140.2		->Change as your server
DNS1=192.168.140.2 		->Change as your server
DNS2=8.8.8.8				->Google DNS
[root@localhost ~]# systemctl restart network
  <hr>
✅ Change hostname:
<p></p>[root@localhost ~]# hostnamectl set-hostname k8smaster.example.com
[root@localhost ~]# bash
[root@k8smaster ~]# 
[root@k8smaster ~]# echo "192.168.60.51 k8smaster.example.com" >> /etc/hosts  	->host file entry.</p>
  <hr>
<p>
✅Firewall Disable and OFF:
[root@k8smaster ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@k8smaster ~]# systemctl stop firewalld
</p>
✅Partition OFF/Disable: 
[root@k8s ~]# swapoff -a; sed -i '/swap/d' /etc/fstab

[root@k8smaster ~]# sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
[root@k8smaster ~]# gatenforce
bash: gatenforce: command not found
[root@k8smaster ~]# 

[root@k8smaster ~]# vi /etc/sysctl.d/kubernetes.conf
Add given line:
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1

[root@k8smaster ~]# systemctl --system
 <hr>
✅ Explanation of each package:
[root@k8smaster ~]# yum install -y yum-utils device-mapper-persistent-data lvm2 
 <hr>
✅ Enable the Docker CE repository

[root@k8smaster ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 <hr>
✅ TO Check Packages: 
[root@k8smaster ~]# cat /etc/yum.repos.d/docker-ce.repo
 <hr>
✅ Install Docker:  
[root@k8smaster ~]# yum install -y docker-ce-19.03.12
 <hr>
✅ Start / Enable/ Status Docker:  
[root@k8smaster ~]# systemctl start docker
[root@k8smaster ~]# systemctl enable docker
[root@k8smaster ~]# systemctl status docker
[root@k8smaster ~]# docker version
 <hr>
✅ Install git for Kubernetes packages: 
[root@k8smaster ~]# yum install -y git
[root@k8smaster ~]# git clone https://github.com/jibon122/k8spackages.git 
 <hr>
✅ git packages: 

✅ Install packages: 
[root@k8smaster k8spackages]# yum install -y cri-tools-1.13.0-0.x86_64.rpm kubeadm-1.18.5-0.x86_64.rpm kubectl-1.18.5-0.x86_64.rpm kubelet-1.18.5-0.x86_64.rpm kubernetes-cni-0.8.6-0.x86_64.rpm

[root@k8smaster k8spackages]# systemctl enable kubelet

[root@k8smaster k8spackages]# systemctl status kubelet 

✅Kubernetes Cluster initialization: 
[root@k8smaster k8spackages]# kubeadm init --apiserver-advertise-address=192.168.60.51 --pod-network-cidr=192.168.0.0/16
 <hr>
✅Check Images creation status: 
[root@k8smaster ~]# watch "docker images"
 <hr>
✅ To start using your cluster, you need to run the following as a regular user: -> from terminal:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
✅Kubernetes status:
[root@k8smaster ~]# kubectl get nodes
[root@k8s k8spackages]# kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml


[root@k8smaster ~]# kubectl get nodes

[root@k8smaster ~]# watch  "kubectl get all -A"

****create pods ****
[root@k8smaster ~]# vi pod1.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    env: prod
spec:
  containers:
    - name: myfirstcontainer
      image: nginx:1.23

[root@localhost ~]# kubectl apply -f pod1.yaml
[root@localhost ~]# kubectl get pods 	-> it shows pod on pending status.

✅ Enable master as a worker: 
kubectl  describe node k8smaster.example.com 
Find Taints -> 
[root@localhost ~]#  kubectl taint nodes k8smaster.example.com node-role.kubernetes.io/master:NoSchedule-

!! Enjoy Single Node Cluster in Kubernetes !!
