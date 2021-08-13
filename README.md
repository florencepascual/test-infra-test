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

### Running the pj-on-kind script

```
# CONFIG_PATH and JOB_CONFIG_PATH need to be absolute path
export CONFIG_PATH="/home/fpascual/test-infra-test/config/prow/config-kubernetes.yaml" 

export JOB_CONFIG_PATH="/home/fpascual/test-infra-test/config/jobs/all-in-one/all-in-one/periodic-all-in-one-test.yaml"

export CONFIG_PATH="/home/fpascual/test-infra-test/config/prow/config-ppc64le.yaml" 
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

