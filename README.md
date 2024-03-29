# k8s-test
The deployment files of this project were based on: https://gerrit.o-ran-sc.org/r/oam and https://gerrit.o-ran-sc.org/r/smo/o1.git.

# Evrioment instalation

First build the docker image:
- docker build -t cpqd-sdnr:latest .

Then create the k8s elements:
- kubectl create -f directory-name

Check the ip range for services and the storage class used in your cluster.

KUBEADM Intallation
--------------------- 
This installation was made on Ubuntu Focal reference: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


- sudo apt install apt-transport-https curl

- curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

- echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list

- sudo mv ~/kubernetes.list /etc/apt/sources.list.d

- sudo apt update

- sudo apt install kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00 docker.io

- sudo systemctl enable docker
-  sudo systemctl start docker
-  swapoff -a

- echo 'Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --fail-swap-on=false"' >>                     /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
      
- sudo kubeadm init --pod-network-cidr=192.168.0.0/16
-  mkdir -p $HOME/.kube
-  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
-  sudo chown $(id -u):$(id -g) $HOME/.kube/config
-  kubectl taint nodes ubuntu node-role.kubernetes.io/master:NoSchedule-

installing the cni
--------------------------
- kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Change the node port range
---------------------------
Reference: http://www.thinkcode.se/blog/2019/02/20/kubernetes-service-node-port-range

The file that has to be changed is /etc/kubernetes/manifests/kube-apiserver.yaml
- insert this line: - --service-node-port-range=00000-39767

After that the api will reset with the apropiate node port range.

NFS installation
---------------------------
First install helm:

- curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
- sudo apt-get install apt-transport-https --yes
- echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee         /etc/apt/sources.list.d/helm-stable-debian.list
- sudo apt-get update
- sudo apt-get install helm

Then install nfs on ubuntu:
- sudo apt update && sudo apt install nfs-common -y

Install nfs-ganesha-server-and-external-provisioner:
- kubectl create ns nfs
- helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
- helm install --version=1.5.0 nfs-provisioner nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner -n nfs --wait

Check if the envrioment is running (kubectl get all -n nfs), then reset the kublet service.
Adicionando e atualizando repositórios Helm charts do Calico, Aether e Incubator:
helm repo add projectcalico https://docs.projectcalico.org/charts;
helm repo add aether https://charts.aetherproject.org;
helm repo add incubator https://charts.helm.sh/incubator;
helm repo update;

Instalando Charts Calico (Foi utilizado a versão 3.25.0):
helm install calico projectcalico/tigera-operator --version v3.25.0
#helm install calico projectcalico/tigera-operator --version v3.23.3 (Obs: se essa versão ficar 3.25.0 ficar instável, utilizar a 3.23.3)

Download e instalação do multus:
cd $HOME
wget https://github.com/k8snetworkplumbingwg/multus-cni/archive/refs/tags/v3.7.1.zip
sudo apt install unzip
unzip v3.7.1.zip
cd multus-cni-3.7.1
sudo updatedb
kubectl apply -f $HOME/multus-cni-3.7.1/images/multus-daemonset.yml

