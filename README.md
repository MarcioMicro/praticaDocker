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

## Criação dos Security Groups
- Para a instância Bastion
   - Apenas a porta 22, a partir de qualquer IP
- Para o Load Balancer
   - Apenas para a porta 80, a partir de qualquer IP
- Para as instâncias privadas
   - Porta 22, a partir da instância Bastion
   - Porta 80, a partir do Load Balancer
- Para o banco de dados
   - Apenas a porta 3306, a partir das instâncias privadas
- Para o volume EFS
   - Apenas para a porta 2049, a partir das instâncias privadas

## Criação do banco de dados no serviço RDS
- Foi criado um banco de dados do tipo MySQL
   - configurou-se identificador, usuário e senha
   - Instância da classe db.t3.micro
   - Configurado na VPC da atividade
   - Configurou-se o Security Group do banco de dados
   - Foi definido um database inicial, de nome wordpress

## Criação do volume EFS
- Foi criado um volume novo na VPC da atividade
- Na guia Network, as subnets foram configuradas com o Security Group do volume EFS
- Foi criado um access point apontando para o volume
   
## Criação das Instâncias
- Todas as instâncias criadas foram com as seguintes configurações:
   - Amazon Linux 2023 AMI
   - t3.small
   - 8 GiB gp3
- A primeira instância criada foi pública, responsável pela comunicação SSH com as demais.
   - Alocada em subnet pública
   - IP público automático
   - Security Group Bastion
- A segunda instância foi criada como privada, para conter o container com wordpress
   - Alocada em subnet privada
   - Sem IP público
   - Security Group privado
   - Criado com o seguinte user data:

```
#!/bin/bash

yum update -y
yum install -y docker
systemctl start docker.service
usermod -a -G docker ec2-user
mkdir -m 777 /mnt/nfs
mount -t nfs4 fs-09f17892f211cfcd1.efs.us-east-1.amazonaws.com:/ /mnt/nfs
mkdir -m 777 /mnt/nfs/wp-content
mkdir -m 777 /mnt/nfs/wp-content/uploads
mkdir -m 777 /mnt/nfs/wp-content/themes
mkdir -m 777 /mnt/nfs/wp-content/plugins
docker run -de WORDPRESS_DB_HOST=praticadocker-rds.cymxgpuymbmd.us-east-1.rds.amazonaws.com \
  -e WORDPRESS_DB_USER=admin \
  -e WORDPRESS_DB_PASSWORD=admin123 \
  -e WORDPRESS_DB_NAME=wordpress \
  -p 80:80 \
  --name praticaDockerWordpress \
  -v /mnt/nfs/wp-content/uploads:/var/www/html/wp-content/uploads \
  -v /mnt/nfs/wp-content/themes:/var/www/html/wp-content/themes  \
  -v /mnt/nfs/wp-content/plugins:/var/www/html/wp-content/plugins \
  wordpress
systemctl enable docker.service
echo "fs-09f17892f211cfcd1.efs.us-east-1.amazonaws.com:/ /mnt/nfs nfs4 defaults,_netdev 0 0" | sudo tee -a /etc/fstab
echo "[Unit]
Description=Docker Container Service
After=docker.service

[Service]
ExecStart=/usr/bin/docker start praticaDockerWordpress

[Install]
WantedBy=multi-user.target" | tee /etc/systemd/system/docker-container.service

sudo systemctl enable docker-container.service
sudo systemctl start docker-container.service

```

- Após a criação da segunda instância, foi criada uma AMI baseada no seu estado atual
- Foi criada, então, a terceira instância, a partir dessa AMI.

## Criação do Load Balancer
- Foi criado um Load Balancer do tipo internet-facing IPV4 na VPC da atividade
- Foram selecionados as duas subnets públicas
- Foi configurado o Security Group correto
- Foi criado um Target Group com as duas instâncias privadas
- Após confirmar tudo, basta acessar o DNS do Load Balancer, que distribuirá o tráfego entre as duas instâncias.

