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
- Create a file `docker/requirements-local.txt` and add the following content:
```
pyodbc==4.0.32
sqlalchemy_dremio==1.2.1
elasticsearch-dbapi[opendistro]==0.2.9
```
- Create this local config file: 
`cp docker/pythonpath_dev/superset_config_local.example docker/pythonpath_dev/superset_config_docker.py`
Change the default SECRET_KEY and set SMTP parameters for sending e-mails inside Superset.

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
- Install nginx to create a reverse proxy (example config):
```
server {   
    charset utf-8;
    client_max_body_size 128M; ## listen for ipv4

    server_name superset.com.br;

    # Include SQL injection protect config
    include /etc/nginx/protect.conf;

    location / {
       proxy_pass  http://localhost:8088;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_redirect default;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection 'upgrade';
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/superset.com.br/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/superset.com.br/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
```
- Install Letsencrypt to create a SSL certificate:
```
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d superset.com.br
```
