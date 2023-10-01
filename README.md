# udacity-microservices-udagram
Udacity's Microservices Udagram Project







### THE BELOW SECTION IS NOT RELATED TO RUBRIC 

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