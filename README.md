# praticaDocker
Documentação de atividade prática do Programa de Bolsas da Compass/UOL sobre Docker

## Criação da VPC
- Para essa atividade, optou-se por criar uma VPC com dois ranges de CIDR pequenos:
   - 192.168.1.0/24: para subnets privadas
   - 192.168.2.0/24: para subnets públicas
- Foi criado e associado um Internet Gateway à VPC.
- Foram configuradas duas subnets privadas e duas subnets públicas, respeitando ranges IP e zonas de disponibilidade distintas.
- Em Route tables:
   - foi criada uma rota para a web para o Internet Gateway;
   - as duas subnets públicas foram associadas.







```
#!/bin/bash

# Instalação do Docker
yum update -y
amazon-linux-extras install docker -y
systemctl start docker
systemctl enable docker

# Instalação do Docker Compose
yum install -y curl
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Criação da pasta do projeto e arquivo docker-compose.yml
mkdir wordpress
cd wordpress
touch docker-compose.yml
cat <<EOF > docker-compose.yml
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: your_password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: your_password
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: your_rds_endpoint
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: your_password
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data:
EOF
```
