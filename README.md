# prow_test


## Kind on x86

### Set Up

```
# Cloning the repo kubernetes/test-infra
git clone https://github.com/kubernetes/test-infra

# Download the dockerd-entrypoint.sh and put in the /usr/local/bin/ directory
sudo wget -o /usr/local/bin/dockerd-entrypoint.sh https://raw.githubusercontent.com/docker-library/docker/master/dockerd-entrypoint.sh

# Download the job 
mkdir -p /home/fpascual/test-infra/config/jobs/all-in-one/all-in-one
curl --oauth2-bearer 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 -o /home/fpascual/test-infra/config/jobs/all-in-one/all-in-one/periodic-all-in-one-test.yaml https://raw.githubusercontent.com/florencepascual/prow_test/main/periodic-all-in-one-test.yaml

# docker, kubectl, kind
docker
kubectl
kind
kind get clusters
# if no kind clusters, create them
kind create cluster
```
on x86
```
export KUBECONFIG='/home/fpascual/.kube/kind-config-mkpod'
```



### Running the pj-on-kind script

```
# CONFIG_PATH and JOB_CONFIG_PATH need to be absolute path
export CONFIG_PATH="/home/fpascual/test-infra/config/prow/config.yaml" JOB_CONFIG_PATH="/home/fpascual/test-infra/config/jobs/all-in-one/all-in-one/periodic-all-in-one-test.yaml"

./test-infra/prow/pj-on-kind.sh docker-all-in-one-test
```

```
# CONFIG_PATH and JOB_CONFIG_PATH need to be absolute path
export CONFIG_PATH="/home/fpascual/test-infra/config/prow/config-ppc64le.yaml" JOB_CONFIG_PATH="/home/fpascual/test-infra/config/jobs/all-in-one/all-in-one/periodic-all-in-one-test-ppc64le.yaml"

./test-infra/prow/pj-on-kind.sh docker-all-in-one-test
```

```
# CONFIG_PATH and JOB_CONFIG_PATH need to be absolute path
export CONFIG_PATH="/home/fpascual/test-infra/config/prow/config-ppc64le.yaml" JOB_CONFIG_PATH="/home/fpascual/test-infra/config/jobs/all-in-one/all-in-one/periodic-all-in-one-test-new.yaml"

./test-infra/prow/pj-on-kind.sh docker-all-in-one-test
```

### Testing

```
kubectl get pods
# get the id of the pod we just created
id='pod-id'
kubectl exec -it $id -- /bin/bash
```

