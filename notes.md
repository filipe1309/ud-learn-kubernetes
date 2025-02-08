# Notes

> notes taken during the course


## Section 5

```sh
kubectl apply -f pod.yaml
```

```sh
kubectl get pods
kubectl run nginx --image=nginx
kubectl describe pod nginx

kubectl get pods -o wide

kubectl run redis --image=redis123 --dry-run=client -o yaml > pod-redis.yaml
```

### ReplicaSets

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: # required, pod template used by the replicaset to create pods when replicas are created or replaced
    # Pod definition
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3 # required
  selector: # required
    matchLabels:
      type: front-end
```

```sh
kubectl apply -f replicaset.yaml
kubectl get rs
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods
kubectl delete rs myapp-replicaset
```

```sh
kubectl replace -f replicaset.yaml
kubectl scale --replicas=6 -f replicaset.yaml
```

```sh
cd replicasets
kubectl create -f replicaset.yaml
kubectl get rs

# delete a pod
# kubectl delete pod <pod-name>
kubectl delete pod myapp-replicaset-5vh7b 
kubectl get pods # new pod will be created

kubectl describe rs myapp-replicaset

kubectl edit rs myapp-replicaset
kubectl delete --all pods
```

```sh
kubectl scale rs myapp-replicaset --replicas=2
```

```sh
kubectl get all
```


### Deployments

```sh
kubectl create -f deployment.yaml
kubectl get deployments
kubectl get pods
kubectl get rs
kubectl describe deployment myapp-deployment
```

```sh
kubectl create deployment redis-deploy --image=redis --replicas=2
kubectl get deploy
```

Rollout

```sh
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

Strategies
- Recreate: Terminate all old pods at once and then create new ones
- Rolling Update: Update pods one by one

```sh
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 # container_name= image_name
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl rollout status deployment/myapp-deployment

# change strategy
kubectl edit deployment myapp-deployment
```

```sh

