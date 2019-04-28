# Kubernetes Spring Boot Example in Google Kubernetes Engine (GKE)

## Create Spring Boot app
You can use the sample project which I have in here.
`git clone https://github.com/TechPrimers/spring-boot-lazy-init-example.git`

## Create Docker image
Command to create docker image using Google JIB plugin
`./mvnw com.google.cloud.tools:jib-maven-plugin:build -Dimage=gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-example:v1`

## Run the Docker image
Command to run the docker image which we created in the previous step
`docker run -ti --rm -p 8080:8080 gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-example:v1`

## Login to the K8s Cluster
Command to login to the K8s cluster from Cloud Shell
`gcloud container clusters get-credentials techprimer-cluster-1 --zone  us-central-1a

## Kubernetes Commands
### List Pods
`kubectl get pods`

### List Deployments
`kubectl get deployments`

### List Services
`kubectl get services`

### Deploy an image
`kubectl run spring-boot-example --image=gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-example:v1 --port=8080`

### Expose Load Balancer
`kubectl expose deployment spring-boot-example --type=LoadBalancer`

### Scale deployments
`kubectl scale deployment spring-boot-example --replicas=3`
