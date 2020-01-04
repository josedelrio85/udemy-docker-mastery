# COMMANDS

kubectl version 

- Interactua con el servidor que tengamos definido en el fichero .kube. Ahora mismo, como ademas de kubernetes tengo instalado microk8s
que monta un cluster local, muestra la version del cliente y del servidor que devuelve microk8s

kubectl run my-nginx --image nginx

kubectl get pods

- The deployment controller creates a ReplicaSet controller that creates the pod. By default creates one replica so only creates one pod.

kubectl get all

- Lista todos los elementos presentes


kubectl delete deployment my-nginx

- Elimina los elementos creados anteriormente


# Scale

kubectl run my-apache --image httpd

kubectl scale deploy/my-apache --replicas 2
kubectl scale deployment my-apache --replicas 2

- Escala las replicas del deployment a 2

    -- deploy = deployment = deployments


## QUAPACHAO!?

- Control Plane

    Deployment updated to 2 replicas

    Replicaset Controller sets pod count to 2

    Control Plane assigns node to pod
    
    Kubelet sees pod is needed, starts container



# Logs

kubectl logs deployment/my-apache

kubectl logs deployment/my-apache --follow --tail 1

    --follow = -f 
    --tail 1 = lista la ultima linea del log


kubectl describe pod <identificador_pod> 

 - Revisar la sección events


# Services

kubectl expose creates a service for existing pods

A service is a stable address for pod(s)

If we want to connect to pod(s), we need a service

## Basic Service Types

- Cluster IP (default)
    - TIP: good in the cluster
    - Only available on the cluster
    - Single, internal virtual IP allocated
    - Only reachable from withing cluster (node and pods)
    - Pods can reach service on apps port number
- NodePort
    - High port allocated on each node
    - Port is open on every node's IP
    - Anyone can connect (if they can reach the port)
    - Other pods need to be updated to this port

These services are always available in kubernetes

- LoadBalancer
    - Most used in the cloud
    - This is for traffic coming into your cluster from an external source
    - Controls a LB endpoint external to the cluster
    - Only available when infra providers gives you a LB(AWS ELB, etc)
    - Creates NodePort + ClusterIP services, tells LB to send to NodePort
- ExternalName
    - Adds CNAME DNS record to CoreDNS only
    - Not used for Pods, but for givin pods a DNS name to use for something outside kubernetes

## Creating a ClusterIP Service

- Let's expose a NodePort so we can access i via the host IP (including localhost)

kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort

-> kubectl get services
httpenv-np   NodePort    10.152.183.19    <none>        8888:31817/TCP   44s

-> curl httpenv:31817

ports => left inside port || right exposed port

- A NodePort service also create a ClusterIP service

- These three service types are addtive, each one creates the ones above it: ClusterIP, NodePort, LoadBalancer


## Add a LoadBalancer service

kubectl expose deployment/httpenv --port 8888 --name httpenv-lb --type LoadBalancer

- In microk8s no built-in LB
- You can still run the command, it will just stay at "pending" (but its NodePort works)


# Kubernetes Services DNS

- This is a DNS-Based Service Discovery

- We've been using hostnames to access Services

    -> curl <hostname>

- But that only works for Services in the same Namespace

    -> kubectl get namespaces

- Services also hava FQDN

    -> curl <hostname>.<namespace>.svc.cluster.local


# Kubernetes managemente techniques

## Run, expose and create generators

- These commands use helper templates called "generators"

- Every resource in Kubernetes has a specifications or "spec"
    
- You can output these templates with --dry-run -o yaml

- You can use those YAML defaults as a starting point 

-> kubectl create deployment sample --image nginx --dry-run -o yaml

    --dry-run => way to show the output of what you typed

    -o => we want an ouput in YAML

- We are saying: "run this command but don't tehnically do it on the cluster"

-> kubectl create job test --image nginx --dry-run -o yaml

    - job is used when you want to create a set of pods that will run once. 
    - it tends to set them with a default to not restart
    - the output is shorter because a job does not create a replicaset or a deployment

-> kubectl expose deployment/test --port 80 --dry-run -o yaml

    - needs to create the deployment first

    -> kubectl create deployment test --image nginx

## Kubectl run command

- It is deprecated but is useful for several purposes


# Three management approaches

- Imperative commands: run, expose, scale, edit, create deployment
    - Best for dev/learning/personal projects
    - Easy to learn, hardest to manage over time
- Imperative objects: create -f file.yaml, replace -f file.yaml, delete...
    - Good for prod of small environments, single file per command
    - Hard to automate
- Declarative objects: apply -f file.yaml, or dir\, diff
    - Best for prod, easier to automate
    - Harder to understand

- Most important rule => don't mix the three approaches

