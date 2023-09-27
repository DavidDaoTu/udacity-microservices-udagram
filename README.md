# udacity-microservices-udagram
Udacity's Microservices Udagram Project


# Notes:
1. After provissioning the Postgres RDS (version 14.8). Edit the security group's inbound rule to allow incoming connections from anywhere (0.0.0.0/0). It will allow your local application to connect to the database.
2. For frontend openssl unsupported error, please run the following

```bash
export NODE_OPTIONS=--openssl-legacy-provider"
```

3. Note that AWS_BUCKET name
```bash
export AWS_BUCKET=mys3udagram
```