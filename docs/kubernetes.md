# K8S Set Up

## Install the required packages in all the nodes

```sh
#Setup
#   1. 3 VMs Ubuntu 16.04.5 or 18.04.1.0, 1 master, 2 nodes.
#   2. Static IPs on individual VMs
#   3. /etc/hosts hosts file includes name to IP mappings for VMs
#   4. Swap is disabled
#   5. Take snapshots prior to installations, this way you can install 
#       and revert to snapshot if needed

#Disable swap, swapoff then edit your fstab removing any entry for swap partitions
#You can recover the space with fdisk. You may want to reboot to ensure your config is ok. 
swapoff -a
vi /etc/fstab

#Add Google's apt repository gpg key
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

#Add the Kubernetes apt repository
sudo bash -c 'cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF'

#Update the package list and use apt-cache to inspect versions available in the repository
sudo apt-get update

sudo apt-get install -y docker.io kubelet kubeadm kubectl
sudo apt-mark hold docker.io kubelet kubeadm kubectl

#Ensure both are set to start when the system starts up.
sudo systemctl enable kubelet.service
sudo systemctl enable docker.service

# Setup Docker daemon. This a change that has been added since the recording of the course.
sudo bash -c 'cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF'

#Restart reload the systemd config and docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Create the Master

```sh
#Only on the master, download the yaml files for the pod network.
#The calico yaml file has changed since the publication of the course and is now avaialble at the URL below.
wget https://docs.projectcalico.org/manifests/calico.yaml


#Look inside calico.yaml and find the network range CALICO_IPV4POOL_CIDR, adjust if needed.
vi calico.yaml

#Create our kubernetes cluster, specifying a pod network range matching that in calico.yaml!
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

#Configure our account on the master to have admin access to the API server from a non-privileged account.
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#Deploy yaml file for your pod network.
#This line of code has be updated since the publication of the course. 
kubectl apply -f calico.yaml
```

## Join the Nodes

```sh
#Disable swap, swapoff then edit your fstab removing any entry for swap partitions
#You can recover the space with fdisk. You may want to reboot to ensure your config is ok. 
swapoff -a
vi /etc/fstab

#Add the Google's apt repository gpg key
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

#Add the kuberentes apt repository
sudo bash -c 'cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF'

#Update the package list
sudo apt-get update

#Install the required packages, if needed we can request a specific version
sudo apt-get install -y docker.io kubelet kubeadm kubectl
sudo apt-mark hold docker.io kubelet kubeadm kubectl


#Ensure both are set to start when the system starts up.
sudo systemctl enable kubelet.service
sudo systemctl enable docker.service

#Setup Docker daemon. This is a change that has been added since the recording of the course.
sudo bash -c 'cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF'

sudo systemctl daemon-reload
sudo systemctl restart docker

#If you didn't keep the output, on the master, you can get the token.
kubeadm token list

#If you need to generate a new token, perhaps the old one timed out/expired.
kubeadm token create

#On the master, you can find the ca cert hash.
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

#Using the master (API Server) IP address or name, the token and the cert has, let's join this Node to our cluster.
sudo kubeadm join 172.16.94.10:6443 \
    --token 9woi9e.gmuuxnbzd8anltdg \
    --discovery-token-ca-cert-hash sha256:f9cb1e56fecaf9989b5e882f54bb4a27d56e1e92ef9d56ef19a6634b507d76a9

#Back on master, this will say NotReady until the networking pod is created on the new node. Has to schedule the pod, then pull the container.
kubectl get nodes

```