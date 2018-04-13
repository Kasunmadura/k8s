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

#### Scalling the deployment

    kubectl scale deployment nginx-deployment --replicas=10
    kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cup-precent=80

RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), then the Deployment controller will balance the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called proportional scaling.

For example, you are running a Deployment with 10 replicas, maxSurge=3, and maxUnavailable=2.


#### Pushing and Resumming a Deployment

     kubectl get deployment
     kubectl get rs
     kubectl rollout pause deployment nginx-deployment
     kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
     kubectl rollout resume deployment nginx-deployment
     kubectl get rs -w
     kubectl get all


##### setup Application envs


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
