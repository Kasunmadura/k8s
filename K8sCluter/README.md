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



### Cluster Communication

1. Everything going with API server.and TLS encryption
2. Most of the installtion methods will handle the certificate creation.
3. once authenticated ,every API call should pass an authorization check.

#### Role base Access Control

1. Kubernetes has an inegrated Role based access Control component
2. Centain roles perform specific actions in Cluster
3. kubernetes has several well thought out. pre-created roles
4. Simple roles might be fine for smaller Cluster
5. if a user doesn't have rights to perform an action but they do have access to perform a composite action that includes it, the user WILL be able to indirect create objects.
6. Carefully consider what you want users to be allowed to do prior to making changes to the existing roles.


#### Securing the Kubectl

1. secure the kubelet on each node.
2. The Kubelets expose HTTPS enpoints which give access to both data nd actions on the nodes.By Degault,these are open.
3. To secure those endpoints, you can enable Kubelet Authntication and Authorization by starting it with an --anonymous-auth=false flag and assinging it an apporpriate  x509 client certificate in its configuration.

#### Securing the Network

1. Network Polices restrict access to the  network for a particular namespace.
2. This allows deverlopers to restric which pods in other namespaces can access pods and ports within the current namespaces.
3. The pod networking CNI must respect these policies which,fortunaely,most of them do.
4. Users can also be assinged quotas or limit ranges.
5. Use plug-Ins for more advanced functionality.
6. That should secure all the communications in a cluster.

#### Vulnerabilites

1. Kubernetes makes extensive use of etcd for storing configuration and secrets. it acts as the key/value store for the entire cluster.
2. Gaining write access to etcd is very much like gaining root on the whole cluter, an even read access can be used by attackers to cause  some seriouse damage.
3. Strong credeentials on your etc server  or cluster is a must.
4. Isolate those behind a firewall that only allows requests from the API servers.
5. Audit logging is also critical
6. Record actions taken by the API for later analiysis in the event
7. Enable audit logging and archive the audit file on a secure server.
8. Rotate your infrastructure credentials frequently.
9. Smaller lifetime windows for secrets and credentials create bigger problems for attackers attemting to use it.
10. You can even set these up  to have short lifetimes and automate their rotation.(5 min)

#### Third Party integrations

1. Always revire 3rd party integrations before enabling them.
2. Integrations to kubernetes can change how secure your cluster is.
3. Add-ons might be noting more that just more pods in the cluster. but those can be powerfull.
4. Don't allow them into the kube-system namespace.


### High Avaliability

1. Create the reliable nodes that will form our cluster.
2. Set up a redundant and reliable storage service with a multinode deployment of etcd.
3. Start replicated and load balanced k8s API server.
4. Set up a master-elected k8s scheduler and controller manager daemons.


#### Step one

    a. Make the Master node reliable
    b. Ensure that the services automatically restart if they fail.
    c. Kubelet already does this. so it's convenient piece to use.
    d. if the kubelet goes down, though , we need something to restart it.
    e. Monit on Debian systems or systemctl on systemd-based systems.

#### Step two

    a. Clustered etcd already replicates the storage to all master instance in your cluster.
    b. to lose data. all 3 nodes would need to have their disks fail at the same time.
    c. the probability that this occurs is relatively low. so just running a replicated etcd cluster is reliable enough.
    e. Add addtional reliablity by increasing the size of the cluster from three to five nodes.
    f. a multinode cluster of etcd.
    g. if you use a cloud provider, then they usually provide this for you
        example: persistent disk on the Google cloud platform
    h. if you are running on pysical machines,you can also use network attached redundant storage using an iSCSI or NFS interface.
    i. RAID array on each physical machine

#### Step Three - Replicated API Service

    a. Create the initial log file so  that Docker will mount a file instend of a directory:

        touch /var/log/kube-apiserver.log

    b. Next, we create a /srv/kubernets/ directory on each node which should include:

        basic_auth.csv - basic auth user and password
        ca.crt - Certificate Authority
        known_tokens.csv - tokens that entities can use to talk to the apiserver
        kubecfg.crt - Client Certificate,public key
        kubecfg.key - Client Certificate, private key
        server.crt - Server Certificate,public key
        server.key - Server Certificate,private key

    c. Either create this manully or copy it from a master node on a working cluster.
    d. once these files exist.copy the kube-apiserver.yaml into /etc/kubernetes/manifests/ on each of our master nodes.
    e. The kubelet monitors this directory, and will automatically create an instance of the kube-apiserver container using the pod definition specified in the  file.
    f. if a network load balancer is setup.then access the cluster using VIP and see traffic balancing between the apiserver instances.
    g. Setting up a aload balancer will depend on the specifics of your platform.
    h. For external uses of the API(eg. the kubectl command line interface,continouse build pipelines or other clients) remember to configure them to talk to the external load balancer's IP address.

#### Step Four Controller/Scheduler Daemons

    a. Now we need to allow our state to change
    b. Controller managers and scheduler.
    c. These processes must not modify the cluster's state simultaneously, use a lease-lock.
    d. Each scheduler and controller manager can be launched with a --leader-elect flag
    e. The scheduler and conttoller-manager can be configured to talk to the API server that is on the same node (i.e 127.0.0.1)
    f. it  can also  be configured to communicate using the load balanced IP address of the API servers.
    g. The scheduler and controller-manager will complete the leader election process mentioned before when using the --lead-elect flag.
    h. In case of failure accessing the API server.the elected leader will not be able to renew the lease,causing a new leader to be elected.
    i. This is especially relavant when configuring the scheduler and controller-manager to access  the API server via 127.0.0.1 and the API server on the same node is unavailable.

#### Installing Configuration Files

    a.Create empty log files on each node, so that Docker will mount the files and not make new directories:
        touch /var/log/kube-scheduler.log
        touch /var/log/kube-controller-manager.log
    b.Set up the describtions of the scheduler and controller manager pods on each node by copying kube-scheduler.yaml and kube-controller-manager.yaml into the /etc/kubernetes/manifests directory

    c.And once that's all done, we've now made our cluster highly available Remember,if a worker node goes down. kubernetes will rapidlu detect that and spin up replacement pods elsewhere in the cluser.


#### End-to-End Testing and Validation

  1. Provide a mechanism to test end-to-end behavaior of the system.
  2. Last signal to ensure  end user operations match specifications.
  3. Primarily a developer tool.
  4. Difficult to run against just "any" deployment -- many sepcific test for cloud providers
      a. Ubuntu has its own Juju-deployed tests
      b. GCE has its own
      c. Many tests offered

#### Kubetest Suite

  1. Ideal for GCE or AWS users
  2. Build
  3. storage
  4. Extract
  5. Brung up the cluster
  6. Test
  7. Dump logs
  8. Tear down


#### Validating Nodes and Cluster

  1. Master
   
    kubectl get nodes
    kubectl describe node master-1

  2. nodes

    ps -aux | grep kube
