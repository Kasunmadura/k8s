Single node k8s

### Minikube

1. Minikube is the recommended method for creating a single node kubnernetes deployment on your local workstation.
2. Its installation is automated and doesn't require a cloud provider, and it works pretty well.


### Kubeadm

1. You can use Kubeadm to deploy multi-node locally if you like,but it's a little more challenging.
2. You will need to select your one CNI (Cluster Network Interface) if you go this route.
3. Fannel is good for this


### K8s hosted and Turnkey solutions

1. Google, AWS, Azure, Stackpoint, AppCode, KUBE2GO ,Platform 9, OpenShift Dedicated.. etc

### Add-on solutions

1. Venders also offer a wide variety of Add-on to k8s. One of the most important ones to consider is the CNI- Container Network Interface.
      a. Calico : secure L3 networking and network policy provide.
      b. Canel  : unites Fannel and Calico
      c. Cilium : L3 network policy plugin that can enforce HTTP/API/L7 policies transparently.Both routing and overlay/encapsulation mode are supported.
      e. CNI-Cenie :k8s to seamlessly connect to a choice of CNI plugin,such as Calico,Canal,Fannel, Romana or Weave


#### Contiv:

      provide configurable networking (L3 using BGP,overlay using vxlan, classic L2 and Cisco-SDN/ACI,).Fully open source.

#### Flanne and Mutus solutions

      overlay network  provide taht can use with k8s and Mutus a Multi plugin for multiple network support in K8s to support all CNI pluging (Calico,Cilium,Contiv,Fannel)

#### Nuage

      is an SDN platform that provides policy-based networking  k8s pods and non-k8s Env

      
