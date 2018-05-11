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


#### Monitoring



Heapster -- Cluster-wide aggregator of monitoring and event data.

1. Kubelet acts as a bridge between the Kubernetes master and the nodes.
2. Manages the pods and containers running on a node.
3. Translated each pod in to the containers making it up.
4. Obtains usage statictics from CAdvicor.
5. Exposes the aggregated pod resource usage statictics via REST API

and we can use  Grafana with InfluxDB

1. Heapster is setup to use this storage backend by default on most Kubernetes clusters
2. InfluxDB and grafana runs in pods.
3. The pod expose itself as Kubernetes service which is how Heapster then discocers it.
4. The Grafana container servers Grafana's UI which provide a dashboard
5. The default dashboard for Kubernetes contains an example dashboard that monitors resource usage of cluster and the pods inside of it. This dashboard can, of course, be fully customized and expanded.

Or Google cloud Monitoring also

1. Google cloud monitoring is hosted monitoring service that allows you to visualize and alert important metrics in your application.
2. Heapster can be set up to automatically push all collcted metrics to google cloud Monitoring.
3. These metices are then Avaliable in the google cloud monitoring console.
4. This storage backend is the easiest to setup and maintain


![Monitoring](https://github.com/Kasunmadura/k8s/blob/master/images/monitoring.png)


CAdvicer is open source container resource resource usage and performance analiysis agent.

    1. Auto discover all containers on a node and collects CPU, Memory , file system ..etc
    2. provide the overall machin usage by analyzing the 'root' container on the machine
    3. Exposes a simple UI for local containers on port 4194


### Service Networking


Expose deployment

    kubectl get pods -o wide
    kubectl get deployment nginx

    kubectl expose deployment nginx --type="NodePort" --port=80

Examples:

1. Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000.

      kubectl expose rc nginx --port=80 --target-port=8000

2.  Create a service for a replication controller identified by type and name specified in "nginx-controller.yaml",
    which serves on port 80 and connects to the containers on port 8000.

      kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

3. Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"

      kubectl expose pod valid-pod --port=444 --name=frontend
4. Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
    "nginx-https"

      kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

5. Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.

      kubectl expose rc streamer --port=4100 --protocol=udp --name=video-stream

6.  Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
    port 8000.

      kubectl expose rs nginx --port=80 --target-port=8000

7. Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000.

      kubectl expose deployment nginx --port=80 --target-port=800

#### Service expose types

1. ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
2. NodePort - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
3. LoadBalancer - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
4. ExternalName - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of kube-dns.



### Ingress


Ingress is an API object that manages external access to the services in a cluster, usually HTTP.It ca n provide load balancing,SSL termination and name-based virtual hosting.
For our purposes, an Edge router is a router that enforces the firewall policy for your cluster.

This could be a gateway managed by a cloud provider or a physical piece of hardware. Our cluster network is a set of links, either logical or physical ,that facilitate communication
within a cluster according to the Kubernetes networking model. Example for a Cluster network include Overlays such as flannel,Like we're using in our testing cluster,Or SDNs such as OpenVSwitch.

Service is Kubernetes Service that identifies a set of pods using label selectors, Unless mentioned otherwise,Services are assumed to have virtual IPs only routable with the cluster network.

### What is ingress ???

Service and pods have IPs only routable by the cluster network.
So an ingress is a collection of rules that allow inbound connection
Can be configured to give service externally reachable URLs, Load balancer traffic. terminate SSL, offers name-based virtual hosting, and the like.
Users requests ingress by POSTing the Ingress resource to a API serves. An ingress controller is responsible for the fulfilling the ingress, usually by way of a load balancer,
though it may also configure the edge router or additional front ends to help handle the traffic in Highly Available manner.

Relatively new resource and not available in any Kubernetes release prior to 1.1
ingress controller to satisfy an ingress object

Most cloud providers deploy an ingress controller on the master.
Each ingress pod must be annotated with the appropriate class


### How to secure an ingress:


1. Specify secret.

  * TLS private Keys
  * Certificate

2. Port 443 ( Assumes TLS termination)
3. Multiple hosts are multiplexed on the same port by hostnames specified through the SNI TLS extension
4. The TLS secret must contain keys named tls.crt and tls.key contain the certificate and private key to use for TLS.


An ingress controller is bootstrapped with a load balancing policy that it applies to all ingress objects.
eg:
    * load balancing algorithms
    * backend weight scheme

persistent sessions and dynamic weight not yet exposed. The service load balancer may provide some of this.
Health check are not exposed directly through the ingress
Readiness probes allow for similar functionality
We can update running ingress also. (kubectl edit ing test). Alternatively, kubeclt replace -f on a modified lngress yaml file.
This command updates a running K8s object.

Remember that ingress is a relatively new concept in K8s. other way to expose a service that doesn't involve the ingress resource:

  * Use Service.Type=LoadBalancer
  * Use Serivce.Type=NodePort
  * Use a Port Proxy.


### Deploying a Load Balancer

    kubectl create -f service-lb.yaml


### Configure and Use Cluster DNS


    kubectl get pods -n kube-system (check cluster-dns)
    kubectl exec -it busybox -- nslookup nginx

    kubectl get services

1s we have create the service then it will start resolve DNS

    kubectl expose deployment dns-target
    kubectl exec -it busybox -- nslookup dnstaget

Troubleshooting

    kubectl exec -it busybox -- cat /etc/resolve.configure
    kubectl get pods -n kube-systems | grep kube-dnstaget

    kubectl logs -n kube-system $(kubectl get pods -n kube-systems -l k8s-app=kube-dns) -c dnsmask


    kubectl get svn -n kube-system
    kubectl get endponts kube-dns -n kube-systemd


### CNI

1. All pods can communicate with all other pods.
2. Each pods has it own ip
3. No need to mapping container ports
4. Backward compatible model with VMs:

    * Port allocation
    * Naming
    * Service Discovery
    * Load balancing
    * Application Configure
    * Migrations


Kubernetes Network model

* Dynamic Port Allocation problems

1. Every application must be configure to know which port , etc
2. APi service must inject dynamic port numbers in to containers
3. Service must be able to find one anther or be configure to do so


* The Kubernetes Way:

1. All containers can communicate with each other without NAT
2. All nodes can communicate with all containers without NAT
3. The IP of a container is the same regardless of with container view it
4. k8s applies ip address at the pod level
5. "IP-per-Pod" containers in a pod share a single ip address, like processes in VM


CNI Container Network interface

* Must be implemented as an executable invoked by the container management systems (in our case k8s )
* Plugin is responsible for
1. inserting the network interface into the container network namespace
2. Making necessary change to the host
3. Assign IP address to the interface
4. Set up route consistent with ip management


### Kubelet

1. Default network plugin
2. Default cluster-wide network
3. Probes for network plugin on startup


### Flannel

* Simple and easy layer 3 network fabric
* Flannel runs each host (via DaemonSet)
  1. Allocates subnets least to each host
  2. Stores network configuration, allocated subnets, other data
  3. Packets forwarded using VxLANs


### Calico

1. Free and open source
2. Simplified  network model
3. Scalable, distributed control plane
4. Policy drive network security
5. uses overlay network sparingly
6. Widely deployed
7. Can be run in policy enforcement mode


other worth mentioning : Cilium, Contiv , Multus, OVN , Romana, Vmware NSX-T  ..etc


K8s require its networking model to be implemented by 3rd party plugin, call CNI
Different CNIs features support different hardware, software ,overlay networks, policies and features


### Kubernetes and persistent Volumes

1. Native pod storage is Ephemeral --like a Pod
2. What happens when a container crashes:
    * Kubectl restart it (possibly on anther node)
    * File system is re-create from images
    * Ephemeral files are gone

#### Docker Volumes

1. Directory on disk
2. Possibly in another container
3. New Volume Drivers

#### Kubernetes Volumes

1. Same lifetime as its pods
2. Data preserved across containers restarts
3. Pod goes away -> Volume goes away
4. Directory with Data
5. Accessible to containers in a pod
6. Implementation details determined by volume types


### Using Volumes

1. Pod spec indicates which volumes to provide for the pod (spec.volumes)
2. Pod spec indicates where to mount these volumes in containers (spec.containers.volumeMounts)
3. Seen from the container's perspective as the file systemd
4. Volumes cannot mount onto other volumes
5. No hard links to other volumes
6. Each pod must specify where each volume is mounted

#### awsElasticBlockStore

1. Mounts and AWS EBS volume to a pod
2. EBS volume is preserved when unmounted
3. Must be created prior to use
4. Nodes must be on AWS EC2 instances in same region
5. Single instance mounting only

    created via command like:

      aws ec2 create-volume --available-zone=us-east-1 --size=10 --volume-type=9p2

#### cephfs

1. allows mounting a CephFS volume to a pod
2. Contents of volume are preserved when unmounted
3. Must have a Ceph server running
4. Share must be exported

#### csi

1. Container Storage interface
2. In-tree CSI volume plugin for volumes on the same node
3. Kubernetes 1.9 +
    * --feature-gates=CSIPersitentVolume=True
4. Metadata fileds specify what is used and How
5. Driver field specify the name of driver
6. volumeHandle identifies volume name
7. readOnly is supported

#### downwardAPI

1. Mount a directory and write data in plan text file


#### emptyDir

1. created when a pod is assigned to a node
2. Exists while pod runs on a particular node
3. initially empty
4. Multiple containers can read/write same volume
5. Volume can be mounted per container --same or different mount points
6. Pod removed -> volume removed
7. Stored on node's local mediume
8. Optional - set emptyDir.medium = Memory for RAM based tmpfs

#### fc (fiber Channel)

1. Allows existing fc volume to be mounted to a pod
2. Single or Multiple Target using targetWWWs
3. FC SAN Zoning must be allocated
4. LUNs must be masked to target WWN

#### flocker

1. Open source clustered container data volume manager
2. Management and orchestration of volumes
3. Allows Flocker dataset to be mounted into a pod
4. Must be created prior to mounting
5. Can be transferred between pods
6. Must have Flocker

#### gcePersistentDisk

1. Mounts a GCE persistent disk to pod
2. Data preserved of volume unmounted
3. Nodes must be on GCE VMs
4. Same project and zone as the persistent disk
5. Multiple concurrnet mounts allowed, but read-only
6. Create with a command like:

    gcloud compute disk create --size=500GB -zone=us-centrall-a my-k8s-disk


#### gitRepo

1. Mounts empltyDir and clones a git repository

eg:

    ...
    volume:
    - name : git-volume
    gitRepo:
      repository : "git@github.com:kasunmadura/k8s.git"
      revision   :  "dd8s9d8as9d8asddjasdjad7asdajdjadjsd9asuda"

#### glusterfs

1. Allows GlusterFS volume to be mounted to a pod
2. Volume data preserved if volume is unmounted
3. Multiple concurrent mounts --read/write --are allowed
4. Must have GlusterFS


#### hostPath

1. Mounts file or Directory from host node's filesystem to a pod
2. Field type - Empty string (for backward compatibility) performs no checks
3. DirectoryOrCreate -- Created if not present
4. Directory-- Directory must exist
5. FileOrCreate -- created if not present
6. File -- File must exist
7. Socket -- Socket must exist
8. CharDevice -- Character device must exist
9. BlockDevice -- Block device must exist
10. HostPath might behave different on different nodes regardless of pod configuration
11. Files and dir created on the host are only writable by root


#### iSCSI

1. Allows an exiting iSCSI volumes to be mounted to a pod
2. Volume data preserved if volume is unmounted
3. Must have an iSCSI provider
4. Multiple read only concurrent connections allowed
5. Only one writer at a time

#### local

1. Alpha, requires "PersistentLocalVolumes" and "VolumeScheduling" feature gates
2. Allows local mounted storage to be mounted to a pods
3. Statically created PersistentLocalVolume
4. Kubernetes is aware of the volume's node contraints
5. Must be on the node
6. Not suitable for all applications
7. 1.9 + Volume binding can be delayed until pod sheduing

#### NFS

1. Allows existing NFS share to be mounted to a pod
2. Volume data preserved if volume unmounted
3. Must have an NFS server

#### PersistenVolmeClaim

1. Used ro mount a PersitentVolume into a pod
2. Users can stake a claim to durable storage without knowing implementation details


#### Projected

1. Volum types currently projected (subject to expansion!):
  * secert
  * downwardAPI
  * configMap


#### ProtworxVolume

1. Elastic block storage layer
2. Storage on a Server
3. Capabilites tier
4. Aggreates capacity
5. Runs in-guest in VMs or on bare metal Linux nodes
6. Can be created dynamically oer pre-provisioned

#### rbd

1. allow Rados Block Device volume to be mounted to pod

#### ScalelIO

1. Software based storage
2. Allows ScaleIO volumes to be mounted to pods
3. Must have existing ScaleIO Cluster
4. Volumes must be pre-created

#### Scret

1. used to pass sensitive information to pods
2. Stored using the k8s API
3. Mount secrets as file for use by pods
4. Volumes are backed by tmpfs so secrets  are never written to non-volatile storage
5. Secrets must be created in K8s API prior to use


#### Storageos

1. Allows exitsting Storage OS volume to be mounted to a pods
2. Runs as container in the k8s Env
3. Data can be repliacted
4. Provisioning and compresssion can improve utilization and reduce cost
5. Provides block storage to containers via system
6. requires 64-Linux


### Volumes and Their Access Modes

1. PersistentVolume -- API for users that abstracts implementation details of storage
2. PersistentVolumeClaim -- Methond for users to claim durable storage regradless  of implementation details

### PersistentVolume (PV)

1. provisioned storage in the Cluster
2. cluster resource
3. Volume plugins have independent lifecycle from pods
4. Volumes share the lifecycle of he pod: PersistentVolumes fo not
5. API object (YAML) details the implemenation

### PersistentVolumeClaim

1. Request for storage
2. Pods consume node resources; PVCs consume PV resorcers
3. Pods can request specific CPU memory ; PVCs can request size and access modes

### PVs and PVCs

1. Users and applications do nto share identical requirements
2. Adminstrators should offer a varity of PVs without users
3. PVs are cluster resource , PVCs are requests for the cluster resource
4. PVCs also act as a "claim check" on a resource
5. PVs and PVCs have a set lifecycle

  * Provision
  * Bind
  * Reclaim

### PV Provisioning

1. Static

  * Cretes PVs
  * in the K8s API Avaliable for consumption

2. Dynamic

  * Used when none of the static PVs match the PVC
  * Based on StoageClasses
  * PVC must request a created and configured storage classic
  * Claims requesting nameless class disble dynamic provistioning

To enable dyanamic storage provistioning, DefaultstorageClass adminsssion controller on the API server must be enabled

### Binding

1. user create PVC
2. Master watches for new PVCs and matches them to PVs
3. Master binds PVC to PV
4. Volume may be more that the request
5. Binds are exclusive
6. PVC -> PC mapping is always 1:1
7. Claims not matched will remain unbound indefinitely

### Pod Uses Volume

1. Pods treat PVCs as volumes
2. Cluster checks claim, mounts appropriate volume to pod


### PersistentVolumes Claim Protection

1. Alpha feature as of k8s 1.9
2. Ensures PVCs actively in use do not get removeed from the system
3. PVC considered active when :

  * The pod status is Pending and the pod is assigned to node
  * The pod status is Runbing

4. if a user detetes a PVC in use, removal is postponed until PVC is not in use by any pod


### Reclaiming

1. User can delete PVC objects
2. Reclaiming policy for a PV tells the clusterwaht to do with volume
3. Polices:
  * Retain
  * Recycle
  * Delete
4. Policies allow for manual reclamation of a resource
  * PVC delete
  * PV still exists ; volume is "released"
  * Not yet avaliable because tht data is still present
  * Admin can manually reclaim the volume
5. Adminstrator can configure custom recycler pod template
6. Must contain a volumes specification
7. Volume plugs that support Delete reclaims policy
  * removes the PersistentVolumes object for k8s
  * Associated storage asstes in the infrastructure
  * Dynamically provisioned volumes inherit reclaim policy their StorageClass
8. Administrator should configure StorageClass according to expectations

### Capacity

1. PVs have a specific storage capacity
2. Set using capacity attribute
3. Storage size is the only resource that can be set or requested


### Access Modes

1.  Must be supported by storage resource provide
2.  ReadWriteOnce -- can be mounted  as read/write by one onde only (RWO)
3.  ReadWriteMany -- can be mounted read-only by many nodes (ROX)
4.  ReadWriteMany -- Can be mounted read/write by many nodes (RWX)

A volume can only be mounted using one access mode at a time, regardless of the modes that are supported



## Authentication and Authorization

1. Transport Layer Security Established
2. Authentication (Authenticator Modules)
3. Admissions modules
4. Authorization
  * ABAC
  * RBAC
  * Webhook

### Kubelet Authentication and Authorization

Then kubernetes endpoint exposes APIs which give access to data of varying sensitivity
How to authenticate and Authorize access:

1. By default, requests are treated as anonymous requests
2. If authenticated , then it authorized the  request
3. default is alwaysAllow
4. Might want to subdivide access beacuse:

  * anonymous auth enabled but anonymouse users should be limited
  * Bearer token auth enabled but some service accounts should be limited
  * Client cerrticate auth enabled, but only some that are signed should be allowed


### Configure Network Polices

1. specification of how groups of pods may communicate
2. Use labels to select pods and define rules
3. Implemented by the network plugin
4. Pods are non-isolated by default
5. Pods are isolated when a Network Policy selects them
