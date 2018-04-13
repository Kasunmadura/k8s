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
     kubectl rollout status deployment nginx-deployment
     kubectl discribe deployment nginx-deployment

deployment rollback from revision

    kubectl rollout history deployment nginx-deployment
    kubectl rollout undo deployment --to-revision=3
