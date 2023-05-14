# praticaDocker
Documentação de atividade prática do Programa de Bolsas da Compass/UOL sobre Docker

## Criação da VPC
- Para essa atividade, optou-se por criar uma VPC com dois ranges de CIDR pequenos:
   - 192.168.1.0/24: para subnets privadas
   - 192.168.2.0/24: para subnets públicas
- Foram configuradas duas subnets privadas e duas subnets públicas, respeitando ranges IP e zonas de disponibilidade distintas.
- Foi criado e associado um Internet Gateway à VPC, para liberar comunicação com a Internet.
- Foi criado um NAT Gateway, alocado um Elastic IP, e associado a uma subnet pública, a fim de garantir tráfego de saída para a Internet às subnets privadas.
- Em Route tables:
  - Foram criadas duas Route Tables, uma pública e uma privada.
    - Na Route Table pública foi criada uma rota para a Internet (0.0.0.0/0) associada ao Internet Gateway. Foram associadas a ela as duas subnets públicas.
    - Na Route Table privada foi criada uma rota para a Internet (0.0.0.0/0) associada ao NAT Gateway. Foram associadas a ela as duas subnets privadas.

## Criação do banco de dados no serviço RDS
   - Foi criado um Database do tipo MySQL
   - configurou-se identificador, usuário e senha
   - Instância da classe db.t3.micro
   - Criou-se um Security Group para o banco de dados
   - Foi definido um database inicial, de nome wordpress
   
## Criação das Instâncias
- Todas as instâncias criadas foram com as seguintes configurações:
  - Amazon Linux 2023 AMI
  - t3.small
  - 8 GiB gp3
- A primeira instância criada foi pública, responsável pela comunicação SSH com as demais.
   - Alocada em subnet pública
   - IP público automático
   - Security Group com SSH liberado para qualquer IP
- A segunda instância foi criada como privada, para conter o container com wordpress
   - 
  





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
```
#!/bin/bash

# Atualiza os pacotes do sistema
yum update -y

# Instala o Docker
yum install -y docker

# Inicia o serviço do Docker
systemctl start docker.service

# Configura o usuário ec2-user para executar comandos do Docker sem precisar de "sudo"
usermod -a -G docker ec2-user

# Baixa e executa o container do WordPress
docker run -e WORDPRESS_DB_HOST=praticadocker-db.cymxgpuymbmd.us-east-1.rds.amazonaws.com \
  -e WORDPRESS_DB_USER=admin \
  -e WORDPRESS_DB_PASSWORD=admin123 \
  -e WORDPRESS_DB_NAME=praticadocker-db \
  -p 80:80 \
  --name meu-wordpress \
  wordpress


# Configura o Docker para iniciar automaticamente na inicialização do sistema
systemctl enable docker.service
```
