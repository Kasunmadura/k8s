# Kubernetes Introdution and K8s Exam practices

## Kubeadm Installtion

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
