# Superset Mz Build and Install Instructions


## Prepare the image on a local machine
- Clone this repository;
- Create a repository called `superset-mz` on AWS ECR;
- Build and push the image to ECR with the commands below (change 688779373772 to AWS account number):
```
aws ecr get-login-password --region sa-east-1 | docker login --username AWS --password-stdin 688779373772.dkr.ecr.sa-east-1.amazonaws.com \
  && docker build -t superset-mz:1.5  . \
  && docker tag superset-mz:1.5 688779373772.dkr.ecr.sa-east-1.amazonaws.com/superset-mz:1.5 \
  && docker push 688779373772.dkr.ecr.sa-east-1.amazonaws.com/superset-mz:1.5
```

## Setup the environment on EC2 for Superset
- Create an IAM Role for this machine for reading on ECR and assing to this EC2;
- Install aws cli on EC2 if isn't installed;
- Log in with the command (change 688779373772 to AWS account number):
```
aws ecr get-login-password --region sa-east-1 | docker login --username AWS --password-stdin 688779373772.dkr.ecr.sa-east-1.amazonaws.com
```
- Clone this repository;
- Add the following content to `docker/requirements-local.txt`:
```
pyodbc==4.0.32
sqlalchemy_dremio==1.2.1
elasticsearch-dbapi[opendistro]==0.2.9
```

- Add MAPBOX_API_KEY to the file `docker/.env-non-dev`;
- Change the file `docker-compose-non-dev.yml`, replacing the image `superset-mz:1.5` to the full uri of the image on ECR `688779373772.dkr.ecr.sa-east-1.amazonaws.com/superset-mz:1.5`;
- Run `docker-compose -f docker-compose-non-dev.yml pull`;
- Run `docker-compose -f docker-compose-non-dev.yml up`;
- Connect to a Dremio instance using the following uri scheme:
```
dremio://username:password@127.0.0.1:31010/dremio;SSL=0  #default
dremio+flight://username:password@127.0.0.1:32010/dremio  #experimental
```

### Additional steps
- Install nginx to create a reverse proxy;
- Install Letsencrypt to create a SSL certificate;
