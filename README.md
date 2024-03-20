# Kubernetes
### **Single Cluster Kubernetes  Install on Ubuntu** 

**Case 1] For Installation of k8s  on Ubuntu Version-- 22.04.4 LTS   you can run Script .**
            
          Step 1 :
 
          Step 2 : sh script.sh
            (It will run and install complete Kubernetes Cluster the steps which are given below ) 
          
          Step 3 :  Run this Command and check the node is ready or not 
                      kubectl get nodes

          Step 4 (optional ) If you pods are in Pending State run this command (For Troubleshooting )
                          kubectl describe pod  <name-pod>
                  
                 If  this error comes
                 (Warning  FailedScheduling  101s  default-scheduler  0/1 nodes are available: 1 node(s) had 
                 untolerated taint 
                 {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not 
                 helpful for scheduling.)
                     
                 Run this command 
                 >kubectl get nodes

                 >kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane:NoSchedule-
                          




                                                  **OR**

**Case 2**
**Complete Steps For Installation** 

Step 1: Update and Upgrade Ubuntu (all nodes)
Begin by ensuring that your system is up to date. Open a terminal and execute the following commands:

sudo apt update
sudo apt upgrade

Step 2: Disable Swap (all nodes)
To enhance Kubernetes performance, disable swap and set essential kernel parameters. Run the following commands on all nodes to disable all swaps:

sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

Step 3: Add Kernel Parameters (all nodes)
Load the required kernel modules on all nodes:

sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

Configure the critical kernel parameters for Kubernetes using the following:

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

Then, reload the changes:

sudo sysctl --system

Step 4: Install Containerd Runtime (all nodes)
We are using the containerd runtime. Install containerd and its dependencies with the following commands:



sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

Enable the Docker repository:

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

Update the package list and install containerd:

sudo apt update
sudo apt install -y containerd.io

Configure containerd to start using systemd as cgroup:

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

Restart and enable the containerd service:

sudo systemctl restart containerd
sudo systemctl enable containerd

Step 5 Install Kubectl, Kubeadm, and Kubelet (all nodes)

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

#If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
#sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


sudo kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


kubectl get nodes

Step :6 Install Kubernetes Network Plugin (master node)
To enable communication between pods in the cluster, you need a network plugin. Install the Calico network plugin with the following command from the master node:

curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml -O

kubectl apply -f calico.yaml

Step 7: Verify the cluster and test (master node)
Finally, we want to verify whether our cluster is successfully created.

kubectl get pods -n kube-system
kubectl get nodes


optional Step 8: If error occur like this.

(Warning  FailedScheduling  101s  default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.)

kubectl get nodes

kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane:NoSchedule-
