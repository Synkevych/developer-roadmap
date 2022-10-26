# Google Cloud Services and Abilities

## AIM

Service that provide

## Cloud Shell

Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

Before all you should setup environment and enable services that your need: 

```sh
# Set the default zone
gcloud config set compute/zone us-central1-f

gcloud services enable compute.googleapis.com
```

<details>
<summary><b>Resources:</b></summary>
<br>


```sh
# create cloud storage bucket
gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID

# list files in the storage bucket
gcloud ls gs://fance-store-$DEVSHELL_PROJECT_ID

# deploy the backend instance
gcloud compute instances create backend \
    --machine-type=n1-standard-1 \
    --tags=backend \
   --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.

# list running instances
gcloud compute instances list

# deploy the frontend instance
gcloud compute instances create frontend \
    --machine-type=n1-standard-1 \
    --tags=frontend \
    --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh

# stop instance
gcloud compute instances stop frontend

# delete instance
gcloud compute instances delete backend

# configure network
gcloud compute firewall-rules create fw-fe \
    --allow tcp:8080 \
    --target-tags=frontend

gcloud compute firewall-rules create fw-be \
    --allow tcp:8081-8082 \
    --target-tags=backend


# Create instance template for scaling
gcloud compute instance-templates create fancy-fe \
    --source-instance=frontend

# list template instances
gcloud compute instance-templates list

# create managed instance group
gcloud compute instance-groups managed create fancy-fe-mig \
    --base-instance-name fancy-fe \
    --size 2 \
    --template fancy-fe

# name ports to identify its 
gcloud compute instance-groups set-named-ports fancy-fe-mig \
    --named-ports frontend:8080

# configure autohealing
# create a health checj that repairs the instance if it returns "unhealthy"
gcloud compute health-checks create http fancy-fe-hc \
    --port 8080 \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3

gcloud compute health-checks create http fancy-be-hc \
    --port 8081 \
    --request-path=/api/orders \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3

# create a firewall rule to allow the health check
gcloud compute firewall-rules create allow-health-check \
    --allow tcp:8080-8081 \
    --source-ranges 130.211.0.0/22,35.191.0.0/16 \
    --network default

# apply the health checks to their respective services
gcloud compute instance-groups managed update fancy-fe-mig \
    --health-check fancy-fe-hc \
    --initial-delay 300
gcloud compute instance-groups managed update fancy-be-mig \
    --health-check fancy-be-hc \
    --initial-delay 300

# create http load balancer

```
</details>

## Storage

* **Cloud SQL** – 
* **Cloud Spanner** – 
* **Firestore** – 
* **BigQuery** – storage and analytics, use SQL queries, build-in machine learning.
* **Bigtable** – 


## Developing and Deployingh

* **App Engine** - development and hosting of a web application
* **Compute Engine** 
* **Cloud Endpoint** –
* **Apigee Engine** – provide a softwere service to other companies, focuses on business problems like rate limiting, quotas, and analytics
* **Cloud Run** - using Artifact Registry as a images store you can run container with your application
* **Cloud Functions**


### Cloud Build

### Kubernetes Engine

Why it is better to use Kubernets ?

? Clusters
? Workloads
? Pods
? Container images  / Container Registry that saves images of your app 
? Cloud Build Artifaccts


```sh 
gcloud config set compute/zone us-central1-f

# Create a GKE cluster with 3 nodes
gcloud services enable container.googleapis.com
gcloud container clusters create fancy-cluster --num-nodes 3

# create docket container with cloud build
gcloud services enable cloudbuild.googleapis.com
cd ~/monolith-to-microservices/monolith
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .

# deploy container to GKE
kubectl create deployment monolith --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0

# expose your wevsite to the interner
kubectl expose deployment monolith --type=LoadBalancer --port 80 --target-port 8080
kubectl get services # will show you external ip address

# scale GKE deployment up to 3 replicas
kubectl scale deployment monolith --replicas=3

# submit changes to a new image version 2.0.0
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .

# update image four your deployment to a new version
kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0
kubectl get pods # to verify changes
kubectl get service monolith # get info about external IP address, or find it in Services & Ingress tab

# remove Google Container Registry Images
gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --quiet
# remove Google Cloud Build artifacts
gcloud builds list | grep 'SOURCE' | cut -d ' ' -f2 | while read line; do gsutil rm $line; done
# remove GKE services
kubectl delete service monolith
kubectl delete deployment monolith
# delete cluster
gcloud container clusters delete fancy-cluster

# update your image without deploying it
kubectl set image deployment/fancy-frontend-916 fancy-frontend-916=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancy-frontend-916:1.0.1
```

## Artifact Registry

What is it, and how use it?


### Resources

[Deploy website on Cloud Run](https://www.cloudskillsboost.google/focuses/10445?parent=catalog)
[Hosting a Web App on Google Cloud Using Compute Engine](https://www.cloudskillsboost.google/focuses/11952?parent=catalog)
