## KIND Cluster Setup Guide

Step 1. vim Install_kind.sh

Step 2. Installing KIND and kubectl and Docker using shell script Link path : KIND_Installation_Script.sh

Step 3. After Running docker ps command ERROR Through permission denied.
sudo usermod -aG docker $USER && newgrp docker

now if you run docker ps. Docker is running.
kubectl version

Step 4: Setting Up the KIND Cluster
Create a kind-config.yaml file:
```yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    image: kindest/node:v1.30.0

  - role: worker
    image: kindest/node:v1.30.0

  - role: worker
    image: kindest/node:v1.30.0
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP

```


Step 5: 
command - kind create cluster --name archanacluster --config=config.yml

command - kubectl cluster-info --context archanacluster

command - kubectl get nodes

Step 6: 
create one namespace called nginx Command - kubectl create ns nginx 

Step 7:
Just Like: Docker run
By this command - kubectl run nginx --image=nginx -n nginx (pod/nginx created)

Step 8 : once pod created how will you access it.
command - kubectl get pods -n nginx

#But Problem is i will forgot command.
SO, USE YML File(that also called MANIFEST FILE) to write command.
Before that delete all command that was excuted already.
command - kubectl delete pod nginx -n nginx (pod "nginx deleted)
command - kubectl delete ns nginx (namespace "nginx" deleted)


Step 9:
Now make folder name is nginx
command - cd nginx
Now make MANIFEST FILE 
command - vim namespace.yml ( i am creating a file in which i will write code to make namespace)

```yaml

kind: Namespace
apiVersion: v1
metadata:
  name: nginx

```

cmd - kubectl apply -f namespace.yml    (namespace/nginx created)

# kubectl apply and kubectl create me difference?
if u use create with kubctl it is created single time.
but if use apply with kubctl then item get created and updated also.

Command - kubectl get namespace 

step 10: Now make pod.yml Mainefest file.
command - vim pod.yml
```yaml

kind: pod
apiVersion: v1
metadata:
  name: nginx-pod
  namespace: nginx
spec:
  containers:
     - name: nginx
       image: nginx:latest
       ports:
        - containerPort: 80

```
command - kubectl apply -f pod.yml  (pod/nginx-pod created)
command - kubectl get pods -n nginx (pod is running)
command - kubectl get pods -v=7 (get more info about pods) (v = stand for verbocity) ( -v = 9 for more detail)

If YOU WANT TO KNOW ALL DETAILS OF POD

VVVVVVVVVVV.IIIIII command - kubectl describe pod/nginx-pod -n nginx

Step 11: Now how can i enter inside the pod?

VVVVVVVVVVV.IIIIII command - kebectl exec -it nginx-pod -n nginx -- bash
use exit command to get out from pod.


Dis-advantage to create kind: cluster

If you create Kind Cluster pods:- You made pods one by one.
Pods crash hone par auto-restart nahi honge ❌
Scaling manually karna padega ❌
Rolling updates nahi milenge ❌

# To resole this problem use Deployemnt 

Deployment → ReplicaSet → Pods
Kind : creates the cluster
kubectl : → communicates with the cluster
Kubernetes Deployment → manages the pods

🚀 what Deployment will do
1. Scaling : command - kubectl scale deployment my-app --replicas=5 (Only this command run 5 pods created)
2. Auto-healing : if any pod get deleted/crashed DEPLOYEMENT will automatically created pod.
3. Rolling Update : kubectl set image deployment/my-app nginx=nginx:latest (NEW/OLDER VERSION CREATED WITHOUT ANY DOWNTIME. Means sare container ko ek sath down nhi karta kuch ko up rakhta hai kuch container ko create karta hai fir baki ke container ko up kar deta hai)
4. Rollback : kubectl rollout undo deployment my-app (Go to previous Version)


Now Steps How to create kind deployment
Step 1: vim deployemt.yml

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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
        image: nginx:latest
        ports:
       - containerPort: 80


```


Step 2 : command - kubectl apply -f deployement.yml
command : kubectl get pods -n nginx
command :  kubectl scale deployment my-app --replicas=5





# What is Job : Ek task jo complete hone ke baad khatam ho jata hai”

1. Pod start hota hai
2. Task run hota hai
3. Task complete → Pod Completed state me chala jata hai
4. Agar fail ho jaye → Job retry karta hai


``` yaml

kind: Job
apiVersion: batch/v1
metadata:
  name: job
spec:
  completions: 1
  template:
    spec:
      containers:
      - name: container
        image: busybox
        command: ["echo", "Hello Kubernetes Job"]
      restartPolicy: Never

```









   


