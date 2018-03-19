## Kubernetes Introduction and K8s Admin exam practices

### Kubeadm Installtion

More infor https://kubernetes.io/docs/setup/independent/install-kubeadm/


#### Install Docker from Ubuntu’s repositories:

    apt-get update
    apt-get install -y docker.io


#### or install Docker CE 17.03 from Docker’s repositories for Ubuntu or Debian:

    apt-get update
    apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository \
       "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) \
       stable"
    apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')


#### Installing kubeadm, kubelet and kubectl

You will install these packages on all of your machines:

    kubeadm: the command to bootstrap the cluster.
    kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
    kubectl: the command line util to talk to your cluster.

Eg: ubuntu Installtion

    apt-get update && apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    apt-get update
    apt-get install -y kubelet kubeadm kubectl

##### Configure cgroup driver used by kubelet on Master Node

Make sure that the cgroup driver used by kubelet is the same as the one used by Docker. Verify that your Docker cgroup driver matches the kubelet config:

    docker info | grep -i cgroup
    cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

If the Docker cgroup driver and the kubelet config don’t match, change the kubelet config to match the Docker cgroup driver. The flag you need to change is --cgroup-driver. If it’s already set, you can update like so:

    sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Otherwise, you will need to open the systemd file and add the flag to an existing environment line.
Then restart kubelet:

    systemctl daemon-reload
    systemctl restart kubelet

### Setup kube Master

    kubeadm init --pod-network-cidr=10.244.0.0/16

Note if you get swap working then you can off your swap (swapoff -a command)

Setup user account

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

setup pod networking communicator fannel

    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
    kubectl get pods
    kubectl get pods --all-namespace

Adding nodes

    kubeadm join --token 388ac2.59769db11b349455 192.168.1.150:6443 --discovery-token-ca-cert-hash sha256:a2a17bdfb29de8c986406c362ca5513b0a94883238ce25c80a1d29ea3a66e70e

From kube Master

    kubectl get nodes

Setup  kubectl autocompletion

    kubectl completion -h
    echo "source <(kubectl completion bash)" >> ~/.bashrc

## k8s Achitechure

![k8s_architecture](https://github.com/Kasunmadura/k8s/blob/master/images/k8s_architecture.png)

#### kube-apiserver :

Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane. .answer all api calls (it use key value storage call etcd)

#### kube-scheduler :

determind which nodes reponsible for pods

#### cloud container manager:

cloud-controller-manager runs controllers that interact with the underlying cloud providers.
The following controllers have cloud provider dependencies:

##### Node Controller:
For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding

##### Route Controller:
For setting up routes in the underlying cloud infrastructure

##### Service Controller:
For creating, updating and deleting cloud provider load balancers

##### Volume Controller:
For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes

#### kube-conroller-manager:

These controllers include in kube controller -

##### Node Controller:
Responsible for noticing and responding when nodes go down.

##### Replication Controller:
Responsible for maintaining the correct number of pods for every replication controller object in the system.

##### Endpoints Controller:
Populates the Endpoints object (that is, joins Services & Pods).

##### Service Account & Token Controllers:
 Create default accounts and API access tokens for new namespaces.

### Node Component

#### kubelet

An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.

#### kube-proxy
kube-proxy enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding.


#### Container Runtime
The container runtime is the software that is responsible for running containers. Kubernetes supports two runtimes: Docker and rkt.
