# udacity-microservices-udagram
Udacity's Microservices Udagram Project


## Part 1: Monolithic notes
1. After provissioning the Postgres RDS (version 14.8). Edit the security group's inbound rule to allow incoming connections from anywhere (0.0.0.0/0). It will allow your local application to connect to the database.
2. For frontend openssl unsupported error, please run the following

```bash
export NODE_OPTIONS=--openssl-legacy-provider"
```

3. Note that AWS_BUCKET name
```bash
export AWS_BUCKET=mys3udagram
```

## Part 2: Microservices notes
1. "npm ERR! Cannot read property '@types/bcrypt' of undefined"
   --> Solution: 
   + Change Node version 16 in Dockerfile
2. "npm ERR! Cannot read property '@angular/common' of undefined"
   --> Solution: 
   + Change RUN npm install -f in Dockerfile.
   + Change "typescript" "~3.5.3" in package.json & package-lock.json

## Part 3: CI notes
1. Remember to tag just built images with DOCKERHUB repository name in .travis.yml, if not we can't push them to DockerHub.

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