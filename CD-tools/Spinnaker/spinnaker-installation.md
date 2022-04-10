# Spinnaker Installation Guide
Spinnaker is a free and open-source continuous delivery software platform originally developed by Netflix and extended by Google.

It is designed to work with Kubernetes, Google Cloud Platform, AWS, Microsoft Azure and Oracle Cloud.

**Developers:**  Netflix Team, Armory, Google, OpsMx

**Initial Release:** November 16,2015

**Platform:** Linux

**Written In:** Java, JavaScript(UI), Kotlin

## Prerequisites
- Machine on which we install Halyard. 
    
    This can be a local machine or VM (Ubuntu 14.04/16.04, Debian, or macOS), or it can be a Docker container. Make sure it has **at least 12GB of memory**.

- A Kubernetes cluster on which we deploy Spinnaker
    
    At least **4 cores and 16GB of RAM** available in the cluster.

## Steps for Installation
- Install Halyard
- Choose a cloud provider
- Choose an environment
- Choose a storage service
- Deploy Spinnaker
- Backup you config
- Configure everything else
- Productionize Spinnaker

## Install Halyard
We will be using a docker container for Halyard deployment. As currently we are deploying it on GCP so we will be using cloud shell docker. But its highly recommended, either deploy it on kubernetes or save its .hal directory, beacuse whenever we need to upgrade or make any change in spinnaker we will be using this container.

Create a config directory for halyard on local system called .hal
```
mkdir ~/.hal
```
Now we will run docker container command for halyard. Also we will mount above created directory
```
docker run -p 8084:8084 -p 9000:9000 --name Halyard -d -v /home/cloud_himanshu2022/spinnaker-demo/.hal:/home/spinnaker/.hal -v /home/cloud_himanshu2022/.kube:/home/spinnaker/.kube  us-docker.pkg.dev/spinnaker-community/docker/halyard:stable
```
In seperate shell we will connect to halyard
```
docker exec -it halyard bash
```
Run the following command to enable command completion:
```
source <(hal --print-bash-completion)
```
## Choose a Cloud Provider
All of Spinnaker’s abstractions and capabilities are built on top of the Cloud Providers that it supports. So, for Spinnaker to do anything you must enable at least one provider, with one Account added for it.

Available cloud providers are:
- App Engine
- Amazon Web Services
- Azure
- Cloud Foundry
- DC/OS
- Google Compute Engine
- Kubernetes
- Oracle

We will be using Kubernetes as a provider for Spinnaker.

First we need to create a service account through which Spinnaker intracts with Kubernetes.
```
CONTEXT=$(kubectl config current-context)

# This service account uses the ClusterAdmin role -- this is not necessary,
# more restrictive roles can by applied.
kubectl apply --context $CONTEXT \
    -f /downloads/kubernetes/service-account.yml


TOKEN=$(kubectl get secret --context $CONTEXT \
   $(kubectl get serviceaccount spinnaker-service-account \
       --context $CONTEXT \
       -n spinnaker \
       -o jsonpath='{.secrets[0].name}') \
   -n spinnaker \
   -o jsonpath='{.data.token}' | base64 --decode)

kubectl config set-credentials ${CONTEXT}-token-user --token $TOKEN

kubectl config set-context $CONTEXT --user ${CONTEXT}-token-user
```
service-account.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: spinnaker-role
rules:
  - apiGroups: ['']
    resources:
      [
        'namespaces',
        'configmaps',
        'events',
        'replicationcontrollers',
        'serviceaccounts',
        'pods/log',
      ]
    verbs: ['get', 'list']
  - apiGroups: ['']
    resources: ['pods', 'services', 'secrets']
    verbs:
      [
        'create',
        'delete',
        'deletecollection',
        'get',
        'list',
        'patch',
        'update',
        'watch',
      ]
  - apiGroups: ['autoscaling']
    resources: ['horizontalpodautoscalers']
    verbs: ['list', 'get']
  - apiGroups: ['apps']
    resources: ['controllerrevisions']
    verbs: ['list']
  - apiGroups: ['extensions', 'apps']
    resources: ['daemonsets', 'deployments', 'deployments/scale', 'ingresses', 'replicasets', 'statefulsets']
    verbs:
      [
        'create',
        'delete',
        'deletecollection',
        'get',
        'list',
        'patch',
        'update',
        'watch',
      ]
  # These permissions are necessary for halyard to operate. We use this role also to deploy Spinnaker itself.
  - apiGroups: ['']
    resources: ['services/proxy', 'pods/portforward']
    verbs:
      [
        'create',
        'delete',
        'deletecollection',
        'get',
        'list',
        'patch',
        'update',
        'watch',
      ]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: spinnaker-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: spinnaker-role
subjects:
  - namespace: spinnaker
    kind: ServiceAccount
    name: spinnaker-service-account
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: spinnaker-service-account
  namespace: spinnaker
```
Adding an account, but before this we need to enable provider into halyard. For this we need to jump back into the halyard container
To enable provider execute below command
```
hal config provider kubernetes enable
```
Then add account
```
CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add my-k8s-account \
    --context $CONTEXT
```
Note: we should provide the kubeconfig inside the halyard container, beacuse context will be gathered form kube config file.

## Choose an environment
In this step, you tell Halyard in what type of environment to install Spinnaker.

The recommended path is a distributed installation onto a Kubernetes cluster, but all of these methods are supported:
- Distributed Installation -- for production use -- will go with this
- Local Installations
- Local git installations 

### Distributed Installation:
First set the $ACCOUNT variable for kubernetes user name that we have created earlier
```
ACCOUNT=my-k8s-account
```
Run the following command to se the installation type
```
hal config deploy edit --type distributed --account-name $ACCOUNT
```
Make sure kubectl is installed on the machine running Halyard.

Optionally, configure Kubernetes liveness probes for your Spinnaker services, setting the initialDelaySeconds to the upper bound of your longest service startup time:
```
hal config deploy edit --liveness-probe-enabled true --liveness-probe-initial-delay-seconds $LONGEST_SERVICE_STARTUP_TIME
```
## Choose a storage service
Spinnaker requires an external storage provider for persisting your Application settings and configured Pipelines. Because these data are sensitive and can be costly to lose, we recommend you use a hosted storage solution you are confident in.

Below is the list of storage services that we can configure for spinnaker.
































kubectl patch service spin-gate -p '{"spec":{"type":"LoadBalancer"}}'



