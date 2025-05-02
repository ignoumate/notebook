### Create Project Structure

```bash
mkdir my-project && cd my-project
bun create elysia api
bun create next-app@latest web
```

### Create a IAM User

1. Visit [AWS IAM](https://us-east-1.console.aws.amazon.com/iam/home?region=ap-south-1)
2. Create new user
3. Add EC2FullyRegistryAccess in permissions
4. Create and Save a Access Key (cli purpose)

### Create Github Private repo

1. Create new private repo for each microservice
2. Add Secrets into Repo settings

```bash
AWS_ACCESS_KEY=XXX
AWS_SECRET_ACCESS_KEY=XXX
```

### Login to AWS Console

```bash
aws configure
AWS Access Key ID:
AWS Secret Access Key:
Default region name: ap-south-1
Default output format: None
```

### Push Images to AWS ECR

1. Visit [AWS ECR](https://ap-south-1.console.aws.amazon.com/ecr/home?region=ap-south-1)
2. Create a new repository
3. Follow steps to build your images locally & push by tagging it to AWS ECR

### Spin up a AWS EC2 Instance

1. Visit [AWS EC2](https://ap-south-1.console.aws.amazon.com/ec2/home?region=ap-south-1)
2. Create new instance (t2.micro free tier, OS: amazon linux)
3. Download `key-pair` .pem key for SSH
4. `ssh -i "path/to/keypair.pem" ec2-user@ec2-XX-XXX-XX-XXX.ap-south-1.compute.amazonaws.com` - to ssh into ec2 instance
5. `sudo yum update -y` - update the system
6. `aws configure` - login with aws credentials
7. Edit inbound rules in security groups, expose port 22, 3001, 3000, 80 & 443, add 0.0.0.0/0 to access the instance from anywhere
8. Create Elastic IP and allocate it to ec2 instance

### Install Docker, Docker-compose

1. sudo yum install docker -y
2. sudo docker --version
3. add user to docker group to avoid writing sudo everytime

```bash
sudo usermod -aG docker $USER
newgrp docker
```

4. `vim docker-compose.yml` - create a docker-compose.yml file

```yml
services:
  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    restart: always
  api:
    image: AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/AWS_ECR_NAME:latest
    depends_on:
      - redis
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    restart: always
    ports:
      - "3001:3001"
```

5. `docker compose up --build`

#### Check if your service is running on http://ELASTIC_IP:PORT

### Install NGINX

```bash
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Configure NGINX

1. Paste this into server block

```conf
server {
    listen       80;
    listen       [::]:80;
    server_name  ignoumate.in www.ignoumate.in;
    root         /usr/share/nginx/html;

    include /etc/nginx/default.d/*.conf;

    location /api/ {
        proxy_pass http://localhost:3001/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```

2. `sudo nginx -t` - to check if nginx.conf syntax is right
3. `sudo systemctl reload nginx`
4. delete 3001 & 3000 ports from instance security groups
5. `sudo yum install -y certbot python3-certbot-nginx`
6. `sudo certbot --nginx -d ignoumate.in -d www.ignoumate.in` - generate a certificate
7. `sudo nginx -t` - to check if nginx.conf syntax is right
8. `sudo systemctl reload nginx`

### Continuous Integration

1. `vim .github/workflows/ci.yml`

```yml
name: CI

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Get short SHA
        id: vars
        run: echo "SHORT_SHA=${GITHUB_SHA::7}" >> $GITHUB_ENV

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: ECR_REPO_NAME
          IMAGE_TAG: ${{ env.SHORT_SHA }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

2. manually SSH into ec2 instance
3. `vim docker-compose.yml` - update tag of service (ie. image: xxx/api:v1 -> xxx/api:v2)
4. `docker-compose down service_name` - to stop the service
5. `docker-compose up -d service_name` - to start the updated service

### Continuous Deployment

1.
