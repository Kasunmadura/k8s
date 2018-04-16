## Deployment

Example deployment file

      apiVersion: apps/v1
      kind: Deployment
      metadata:
      name: nginx-deployment
      labels:
      app: nginx
      spec:
      replicas: 3
      selector:
      matchLabels:
      app: nginx
      template:
      metadata:
      labels:
        app: nginx
      spec:
      containers:
      - name: nginx
        image: nginx:1.8.1
        ports:
        - containerPort: 80

### Deployment run using yaml file

      kubectl create -f nginx-deployment.yaml
      kubectl get deployment nginx-deployment
      kubectl rollout status deployment nginx-deployment
      kubectl describe deployment nginx-deployment

### Generate yaml file from running deployment

      kubectl get deployment nginx-deployment -o yaml

### Rollout new changes and troubleshoot commands


      kubectl set image deployment nginx-deployment nginx=nginx:1.9
      kubectl rollout status deployment nginx-deployment
      kubectl discribe deployment nginx-deployment


change the yaml file maually and run below command to rollout

     kubectl apply -f nginx-deployment.yaml

or

     kubectl edit deployment nginx-deployment
     kubectl rollout status deployment nginx-deployment
     kubectl discribe deployment nginx-deployment

deployment rollback from revision

    kubectl rollout history deployment nginx-deployment
    kubectl rollout history deployment nginx-deployment --revision=2
    kubectl rollout undo deployment --to-revision=3


get details about pods and replicas

    kubectl get rs
    kubectl get pods
    kubectl get deploy

#### Scaling the deployment

    kubectl scale deployment nginx-deployment --replicas=10
    kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cup-precent=80

RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), then the Deployment controller will balance the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called proportional scaling.

For example, you are running a Deployment with 10 replicas, maxSurge=3, and maxUnavailable=2.


#### Pushing and Resuming a Deployment

     kubectl get deployment
     kubectl get rs
     kubectl rollout pause deployment nginx-deployment
     kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
     kubectl rollout resume deployment nginx-deployment
     kubectl get rs -w
     kubectl get all


##### Setup Application envs


     kubectl create configmap my-map --from-literal=school=Kegaluvidiyalaya

     or yaml

     kubectl create -f my-map.yaml

      apiVersion: v1
      data:
       school: Kegaluvidiyalaya
      kind: ConfigMap
      metadata:
       name: my-map

  Get status configs on configmaps

      kubectl get configmaps
      kubectl describe configmaps
      kubectl create -f pod-config.yaml
      kubectl logs config-test-pod



#### Self-Healing

We could try to stop one of pod and shutdown one of node in our cluster and check whether how self healing working on the Kubernetes.


     kubectl get pods
     kubectl delete pods nginx-deployment-75675f5897-7l6nn
     kubectl get pods


#### Labels and Selectors


    kubectl lable  -l pod app=nginx  tier=frontend
    kubectl get pods -l tier=frontend
    kubectl get pods -l 'environment in (production),tier in (frontend)'
    kubectl get pods -l 'environment, environment notin(qa)'
    kubectl get pods -l environment=production,tier=frontend
    kubectl label pod mysql-test-3221451521 instance=testing --overwrite


#### DaemonSets


    kubectl get daemonsets -n kube-system
    kubectl describe daemonsets kube-flannel-ds -n kube-system


#### Resource Limits and Pod Scheduling


Remove from node 'foo' the taint with key 'dedicated' and effect 'NoSchedule' if one exists.

    kubectl taint nodes foo dedicated:NoSchedule-


Remove from node 'foo' all the taints with key 'dedicated'

    kubectl taint nodes foo dedicated-

Add a taint with key 'dedicated' on nodes having label mylabel=X

  kubectl taint node -l myLabel=X  dedicated=foo:PreferNoSchedule


#### Manually Scheduling a Pod

Setup label for the node

    kubectl lable node k8s-node-2 net=gigabit
    kubectl describe node k8s-node-2

Use nodeSelector for point pod to specific node. Before that we have to setup label for that node.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webhead
    spec:
      replicas: 2
      selector:
        matchLabels:
          run: webhead
      template:
        metadata:
          labels:
            run: webhead
        spec:
          containers:
          - name: webhead
            image: nginx
            ports:
            - containerPort: 80
          nodeSelector:
            net: gigabit


Then we could see about pods starting on the k8s-node-2 server CreationPolicy

      kubectl get pods
      kubectl describe pods webhead-7765695f45-hh9n7
