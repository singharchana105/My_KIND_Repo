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

If you create Kind Cluster pods:- You made pods one by one. and if

1. Pods crash hone par auto-restart nahi honge ❌

2. Scaling manually karna padega ❌

3. Rolling updates nahi milenge ❌

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
  name: demo-job
spec:
  completions: 1
  parallelism: 1
  template:
   metadata:
      name: demo-job-pod
      labels:
        app: batch-task
    spec:
      containers:
      - name: batch-container
        image: busybox:latest
        command: ["sh", "-c" , "echo Hello Dosto! && sleep 10"]
      restartPolicy: Never

```
command :- kubectl get pods -n nginx

command : kubectl logs pod/pod ka name -n nginx( Iss command se jo output aana tha job command me vo dikhayega)

Note : pods ki bhi lifecycle hoti hai.
1. contener creating
2. Running
3. Terminating 
4. Completed
5. Imagepullbackup


# CronJob : A task that you can schedule

Lets go and make a cron job for taking backup of one folder in another folder every minute

vim cron-job.yml

```yaml

kind: Cronjob
apiversion: batch/v1
metadata:
  name: minute-backup
  namespace: nginx

spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
     template:
      metadata:
       name: minute-backup
       labels:
         app: minute-backup
 spec:
   containers:
    - name: backup-container
      image: busybox
      command:
      - sh
      - -c
      - >
        echo "Backup Started" ;
        mkdir -p /backups &&
        mkdir -p /data &&
        cp -r /demo-data /backups &&
        echo "Backup Completed" ;
     volumeMounts:
      - name: data-volume
        mountPath: /demo-data
      - name: backup-volume
        mountPath: /backups
  restartPolicy: OnFailure
  volumes:
   - name: data-volume
     hostPath:
       path: /demo-data
       type: DirectoryOrCreate
   - name: backup-volume
     hostPath:
       path: /backups
       type: DirectoryOrCreate


```

##Persistent Volume (PV) && Persistent Volume Claim (PVC)##
Both are used to store data of pods, so that if pods get down/crashed data will not lost.

Persistent Volume (PV) - Actual Storage (disk space in cluster). Host system EC2 have 30GB storage. Then 1GB data ka PV bana liye.

Persistent Volume Claim (PVC) - A request for that storage. (ab 1GB ko pods ko assign karenge)

eg:- PV - a storage resource (like a hard-derive)
     PVC - a user asking for some storage.

Now lets make Menistfile of PV

Step1 vim persistentVolume.yml

  ```YAML

kind:PersistentVolume
apiVersion: v1
metadata:
  name: local-pv
  namespace: nginx
  labels:
    app: local
spec:
   capacity:
      storage: 1Gi
   accessModes:
      - ReadWriteOnce
   persistentvolumeReclaimPolicy: Retain
   storageClassName: local-storage
  hostPath:
     path: /mnt/data

```

Apply it :- kubectl apply -f persistentVolume.yml
Apply it - kubectl get pv 
     

Now I have to claim that PV

For that i have to make one more Manifest file. 
vim persitentVolumeClaim.yml

```YMAL
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage


```
Apply it:- kubectl apply -f persitentVolumeClaim.yml


pehle jo Deployment banye the uska data delete ho jata tha usko Volumemount kardenge iss PVC ke sath. 

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
      volumeMounts:
       - mountPath: /var/www/html
         name: my-volume
      volumes:
        - name: my-volume
          persistentVolumeClaim:
             claimName: local-pvc


```

command - kubectl apply -f Deployment.yml
command - kubectl get pods -n nginx
command - kubectl describe pod/podname -n nginx


Now how we get data from /var/www/html in mnt/data

command - kubectl get pods -n nginix -o wide (find node kon se cluster par chal rha hai.)

command - kubectl get nodes

command - docker ps (find container id of that node)

command - docker exec -it containerID bsh  (By this command you will go inside the container )

command - ls , cd mnt,ls,data exit



Now ab iss pods ko user kyese access karenge. Using SERVICES this will expose to outer world.

vim service.yml

```yaml
kind: Service
apiVersion: v1
mrtadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
    -protocol:TCP
     port: 80
     targetPort: 80
  type: ClusterIP

```

command - kubectl apply -f service.yml
command - kubectl get all -n nginx


Now its time to access my service on the browser

command - kubectl port-forward service/nginx-service -n nginx 80:80 --address=0.0.0.0 ( unable to listen on port 80 permission denied)

command - sudo -E kubectl port-forward service/nginx-service -n nginx 80:80 --address=0.0.0.0 ( listener failed bind: address already in use unable to listen on any requested port 80 80)

command - sudo -E kubectl port-forward service/nginx-service -n nginx 81:80 --address=0.0.0.0 (add security rule in EC2 for port 81)

site will running in browser PublicIP:81 




## Maka a mini project from github

Step 1 : git clone URL

Step 2 : docker build -t note-app-k8s .

Step 3 : docker images   ( images note-app-k8s locally not working. isko docker hub par phuchana padega)

Step 4 : Login DockerHub. Go to account setting and go in peronal access tokens. Generate token. it will give you Run and password

Step 5 : paste run of Tocken command in CLI  and password. Login succeeded

Step 6 : docker build -t note-app-k8s .

(permission denied ka error may be aaye so for that command : sudo usermod -aG docker $USER)

Step 7 : docker image tag note-app-k8s:latest archanakidocker/note-app-k8s:latest (tag your image with docker hub)

Step 8 : docker images

Step 9 : docker push archanakidocker/note-app-k8s:latest   (Pushed in my docker hub repo)

Step 10. mkdir k8s, cd k8s,

Step 11. vim deployment.yml

```yml

kind: Deployment
apiVersion: apps/v1
metadata:
  name: notes-app-deployment
  labels:
    app: notes-app
   namespace: notes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: archanakidocker/note-app-k8s
        ports:
        - containerPort: 8000

```

command- kubectl apply -f deployment.yml

Step 12: vim namespace.yml

```yml
kind: namespace
apiVersion: v1
metadata:
  name: notes-app

```

command- kubectl apply -f namespace.yml

Step 13: vim service.yml

```yml
kind: service
apiVersion: v1
metadata:
  name: notes-app
  namespace: notes-app
spec:
  selector:
    app: notes-app
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
  type: ClusterIP


```
command- kubectl apply -f service.yml


 Step 14: kubectl get pods -n notes-app    (container is creating)

 Step 15: kubectl port-forward service/notes-app-service -n notes-app 8000:8000 --address=0.0.0.0

 Step 16: Go in EC2 add security group port no- 8000

 Step 17 : No copy Ip of EC2 192.62.36.45:8000 ( Application running)



 # Ingress ? Resource that manage traffic and routes.
 
 /nginx -> open nginx 
 
 /app -> open jango app

 Namespace should be same.

# Step 1. Vim deployment.yml  -> namespace change notes-app to nginx.

``` 
kind: Deployment
apiVersion: apps/v1
metadata:
  name: notes-app-deployment
  labels:
    app: notes-app
   namespace: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: archanakidocker/note-app-k8s
        ports:
        - containerPort: 8000
```

 Step 2. vim service.yml     -> namespace change note-app to nginx

```yml
kind: service
apiVersion: v1
metadata:
  name: notes-app
  namespace: nginx
spec:
  selector:
    app: notes-app
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000
  type: ClusterIP

```
 
Step 3. Note : kubectl get deployment -n notes-app 
 
        kubectl delete deployment/notes-app-deployment -n  notes-app

        similary  kubectl delete service/notes-app-service -n notes-app

        similarly kubectl delete ns notes-app

 I am delete this because i have changes there namesapce.

 Step 4. 
 kubectl apply -f deployment.yml -f service.yml

 kubectl get pods -n nginx

 kubectl get svc -n nginx

 Step 5. Ingress controller are used to help in routing at cluster level.

command for ingress controller - go to folder    ->     cd /home/ubuntu/kubernetes-in-one-shot

Run this    ->     kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml

Now ingress controller is all set. kubectl get ns -> ingress-nginx namespace created.

kubectl get pods -n ingress-nginx

kubectl get svc -n ingress-nginx

Step 6. Now go to nginx folder and create vim ingress.yml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-notes-ingress
  namespace: nginx
  annotation:
    nginx.ingress.kubernetes.io/rewrite-targets: /
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: /nginx
        backend:
          service:
            name: nginx-service
            port:
              number: 80
      - pathType: Prefix
        path: /
        backend:
          service:
            name: notes-app-service
            port:
              number: 8000

```

kubectl apply -f ingress.yml

kubectl get ing -n nginx  (ip address is missing so for getting it ingress controller will exposes)

kubectl get all -n nginx

Step 7. Now Ingress controller has exposed

kubectl get svc -n ingress-nginx   ( you will get ingress-nginx-controller name)

Exposed command - kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 80:80 --address=0.0.0.0   (if this giving an error of permission denied)

command - sudo -E kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 80:80 --address=0.0.0.0  (port forwarding)

it means that in port 80 ingress controller run.

Step 8. In Ec2 add port 80 in inbound rule.



# StatefulSets: Pods carry state. when i will delete one pod in statefulset. same number with same name pod will created.

jango,flask,springboot all these application ka state maintain nhi hua tab bhi chalta hai. -> Deployment, Replicasets, Daemonsets

But mysql,mongoDB are statefull application. -> statefulsets

# Make statefulsets of MYSQL Project

Step 1. vim namespace.yml

--- 
kind: Namespace
apiVersion: v1
metadata: 
  name: mysql

---
kubectl apply -f namespace 

Step 2. vim statefulsets.yml

--- 
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: mysql-statefulset
  namespace: mysql
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mysql
   
  template:
    metadata:
      labels:
        app: mysql
    spec:
      
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
       env:
       -name: MYSQL_ROOT_PASSWORD
        value: root
       - name: MySQL_DATABASE
        value: devops

       volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumeClaimTemplates:
        - metadata:
           name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
          resources:
          requests:
           storage: 1Gi

 
 

 






   


