# Spinnaker Installation Commands

Before starting we must have a kubernetes cluster ready.
```
mkdir ~/.hal
```
Now connect Kubernetes cluster into the shell. We are authenticating it because we need kube config file to be mount inside the container.
```
gcloud container clusters get-credentials <CLUSTER_NAME> --zone <CLUSTER_ZONE> --project <PROJEVT_ID>
```
After this run docker command to create halyard container 
```
docker run -p 8084:8084 -p 9000:9000 --name halyard -d -v /home/cloud_himanshu2022/spinnaker-demo/.hal:/home/spinnaker/.hal -v /home/cloud_himanshu2022/.kube:/home/spinnaker/.kube  us-docker.pkg.dev/spinnaker-community/docker/halyard:stable
```
Login into halyard container
```
docker exec -it halyard bash
```
```
source <(hal --print-bash-completion)
```
## Choose a Cloud Provider
Create a service-account.yaml file with below context
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
```
CONTEXT=$(kubectl config current-context)

# This service account uses the ClusterAdmin role -- this is not necessary,
# more restrictive roles can by applied.
kubectl apply --context $CONTEXT \
    -f service-account.yaml


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
```
hal config provider kubernetes enable
```

```
CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add my-k8s-account \
    --context $CONTEXT
```
## Choose an environment
```
ACCOUNT=my-k8s-account
```
Set deployment type as distributed system
```
hal config deploy edit --type distributed --account-name $ACCOUNT
```
```
hal config deploy edit --liveness-probe-enabled true --liveness-probe-initial-delay-seconds $LONGEST_SERVICE_STARTUP_TIME
```
## Choose a storage service
### Google Cloud Storage
Create Service account on gcloud to access cloud storage.
```
SERVICE_ACCOUNT_NAME=spinnaker-gcs-account
SERVICE_ACCOUNT_DEST=~/.gcp/gcs-account.json

gcloud iam service-accounts create \
    $SERVICE_ACCOUNT_NAME \
    --display-name $SERVICE_ACCOUNT_NAME

SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:$SERVICE_ACCOUNT_NAME" \
    --format='value(email)')

PROJECT=$(gcloud config get-value project)

gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin --member serviceAccount:$SA_EMAIL

mkdir -p $(dirname $SERVICE_ACCOUNT_DEST)

gcloud iam service-accounts keys create $SERVICE_ACCOUNT_DEST \
    --iam-account $SA_EMAIL
```
Now copy cred file to .hal directory
```
cp ~/.gcp/gcs-account.json ~/<Directory>/.hal/gcs-account.json
```
Get project value to fill in hal container
```
gcloud config get-value project
```
```
PROJECT=$(gcloud config get-value project)
BUCKET_LOCATION=us
SERVICE_ACCOUNT_DEST=# see Prerequisites section above
```
Configure Halyard for storage type and handover the json key.
```
hal config storage gcs edit --project $PROJECT \
    --bucket-location $BUCKET_LOCATION \
    --json-path $SERVICE_ACCOUNT_DEST
```
```
hal config storage edit --type gcs
```
## Deploy Spinnaker
```
hal version list
hal config version edit --version $VERSION
```
Deploy Spinnaker
```
hal deploy apply
```
Change current namespace to the spinnaker namespace
```
kubectl config set-context --current --namespace=spinnaker
```
Login into the cluster and edit services to have load balancer 
```
kubectl patch service spin-deck -p '{"spec":{"type":"LoadBalancer"}}' -n spinnaker
kubectl patch service spin-gate -p '{"spec":{"type":"LoadBalancer"}}' -n spinnaker
```
Update config and redeploy
```
hal config security ui edit --override-base-url "http://<LoadBalancerIP>:9000"
hal config security api edit --override-base-url "http://<LoadBalancerIP>:8084"
hal deploy apply
```


