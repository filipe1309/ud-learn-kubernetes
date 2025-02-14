# Notes

> notes taken during the course


## Section 5

| Kind | Version |
| --- | --- |
| Pod | v1 |
| Service | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

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

### Section 6: Networking in Kubernetes

IP Addrees is assignd to a Pod (not to a container like in docker)

When a Kubernetes its created, 

- each node receives its won IP address (ex 198.168.1.2…)
- a privite network is created with the IP 10.244.0.0, and the Pods are attached to it with sub-nets like 10.244.0.1, 10.244.0.2 etc
- When the cluster has more than 1 node (worker) the pods IP will conflict!!!!
    - because 2 or more Pod will have the same IP Address
    - So kubernetes expect us to set the network correctely
        - all Containers/Pods can communicate to one another without NAT
        - all Nodes can communicate with all containers and vice-versa without NAT
        - There are several pre-built solutions available: cisco, flannel, nsx, cilium…
            - It will create a different private network for each Node (ex 10.244.0.0, 10.244.0.1)

### Section 7: Services

Types

- ClusterIP
- NodePort
- LoadBalancer

#### NodePort

- Expose the service on each Node's IP at a static port (NodePort)
- A ClusterIP service, to which the NodePort service will route, is automatically created
- You'll be able to contact the NodePort service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`


NodePort = Node port (30000 - 32767, if not defined it will be a random number in this range)

Port = Service Port

TargetPort = Pod Port (default equals to Port)

NodePort → Port → TargetPort

curl NODE_IP:NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30004
  selector:
    app: myapp
```

```sh
kubectl apply -f service.yaml
kubectl get svc
```

With kubernetes on Docker Desktop: http://localhost:30004/  

With minikube get the address with `minikube service myapp-service --url`, ex: http://192.168.99.101:30004  

#### ClusterIP

- only accessible within the cluster
- default type of service
- Expose the service on a Cluster-internal IP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service-clusterip
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: myapp
```

#### LoadBalancer

- Expose the service externally using a cloud provider's load balancer
- NodePort and ClusterIP services, to which the external load balancer will route, are automatically created

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service-loadbalancer
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: myapp
```


```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service 
  namespace: default
spec:
  ports:
  - nodePort: 30080
    port: 8080
    targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort
```
