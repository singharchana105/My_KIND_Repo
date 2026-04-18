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
    image: kindest/node:v1.35.2

  - role: worker
    image: kindest/node:v1.35.2

  - role: worker
    image: kindest/node:v1.35.2
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP

```

Step 5: 
kind create cluster --name archanacluster --config=config.yml




