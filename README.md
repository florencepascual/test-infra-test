# prow_test


## Kind on x86

### Set Up

```
# Cloning the repo kubernetes/test-infra for the mkpj, mkppod and pj-on-kind.sh
git clone https://github.com/kubernetes/test-infra

# Download the prow job config file
mkdir -p /home/fpascual/test-infra/config/jobs/docker-in-docker
curl -o /home/fpascual/test-infra/config/jobs/docker-in-docker/periodic-all-in-one.yaml https://raw.githubusercontent.com/florencepascual/test-infra-1/master/config/jobs/periodic/docker-in-docker/periodic-all-in-one.yaml

# download ppc64le config file
curl -o /home/fpascual/test-infra/config/prow/config-ppc64le.yaml https://raw.githubusercontent.com/ppc64le-cloud/test-infra/master/config/prow/config.yaml
# rename kubernetes file
mv /home/fpascual/test-infra/config/prow/config.yaml /home/fpascual/test-infra/config/prow/config-kubernetes.yaml 

# docker, kubectl, kind
docker
kubectl
kind
kind get clusters
# if no kind clusters, create them
kind create cluster
```
on x86 to set KUBECONFIG
```
export KUBECONFIG='/home/fpascual/.kube/kind-config-mkpod'
```

on power

```
sudo minikube start
rm pod.yaml
nano pod.yaml
kubectl apply -f pod.yaml
```

### Secret

```
nano docker-secret.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-secret
type: kubernetes.io/basic-auth
stringData:
  username: florencepascual
  password: 
```
https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-config-file/

```
kubectl apply -f ./docker-secret.yaml
```

```
docker logout
docker login -u florencepascual -p dddc5ad3-40c2-46ee-ae79-11855453cd1a

kubectl create secret generic docker-token \
    --from-file=.dockerconfigjson=/home/fpascual/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
```

### Running the pj-on-kind script

```
# CONFIG_PATH and JOB_CONFIG_PATH need to be absolute path
export CONFIG_PATH="/home/fpascual/test-infra-test/config/prow/config-kubernetes.yaml" 
export CONFIG_PATH="/home/fpascual/test-infra-test/config/prow/config-ppc64le.yaml" 

export JOB_CONFIG_PATH="/home/fpascual/test-infra-test/config/jobs/all-in-one/all-in-one/periodic-all-in-one-test.yaml"

export CONFIG_PATH="/home/fpascual/test-infra-test/config/prow/config.yaml" 
export JOB_CONFIG_PATH="/home/fpascual/test-infra-test/config/jobs/periodic/docker-in-docker/periodic-all-in-one.yaml"

./test-infra-test/hack/test-pj.sh docker-all-in-one-test


rm test-infra-test/config/jobs/periodic/docker-in-docker/periodic-all-in-one.yaml
nano test-infra-test/config/jobs/periodic/docker-in-docker/periodic-all-in-one.yaml
```

### Testing manually

```
kubectl get pods
# get the id of the pod we just created
id='pod-id'
kubectl exec -it $id -- /bin/bash

# get the logs of the container test
kubectl logs $id test

# delete a pod
kubectl delete pod $id
kubectl delete --all pods 
```




## Test prow job on ppc64le

Need : 2 machines (1 ppc and 1 x86)
On ppc : start cluster 
```
# start cluster
sudo kubeadm init 

# to restart the cluster
sudo kubeadm reset
rm .kube/config/admin.conf

# set kubeconfig
mkdir -p /home/fpascual/.kube/config
sudo cp -i /etc/kubernetes/admin.conf /home/fpascual/.kube/config
sudo chown $(id -u):$(id -g) /home/fpascual/.kube/config
sudo chown $(id -u):$(id -g) /home/fpascual/.kube/config/admin.conf
export KUBECONFIG=/home/fpascual/.kube/config/admin.conf

# check the cluster is running
kubectl cluster-info

# on master node
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
# check all pods are running
kubectl get pods --all-namespaces
# check master node is ready 
kubectl get nodes -o wide
# enable scheduling (https://github.com/calebhailey/homelab/issues/3)
kubectl taint nodes --all node-role.kubernetes.io/master-

# on worker node, not necessary
# run the command that was the output by kubeadm init as sudo
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

# if you do not have the token 
kubeadm token list
```

on x86 : connect to the cluster which is running on ppc64
```
rm -rf /home/fpascual/.kube/config/admin.conf
nano /home/fpascual/.kube/config/admin.conf
# copy the admin.conf which is in the ppc64 machine
export KUBECONFIG=/home/fpascual/.kube/config/admin.conf
# check the cluster is connected
kubectl cluster-info

# get the secrets
# docker-token
docker login -u florencepascual --password-stdin
kubectl create secret generic docker-token --from-file=.dockerconfigjson=/home/fpascual/.docker/config.json --type=kubernetes.io/dockerconfigjson
# secret-s3
nano secret-s3.yaml
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-s3
type: Opaque
data:
  password: 
```
```
kubectl apply -f secret-s3.yaml

```

apiVersion: v1
kind: Secret
metadata:
  name: s3-auth
stringData:
  service-account.json: |
    {
      "region": "us-south",
      "access_key": "",
      "endpoint": "s3.us-south.cloud-object-storage.appdomain.cloud",
      "insecure": false,
      "s3_force_path_style": true,
      "secret_key": ""
    }