#Install kops AWS 

$aws s3api create-bucket \
    --bucket shivify-state-store \
    --region us-east-1

$curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
$chmod +x ./kops
$sudo mv ./kops /usr/local/bin/

$curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$chmod +x ./kubectl
$sudo mv ./kubectl /usr/local/bin/kubectl

$export NAME=fleetman.k8s.local
$export KOPS_STATE_STORE=s3://shivify-state-store

$aws ec2 describe-availability-zones --region us-east-1

kops create cluster --zones=us-east-1a,us-east-1b,us-east-1c ${NAME}

$ kops create cluster --zones=us-east-1a,us-east-1b,us-east-1c ${NAME}

SSH public key must be specified when running with AWS (create with `kops create secret --name fleetman.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub`)

$ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa
$kops create secret --name fleetman.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub

$kops edit cluster ${NAME}

$kops edit ig nodes --name ${NAME}

$kops get ig --name ${NAME}
NAME                    ROLE    MACHINETYPE     MIN     MAX     ZONES
master-us-east-1a       Master  c4.large        1       1       us-east-1a
nodes                   Node    t2.medium       3       5       us-east-1a,us-east-1b,us-east-1c

$kops update cluster ${NAME} --yes
$ kops validate cluster

 
