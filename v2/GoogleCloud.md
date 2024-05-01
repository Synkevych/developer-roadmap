# Google Cloud Services and Abilities

Basic terms and technologies that need to be understood by achieving a skill badge.

(User Profile)[https://www.cloudskillsboost.google/public_profiles/777e21b3-a400-4bed-ad95-e9eaa0ee3055]

## Identety and Access Management

Service that provide ability to change access to the provided resources: 

Type of recources:
  * organisations node
  * folders
  * project
  * recources

Type of roles: 
  * basic – viewer, editor, owner, search
  * predefined
  * custom 
  * service account

Add to project a user with editor role:

```sh
gcloud projects add-iam-policy-binding my-project  --member=user:my-user@example.com --role='roles/editor'
```

## Cloud Shell

Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

### Set compute zone  

```sh
# Set the default zone
gcloud config set compute/zone us-central1-f

gcloud services enable compute.googleapis.com
```

### Create MySQL database

```sh
gcloud services enable sqladmin.googleapis.com
gcloud sql instances create griffin-dev-db --region=us-east1 --database-version=MYSQL_8_0 --cpu=2 --memory=4GB --root-password=password123
gcloud sql connect griffin-dev-db --user=root --quiet
```

### Create a VM

```sh 
gcloud compute instances create griffin-bastion-wp --network-interface network=griffin-dev-vpc,subnet=griffin-dev-mgmt --network-interface network=griffin-prod-vpc,subnet=griffin-prod-mgmt  --zone=us-east1-b --machine-type=n1-standard-1
	gcloud compute instances list --sort-by=ZONE

kubectl create -f wp-env.yaml
kubectl apply -f wp-deployment.yaml
```

### Create a Kubernetes Cluster 

```sh
gcloud config set compute/zone us-east1-b

gcloud container clusters create griffin-dev \
  --machine-type n1-standard-4 \
  --num-nodes 2 \
  --network griffin-dev-vpc\
  --subnetwork griffin-dev-wp\
  --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
```

<details>
<summary><b>Using gcloud create website cluster</b></summary>
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
* **BigQuery** – storage and analytics, use SQL queries, build-in machine learning. Fully-managed petabyte-scale data warehouse that runs on the Google Cloud. Data analysts and data scientists can quickly query and filter large datasets, aggregate results, and perform complex operations without having to worry about setting up and managing servers. It comes in the form of a command line tool (pre installed in cloudshell) or a web console—both ready for managing and querying data housed in Google Cloud projects.
* **Bigtable** – 

### Command to working with backet from Cloud Shell

```
gsutil -m cp -r gs://cloud-training/folder .
```

### Commands to work with PostgreSQL instance

```
# get Information about the sql instance 
gcloud sql instances describe $CLOUD_SQL_INSTANCE

# create a backup by providing date 
gcloud sql instances patch $CLOUD_SQL_INSTANCE  --backup-start-time=HH:MM

# enable point in time clone 
gcloud sql instances patch $CLOUD_SQL_INSTANCE \
     --enable-point-in-time-recovery \
     --retained-transaction-log-days=1

# crate point in time clone
gcloud sql instances clone $CLOUD_SQL_INSTANCE $NEW_INSTANCE_NAME --point-in-time 'TIMESTAMP'
```

## VPC Networks

Working with networks using Cloud Shell 
```sh
# Create network:
	gcloud compute networks create griffin-dev-vpc --subnet-mode=custom;
	gcloud compute networks create griffin-prod-vpc --subnet-mode=custom;
	gcloud compute networks list

# Create subnets:
	gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region=us-east1 --range=192.168.16.0/20;
	gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region=us-east1 --range=192.168.32.0/20;
	gcloud compute networks subnets create griffin-prod-wp --network=griffin-prod-vpc --region=us-east1 --range=192.168.48.0/20;
	gcloud compute networks subnets create griffin-prod-mgmt --network=griffin-prod-vpc --region=us-east1 --range=192.168.64.0/20;
# Create firewall rules for make network reachable:
	gcloud compute firewall-rules create griffin-dev-vpc-firewall --network griffin-dev-vpc --allow tcp:22,tcp:3389,icmp;
	gcloud compute firewall-rules create griffin-prod-vpc-firewall --network griffin-prod-vpc --allow tcp:22,tcp:3389,icmp;
	gcloud compute firewall-rules list --sort-by=NETWORK
  ```

## Developing and Deployingh

* **App Engine** - development and hosting of a web application
* **Compute Engine** 
* **Cloud Endpoint** –
* **Apigee Engine** – provide a softwere service to other companies, focuses on business problems like rate limiting, quotas, and analytics
* **Cloud Run** - using Artifact Registry as a images store you can run container with your application
* **Cloud Functions**


### Cloud Build

> Serverless CI/CD platform for build, test and deploy your builds on Google Cloud. You can configure builds to fetch dependencies, run unit tests, static analyses, and integration tests, and create artifacts with build tools such as docker, gradle, maven, bazel, and gulp.

### Container Registry (Docker images)

### Commands

```sh
# Build the image using carrent dir as a source.
docker build -t gcr.io/PROJECT_ID/hello-node:v1 .
# Check the build by run it
docker run -d -p 8080:8080 gcr.io/PROJECT_ID/hello-node:v1
# Push the build to Container Registry
docker push gcr.io/qwiklabs-gcp-01-21557d62afb4/hello-node:v1
```

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

# expose your website to the interner
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
I's a recources repo like github or dockerhub, but private and used only in the created project.


## Create a Cloud Run service that converts files to PDF files in the cloud 

Build a container and push it to Container Registry:

`gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter`

To deploy container to Cloud Run as a service:

```sh
gcloud run deploy pdf-converter \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
  --platform managed \
  --region us-east1 \
  --no-allow-unauthenticated \
  --max-instances=1

gcloud run deploy pdf-converter \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
  --platform managed \
  --region us-east1 \
  --memory=2Gi \
  --no-allow-unauthenticated \
  --max-instances=1 \
  --set-env-vars PDF_BUCKET=$GOOGLE_CLOUD_PROJECT-processed
```

Check working:

`curl -X POST -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL`

Create a bucket for file storage: 

`gsutil mb gs://$GOOGLE_CLOUD_PROJECT-upload`

Create a Pub/Sub notification whenever a new file finished uploading to the bucker(-t is a label):

`gsutil notification create -t new-doc -f json -e OBJECT_FINALIZE gs://$GOOGLE_CLOUD_PROJECT-upload`

Create a new service account which Pub/Sub will use to trigger the Cloud Run services: 

`gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"`

Give the new service account permission to invoke the PDF converter service:

`gcloud beta run services add-iam-policy-binding pdf-converter --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com --role=roles/run.invoker --platform managed --region us-east1`

Enable your project to crate Cloud Pub/Sub authentication token:

```
PROJECT_NUMBER=[project_number]
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com --role=roles/iam.serviceAccountTokenCreator
```

Create a Pub/Sub subscription so that the PDF converter can run whenever a message is published to the topic “new-doc”

`gcloud beta pubsub subscriptions create pdf-conv-sub --topic new-doc --push-endpoint=$SERVICE_URL --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com`


## Questions

[] Identity Aware Proxy
[] Bastion host strategy
[] A "zero trust" solution: <https://cloud.google.com/beyondcorp-enterprise>
[] Benefits of using Cloud SQL against local PostgreSQL
[] What is BigQuery and why it is better that rational databases like postgresql or mysql?

### Resources

[Deploy website on Cloud Run](https://www.cloudskillsboost.google/focuses/10445?parent=catalog)
[Hosting a Web App on Google Cloud Using Compute Engine](https://www.cloudskillsboost.google/focuses/11952?parent=catalog)
[Create PDF Files using Cloud Run](https://www.cloudskillsboost.google/focuses/8390?parent=catalog)
[Importing Data to a Firestore Database](https://www.cloudskillsboost.google/focuses/8392?parent=catalog)
