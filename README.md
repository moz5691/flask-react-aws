# Deploying a Flask and React Microservice to AWS ECS

[![pipeline status](https://gitlab.com/testdriven/flask-react-auth/badges/master/pipeline.svg)](https://gitlab.com/testdriven/flask-react-auth/commits/master)

https://testdriven.io/courses/auth-flask-react/

- aws commands

```sh
$ aws sts get-caller-identity
```

- Create repository in AWS

```sh
$ aws ecr create-repository --repository-name <name> --region <region>
$ aws ecr create-repository --repository-name tdd-users --region us-east-1
$ aws ecr create-repository --repository-name tdd-client --region us-east-1
```

- Build docker image

```sh
  $ docker build \
   -f services/users/Dockerfile \
   -t 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-users:dev \
   ./services/users

  $ docker build \
  -f services/client/Dockerfile \
  -t 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-client:dev \
  ./services/client
```

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 794200276070.dkr.ecr.us-east-1.amazonaws.com

aws ecr get-login-password --region us-east-1 | docker
login --username AWS --password-stdin 794200276070.dkr.ecr.us-east-1.amazonaws.com

docker push 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-users:dev
docker push 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-client:dev

#### docker-compose

```sh
$ export REACT_APP_USERS_SERVICE_URL=http://localhost:5001
$ docker-compose -f docker-compose.prod.yml up -d --build
```

#### Test

```sh
$ docker-compose down
$ export REACT_APP_USERS_SERVICE_URL=http://localhost:5001
$ docker-compose -f docker-compose.prod.yml up -d --build

$ docker-compose -f docker-compose.prod.yml exec users python manage.py recreate_db
$ docker-compose -f docker-compose.prod.yml exec users python manage.py seed_db

```

#### ECR build

```sh
$ docker build \
  -f services/users/Dockerfile.prod \
  -t 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-users:prod \
  ./services/users

$ docker build \
  -f services/client/Dockerfile.prod \
  -t 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-client:prod \
  --build-arg NODE_ENV=production \
  --build-arg REACT_APP_USERS_SERVICE_URL=${REACT_APP_USERS_SERVICE_URL} \
  ./services/client

$ docker push 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-users:prod

$ docker push 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-client:prod
```

test added

#### ECR build for Fargate

```sh
$ export REACT_APP_USERS_SERVICE_URL=http://LOAD_BALANCER_DNS_NAME

$ docker build \
  -f services/users/Dockerfile.prod \
  -t 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-users-fargate:prod \
  ./services/users

$ docker build \
  -f services/client/Dockerfile.prod \
  -t 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-client-fargate:prod \
  --build-arg NODE_ENV=production \
  --build-arg REACT_APP_USERS_SERVICE_URL=${REACT_APP_USERS_SERVICE_URL} \
  ./services/client

$ docker push 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-users-fargate:prod

$ docker push 794200276070.dkr.ecr.us-east-1.amazonaws.com/tdd-client-fargate:prod

```

#### Query database

```sh
aws --region us-east-1 rds describe-db-instances \
  --db-instance-identifier flask-react-db-fargate \
  --query 'DBInstances[].{DBInstanceStatus:DBInstanceStatus}'



aws --region us-east-1 rds describe-db-instances \
  --db-instance-identifier flask-react-db-fargate \
  --query 'DBInstances[].{Address:Endpoint.Address}'

```

postgres://webapp:password@flask-react-db-fargate.cecs7qtovf8h.us-east-1.rds.amazonaws.com:5432/users_prod
