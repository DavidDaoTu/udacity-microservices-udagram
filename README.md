# udacity-microservices-udagram
Udacity's Microservices Udagram Project

First of all, my Udagram microservices deployed at: http://ab70b2cdab75f4d27bd7b6a11072ea0e-962554426.us-east-1.elb.amazonaws.com/home

## PROJECT RUBRIC FOR REVIEWERS

### I. Containers and Microservices

**Success Criteria:**

1. Divide an application into microservices: /feed and /user backends are separated into independent projects.

- **Solution**: I seperated ***udagram-api-feed*; *udagram-api-user*** folders as the snapshot below

   <p align="center">
   <img src="Screenshots/microservice_app.png" width="100%" title="Divide an application into microservices" alt="Divide an application into microservices"/>
   </p>


2. Build and run a container image using Docker

- **Solution**: I included Dockerfiles in each service as udagram-api-feed,
udagram-api-user, udagram-frontend, udagram-reverseproxy folders.

   <p align="center">
   <img src="Screenshots/Dockerfiles_in_each_services.png" width="100%" title="Project includes Dockerfiles to successfully create Docker images for /feed, /user backends, project frontend, and reverse proxy" alt="Project includes Dockerfiles to successfully create Docker images for /feed, /user backends, project frontend, and reverse proxy"/>
   </p>
   
   I already built Docker images & pushed microservices images to my DockerHub.
   <p align="center">
   <img src="Screenshots/DockerHub.png" width="100%" title="Screenshot of DockerHub shows the images" alt="Screenshot of DockerHub shows the images"/>
   </p>

### II. Independent Releases and Deployments

**Success Criteria:**

1. Use Travis to build a CI/CD pipeline
- **Solution**: My project consists **.travis.yml** file.
  
   <p align="center">
   <img src="Screenshots/travis_ci.png" width="100%" title="Project includes a .travis.yml file" alt="Project includes a .travis.yml file"/>
   </p>

   The snapshot below show TravisCI interface showing a sucessful build Docker images:
   <p align="center">
   <img src="Screenshots/travis_ci_interface.png" width="100%" title="Screenshot of the Travis CI interface shows a successful build" alt="Screenshot of the Travis CI interface shows a successful build"/>
   </p>

   Then TravisCI also successfully push Docker images to my DockerHub with tag latest: digest: sha256...
   <p align="center">
   <img src="Screenshots/continuous_deployment.png" width="100%" title="Screenshot of the Travis CI interface shows a successful deploy" alt="Screenshot of the Travis CI interface shows a successful deploy"/>
   </p>


### III. Service Orchestration with Kubernetes

**Success Criteria:**
1. Deploy microservices using a Kubernetes cluster on AWS
- **Solution**: The below presents the success deployement & pods with status running on AWS EKS
   <p align="center">
   <img src="Screenshots/kubectl_get.png" width="100%" title="A screenshot of kubectl commands show the Frontend and API projects deployed in Kubernetes" alt="A screenshots of kubectl commands show the Frontend and API projects deployed in Kubernetes"/>
   </p>

   The output of kubectl describe services does not expose any sensitive strings such as database passwords:
   <p align="center">
   <img src="Screenshots/kubectl_describe_service.png" width="100%" title="A screenshot of kubectl commands show the Frontend and API projects deployed in Kubernetes" alt="A screenshots of kubectl commands show the Frontend and API projects deployed in Kubernetes"/>
   </p>

2. Use a reverse proxy to direct requests to the appropriate backend
- **Solution**: I also expose External IP of frontend & reverse proxy service, please check below snapshots:
  
   <p align="center">
   <img src="Screenshots/kubectl_get_services.png" width="100%" title="A screenshot of kubectl get services" alt="A screenshot of kubectl get services"/>
   </p>

   In Frontend console log, I printed the External IP of reverseproxy's URL
   <p align="center">
   <img src="Screenshots/reverseproxy_external_ip.png" width="100%" title="A screenshot of External IP's reverseproxy" alt="A screenshot of deployement"/>
   </p>


3. Configure scaling and self-healing for each service
- **Solution**: Kubernetes services are replicated. At least one of the Kubernetes services has replicas: defined with a value greater than 1 in its deployment.yml file
   <p align="center">
   <img src="Screenshots/replicas_deployment.png" width="100%" title="A screenshot of replicas in deployment file" alt="A screenshot of replicas in deployment file"/>
   </p>

   Screenshot of Kubernetes cluster of command kubectl describe hpa has autoscaling configured with CPU metrics
   <p align="center">
   <img src="Screenshots/kubectl_describe_hpa_fixed.png" width="100%" title="A screenshot of kubectl describe hpa" alt="A screenshot of kubectl describe hpa"/>
   </p>

   I also show the kubectl get hpa command:
   <p align="center">
   <img src="Screenshots/kubectl_get_hpa_fixed.png" width="100%" title="A screenshot of kubectl get hpa" alt="A screenshot of kubectl get hpa"/>
   </p>


### IV. Debugging, Monitoring, and Logging

**Success Criteria:**

1. Use logs to capture metrics for debugging a microservices deployment

- **Solution**: Screenshot of one of the backend API pod logs indicates user activity that is logged when an API call is made.
   
   When accessing the deployed web: "http://ab70b2cdab75f4d27bd7b6a11072ea0e-962554426.us-east-1.elb.amazonaws.com", backend feed API is called & shown as below:

   <p align="center">
   <img src="Screenshots/kubectl_logs_feed.png" width="100%" title="A screenshot of kubectl logs backend feed API's pod" alt="A screenshot of kubectl logs backend feed API's pod"/>
   </p>

   In addition to that, when I do a log-in activity on the Frontend, so the backend-user API call shown as below:
   <p align="center">
   <img src="Screenshots/kubectl_logs_user.png" width="100%" title="A screenshot of kubectl logs backend user API's pod" alt="A screenshot of kubectl logs backend user API's pod"/>
   </p>


> **_Note that:_**
   Because of ***"Cross-Origin Read Blocking (CORB)"*** so my deployment on AWS EKS is not so success to load posts even I enabled S3 CORS. 
   I just wanna say that my local deployment connected to AWS S3 & AWS RDS Postgres is working very well as below:

   <p align="center">
   <img src="Screenshots/local_deployment.png" width="100%" title="A screenshot of local deployement" alt="A screenshot of local deployement"/>
   </p>

>=========================> END OF PROJECT RUBRIC <=====================
### !! THE BELOW SECTION IS NOT RELATED TO RUBRIC !!

=============  Only use for my notes, reviewers don't need to care ============= 
#### Part 1: Monolithic notes
1. After provissioning the Postgres RDS (version 14.8). Edit the security group's inbound rule to allow incoming connections from anywhere (0.0.0.0/0). It will allow your local application to connect to the database.
2. For frontend openssl unsupported error, please run the following

```bash
# Only used for local machine. Don't set this as environemt in configmap of kubectl
export NODE_OPTIONS=--openssl-legacy-provider"

```

3. Note that AWS_BUCKET name
```bash
export AWS_BUCKET=mys3udagram
```

#### Part 2: Microservices notes
1. "npm ERR! Cannot read property '@types/bcrypt' of undefined"
   --> Solution: 
   + Change Node version 16 in Dockerfile
2. "npm ERR! Cannot read property '@angular/common' of undefined"
   --> Solution: 
   + Change RUN npm install -f in Dockerfile.
   + Change "typescript" "~3.5.3" in package.json & package-lock.json

#### Part 3: CI notes
1. Remember to add DOCKER HUB environment varialbes ($YOUR_DOCKER_HUB, $DOCKER_PASSWORD, $DOCKER_USERNAME) in travisci webpage.
2. Remember to tag just built images with DOCKERHUB repository name in .travis.yml, if not we can't push them to DockerHub.

```bash
script:
  - docker --version # print the version for logging  
  - echo "Building udagram microservices app"
  ## Run this command from the directory where you have the "docker-compose-build.yaml" file present
  - docker-compose -f docker-compose-build.yaml build --parallel
  - docker tag reverseproxy $YOUR_DOCKER_HUB/reverseproxy:latest
  - docker tag udagram-api-user $YOUR_DOCKER_HUB/udagram-api-user:latest
  - docker tag udagram-api-feed $YOUR_DOCKER_HUB/udagram-api-feed:latest
  - docker tag udagram-frontend $YOUR_DOCKER_HUB/udagram-frontend:latest

after_success:
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
  - docker push $YOUR_DOCKER_HUB/reverseproxy
  - docker push $YOUR_DOCKER_HUB/udagram-api-user
  - docker push $YOUR_DOCKER_HUB/udagram-api-feed
  - docker push $YOUR_DOCKER_HUB/udagram-frontend
```


#### Part 4: EKS deployment notes
1. Using AWS CLI to create EKS Cluster with our defined role (our policy).


- Following this [guidelines](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)


```bash
$ aws eks create-cluster --region us-east-1 --name my-EKSCluster --kubernetes-version 1.27 \
   --role-arn arn:aws:iam::074088802078:role/myAmazonEKSClusterRole \
   --resources-vpc-config subnetIds=subnet-0df98f9a562d3dc06,subnet-0a364f3b6fd600c7e,subnet-0c96bdcdbef4ff101,subnet-0ec7a576d92f670fc,subnet-0cbd4ae809979b80b

$ aws eks update-kubeconfig --region us-east-1 --name my-EKSCluster
$ kubectl get nodes
```


2. EKS environment , secret variables & credential configuration

See this document: https://kubernetes.io/docs/concepts/configuration/

- Create a "env-configmap.yaml" file in the project's directory to save all our configuration values (non-confidential environments variables) in that file. Do not store the PostgreSQL username and passwords in the env-configmap.yaml file


- Create a "env-secret.yaml" file to store the confidential values, such as login credentials. Unlike the AWS credentials, these values do not need to be Base64 encoded.

- Secret: Create a "aws-secret.yaml" file to store your AWS login credentials. Replace ___INSERT_AWS_CREDENTIALS_FILE__BASE64____ with the Base64 encoded credentials (not the regular username/password).
  
See this https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables for more details:


```bash
## Use a combination of head/tail command to identify lines you want to convert to base64
## You just need two correct lines: a right pair of aws_access_key_id and aws_secret_access_key
cat ~/.aws/credentials | tail -n 5 | head -n 2
## Convert 
cat ~/.aws/credentials | tail -n 5 | head -n 2 | base64
```

For instance:
```bash
$ cat ~/.aws/credentials | tail -n 2 | base64
```

Or we can do

```bash
$ echo -n "postgres" | base64
$ echo -n "my-db-udagram.ctwfguulwmzc.us-east-1.rds.amazonaws.com" | base64
$ echo -n "AKIARCQAJM4PEQBTMWM3" | base64

```
Once, all deployment and service files are ready, you can use commands like:

```bash
## Apply env variables and secrets
kubectl apply -f aws-secret.yaml
kubectl apply -f env-secret.yaml
kubectl apply -f env-configmap.yaml

```

Check Secret:

```bash
$ kubectl describe secret

```

Check configmap
```bash
$ kubectl describe configmap
```




3. Deployment

+ Configure all key-value pairs in a secret as container environment variables: https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables

> **Note**: In order to fix error "FailedGetResourceMetric" with auto-scaling. We need to install ***"Kubernetes Metrics Server"*** (https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html). 

```bash
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
$ kubectl get deployment metrics-server -n kube-system
```
Then we deploy our pods.

```bash
## Deployments - Double check the Dockerhub image name and version in the deployment files
$ kubectl apply -f backend-feed-deployment.yaml
$ kubectl apply -f backend-user-deployment.yaml
$ kubectl apply -f frontend-deployment.yaml
$ kubectl apply -f reverseproxy-deployment.yaml

```

```bash
## Service
$ kubectl apply -f backend-feed-service.yaml
$ kubectl apply -f backend-user-service.yaml
$ kubectl apply -f frontend-service.yaml
$ kubectl apply -f reverseproxy-service.yaml

```

```bash
## Check the deployment names and their pod status
$ kubectl get deployments
```

How to debug:

```bash
$ kubectl get pods
$ kubectl describe pod [pod-name]
## An example:
## kubectl logs backend-user-5667798847-knvqz
## Error from server (BadRequest): container "backend-user" in pod "backend-user-5667798847-knvqz" is waiting to start: trying and failing to pull image

```
In case of ImagePullBackOff or ErrImagePull or CrashLoopBackOff, review your deployment.yaml file(s) if they have the correct image path and environment variable names.

Check if the backend pods are experiencing CrashLoopBackOff due to an insufficient memory error. If yes, then you can increase the memory limits as shown in this

```bash
$ kubectl get pods
$ kubectl logs [pod-name] -p
## Once you increase the memory, check the updated deployment as:
$ kubectl get pod [pod-name] --output=yaml
## You can autoscale, if required, as
$ kubectl autoscale deployment backend-user --cpu-percent=70 --min=3 --max=5

```

Look at what's there inside the running container. Open a Shell to a running container as:

```bash
$ kubectl get pods
## Assuming "backend-feed-68d5c9fdd6-dkg8c" is a pod
$ kubectl exec --stdin --tty backend-feed-68d5c9fdd6-dkg8c -- /bin/sh
## See what values are set for environment variables in the container
$ printenv | grep POST
## Or, you can try "curl <cluster-IP-of-backend>:8080/api/v0/feed " to check if services are running.
## This is helpful to see is backend is working by opening a bash into the frontend container
```

4. Connect to the Kubernetes Services to Access the Application

If the deployment is successful, and services are created, there are two options to access the application:

+ Port Forwarding: You can forward a local port to a port on the "frontend" pod, as explained here (https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/#forward-a-local-port-to-a-port-on-the-pod).

+ Expose External IP: You can expose the "frontend" deployment using a Load Balancer's External IP.

We use "Expose External IP" method#2
Refer to this tutorial (https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/) to expose an External IP address of the frontend service.

```bash
## Check the deployment names and their pod status
$ kubectl get deployments
## Create a Service object that exposes the frontend deployment
## The command below will ceates an external load balancer and assigns a fixed, external IP to the Service.
$ kubectl expose deployment frontend --type=LoadBalancer --name=publicfrontend
## Repeat the process for the *reverseproxy* deployment. 
## Check name, ClusterIP, and External IP of all deployments
$ kubectl get services 
$ kubectl get pods # It should show the STATUS as Running
```

Connecting services deployed on k8s cluster as below:
   <p align="center">
   <img src="Screenshots/deployments.png" width="100%" title="Connecting services deployed on K8S cluster" alt="Connecting services deployed on "/>
   </p>
   
### Update the Environment Variables and Re-Deploy the Application
Once both the services - frontend and reverseproxy - have an External IP, change the API endpoints in the following places locally.

1. Update **udagram-frontend/src/environments/environment.ts** file - Replace the keyword **localhost** in the **http://localhost:8080/api/v0** string with the External-IP of the **reverseproxy deployment**. For example,
```js
## Assuming http://ae0c61849655c46e581aa91d3c03386d-513419455.us-east-1.elb.amazonaws.com is the External-IP of the *reverseproxy* service.
apiHost: 'http://ae0c61849655c46e581aa91d3c03386d-513419455.us-east-1.elb.amazonaws.com:8080/api/v0'
```
Here, we are using the External-IP to connect the frontend to the reverseproxy.

2. Update udagram-frontend/src/environments/environment.prod.ts file in the same way as done for the environment.ts file.

3. Build a new frontend image, and push it to the Dockerhub. While building a new image, it is recommended to use a different tag each time, as shown in the example below:

```bash
## Run these commands from the /udagram-frontend directory
docker build . -t [Dockerhub-username]/udagram-frontend:v6
docker push [Dockerhub-username]/udagram-frontend:v6
```
4. Next, re-deploy the new frontend image to the k8s cluster. You will have to update the image tag in the frontend-deployment.yaml file.

```bash
## Run these commands from the /udagram-deployment directory
## Rolling update the containers of "frontend" deployment
kubectl set image deployment frontend frontend=[Dockerhub-username]/udagram-frontend:v6
```
Check your deployed application at the External IP of your publicfrontend service.

