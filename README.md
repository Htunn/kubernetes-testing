# kubernetes-testing
# Multi-node Cluster
# Prerequisites

Ubuntu Server ISO 18.04 LTS for local VM or Public Cloud Deployment

minimum 8 GB RAM

2 GB for each VM

Disable swap 
   > swapoff -a (or) comment swap partition on /etc/fstab

# System Topology
192.168.2.2 k8s-master 

192.168.2.3 k8s-node-01 

192.168.2.4 k8s-node-02

192.168.2.5 k8s-node-03

# Setup Kubernetes on Ubuntu 18.04

# Update system packages
sudo apt-get update

sudo apt-get upgrade

sudo apt-get install linux-image-extra-virtual

sudo reboot

# add user to manage k8s cluster
sudo useradd -s /bin/bash -m k8s-admin

sudo passwd k8s-admin

sudo usermod -aG sudo k8s-admin

echo "k8s-admin ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/k8s-admin

# Install Docker or other container runtimes 
# Install dependencies

sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common

# Import Docker repository GPG key:

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs)  stable"

# Install docker:

sudo apt-get update

sudo apt-get install docker-ce

sudo usermod -aG docker k8s-admin


# Install and Configure Kubernetes Master

# On Master node only
# Add Kubernetes repository

cat << EOF > /etc/apt/sources.list.d/kubernetes.list

deb http://apt.kubernetes.io/ kubernetes-xenial main

EOF
  
# Import GPG Key

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Install Kubernetes Master Components

sudo apt update

sudo apt install kubectl kubelet kubeadm kubernetes-cni

# Initialize K8s Cluster
# Export env variables (optional) to ~/.bashrc or ~/.zshrc

export API_ADDR=`ifconfig eth0 | grep 'inet'| cut -d':' -f2 | awk '{print $2}'`

export DNS_DOMAIN="k8s.local"

export POD_NET="10.4.0.0/16"

export SRV_NET="10.5.0.0/16"

# Initialize k8s Cluster

kubeadm init --pod-network-cidr ${POD_NET} --service-cidr ${SRV_NET} --service-dns-domain "${DNS_DOMAIN}" --apiserver-advertise-address ${API_ADDR}

# copy token for joing k8s cluster
Example:

kubeadm join 192.168.2.2:6443 --token 9y4vc8.h7jdjle1xdovrd0z --discovery-token-ca-cert-hash \
sha256:cff9d1444a56b24b4a8839ff3330ab7177065c90753ef3e4e614566695db273c

# Configure Access for k8s-admin user on the Master server

su - k8s-admin

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=$HOME/.kube/config

echo "export KUBECONFIG=$HOME/.kube/config" | tee -a ~/.bashrc

# Deploy Weave Net POD Network to the Cluster ( Run as normal user)

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# check weave net pod

kubectl get pod -n kube-system | grep 'weav'


# Setup Kubernetes Worker Nodes

1. Ensure docker is installed ( refer above step )
2. add k8s repo ( refer above step )
3. Install kubernetes components ( refer above step )
4. join the node to the cluster

Example:

kubeadm join 192.168.2.2:6443 --token 9y4vc8.h7jdjle1xdovrd0z \
 --discovery-token-ca-cert-hash sha256:cff9d1444a56b24b4a8839ff3330ab7177065c90753ef3e4e614566695db273c
 
 # Check nodes status on the master
 
 kubectl get nodes
 
 # check weave net 
 
 ip add | grep 'weav'
 

# Test Kubernetes Deployment

# create test pod to confirm that our cluster

# Create a test namespace:

kubectl create namespace test-namespace

# create a pod using deployment

kubectl create -n test-namespace -f http-app-deployment.yml

kubectl -n test-namespace get deployments


# create a service which exposes the Pods on a particular port

kubectl -n test-namespace create -f http-app-service.yml

kubectl -n test-namespace get svc


# k8s shell completion 

# for bash

echo "source <(kubectl completion bash)" >> ~/.bashrc

# for zsh

if [ $commands[kubectl] ]; then

source <(kubectl completion zsh)

fi





 



