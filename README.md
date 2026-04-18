## KIND Cluster Setup Guide

## Step 1. vim Install_kind.sh

## Step 2. Installing KIND and kubectl and Docker using shell script Link 


# Step 3. After Running docker ps command ERROR Through permission denied.
sudo usermod -aG docker $USER && newgrp docker
now if you run docker ps. Docker is running.
kubectl version
