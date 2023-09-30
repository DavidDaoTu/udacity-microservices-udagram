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