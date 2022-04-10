# Helm for CloudBuild
How to use helm with cloudbuild?

To use helm with Cloud Build we need to push an image into Container Registry for Helm tool builder
Below are the cmmands for the same:
```
PROJECT_ID=<project-id>
git clone https://github.com/GoogleCloudPlatform/cloud-builders-community.git
cd cloud-builders-community/helm
docker build -t gcr.io/$PROJECT_ID/helm .
docker push gcr.io/$PROJECT_ID/helm
```

Now we need to add helm section in cloudbuild.yaml