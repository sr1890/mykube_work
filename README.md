
######### Kube Command ######### 

Pod: 
=======================================================================

kubectl apply -f pods02.yaml
kubectl get po,svc,deploy
kubectl get po -o wide
kubectl describe po
kubectl exec -it webserver -c webwatcher -- /bin/bash
kubectl describe po mc1
kubectl exec mc1 -c 1st -- /bin/cat /usr/share/nginx/html/index.html
kubectl exec mc1 -c 2nd -- /bin/cat /html/index.html

replicaset: 
=======================================================================

kubectl get rs
kubectl get pods web-6n9cj -o yaml | grep -A 5 owner
kubectl edit pods web-44cjb
kubectl scale --replicas=5 -f nginx_replicaset.yaml
kubectl autoscale rs web --max=5


Deployment
=======================================================================

kubectl create -f nginx-dep.yaml
kubectl get deployments
kubectl describe deploy
kubectl scale deployments/nginx-deployment --replicas=4
kubectl describe deployments/nginx-deployment
kubectl get pods -o wide
kubectl scale deployments/nginx-deployment --replicas=2
kubectl set image  deployments/nginx-deployment nginx=nginx:1.9.1
kubectl describe pods

rollout:
=======================================================================-

kubectl rollout undo deployments/nginx-deployment
kubectl rollout status deployments/nginx-deployment 
kubectl describe deployments | grep Strategy

cleanup:
=======================================================================-

kubectl delete service nginx-deployment
kubectl delete deployment nginx-deployment 

Replica set:
=======================================================================

kubectl get po -l tier=frontend
kubectl get rs
kubectl get pods -n kube-system
kubectl get pods web- -o yaml | grep -A 5 owner


Service: 
=======================================================================

kubectl get svc my-nginx
kubectl describe svc my-nginx
kubectl exec my-nginx-3800858182-jr4a2 -- printenv | grep SERVICE
kubectl scale deployment nginx-deployment --replicas=0; kubectl scale deployment nginx-deployment --replicas=2;
kubectl get deployments
kubectl get pods -l run=nginx-deployment -o wide
kubectl exec nginx-deployment-85c68f75c-sqklm -- printenv | grep SERVICE

DNS:
========================================================================

kubectl get pods -l run=my-nginx -o yaml | grep podIP
kubectl get services kube-dns --namespace=kube-system
kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup my-nginx

Stateful:
========================================================================
Start the NFS using the command:

docker run -d --net=host \
   --privileged --name nfs-server \
   katacoda/contained-nfs-server:centos7 \
   /exports/data-0001 /exports/data-0002 
 
kubectl get pv
kubectl get pvc
kubectl exec -it shell-demo -- /bin/bash
ip=$(kubectl get pod www2 -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); curl $ip

DaemonSet
========================================================================
kubectl get daemonsets/prometheus-daemonset
kubectl get pods -lname=prometheus-exporter

Ingress: 
=======================================================================

kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)

kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"
  
alias:
=======================================================================

$ alias k=kubectl
$ alias kaf='k apply -f'
$ alias kdf='k delete -f'
$ alias kdp='k delete po'
$ alias kgp='k get po

Labels:
=======================================================================

kubectl label nodes gke-k8s-lab1-default-pool-917e857d-qd9k mynode=worker-1
kubectl label nodes gke-k8s-lab1-default-pool-917e857d-l1hk role=dev
kubectl label nodes gke-k8s-lab1-default-pool-917e857d-qq7q role=dev
kubectl taint nodes gke-k8s-lab1-default-pool-917e857d-l1hk role=dev:NoSchedule
kubectl apply -f pod-nginx.yaml
kubectl get nodes --show-labels


upgrade kubeadm:
=======================================================================
root@mastercka#  apt-get update
root@mastercka#  apt-get upgrade -y kubelet kubeadm
root@mastercka#  kubeadm upgrade plan
root@mastercka# kubeadm upgrade apply v1.13.0

We have upgraded master node but we need to manually update CNI (Container Network Interface) after this step before updating nodes. For example I use flanneld for my CNI and you can get latest version from https://github.com/coreos/flannel/releases and check your cluster using

kubectl get daemonsets -n kube-system kube-flannel-ds-amd64  -o yaml |grep image:

Now it is time to update nodes:
serra@mastercka$ kubectl get nodes
serra@mastercka$ kubectl drain node1cka.serra.pw --ignore-daemonsets
serra@mastercka$  kubectl get nodes
node1cka# apt-get update && apt-get upgrade -y kubelet kubeadm 
node1cka#  kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2) 
node1cka:~#  systemctl restart kubelet
node1cka:~# systemctl status kubelet
serra@mastercka:~$  kubectl uncordon node1cka.serra.pw 
If we repeat above steps for all nodes in your cluster at the end of it hopefully you will get all nodes upgraded to 1.13.0


Others:
========================================================================

#Show the capacity of all our nodes as a stream of JSON objects:

kubectl get nodes -o json | jq ".items[] | {name:.metadata.name} + .status.capacity"

#List the pods in the kube-system namespace:

kubectl -n kube-system get pods

#Monitoring: Weave Scope is a visualization and monitoring tool for Docker and Kubernetes:

kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl get svc -n weave -o yaml > svc.yaml && sed -i "s/ClusterIP/NodePort/g" svc.yaml && kubectl replace -f svc.yaml

# grab the name of your active pod
$ PODNAME=$(kubectl get pods --output=template \
     --template="{{with index .items 0}}{{.metadata.name}}{{end}}")

# open a port-forward session to the pod
$ kubectl port-forward $PODNAME 3000:3000

#Port forwarding

pod=$(kubectl get pod -n weave --selector=name=weave-scope-app -o jsonpath={.items..metadata.name})

kubectl port-forward -n weave "$(kubectl get -n weave pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}')" 4040


GCP Create cluster:

gcloud container clusters create k8s-lab1 --disk-size 10 --zone asia-east1-a --machine-type n1-standard-2 --num-nodes 3 --scopes compute-rw
gcloud container clusters delete k8s-lab1 --zone asia-east1-a
gcloud container clusters get-credentials k8s-lab1 --zone asia-east1-a


#Create an NGINX Pod
	kubectl run --generator=run-pod/v1 nginx --image=nginx
#Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
	kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
#Create a deployment
	kubectl create deployment --image=nginx nginx
#Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
	kubectl create deployment --image=nginx nginx --dry-run -o yaml
#Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
	kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml


###############################################################
