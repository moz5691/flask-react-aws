# Deploying a Flask and React Microservice to AWS ECS

[![pipeline status](https://gitlab.com/testdriven/flask-react-auth/badges/master/pipeline.svg)](https://gitlab.com/testdriven/flask-react-auth/commits/master)

https://testdriven.io/courses/auth-flask-react/

- aws commands

```sh
$ aws sts get-caller-identity
```

{
"UserId": "AIDA3R2QGPRTLWDUFYCBT",
"Account": "794200276070",
"Arn": "arn:aws:iam::794200276070:user/sls-admin"
}

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
